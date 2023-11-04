# Gestor de CDP - Documentación Detallada

* **Nombre del Contrato:** cdpManager.sol
* **Tipo/Categoría:** Gestión de Vault 
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss-cdp-manager/tree/master/src)
* ****[**Etherscan**](https://etherscan.io/address/0x5ef30b9986345249bc32d8928b7ee64de9435e39)

## 1. Introducción (Resumen)

**Resumen:** El `DssCdpManager` (alias `manager`, “administrador” o “gestor” en español) fue creado para permitir un proceso formalizado de transferencia de _vaults_ entre dueños, como la transferencia de activos. Se recomienda que todas las interacciones con _vaults_ se hagan a través del Gestor de CDP. Una vez que el colateral bloqueado ha sido depositado en el Protocolo de Maker, los usuarios pueden hacer uso de las siguientes características: 

* Propiedad de Múltiples _vaults_ e identificación numérica (los usuarios pueden poseer N número de _Vaults_)
* Transferibilidad de _Vaults_


![Diagrama del Sistema MCD: Diagrama de interacción entre usuarios de _Vaults_ y Gestores de _Vaults_](<../../.gitbook/assets/CDP Manager.png>)

**Nota:** El diagrama del sistema MCD anterior muestra que los usuarios de _vaults_ pasan a través del _proxy_ para poder interactuar con el Gestor CDP pero también es posible utilizar directamente el contrato del Gestor de CDP.   

## 2. Detalles del Contrato

### Funcionalidades Clave (definidas en el contrato inteligente)

* `cdpAllow(uint cdp, address usr, uint ok)`: Permitir/Denegar (`ok`) una dirección `usr` para gestionar el `cdp`.
* `urnAllow(address usr, uint ok)` : Permitir/Denegar (`ok`) que una dirección `usr` interactúe con una _urn_, con el propósito de entrar (`src`) o salir (`dst).`
* `open(bytes32 ilk, address usr)`: Abre un nuevo _Vault_ para que `usr` sea usado por un tipo de colateral `ilk`.
* `give(uint cdp, address dst)`: Transferencias de `cdp` a `dst`.
* `frob(uint cdp, int dink, int dart)`: Incrementar/disminuir la cantidad de `ink` de colateral bloqueado e incrementar/disminuir la cantidad de deuda `art` en el `cdp` depositando el DAI generado o colateral liberado en la dirección `cdp`.    
* `frob(uint cdp, address dst, int dink, int dart)`: Incrementar/disminuir la cantidad de colateral `ink` bloqueado e incrementar/disminuir la cantidad de deuda `art` en el `cdp`, depositando el DAI generado o colateral liberado en la dirección `dst` **especifica**.
* `flux(bytes32 ilk, uint cdp, address dst, uint wad)`: Mueve la cantidad `wad` (precisión 18) de colateral `ilk` desde `cdp` a `dst`.
* `flux(uint cdp, address dst, uint wad)`: Mueve la cantidad `wad` de colateral `cdp` desde `cdp` a `dst`.
* `move(uint cdp, address dst, uint rad)`: Mueve la cantidad `rad` (precisión 45) de DAI desde `cdp` a `dst`.
* `quit(uint cdp, address dst)`: Mueve el colateral bloqueado y la deuda generada desde `cdp` a `dst`.

**Nota:** `dst` se refiere a la dirección de destino.

### Diseño de Almacenamiento

* `vat` : dirección del contrato principal que contiene las _vaults_.
* `cdpi`: id incremental automática.
* `urns`: Mapeo `CDPId => UrnHandler`
* `list`: Mapeo `CDPId => Prev & Next CDPIds` (lista doblemente enlazada)
* `owns`: Mapeo `CDPId => Owner`
* `ilks`: Mapeo `CDPId => Ilk` (tipo de colateral)
* `first` : Mapeo `Owner => First CDPId`
* `last`: Mapeo `Owner => Last CDPId`
* `count`: Mapeo `Owner => Amount of CDPs`
* `allows`: Mapeo `Owner => CDPId => Allowed Addr => True/False`

## 3. Mecanismos y Conceptos Clave
 
### Resumen

El Gestor de CDP fue creado como una forma de permitir que las _vaults_ sean tratadas mas como activos que puedan ser intercambiados. Originalmente, los principales contratos [dss](https://github.com/makerdao/dss/tree/master/src) no tenían la funcionalidad para permitir transferir posiciones de la _vault_. El Gestor CDP fue creado para _wrap_ (“enrollar”) esta funcionalidad y permitir la transferencia entre usuarios.     

### Objeto de Alto Nivel 

* `manager` recibe la dirección `vat` en su creación y actúa como un contrato interfaz entre este y los usuarios.   
* `manager` mantiene un registro interno de `id => owner` and `id => urn` permitiendo al `owner` (“dueño”) ejecutar las funciones `vat` para su `urn` mediante el `manager`. 
* `manager` mantiene una estructura de lista doblemente enlazada que permite la recuperación de todas las _vaults_ que un `owner` (“dueño”) tiene mediante llamadas _on-chain_. 
  * En resumen, para esto sirve `GetCdps`. Este contrato es un contrato de ayuda que permite la obtención de todas las _vaults_ en una sola llamada.  

### **Ejemplo de Uso de Gestor** de CDP **(ruta común):**

* Un usuario ejecuta `open` y obtiene un `CDPId` a cambio. 
* Después de esto, el `CDPId` se asocia con una `urn` con `manager.urns(cdpId)` y luego _`join`'s_ (“se une”) colateral a este. 
* El usuario luego puede ejecutar `frob` para elegir cual dirección `dst` quiere utilizar para enviar el DAI generado. 
* Si el usuario ejecuta `frob` sin `dst`, entonces el DAI generado se mantendrá en la `urn` de la _vault_. En este caso, el usuario puede utilizar `move` para moverlo más adelante.
* Ten en cuenta que este es el mismo proceso para el colateral que es liberado después del `frob` (para la función `frob`, que no requiere la dirección `dst`). El usuario puede utilizar _`flux`_ para enviarlo a otra dirección más adelante. 
* En el caso donde un usuario quiere abandonar el `manager`,  pueden utilizar `quit` (“renunciar”) como una forma de migrar la posición de su _Vault_ a otra dirección `dst`.

## 4. Gotchas (Posibles Fuentes de Error del Usuario)

* Para los desarrolladores que se quieren integrar con `manager`, necesitarán entender que las 'acciones de _vault_' están todavía en el ambiente `urn`. A pesar de esto, el `manager` intenta abstraer el uso de `urn` mediante un `CDPId`. Esto significa que los desarrolladores tendrán que obtener `urn` (`urn = manager.urns(cdpId)`) para permitir _`join`ing_ (“la unión”) del colateral con esa _vault_.   
* Como el `manager` asigna un `ilk` específico por `CDPId` y no permite que otros lo utilicen como propio, hay una segunda función `flux` que espera un parámetro `ilk`. Esta función tiene el simple propósito de tomar el colateral que fue enviado por error a una _vault_ que no pueda manejarlo / que sea incompatible.  
* **Función(es) _Frob_:**
  * Cuando utilizas `frob` en el Gestor de CDP, generas DAI nuevo en el `vat` mediante el Gestor de CDP que después es depositado en la `urn` que el Gestor CDP maneja. Este proceso depende de cuál función `frob` tilizas (existen **dos** funciones `frob`). En resumen, una permite una dirección de destino y la otra no la requiere.   
 * Si utilizas la función `frob` que tiene la dirección de destino (`dst`), estás diciendo que puedes enviar cualquier DAI generado o colateral que ha sido liberado. La segunda función `frob` está pensada para dejar el colateral en la dirección `urn` porque la `urn` es propiedad del Gestor de CDP. En este caso necesitarás utilizar las funciones `flux` o `move` manualmente para sacar el DAI o el Colateral. Estas funciones (`flux` y `move`) pueden ser más beneficiosas para un desarrollador que trabaje con la función _proxy_, ya que permite más flexibilidad. Por ejemplo, al usar estas funciones puedes mover una cantidad específica de colateral y puede causar que las otras funciones lo hagan. En general, trabajar con ella puede brindar un poco más de flexibilidad en cuanto a las necesidades específicas del desarrollador.         
* Como se menciono anteriormente en el resumen, el contrato principal [dss](https://github.com/makerdao/dss/tree/master/src) originalmente no tenía la funcionalidad de permitir transferir posiciones de _vaults_. Desde entonces, los contratos principales también han implementado una funcionalidad de transferencia nativa llamada _`fork`_ que permite transferir desde una _vault_ a otra direccion. Sin embargo, hay una restricción, que es que el dueño de la dirección que estará recibiendo la _vault_ necesita proporcionar autorización de que en efecto quiere recibirla. Esto fue creado para una situación en la que el usuario está transfiriendo colateral que está bloqueado, así como la deuda generada. Si simplemente estás moviendo colateral a otra dirección, no hay problema, pero en el caso de que también estés transfiriendo la deuda generada, hay una posibilidad de poner a una _vault_ perfectamente segura en una posición peligrosa. Esto hace que la funcionalidad del contrato sea un poco más restrictiva. Por lo tanto, el Gestor de CDP es una buena opción de mantener una forma simple de transferir _vaults_ y reconocerlas mediante una ID numérica.      

## 5. Modos de Fallo (Limites en las Condiciones de Funcionamiento y Factores de Riesgo Externos)

### **Problemas Potenciales Alrededor de la Reorganización de la _Chain_**

Cuando `open` es ejecutado, una nueva `urn` es creada y se le asigna un `cdpId` para un `owner` específico. Si el usuario utiliza `join` para agregar colateral a `urn` inmediatamente después de la transacción haya sido minada, hay una posibilidad de que ocurra una reorganización de la cadena. Esto podría resultar en que el usuario pierda la propiedad del par `cdpId`/`urn`, por lo tanto, pierda sus colaterales. Sin embargo, solo puede surgir este problema cuando se evita el uso de las [funciones _proxy_](https://github.com/makerdao/dss-proxy-actions) mediante un [perfil _proxy_](https://github.com/dapphub/ds-proxy) ya que el usuario `open` (“abrirá”) el `cdp` y `join` (“unirá”) colateral en la misma transacción.     