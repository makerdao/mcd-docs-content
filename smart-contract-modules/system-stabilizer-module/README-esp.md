# Módulo Estabilizador del Sistema

* **Nombre del Modulo:** Estabilizador del Sistema 
* **Tipo/Categoría:** DSS —> Modulo Estabilizador del Sistema (Vow.sol, Flap.sol, Flop.sol)
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* **Fuente del Contrato:**&#x20;
  * ****[**Vow**](https://github.com/makerdao/dss/blob/master/src/vow.sol)
  * ****[**Flop** ](https://github.com/makerdao/dss/blob/master/src/flop.sol)
  * ****[**Flap** ](https://github.com/makerdao/dss/blob/master/src/flap.sol)

## 1. Introducción (Sumario)

El propósito del Módulo Estabilizador del Sistema es el de corregir el sistema cuando el valor del colateral que respalda el DAI cae por debajo del nivel de liquidación (determinado por la Gobernanza) cuando la estabilidad del sistema está en riesgo. El Módulo Estabilizador del Sistema crea incentivos para que los _Keepers_ de las Subastas (actores externos) intervengan y lleven al sistema devuelta a un estado seguro (equilibrio del sistema) participando en las subastas de deuda y de excedentes, y que a su vez obtengan beneficios al hacerlo. 

Para entender mejor cómo funcionan los _Keepers_ de las Subastas en relación al Mecanismo Estabilizador del Sistema, puedes leer los siguientes artículos: 

* [Subastas y _Keepers_ en el MCD 101](https://github.com/makerdao/developerguides/blob/master/keepers/auctions/auctions-101.md)
* [Cómo ejecutar tu propio Bot _Keeper_ de Subastas en MCD](https://blog.makerdao.com/how-to-run-your-own-auction-keeper-bot-in-mcd/)
* [Guía de Configuración para Desarrolladores: Bot _Keeper_ de Subastas](https://github.com/makerdao/developerguides/blob/master/keepers/auction-keeper-bot-setup-guide.md)

## 2. Detalles del Modulo

El Módulo Estabilizador del Sistema tiene tres componentes principales que consisten en los contratos `Vow`, `Flop` y `Flap`.

#### Documentación de los Componentes del Estabilizador del Sistema

* [**Documentación Vow**](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module/vow-detailed-documentation-esp)
* [**Documentación Flop**](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module/flop-detailed-documentation-esp)
* [**Documentación Flap**](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module/flap-detailed-documentation-esp)

## 3. Mecanismos y Conceptos Claves

#### Resumen de los **Componentes del Módulo Estabilizador del Sistema**

1. `Vow` representa el **balance** general del Protocolo de Maker (tanto el sistema de excedentes como el sistema de deuda). El propósito de `vow` es el de cubrir los déficits mediante las subastas de deuda y liquidar excedentes mediante las subastas de excedentes.  
2. `Flopper` (Subasta de Deuda) es utilizado para deshacerse de la deuda del `Vow` subastando MKR por una cantidad fija de DAI del sistema interno. Cuando inician las subastas `flop`, los participantes compiten con ofertas decrecientes de MKR. Después de la finalización de la subasta, el Flopper envía el DAI interno recibido hacia el `Vow` para cancelar su deuda. Por último, el Flopper acuña el MKR del ganador de la subasta. 
3. `Flapper` (Subasta de Excedentes) es utilizado para deshacerse del excedente del `Vow` subastando una cantidad fija de DAI por MKR. Cuando las subastas `flap` inician, los participantes compiten con cantidades crecientes de MKR. Después de la finalización de la subasta, el Flapper quema el MKR obtenido  y envía DAI interno al ganador de la subasta.    

#### Cómo el Módulo Estabilizador del Sistema interactúa con otros Módulos DSS

![](<../../.gitbook/assets/Screen Shot 2019-11-12 at 11.28.41 PM.png>)

#### Cómo los contratos `Vow`, `Flop` y `Flap` ayudan al funcionamiento del sistema MCD

![](<../../.gitbook/assets/Screen Shot 2019-11-12 at 11.33.23 PM.png>)

#### Vow

Cuando el Protocolo de Maker recibe deudas del sistema y excedentes del sistema a través de las subastas de colateral y la acumulación de la tasa de estabilidad, se desvía del equilibrio del sistema. El trabajo del Vow es traerlo de vuelta al equilibrio del sistema. 

**La Prioridad del Vow**

1. Poner en marcha las subastas de excedentes (Flop y Flap) que, a su vez, corrigen los desequilibrios del sistema.  

#### Flop

El propósito de las subastas de deuda es cubrir el déficit del sistema, que se asemeja a `Sin` (la deuda total en la fila de espera). Vende una cantidad de MKR acuñado y compra Dai que será cancelado 1 a 1 con `Sin`.

**Prioridades del _Flopper_:**

1. Acumular una cantidad de DAI equivalente a la cantidad de deuda incobrable lo más rápido posible.  
 
2. Minimizar la cantidad de inflación de MKR 

![Un diagrama que detalla las interacciones de un usuario con el Flopper y el Vow.](../../.gitbook/assets/flop\_auction\_interaction\_diagram.png)

#### Flap

El propósito de las subastas de excedentes es el de liberar DAI excedente del `Vow` mientras los usuarios ofertan con MKR, que será quemado, reduciendo la oferta total de MKR. Venderá una cantidad fija de DAI para comprar y quemar MKR.

**La Prioridad del Flapper:**

1. Reducir mecánicamente la oferta de MKR al subastar los excedentes de DAI.

![Un diagrama que detalla las interacciones de un usuario con el Flapper y el Vow.](../../.gitbook/assets/Flap\_auction\_interaction\_.png)

## 4. Gotchas (Posibles Fuentes de Error del Usuario)

#### **Keepers** de Subastas

En el contexto de la ejecución de un _Keeper_ de Subastas para realizar ofertas dentro de una subasta, un modo de fallo primario ocurriría cuando un _Keeper_ especifica un precio que no es rentable para MKR (más información [aquí](https://github.com/makerdao/developerguides/tree/master/keepers)). 

* Este modo de fallo es debido al hecho de que no hay nada que el sistema pueda hacer para evitar que un usuario pague significativamente más del precio justo de mercado por el token en una subasta (esto se aplica en todos los tipos de subastas: `flip`, `flop`, y `flap`).
* Los _Keepers_ que tienen un mal rendimiento están en riesgo principalmente durante la fase `dent`, ya que podrían devolver mucho colateral al CDP original y acabar pagando de más (es decir, pagar demasiado DAI (`bid`) por muy pocas _gems_ (`lot`)).     
  * **Nota:** Esto no aplica al contrato Flap (solo Flop y Flip)

**Cuando ningún _Keeper_ de Subastas oferta**

* Tanto para las subastas `Flip` como para las subastas `Flap`, la función `tick` reiniciará una subasta si han habido 0 ofertas y ha pasado el `end` original.  
* En caso de que una subasta `Flop` expire sin recibir ninguna oferta, cualquiera puede reiniciar la subasta llamando a `tick`. Junto con el reinicio de la subasta, también se necesita un incremento del tamaño inicial de `lot` por `pad` (a 50% por defecto). Este proceso extra se debe a que la falta de ofertas por el `lot` podría ser debido a circunstancias del mercado, donde el valor de `lot` es demasiado bajo y ya no es suficiente para recuperar la cantidad de `bid`.

## 5. Modos de Falla (Limites en las Condiciones de Funcionamiento y Factores de Riesgo Externos)

### Contrapartidas de la Gobernanza 

* `beg`
  * Si es demasiado alto, desalienta las ofertas y genera un mecanismo de subasta menos eficiente.  
  * Si es demasiado bajo, no sería tan grave, pero fomentaría ofertas más pequeñas y aumentaría la probabilidad de que llegue el `Bid.end`  antes de encontrar el precio _"verdadero"_. 
* `ttl`
  * Si es demasiado alto, podría causar que el ganador de la subasta espere demasiado para cobrar sus ganancias (dependiendo de la situación, posiblemente sometiéndolo a la volatilidad del mercado).  
  * Si es demasiado bajo, aumentaría la probabilidad de que las ofertas caduquen antes que otros participantes tengan la oportunidad de responder.   
* `tau`
  * Si es demasiado alto, no tendría un impacto tan significativo, ya que las subastas seguirían funcionando con normalidad basándose en el `Bid.tic`.
  * Si es demasiado bajo, aumentaría la posibilidad de que una subasta termine antes de encontrar el precio _”verdadero"_.

#### Otros Modos de Falla no tan comunes (Esto es para todos los tipos de subasta: Flip, Flop, Flap)

**Posibles Problemas de Congestión en la Red**

Cuando se subastan tokens de colateral, las ofertas finalizan después de que haya transcurrido un intervalo de tiempo (`ttl`). Por lo tanto, en el caso de que haya una congestión extrema en la red, el `ttl` y las subastas se ven afectadas porque les puede tomar mas de tres horas confirmar una transacción. Por lo tanto, debido a la congestión de la red de Ethereum, esto puede dar a lugar a que las subastas se liquiden por menos que el valor justo de mercado. Debido a este potencial problema, los participantes de la subasta necesitan calcular los riesgos de la congestión de red cuando hacen sus ofertas. 

**Ejemplo:**

Cuando se produce una gran congestión en la red, cuando las APIs comunes podrían cerrarse debido al rechazo de servicio (DoS) y cuando los precios del gas son altos, puede ser que ofertar se vuelva extremadamente caro, independiente de que las ofertas ganen o pierdan. Desafortunadamente, esto también genera que solo los que tengan la experiencia y el conocimiento técnico para construir sus propias transacciones puedan usar y participar en las subastas. Por lo tanto, las subastas finalizan por debajo del valor justo de mercado.

**Fuente de la Sección de Auditoría:** La dependencia de la marca de tiempo en subastas permite la denegación del servicio [(Pagina  8)](https://github.com/makerdao/audits/blob/master/mcd/trail-of-bits.pdf)

**Reordenamiento de Transacciones / Ataques _Front Running_ a las Subastas**

 Los ataques _Front Running_ son posibles. Para disminuir su probabilidad, revisamos y evaluamos otras opciones de subastas como la [holandesa](https://www.investopedia.com/terms/d/dutchauction.asp) y/o _commit reveal_, pero actualmente no creemos que valga la pena retrasar todo el sistema por este cambio. Debido a la naturaleza modular del sistema y al hecho de que los módulos de subastas puedan ser cambiados, será posible para la Gobernanza actualizar el proceso de subastas en el futuro.    