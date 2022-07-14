# _Flapper_ - Documentación Detallada

* **Nombre del Contrato:** flap.sol
* **Tipo/Categoría:** DSS —> System Stabilizer Module
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/flap.sol)
* ****[**Etherscan**](https://etherscan.io/address/0xc4269cc7acdedc3794b221aa4d9205f564e27f0d#code)

## 1. Introducción (Sumario)

**Sumario:** _Flapper_ es una Subasta de Excedentes. Estas subastas se utilizan para subastar un monto fijo de DAI excedente en el sistema por MKR. Este DAI excedente proviene de las **Tarifas de Estabilidad** que se acumulan en las _vaults_. En este tipo de subasta, los postores compiten con grandes cantidades de MKR. Una vez que la subasta finaliza, el DAI subastado se envía al postor ganador. Luego, el sistema quema el MKR recibido del postor ganador.

![Interacciones de la _Flap_ con el Protocolo de Maker](https://github.com/makerdao/mcd-docs-content/raw/master/.gitbook/assets/Screen%20Shot%202019-11-17%20at%202.14.34%20PM.png)

## 2. Detalles del Contrato

### _Flapper_ (Glosario)

* `Flap` - Subasta de Excedentes (vendiendo _stablecoins_ por MKR) \[contrato]
* `wards [usr: address]` - `rely`/`deny`/`auth` Mecanismos de autorización \[uint - "entero no asignado"]
* `Bid` - Estado de una subasta específica\[apuesta]
  * `bid` - cantidad que es ofrecida por el `lot` (lote de MKR) \[uint]
  * `lot` - monto del _lot_ (lote de DAI) \[uint]
  * `guy` - postor más alto \[_address_]
  * `tic` - caducidad de la apuesta \[uint48]
  * `end` - cuándo va a finalizar la subasta \[uint48]
* `bids (id: uint)` - almacenamiento de todos los `Bid`s por `id` \[mapeo]
* `vat` - almacenamiento de las _address_ de los Vat \[_address_]
* `ttl` - tiempo de vida de la apuesta / tiempo máximo de duración de la apuesta (por defecto: 3 horas) \[uint48]
* `lot` - monto del _lot_ (lote de DAI) \[uint]
* `beg` - incremento mínimo de apuesta (por defecto: 5%) \[uint]
* `tau` - duración máxima de la subasta (por defecto: 2 días) \[uint48]
* `kick` - empezar una subasta / poner un nuevo `lot` de DAI para la subasta \[función]
* `tend` - hacer una apuesta, aumentando así el tamaño de la oferta / presentar una apuesta en MKR (aumento de la `bid`) \[función]
* `deal` - reclamo de una apuesta ganadora / resolver una subasta finalizada \[función]
* `gem` - Token MKR \[_address_]
* `kicks` - recuento total de la subasta \[uint]
* `live` - bandera de jaula \[uint]
* `file` - usado por la Gobernanza para establecer `beg`, `ttl` y `tau` \[función]
* `yank` - es utilizado durante un Asentamiento Global para mover la subasta de la fase `tend` al `End` recuperando el colateral y pagando el DAI al mejor postor. \[función]
* `tick()` - reestablece el valor `end` si no hubo ninguna apuesta (0 `bid`s) y el `end` original ya ha pasado.

### **Parámetros establecidos por la Gobernanza**

Los votantes de la Gobernanza de Maker determinan el límite del excedente. Las subastas de excedentes son activadas cuando el sistema tiene una cantidad de DAI superior a ese límite establecido.

* **Parámetros establecidos a través de `file`:**
  * `beg`
  * `ttl`
  * `tau`

**Nota:** La Gobernanza del MKR también determina el `Vow.bump`, el cual establece el `Bid.lot` para cada Subasta _Flap_ y el `Vow.hump` que determina el _buffer_ de excedentes.

### Autorizaciones

`auth` - chequea si una _address_ puede llamar a este método \[función modificadora]

* `rely` - permite a una _address_ a llamar métodos de autorización (`auth`) \[función]
* `deny` - rechaza a una _address_ a llamar métodos de autorización (`auth`) \[función]

## 3. Conceptos y Mecanismos Claves

El mecanismo comienza con los poseedores de MKR del sistema (los votantes de la Gobernanza de Maker). Los poseedores de MKR especificarán la cantidad de excedente permitido en el sistema a través del sistema de votación. Una vez que se llega a un acuerdo sobre lo que se tiene que establecer, las subastas de excedentes son activadas cuando el sistema tiene un excedente en DAI mayor a la cantidad decidida durante la votación. El excedente del sistema es determinado en `Vow` y cuando `Vow` [no tiene deuda en el sistema](https://github.com/makerdao/dss/blob/master/src/vow.sol#L128) y ha acumulado suficiente DAI para [superar el tamaño de la Subasta de Excedente (`bump`) más el _buffer_ (`hump`)](https://github.com/makerdao/dss/blob/master/src/vow.sol#L127).

Para determinar si el sistema tiene excedente neto, se deben conciliar tanto los ingresos como la deuda del sistema. Resumiendo, cualquier usuario puede hacer esto enviando la [transacción de recuperación](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module/vow-detailed-documentation#liquidations-manager-esp) al contrato del sistema denominado "_Vow_". Siempre que exista un excedente neto en el sistema, la subasta de excedentes comenzará cuando cualquier usuario envíe la transacción **flap** al [contrato de _Vow_](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module/vow-detailed-documentation-esp).

Una vez comenzada la subasta, un monto fijado (`lot`) de DAI se pone a la venta. Los postores compiten por esta cantidad de `lot` de DAI fijado con cantidades de `bid`s (apuestas) en MKR crecientes. En otras palabras, esto significa que los postores seguirán colocando montos de oferta de MKR en incrementos mayores que el monto mínimo de aumento de la oferta que se ha establecido (este es el `beg` en acción). 

La subasta de excedentes termina oficialmente cuando la duración de apuesta termina (`ttl`) sin que se haga otra oferta **O** cuando se haya alcanzado la duración de una subasta (`tau`). Cuando al subasta finaliza, el MKR recibido por el excedente de DAI, es enviado para ser quemado, disminuyendo así el suministro total de MKR.

![Un diagrama que detalla las interacciones de un usuario con la _Flapper_ y el _Vow_.](https://i.imgur.com/Lx53O0j.png)

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

#### **_Keepers_**

En el contexto de que se esté dirigiendo un _keeper_ (puedes encontrar más información [aquí](https://github.com/makerdao/developerguides/tree/master/keepers)), para poder realizar ofertas dentro de una subasta, un modo de falla principal podría ocurrir cuando un _keeper_ especifica un precio no rentable para MKR.

* Este modo de falla se debe a que no hay nada que el sistema pueda hacer si un usuario desea pagar una suma significativamente superior a la del mercado por un token en una subasta (esto aplica para todos los tipos de subastas, `flip`, `flop` y `flap`).
* Los _keepers_ que no se están desempeñando bien en una subasta `flap`, corren el riesgo de pagar MKR de más por el DAI ya que no hay un límite máximo del tamaño de la `bid` (oferta) más que el saldo de MKR.

#### **Incrementos de la `Bid` en una Subasta**

Durante `tend`, los montos de `bid` se incrementarán por un porcentaje de `beg` con cada nuevo `tend`. El postor debe conocer el `id` de la subasta, especificar el monto correcto del `lot` para la subasta, ofertar al menos un % de `beg` más que la última oferta y debe tener un saldo de MKR suficiente.

Un riesgo puede ser el _"front-running"_ o mineros maliciosos. En este escenario, la oferta de un _keeper_ honesto \ (oferta anterior + % de `beg`) se comprometería después de la oferta del _keeper_ deshonesto por lo mismo, evitando así que la oferta del _keeper_ honesto sea aceptada y obligándolos a volver a ofertar con un precio más alto ((Oferta anterior + % de `beg`) + % de `beg`). El _keeper_ deshonesto tendría que pagar tarifas de _gas_ más altas para tratar de que un minero coloque su transacción primero o se ponga de acuerdo con un minero para asegurarse de que su transacción sea la primera. Esto podría volverse especialmente importante a medida que la oferta alcanza la tasa del mercado actual para MKR<>DAI.

**Ejemplo Rápido**:

El `beg` podría establecerse en un 3%, significando que el postor actual ha realizado una oferta de 1 MKR; entonces, la próxima oferta debería de ser de, al menos, 1.03 MKR. En general, el propósito del sistema de incremento de oferta se debe a que se busca incentivar la oferta prematura y hacer que el proceso de la subasta fluya rápidamente.

#### Realizar ofertas incorrectamente

Los postores envían tokens MKR de sus _addresses_ al sistema/subasta específica. Si una oferta es superada por otra, la oferta perdedora se reembolsa a la _address_ del postor. Es importante destacar que, sin embargo, una vez que se hace una oferta, esta no puede cancelarse. La única forma posible de recuperar dicha oferta es si se sobrepuja (o si el sistema entra en un Asentamiento Global).

### **Ilustración del flujo de las ofertas/apuestas:**

1. El _Vow_ `kick` (realiza) una nueva Subasta _Flap_.
2. El Postor 1 envía una oferta (MKR) que incrementa el `bid` por arriba del valor inicial 0 que se estableció durante el `kick`. El saldo de MKR del Postor 1 disminuye y el balance de la _Flap_ se incrementa por el monto de la oferta. Se restablece el `bid.guy` del _address_ del _Vow_ a la del Postor 1 y `bid.tic` se restablece a `now` (ahora) + `ttl`.
3. El Postor 2 hace una oferta que supera a la del Postor 1 de, por lo menos, el monto del `beg`. El saldo de MKR del Postor 2 disminuye y el saldo del Postor 1 se incrementa por el monto de la `bid` que realizó el Postor 1. La diferencia entre la `bid` del Postor 2 y el Postor 1 es enviada por el Postor 2 a la _Flap_.
4. Luego, el Postor 1 hace un `bid` superior a la del Postor 2 de, por lo menos, el monto del `beg`. El saldo de MKR del Postor 1 disminuye y el saldo del Postor 2 se incrementa por el monto de la `bid` que realizó el Postor 2. El monto por el cual el Postor 1 incrementó la `bid` es enviado por el Postor 1 a la _Flap_.
5. El Postor 2, al igual que el resto de postores que estén participando en la subasta, deciden que ya no vale la pena seguir apostando más, por lo que dejan de ofertar. Una vez que el `Bid.tic` expira, el Postor 1 llama a la función `deal` y los tokens de DAI excedentes son enviados a la _address_ del postor ganador (el Postor 1, en este ejemplo) en el `Vat` y, entonces, el sistema quema el MKR recibido del postor ganador. `gem.burn(address(this), bids[id].bid)`.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

#### 1. Ve la [documentación del Módulo Estabilizador del Sistema](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module-esp)

#### 2. Otros Modos de Falla

* **Resultado del quemado de MKR**
  * Existe la posibilidad de que surja una situación en la que el token MKR haga que la transacción se revierta (por ejemplo, se detiene o se revoca el permiso de _Vow_ para llamar a _burn()_). En un caso como este, el trato no puede tener éxito hasta que alguien solucione el problema con el token MKR. En caso de interrupción, esto podría incluir el despliegue de un nuevo token MKR. Esta nueva implementación podría ser completada por cualquier persona que utilice el sistema MCD, pero la Gobernanza tendría que agregarlo al sistema. A continuación, tendría que reemplazar las antiguas subastas de excedentes y de deudas por las nuevas utilizando el nuevo token MKR. Por último, es crucial habilitar la posibilidad de votar también con la nueva versión.

* **Cuando hay una gran cantidad de excedente**
  * Esto daría lugar a muchas subastas _Flap_, ya que el excedente sobre `bump` + `hump` siempre se subasta en incrementos de `bump`. Sin embargo, las subastas se realizan al mismo tiempo, por lo que esto "inundaría el mercado de _keepers_" y, posiblemente, daría lugar a que se hicieran muy pocas ofertas en cualquier subasta. Esto podría ocurrir porque los _keepers_ no pujen en varias subastas a la vez, lo que provocaría una congestión en la red porque todos los _keepers_ intentan pujar en todas las subastas. Esto también podría dar lugar a una posible colusión entre los _keepers_ (si la _pool_ de capital es lo suficientemente grande, es posible que estén más dispuestos a trabajar juntos para dividirlo equitativamente a expensas del sistema).