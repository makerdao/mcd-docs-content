# Acciones _Proxy_ - Documentación Detallada

* **Nombre del Contrato:** DssProxyActions.sol
* **Tipo/Categoría:** Módulo _Proxy_ 
* [**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* [**Fuente del Contrato**](https://github.com/makerdao/dss-proxy-actions/blob/master/src/DssProxyActions.sol)
* **Etherscan**
  * \*\*\*\*[**Acciones _Proxy_** ](https://etherscan.io/address/0x82ecd135dce65fbc6dbdd0e4237e0af93ffd5038)
  * \*\*\*\*[**Acciones _Proxy_ End** ](https://etherscan.io/address/0x069b2fb501b6f16d1f5fe245b16f6993808f1008)
  * \*\*\*\*[**Acciones _Proxy_ DSR**](https://etherscan.io/address/0x07ee93aeea0a36fff2a9b95dd22bd6049ee54f26)

## 1. Introducción \(Resumen\)
 
El contrato de Acciones _Proxy_ es una _wrapper_ ("envoltura") generalizada para el Protocolo de Maker. Es básicamente un conjunto de funciones _proxy_ para MCD \(usando el gestor-dss-cdp\). El propósito del contrato es ser utilizado mediante un _ds-proxy_ personal y puede ser comparado con el _Sag-Proxy_ original ya que ofrece la habilidad de ejecutar una secuencia de acciones automáticamente.  

## 2. Detalles del Contrato

### Glosario \(Acciones _Proxy_\)

`manager` - permite un proceso formalizado para que los CDPs sean transferidos entre usuarios.<

`ilk` - un tipo de colateral.

`usr` - dirección de usuario.

`cdp` - Posición de Deuda Colateralizada \(ahora conocida como `Vault`\).

`dst` - se refiere a la dirección de destino.

`wad` - cantidad de _tokens_, usualmente como un entero de punto fijo con 10^18v decimales. 

`rad` - un entero de punto fijo, con 10^45 decimales.  

`dink` - cambio en el colateral. 

`dart` - cambio en la deuda.

`ethJoin` - permite usar Ether nativo con el sistema.  

`gemJoin` - permite depositar _tokens_ ERC20 estándar para utilizarlos con el sistema. 

`daiJoin` - permite a los usuarios retirar sus DAI del sistema en un _token_ ERC20 estándar. 

### Funcionalidades Clave \(como se definen en el _smart contract_\)

### DssProxyActions

* `open()`: crea un `UrnHandler` \(`cdp`\) para la dirección `usr` \(para un `ilk` específico\) y permite al usuario gestionarlo mediante el registro interno del _`manager`_ ("gestor").
* `give()`: transfiere la propiedad del `cdp` a la dirección `usr` en el registro del _`manager`_.  
* `giveToProxy()`: transfiere la propiedad de `cdp` al _proxy_ de la dirección `usr`\(mediante `proxyRegistry`\) en el registro del _`manager`_.  
* `cdpAllow()`: permite/niega a la dirección `usr` gestionar el `cdp`.  
* `urnAllow()`: permite/niega que la dirección `usr` gestione la dirección `msg.sender` como `dst` para `quit`.
* `flux()`: mueve la cantidad `wad` de colateral desde la dirección `cdp` a la dirección `dst`.  
* `move()`: mueve la cantidad `rad` de DAI de la dirección `cdp` a la dirección `dst`.
* `frob()`: ejecuta `frob` a la disección `cdp` asignando el colateral liberado y/o DAI retirado a la misma dirección.  
* `quit()`: mueve el balance del colateral `cdp` y la deuda a la dirección `dst`.  
* `enter()`: mueve el balance del colateral `src` y la deuda a `cdp`.
* `shift()`: mueve el balance del colateral `cdpSrc` y la deuda a `cdpDst`.
* `lockETH()`: deposita la cantidad `msg.value` de ETH en el adaptador `ethJoin` y ejecuta `frob` a `cdp` incrementando el valor bloqueado.  
* `safeLockETH()`: igual que `lockETH`, pero requiere `owner == cdp owner`.
* `lockGem()`: deposita la cantidad de colateral `wad` en el adaptador `gemJoin` y ejecuta `frob` a `cdp` incrementando el valor bloqueado. 
* `safeLockGem()`: igual que `lockGem`, pero requiere`owner == cdp owner`.
* `freeETH()`: ejecuta `frob` a `cdp` disminuyendo el colateral bloqueado y retira la cantidad `wad` de ETH del adaptador `ethJoin`. 
* `freeGem()`: ejecuta `frob` a `cdp` disminuyendo el colateral bloqueado y retira la cantidad `wad` de ETH del adaptador `gemJoin`. 
* `draw()`: actualiza la tasa de tarifa del colateral, ejecuta `frob` a `cdp` incrementando deuda y saca la cantidad `wad` de token DAI \(acuñándolo\) del adaptador `daiJoin`.  
* `wipe()`: une la cantidad `wad` de token DAI al adaptador `daiJoin` \(quemándolo\) y ejecuta `frob` a `cdp` para que disminuya la deuda.
* `safeWipe()`: igual que `wipe`, pero requiere `owner == cdp owner`.
* `wipeAll()`: une toda la cantidad necesaria de token DAI al adaptador `daiJoin` \(quemándolo\) y ejecuta `frob` a `cdp` fijando la deuda en cero.  
* `safeWipeAll()`: igual que `wipeAll`, pero requiere `owner == cdp owner`. 
* `lockETHAndDraw()`: combina `lockETH` y `draw`.
* `openLockETHAndDraw()`: combina `open`, `lockETH` y `draw`.
* `lockGemAndDraw()`: combina `lockGem` y `draw`.
* `openLockGemAndDraw()`: combina `open`, `lockGem` y `draw`.
* `wipeAndFreeETH()`: combina `wipe` y `freeETH`.
* `wipeAllAndFreeETH()`: combina `wipeAll` y `freeETH`.
* `wipeAndFreeGem()`: combina `wipe` y `freeGem`.
* `wipeAllAndFreeGem()`: combina `wipeAll` y `freeGem`.

### **DssProxyActionsFlip**

* `exitETH()`: saca la cantidad `wad`de ETH del adaptador `ethJoin`. Esto es recibido en la _urn_ `cdp` después que la subasta de liquidación ha terminado. 
* `exitGem()`: saca la cantidad `wad` de colateral del adaptador `gemJoin`. Esto es recibido en la _urn_ `cdp` después que la subasta de liquidación ha terminado. 

### **DssProxyActionsEnd**

* `freeETH()`: una vez que el sistema está enjaulado, este recupera el ETH restante del `cdp` \(paga la deuda restante, si existe\).
* `freeGem()`: una vez que el sistema está enjaulado, este recupera el _token_ restante del `cdp` \(paga la deuda restante, si existe\).  
* `pack()`: una vez que el sistema está enjaulado, se empaqueta esta cantidad _`wad`_ de DAI para que esté lista para cobrar. 
* `cashETH()`: una vez que el sistema está enjaulado, este cobra la cantidad _`wad`_ de DAI previamente empaqueteda y devuelve el equivalente en ETH.  
* `cashGem()`: una vez que el sistema está enjaulado, este cobra la cantidad _`wad`_ de DAI previamente empaquetada y devuelve el equivalente en token _gem_.  

### **DssProxyActionsDsr**

* `join()`: une la cantidad `wad` de token DAI al adaptador _`daiJoin`_ \(quemándolo\) y mueve el balance al `pot` para las Tasas de Ahorro DAI.  
* `exit()`: recupera la cantidad `wad` de DAI del `pot` y saca token DAI del adaptador `daiJoin` \(acuñandolo\).
* `exitAll()`: realiza las mismas acciones que `exit` pero para toda la cantidad disponible. 

## 3. Mecanismos y Conceptos Clave

El `dss-proxy-actions` fue diseñado para ser utilizado por el _Ds-Proxy_, que es propiedad individual de los usuarios para interactuar más fácilmente con el Protocolo de Maker. Ten en cuenta que no está destinado a utilizarse directamente \(esto será tratado más adelante\). El contrato `dss-proxy-actions` fue desarrollado para servir como una biblioteca para los _ds proxies_ de los usuarios. En general, los _ds proxy_ reciben dos parámetros: 

* **Dirección de la biblioteca _Proxy_**
  * En este caso, la biblioteca de acciones _dss proxy_
* **Datos de la llamada**
  * Funciones y parámetros que se desean ejecutar 

**Para más información, consulta el _ds-proxy_** [**aquí.**](https://github.com/dapphub/ds-proxy)

#### **Resumen DSProxy \(en relación con el contrato _dss-proxy-actions_\)**

El propósito del _ds-proxy_ es ejecutar transacciones y secuencias de transacciones por _proxy_. El contrato permite la ejecución de código utilizando una identidad persistente. Esto puede ser muy útil para ejecutar una secuencia de acciones atómicas. Dado que el dueño del _proxy_ puede cambiarse, esto permite modelos de propiedad dinámicos; es decir, _multisig_. 
* En el próximo ejemplo, veremos cómo la **función _execute_** utiliza el _proxy_ para ejecutar: calldata \_data on contract \_code.
* **Los parámetros de la función son:**
  * address \_target
  * bytes memory \_data
* Para la _address-target_ que pases, esa será la biblioteca que deseas utilizar, en este caso la biblioteca de acciones _proxy_. 
* Para la _Memory data_ que pases, esos serán los datos de llamada de lo que deseas ejecutar exactamente.  
  * **Ejemplo:** deseas abrir una _vault_, entonces los bytes de _memory data_ que pasarás serán un decodificador ABI que ejecuta la función _open_ con ciertos parámetros.  

* **Nota:** Esto se usa para ambos SCD y MCD.  

#### **Ejemplo de Uso de la Acción _Proxy_ \(Cómo se ve una llamada _proxy_\)**

```text
proxy.execute(dssProxyActions, abi.encodeWithSignature("open(address,bytes32,address)", address(manager), bytes32("ETH-A"), address(0x123)))
```

* Tu _ds-proxy_ es solo para tí, lo creamos para una _wallet_, así que cada _wallet_ tiene su propio _ds proxy_ que nadie más debería ser capaz de ejecutar con ese _proxy_.
* En MCD, las _vaults_ no serán propiedad de tu _wallet_ sino de tu _ds proxy_, que te permite ejecutar cualquier función mediante el _ds proxy_, así como realizar acciones dentro de tu _vault_ y/o agrupar muchas acciones dentro de una sola transacción.  
* **La ejecución se ve así:**
  * Ejecución de _Proxy_ \(llamada a la _blockchain_ de Ethereum\) donde el primer parámetro es el contrato que estés utilizando para la biblioteca \(en este caso, _dss proxy actions_\). No es que esto sea algo que el _frontend_ vaya a hacer por ti. **Ejemplo:** Cuando deseas abrir una _vault_, enviará la transacción al _proxy_ para ejecutar la función _execute_ y luego pasará la dirección _dss proxy action_; y el segundo parámetro que pasará es la misma función que tu _ds proxy_ necesita para ejecutar desde las _dss proxy actions_. En este caso, queremos ejecutar la función _open_ del _dss proxy action_ - así que tu _proxy_ delegará la llamada a la función _open_ de la biblioteca de _dss proxy actions_. Necesitamos hacerlo de esta forma porque el segundo parámetro es el parámetro de formato del _call data_ de bytes, por lo que esta función nosotros la pasamos por _ABI Encode_ con la firma _abierta_. Entonces pasamos la firma y después los parámetros actuales que queremos pasar a esta función. En este caso: el gestor, el primer parámetro de función _open_, el tipo de colateral y la dirección para la que deseas crear la _vault_ \(En este caso, la dirección es 0x123\)

 * **Nota:** _UI_ decide qué acción _proxy_ utilizará el usuario.

## 4. **Gotchas**
   
* **Utilizar _dss-proxy-actions_ directamente puede resultar en la perdida de control de tu _vault_.** 
  * Si abres una nueva _vault_ mediante las _dss proxy actions_\(centralizadas\) sin un _ds proxy_ estarías creando una _vault_ que es propiedad de las _dss proxy actions_ que cualquiera puede llamar públicamente. Sería propiedad del contrato de _dss proxy actions_ y cualquiera podría ejecutar acciones en tu _vault_. Por lo tanto, hay un riesgo significativo si utilizas las _dss proxy actions_ directamente.  
* Cuando interactúas con las _dss-proxy-actions_ necesitas un cierto permiso para obtener fondos en DAI o MKR de la _wallet_ del usuario. Necesitas permiso de tu _wallet_ al _ds-proxy_ \(no _dss-proxy-actions_\). Porque cuando ejecutas las _dss-proxy-actions_, en realidad estás realizando esa acción en el entorno de tu _ds-proxy_, que es delegar llamadas o importar la función desde las _proxy actions_ y no ejecutarlas directamente. 

## 5. **Modos de Falla**

* **_Ds proxy_ es un _proxy_ de propósito general**
  * Esto significa que como usuario del _ds-proxy_, puedes ejecutar lo que quieras, sean _Dss-proxy-actions_ o cualquier pieza de código. Por lo tanto, los usuarios son responsables de lo que estén ejecutando y necesitan tener confianza en la _UI_ que están usando \(similar a cualquier otra transacción que estás ejecutando desde tu _wallet_\).
  * En términos de modos de falla, esto significa que puedes ejecutar una _proxy action_ maliciosa, así como una acción directa que podría potencialmente enviar tus ETH a una dirección aleatoria. Para ser precavido, debes revisar los datos de llamada de tus _wallets_ y/o auditar lo que hace tu _wallet_, ya que podrían presentar a los usuarios datos de llamadas aleatorias no deseadas, así como ejecutar acciones no deseadas.  
  * **En líneas generales, este es el momento de decirte que siempre hay un riesgo cuando se utiliza un _ds proxy_** 