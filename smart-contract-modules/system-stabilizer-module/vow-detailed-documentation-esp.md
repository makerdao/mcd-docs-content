# Vow – Documentación Detallada 

* **Nombre del Contrato:** vow.sol
* **Tipo/Categoría:** DSS —> Modulo Estabilizador del Sistema
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato:**](https://github.com/makerdao/dss/blob/master/src/vow.sol)
* ****[**Etherscan**](https://etherscan.io/address/0xa950524441892a31ebddf91d3ceefa04bf454466)

## 1. Introducción (Resumen)

El contrato `Vow` representa la Hoja de Balance del Protocolo Maker. En particular, el `Vow` actúa como recipiente de tanto el excedente del sistema como de la deuda del sistema. Sus principales funciones son cubrir déficits mediante subastas de deuda (`Flop`) y liberar los excedentes mediante las subastas de excedentes (`Flap`).

![Interacción de los Contratos Vow.sol](https://raw.githubusercontent.com/makerdao/mcd-docs-content/master/.gitbook/assets/Screen_Shot_2019-11-04_at_5.34.15_PM.png)

**En la imagen pueden ver:**

* Llamadas inter-contractuales necesarias para el funcionamiento del módulo.

**No aparece en la imagen:**

* Llamadas `cage` desde `Vow` a `Flap` y `Flop`.
* Interacciones de subastas y usuarios de subastas .
* Llamadas de Gobernanza / interacciones.

## 2. Detalles del Contrato 

### **Vow (Glosario)**

* `sin`: la fila de espera de la deuda del sistema.  
* `Sin`: la cantidad total de deuda en la fila de espera. 
* `Ash`: la cantidad total de deuda en la subasta. 
* `wait`: longitud de la fila de espera de la deuda.
* `sump`:  tamaño de oferta de la subasta de deuda; es decir, la cantidad de deuda fija en ser cubierta por cualquier subasta de deuda.
* `dump`: tamaño del lote de la subasta de deuda; es decir, la cantidad inicial de MKR ofrecida para cubrir el `lot`/`sump`.
* `bump`: tamaño del lote de la subasta de excedente; es decir, la cantidad fija de excedentes que se venderán en cualquier subasta de excedentes.  
* `hump`: buffer de excedentes, debe ser superado antes de que se puedan llevar a cabo las subastas de excedentes.  

**Otros términos incluidos en el diagrama anterior:**

* `move`: transfiere _stablecoins_ entre usuarios.
* `kick`: inicia una subasta.

### **Administrador de Liquidaciones**

* **Fess** - Empuja deuda incobrable a la fila de espera de las subastas (añade deuda a la fila).
* **Flog** - Libera deuda de la fila de espera para la subasta (produce deuda de la fila).
* **Heal** - `vow` llama a `heal` en el contrato `vat` para cancelar el excedente y la deuda. (Optimiza el buffer de deuda (`vat.heal`)).
* **Kiss** - Cancela el excedente y la deuda en las subastas. Libera deuda de la subasta y _Heal_ (`vat.heal`).  
* **Flap** - Activa una subasta de excedentes (`flapper.kick`).
* **Flop** - Activa una subasta de déficit (`flopper.kick`).

#### **Autorización**

El contrato `vow` llama a `kick` en `flop` y `flap` para iniciar una subasta (subastas de deuda y subastas de excedentes, respectivamente).

* `Flopper` (Subastas de Deuda) – Si el déficit no es cubierto en la porción de la subasta a plazo, en la subasta `flip`, entonces las subastas de deuda son utilizadas para deshacerse de la **deuda** del Vow al subastar MKR por una cantidad fija de DAI. Una vez que la subasta termine, el `Flop`per enviará el DAI recibido hacia el `Vow` para cancelar su deuda. Por último, el `Flop`per acuñará MKR para el ganador de la subasta. 
* `Flapper` (Subastas de Excedentes) – Estas son utilizadas para deshacerse del **excedente** de `Vow` al subastar una cantidad fija de DAI por MKR. Una vez que la subasta termina, el `Flap`per quema el MKR ganado en la subasta y envía DAI interno al ganador.

#### **Datos** del Sistema

* **Configuración del sistema**

    `Vow.wait` - Retraso _Flop_ `Vow.sump` - Tamaño de oferta fija Flop

    `Vow.dump` - Tamaño inicial del lote _Flop_ `Vow.bump` - Tamaño del lote inicial _Flap_ `Vow.hump`- Buffer de Excedentes

#### Cola de Deuda (**SIN)**

Cuando una _vault_ es liquidada (`bite` - [documentación](https://docs.makerdao.com/smart-contract-modules/core-module/cat-detailed-documentation#bite-bytes32-ilk-address-urn)), la deuda embargada, se pone en fila de espera de una subasta en un `Vow` (etiquetado como `sin[marca de tiempo]`- la unidad de deuda del sistema). Esto ocurre en la marca de tiempo del bloque de la acción `bite`. Puede ser liberado para subasta mediante `flog` (`flog` libera la deuda en cola para la subasta) una vez que el tiempo permitido `Vow.wait` (el retraso flop) haya terminado.      
`Sin` es almacenado cuando está en la cola de la deuda, pero la deuda disponible para la subasta no se almacena en ningún sitio. Esto es porque la deuda que es elegible para subastar se obtiene de comparar el `Sin` (es decir, deuda en la fila de espera) con el balance de DAI del `Vow` registrado en `Vat.dai[Vow]`. Por ejemplo, si `Vat.sin[Vow]` es más grande que la suma de `Vow.Sin`  y el `Ash` (deuda en subasta actualmente), entonces la diferencia puede ser elegible para una subasta `Flop`.          

**Notas:**

* En el caso de que un `cat.bite` / `vow.fess` sea ejecutado, la deuda `tab` es añadida a `sin[now]` y `Sin`, lo que bloquea esa cantidad de `tab` para enviarla a la subasta `flop`  y se recupera todo el DAI con una subasta `flip`. En teoría, desbloquear la cantidad de `tab` en `Sin` no debería ser necesario, pero en la práctica en realidad sí lo es. Si esta deuda no es desbloqueada, entonces cuando tengamos la necesidad real de enviar una subasta `flop`, podríamos tener un gran `Sin` que lo bloquee. Para resumir, esto significa que cada registro de `sin[era]` que tenga una cantidad > 0 debería ser _`flog`'ed_ antes de impulsar una subasta `flop` (esto es porque para impulsar todo el asunto, necesitas que cada registro esté en 0, de lo contrario se estaría bloqueando la deuda).  
* No es necesario que cada `sin[era]` sea un solo `bite`, ya que se agruparán todos los `bite`’s que estén en el mismo bloque de Ethereum juntos. 
* El `auction-keeper` (_keeper_ de subasta) va a `flog` cada `era` con `Sin` positivo si el `woe`+ `Sin` >= `sump`, donde `woe` = `vat.sin[vow]` - `vow.Sin` - `vow.Ash`**.** Donde los componentes dentro de vat.sin(vow)` - `vow.Sin` - `vow.Ash` son definidos como: 
    * **`vat.sin(vow)`**- deuda incobrable total
    * **`vow.Sin`** - deuda bloqueada
    * **`vow.Ash`** - deuda en subastas
* `Vow.sin` registra porciones individuales de deuda (marcados con una marca de tiempo). Estas no son subastadas directamente, pero son borradas cuando se llama `flog`.
* Si el `Sin` no es cubierto mediante una subasta `flip` dentro del tiempo de espera designado (`tau`), el `Sin` “madura” y es marcado como deuda incobrable para el `Vow`. Esta deuda incobrable puede cubrirse a través de una subasta de deuda (`flop`) cuando supera un valor mínimo (el tamaño de `lot`). En resumen, el tiempo que transcurre entre que la deuda sea añadida a la cola `sin[]` y se convierta en “madura” (cuando utiliza `flog` en la cola y es elegible para la subasta `Flop`) es la cantidad de tiempo que la subasta `Flip` tiene para cubrir esa deuda. Esto es debido al hecho de que cuando una subasta `Flip` recibe <<DAI, disminuye el balance DAI de `Vow` en el `Vat`.   
* **Nota:** En este caso, hay un riesgo de que se produzca una situación en la que el `Vow.wait` es diferente al `Flip.tau`. El riesgo principal está relacionado con que `wait` < `tau`, lo que resultaría en que las subastas de deuda se ejecutaran antes de que las subastas de colateral incautado asociadas pudieran completarse. 

**El `Sin` general puede afectar al sistema de la siguiente manera:**

1. Puede suceder que cada `Vow` esté separado con su propio `sin`
2. En caso de una actualización, si es removido un `Vow` que tiene un `sin`, esto puede crear deuda incobrable sin seguimiento en el sistema. 

#### **Contabilidad**

`Vow.Sin` - Calcula la cola de deuda total en el sistema. `Vow.Ash`- Calcula la deuda total en subasta.

## 3. Mecanismos y Conceptos Clave

Es importante tener en cuenta que el Protocolo de Maker se desviará de su equilibrio. Esto ocurre cuando recibe deuda del sistema y excedentes del sistema a través de las subastas de colaterales y de la acumulación de tarifas de estabilidad provenientes de _vaults_. El contrato `Vow` contiene la lógica para activar tanto las subastas de deuda (`flop`) como las subastas de excedentes (`flap`), que funcionan para corregir los desequilibrios monetarios del sistema.   

**Resumen**

* **Deuda del Sistema:** en caso de que las _vaults_ sean ”bitten” (liquidadas), su deuda es asumida por el contrato `Vow` como un `Sin` (la unidad de deuda del sistema). Luego, la cantidad de `Sin` es colocada en la cola de `Sin`.**Nota:** cuando el `Sin` no es cubierto por una subasta `flip` (dentro del tiempo de espera `wait` dedicado) se considera que el `Sin` tiene deuda incobrable con el `Vow`. Esta deuda incobrable es cubierta a través de una subasta de deuda (`flop`) cuando excede un valor mínimo (el tamaño de `lot`).    
* **Excedentes del Sistema:** se producen por la acumulación de tarifas de estabilidad, resultando en DAI interno adicional en el `Vow`. Después, estos excedentes son liberados a través de una subasta de excedentes (`flap`).  

## 4. Gotchas (Fuentes potenciales de errores de usuario)

* Cuando el `Vow` es actualizado, hay múltiples referencias a el, que deben ser actualizadas al mismo tiempo (`End`, `Jug`, `Pot`).
* El `Vow` es el único usuario con un balance `Sin` igual a cero (no una invariante `vat`, ya que pueden haber múltiples `Vow`s).
* El almacenamiento Ilk se reparte a través de los módulos `Vat`, `Jug`, `Pot` y `Vow`. El `cat` también almacena las penalizaciones de liquidación y el tamaño máximo de subasta.   
* Una porción de la Tarifa de Estabilidad esta destinada a la Tasa de Ahorro de DAI (DSR por sus siglas en ingles) al aumentar la cantidad de `Sin` en el `Vow`, en cada llamada `Pot.drip( )`.
* Establecer un valor incorrecto para el `vow` puede causar que el excedente se pierda o sea robado.

## 5. Modos de Falla (Limites en las Condiciones de Funcionamiento y Factores de Riesgo Externos)

#### Liquidación de Vault

* Un modo de falla puede surgir si ningún actor llama a `kiss`, `flog` o `heal` para reconciliar/poner en la cola la deuda.

#### Subastas

* Un modo de falla podría surgir si un usuario no llama a `flap` o `flop` para iniciar las subastas.
* Cuando `Vow.wait` es fijado demasiado alto (`wait` es muy largo), las subastas `flop` ya no pueden ocurrir. Esto supone un riesgo de sub-colateralización. 
* Cuando `Vow.wait` es fijado demasiado bajo, puede provocar muchas subastas `flop` y evita que ocurran subastas `flap`.
* Cuando `Vow.bump` es fijado demasiado alto, puede resultar en que las subastas `flap` no se produzcan. Por lo tanto, si no hay subastas `flap`, no habrá ofertas de MKR como parte del proceso y, por lo tanto, no se quemará MKR automáticamente como resultado de una subasta exitosa. 
* Cuando `Vow.bump` es fijado demasiado bajo, las subastas `flap` no son rentables para los participantes (el tamaño de `lot` vale menos que el costo del gas). Por lo tanto, no se harán ofertas por MKR durante la subasta `flap` y como resultado, no habrá quema automática de MKR.  
* Cuando `Vow.sump` es fijado demasiado alto, las subastas `flop` no son posibles. Esto resulta en que el sistema no pueda recuperarse de un estado de sub-colateralización.  
* Cuando `Vow.sump` es fijado demasiado bajo, las subastas `flop` no son rentables para los participantes (donde el tamaño de `lot` vale menos que el costo del gas). Esto genera inflación del MKR, debido a la acuñación automática de MKR.
* Cuando `Vow.dump` es fijado demasiado alto, las subastas `flop` corren el riesgo de no poder cerrarse o de acuñar una gran cantidad de MKR, creando a su vez un riesgo de dilución de MKR y la posibilidad de un ataque a la Gobernanza.
* Cuando `Vow.dump` es fijado demasiado bajo, las subastas `flop` tienen que ser `kick`ed (activadas) demasiadas veces antes de que sean interesantes para los _keepers_.
* Cuando `Vow.hump` es fijado demasiado alto, las subastas `flap` nunca ocurrirán. Si una subasta `flap` no ocurre, no hay venta de excedente y, por lo tanto, no hay quema de las ofertas de MKR. 
* Cuando `Vow.hump` es fijado demasiado bajo, puede causar que excedentes sean subastados mediante subastas `flap`, antes de ser utilizados para cancelar el `sin` de las liquidaciones, haciendo necesario realizar subastas `flop` y generando un funcionamiento ineficiente del sistema. 