# Glosario MCD

## 1. [Glosario General de Términos del MCD](https://makerdao.world/en/learn/Glossary/)

## 2. Glosario Principal de _Smart Contracts_ de MCD:

### **General**

* `guy`, `usr`: una _address_.
* `wad`: una cantidad de tokens, usualmente, como un entero de 18 decimales.
* `ray`: un número entero con 27 decimales.
* `rad`:  un número entero con 45 decimales.
* `file`: administra algunos valores de configuración.

### **Autorizaciones**

* `auth`: chequea si una _address_ puede llamar a este método.
* `ward`: una _address_ la cual tiene permitido llamar a los métodos de autorización.
* `rely`: permite a una _address_ llamar a los métodos de autorización.
* `deny`: rechaza a una _address_ que pueda llamar a los métodos de autorización.
* `Authority` - chequea si una _address_ puede llamar a este método.
* `Kiss` - cancela las subastas de excedentes y de deudas.

### **_Vat_ (Motor de _Vault_)**

* `gem`: tokens de colaterales.
* `dai`: tokens de _stablecoins_.
* `sin`: _stablecoins_ no respaldadas (deuda del sistema que no pertenece a ninguna `urn`).
* `ilk`: un tipo de colateral.
  * `rate`: multiplicador de deuda de _stablecoins_ (tarifas de estabilidad acumuladas).
  * `take`: multiplicador del balance de colaterales.
  * `Ink`: saldo total de colaterales.
  * `Art`: deuda normalizada total de _stablecoins_.
* `init`: crea un nuevo tipo de colateral.
* `urn`: una _vault_ específica.
  * `ink`: balance del colateral.
  * `art`: deuda pendiente normalizada de _stablecoins_.
* `slip`: modifica el saldo del colateral de un usuario.
* `flux`: transfiere colaterales entre usuarios.
* `move`: transfiere _stablecoins_ entre usuarios.
* `grab`: liquida una _vault_.
* `heal`: crea / destruye iguales cantidades de _stablecoin_ y de deuda del sistema (`vice`).
* `fold`: modifica el multiplicador de deuda, creando / destruyendo la deuda correspondiente.
* `toll`: modifica al multiplicador de colaterales, creando / destruyendo el colateral correspondiente.
* `suck`: acuña _stablecoins_ no respaldadas (contabilizado con `vice`).
* `spot`: precio de colateral con un margen de seguridad, es decir, el máximo de _stablecoin_ permitido por unidad de colateral.
* `line`: el techo de deuda para un tipo específico de colateral.
* `Line`: el techo de deuda total para todos los tipos de colaterales.
* `dust`: la deuda mínima posible para una _vault_.
* `frob`: modifica una _vault_.
  * `lock`: transfiere colateral a una _vault_.
  * `free`: transfiere colateral de una _vault_.
  * `draw`: incrementa la deuda de una _vault_, creando DAI.
  * `wipe`: disminuye la deuda de una _vault_, destruyendo DAI.
  * `dink`: cambia a colateral.
  * `dart`: cambia a deuda.
  * `calm`: devuelve "true" (verdadero) cuando la _vault_ se encuentra debajo una colateral o del techo de deuda total.
  * `cool`: devuelve "true" (verdadero) cuando la deuda de _stablecoin_ no incrementa.
  * `firm`:  devuelve "true" (verdadero) cuando el saldo de un colateral no disminuye.
  * `safe`:  devuelve "true" (verdadero) cuando el ratio de deuda de colateral de una _vault_ está por encima del ratio de liquidación de un colateral.
* `fork`: para dividir una _vault_ - aprobación binaria o división/fusión de _vaults_.
  * `dink`: cantidad de colateral a intercambiar.
  * `dart`: cantidad de deuda de _stablecoin_ a intercambiar.
* `wish`: chequea si una _address_ tiene permitido modificar el `gem` o saldo de DAI de otra _address_.
  * `hope`: habilita `wish` para un par de _addresses_.
  * `nope`: deshabilita `wish` para un par de _addresses_.

**Nota:** `art` y `Art` representan la deuda normalizada, es decir un valor que, cuando es multiplicado por la tasa correcta, trae la deuda actual de _stablecoin_.

#### **Contabilidad**

* `debt`: la suma de todo el `dai` (la cantidad total de DAI emitidos).
* `vice`: la suma de todo el `sin` (la cantidad total de deuda del sistema).
* `ilk.Art`: la suma de todo el `art` en las `urn`s para `ilk`.
* `debt`: es `vice` más `ilk.Art * ilk.rate` a través de todos los `ilk`'s.

#### **Colateral**

* `gem`: puede ser transferido a cualquier _address_ por su dueño.

#### **DAI**

* `dai`: solo puede moverse con el consentimiento de su dueño / puede ser transferido a cualquier _address_ por su dueño.

**Otros**

* `LogNote`: un registro de propósito general que se puede agregar a cualquier función de un contrato.

### **_Jug_ (Tarifas de Estabilidad)**

#### Estructura

`Ilk`: contiene dos valores `uint256`: `duty`, la prima de riesgo específica de colaterales, y `rho`, la marca de tiempo de la última actualización de tarifas.

`VatLike`: simulación de contrato para hacer que las interfaces de _Vat_ puedan ser llamadas desde el código sin necesidad de una dependencia explícita del contrato de _Vat_ en sí.

#### Almacenamiento

`wards`: `mapping(address => uint)` que indica qué _addresses_ pueden llamar a las funciones administrativas.

`ilks`: `mapping (bytes32 => Ilk)` que almacena una estructura `Ilk` para cada tipo de colateral.

`vat`: un `VatLike` que señala al contrato [_Vat_](https://www.notion.so/makerdao/Vat-Core-Accounting-5314995c98e544c4aaa0ebedb01988ad) del sistema.

`vow`: la `address` del contrato _Vow_.

`base`: un `uint256` que especifica una tarifa que aplica a todos los tipos de colaterales.

#### Administrativo

Estos métodos requieren de `wards[msg.sender] == 1` (es decir que solo usuarios autorizados pueden llamarlos).

`rely`/`deny`: agrega o remueve usuarios autorizados (por medio de modificaciones al mapeo de los `wards`).

`init(bytes32)`: comienza la recolección de tarifas de estabilidad para un tipo de colateral en particular.

`file(bytes32, bytes32, uint)`: establece `duty` para un tipo particular de colateral.

`file(bytes32, data)`: establece el valor de `base`.

`file(bytes32, address)`: establece el valor de `vow`.

#### Métodos de Recolección de Tarifas

`drip(bytes32)`: recolecta tarifas de estabilidad para un tipo de colateral en particular.

### **_Cat_ (Liquidaciones)**

* `mat`: el ratio de liquidación.
* `chop`: la penalidad de liquidación.
* `lump`: la cantidad de liquidación, es decir la cantidad de deuda fija que debe ser cubierta por un evento de liquidación.
* `bite`: inicia la liquidación de una _vault_.
* `flip`: liquida el colateral de una _vault_ para cubrir la cantidad de deuda fija.
* 
### **_Vow_ (Liquidación)**

* `sin`: la cola de espera de deuda del sistema.
* `Sin`: el monto total de deuda en cola de espera.
* `Ash`: el monto total de deuda en subasta.
* `wait`: longitud de la cola de espera de la deuda.
* `sump`: tamaño de la oferta de la subasta de deuda, es decir,la cantidad de deuda fija que debe ser cubierta por cualquier subasta de deuda.
* `dump`: tamaño del lote de la suabsta de deuda; es decir, el monto inicial de MKR ofrecido para cubrir el `lot`/`sump`.
* `bump`: tamaño del lote de la subasta de excedentes; es decir, la cantidad fija de excedente que debe ser vendido en cualquier subasta de excedentes.
* `hump`: _buffer_ de excedentes, debe excederse antes de que las subastas de excedentes puedan realizarse.

**Otro términos incluídos en la documentación del _Vow_:**

* `move`: transfiere _stablecoins_ entre usuarios.
* `kick`: comienza una subasta.
* `woe`: indica, específicamente, la deuda incobrable o puede ser utilizado como un nombre de variable para cualquier monto de deuda.

### _Flipper_ (Subastas de Colaterales)

* `wards [usr: address]`, `rely`/`deny`/`auth`: Mecanismos de autorización
* `Bid`: Estado de una Subasta en particular {`bid`, `lot`, `guy`, `tic`, `end`, `usr`, `gal`, `tab`}
  * `bid`: monto de la oferta (DAI)/ DAI pagado
  * `lot`: cantidad a ser subastada / _gems_ de colaterales a ser subastados
  * `guy`: el mayor postor (_address_)
  * `tic`: expiración de la oferta
  * `end`: cuándo finalizará la subasta / duración máxima de la subasta
  * `usr`: _address_ de la _vault_ que está siendo subastada. Recibe _gems_ durante la fase de `dent`.
  * `gal`: destinatario de los ingresos de la subasta / recibe los ingresos de DAI (este es el contrato _Vow_).
  * `tab`: total de DAI deseado de la subasta / total de DAI que se recibirá (en subasta _flip_).
* `bids[id: uint]`: almacenamiento de todas las ofertas.
* `vat`: almacenamiento de la _address_ del _Vat_.
* `ilk`: id del Ilk por el cual el _Flipper_ es responsable.
* `beg`: incremento mínimo de oferta (por defecto: 5%).
* `ttl`: duración de la subasta (por defecto: 3 horas).
* `tau`: longitud de la subasta (por defecto: 2 días).
* `kicks`: Contador total de la subasta, utilizado para registrar los `id`s de la subasta.
* `kick`: función utilizada por `Cat` para comenzar una subasta / pone colateral a subastarse.
* `tick`: reinicia una subasta si esta no obtuvo ofertas y su tiempo de `end` (final) ya ha pasado.
* `tend`: primera fase de la subasta. Incremento del DAI de las `bid` (ofertas) por un `lot` (lote) de _Gems_.
* `dent`: segunda fase de la subasta. Establecer la `bid` de DAI por un `lot` decreciente de _Gems_.
* `file`: función utilizada por la Gobernanza para establecer el `beg`, `ttl` y `tau`.
* `deal`: reclamar una oferta ganadora / liquida una subasta finalizada.
* `yank`: utilizado durante el Asentamiento Global para pasar a las subasta de la fase `tend` a la `End` recuperando el colateral y reembolsando el DAI al mejor postor.

### _Flapper_ (Subasta de Excedentes)

* `Flap`: subasta de excedentes (vendiendo _stablecoins_ por MKR) \[contrato]
* `wards [usr: address]`: `rely`/`deny`/`auth` Mecanismos de Autorización \[uint]
* `Bid`: Estado de una Subasta en específico \[oferta]
  * `bid`: cantidad ofertado por el `lot` (MKR) \[uint]
  * `lot`: monto del `lot` (DAI) \[uint]
  * `guy`: mejor postor \[_address_]
  * `tic`: expiración de la oferta \[uint48]
  * `end`: cuándo finalizará la subasta \[uint48]
* `bids (id: uint)`: almacenamiento de todas las `Bid`s por `id` \[mapeo]
* `vat`: almacenamiento de la _address_ del _Vat_ \[_address_]
* `ttl`: tiempo de vida de la oferta / duración máxima de la oferta (por defecto: 3 horas) \[uint48]
* `lot`: monto del `lot` (DAI) \[uint]
* `beg`: incremento mínimo de la oferta (por defecto: 5%) \[uint]
* `tau`: duración máxima de subasta (por defecto: 2 days) \[uint48]
* `kick`: comienza una subasta / pone un nuevo `lot` de DAI para subastarse \[función]
* `tend`: hace una oferta, aumentando así el tamaño de la oferta / enviar una oferta MKR (aumentando la `bid`) \[función]
* `deal`: reclamar una oferta ganadora / liquidando una subasta finalizada \[función]
* `gem`: Token MKR \[_address_]
* `kicks`: contador total de subasta \[uint]
* `live`: bandera de _cage_ (jaula) \[uint]
* `file`: utilizado por la Gobernanza para establecer `beg`, `ttl` y `tau` \[función]
* `yank`: utilizado durante el Asentamiento Global para pasar a las subasta de la fase `tend` a la `End` recuperando el colateral y reembolsando el DAI al mejor postor. \[función]
* `tick()`: reinicia el valor `end` si no hubieron ofertas y el tiempo de `end` (final) ya ha pasado.

### _Flopper_ (Subastas de Deuda)

* `flop`: subasta de deuda (cubriendo la deuda al inflar el MKR y vendiéndolo por _stablecoins_)
* `lot`: cantidad a subastar / _gems_ a la venta (MKR)
* `guy`: mejor postor (_address_)
* `gal`: destinatario de los ingresos de la subasta / recibe el ingreso de DAI (este es el contrato _Vow_)
* `ttl`: tiempo de vida de la oferta (duración máxima de la oferta / tiempo de vida de una única oferta)
* `beg`: disminución mínima de oferta
* `pad`: incremento del tamaño del `lot` durante el `tick` (por defecto: 50%)
* `tau`: duración máxima de la subasta
* `end`: cuando finalizará la subasta / duración máxima de la subasta
* `kick`: comienzo de la subasta / poner un nueva `bid` (oferta) de MKR para subastar
* `dent`: hacer una oferta, disminución del tamaño del lote (presentar una `bid` -oferta- fija de DAI con tamaño de `lot` decreciente)
* `deal`: reclamar una oferta ganadora / liquida una subasta finalizada
* `vat`: la _address_ del _Vat_
* `gem`: token MKR (_address_)
* `kicks`: contador total de la subasta, utilizado para registrar los `id`s de la subasta
* `live`: bandera de _cage_ (jaula)
* `wards [usr: address]`, `rely`/`deny`/`auth`: mecanismos de autenticación
* `Bid`: estados de una subasta específica {`bid`, `lot`, `guy`, `tic`, `end`}
* `bid`: monto de oferta en DAI / DAI pagado
* `tic`: expiración de oferta
* `tick`: reinicia una subasta

### **_End_ (Asentamiento Global / Apagado)**

`cage`: Bloquea el sistema e inicia el apagado. Esto se realiza congelando las acciones del usuario, cancelando las subastas `flap` y `flop`, bloqueando el resto de los contratos del sistema, deshabilitando ciertas acciones de la Gobernanza que puedan interferir con el proceso de liquidación/asentamiento y comenzando el período de enfriamiento.

`cage(ilk)`: Etiqueta los precios del Ilk / Establece el precio final para un ilk (`tag`).

`skim`: Establece una _vault_ al precio etiquetado / Cancela el DAI adeudado de una _vault

`free`: Elimina el colateral (restante) de una _vault_ liquidada. Esto ocurre luego de que no haya deuda en una _vault_.

`thaw`: Corrige al suministro de DAI luego del "desnatado/limpieza" / Corrige al suministro total pendiente de _stablecoin_.

`flow`: Calcula el precio fijo de un ilk, posiblemente ajustando el precio de la `cage` con excedente/déficit.

`pack`: Bloquea el DAI por delante de `cash` / Pone algunas _stablecoins_ en una `bag` (bolsa) en preparación para `cash`.

`cash`: Intercambia los DAI empaquetados (`pack`) por colateral / Intercambia algo de DAI de la `bag` para un `gem` en particular, en parte proporcional al tamaño de la `bag`.

`file`: La configuración de la Gobernanza. Establece varios valores de parámetros.

`skip`: Opcionalmente, cancela subasta activas.

#### **Otros**

`wards(usr: address)`: Mecanismos de autenticación

`vat`: Contrato _Vat_

`cat`: Contrato _Cat_

`vow`: Contrato _Vow_

`spot`: Contrato _Spotter_

`live`: Bandera de _cage_ (jaula)

* Los contratos "Live" (vivo/activo) tienen `live` = 1, indicando que el sistema está corriendo con normalidad. Por lo tanto, cuando se invoca a `cage()`, este establece la bandera a 0. Esto incluye al contrato `End`, lo que significa que `cage()` solo puede ser invocado por única vez y que, las funciones subsecuentes, no pueden invocarse hasta que no estemos "dead" (muerto/desactivado) y en el proceso `End`.

`ilk`: Un tipo de Colateral

`when`: Tiempo de la `cage` / el tiempo de liquidación.

`wait`: Duración del enfriamiento del procesamiento / la longitud del enfriamiento del procesamiento

`debt`: DAI pendiente luego del procesamiento / suministro de _stablecoin_ pendiente, luego de que el excedente/déficit del sistema haya sido absorbido.

`tag`: Precio de la `cage` / precio por tipo de colateral al momento de la liquidación.

`gap`: Déficit de colateral / déficit por colateral considerando _vaults_ subcolateralizados.

`Art`: Deuda total por Ilk / deuda de _stablecoin_ pendiente.

`fix`: Precio final del `cash` / el precio del `cash` para un ilk (monto por _stablecoin_).

`bag(usr: address)`: DAI empaquetado (`pack`) por `cash` / _stablecoins_ no transferibles listas para ser intercambiadas por colateral.

`out`: Retiro de `cash` / el monto de _stablecoins_ ya intercambiadas para una _address_ determinada.

`skip`: Opcionalmente, cancela subasta activas.

`wad`: Una cantidad de tokens, generalmente como un número entero con 10^18 lugares decimales.

`urn`: Una _vault_ específica.

`tend`: Realizar una oferta, incrementando el tamaño de la oferta.

`bid`: La cantidad ofrecida por el `lot`.

`lot`: La cantidad a ser subastada.

`dent`:  Realizar una oferta, disminuyendo el tamaño del `lot`.

### _Join_ (Adaptadores de Token)

* `vat`: almacenamiento de la _address_ del _Vat_.
* `ilk`: `id` del `Ilk` por el cual un `GemJoin` está siendo creado.
* `gem`: la _address_ del `ilk` para transferir.
* `dai`: la _address_ del token del `dai`.
* `one`: un uint de 10^27 utilizado para cálculos en el `DaiJoin`.
* `live`: una bandera de acceso para el adaptador `join`.
* `dec`: decimales para el _Gem_.

### _Cat_ (Liquidaciones)

* `mul(uint, uint)`/`rmul(uint, uint)`: revertirá en caso de escasez o desbordamiento.
* `bite(bytes32, address)`: revertirá si `lot` o `art` son mayores o iguales a 2^255.
* `bite`: no dejará a una _vault_ con deuda y sin colateral.
* `wards`: tienen permitido llamar a funciones protegidas (Administración y `cage()`)
* `ilks`: almacena la estructura de `Ilk`.
  * `Ilk` es la estructura con la _address_ del contrato de la subasta de colaterales (`flip`), la penalidad por la liquidación del colateral (`chop`) y el tamaño máximo de colateral que puede ser subastado en una vez (`lump`).
* `live`: debe ser `1` para que el `Cat` muerda (`bite`) (ver `cage` en mecanismos)
* `vat`: _address_ que se ajusta a una interfaz `VatLike` (ver la documentación del `vat`). Se establece en el constructor y **no puede ser cambiada**.
* `vow`: _address_ que se ajusta a una interfaz `VowLike` (ver la documentación del `vow`).

**Eventos**

* `Bite`: emitido cuando `bite(bytes32, address)` es ejecutado con éxito. Contiene:
  * `ilk`: Colateral
  * `urn`: _Address_ de la _vault_
  * `ink`: ver `lot` en `bite`
  * `art`: ver `art` en `bite`
  * `tab`: ver `tab` en `bite`
  * `flip`: _address_ del contrato de la subasta
  * `id`: ID de la subasta en el `Flipper`

### _Spot_ (Enlace de Oráculos y Contratos)

* `ilk`: un tipo de colateral determinado.
* `ilk.pip`: el contrato que contiene el precio actual de un `ilk` determinado.
* `ilk.mat`: el ratio de liquidación de un `ilk` determinado.
* `vat`: el núcleo del sistema MCD.
* `par`: la relación entre el DAI y una unidad de valor del precio. (Similar al TRFM)

**Colateral**

* Solo los usuarios autorizados pueden actualizar las variables en el contrato.

### _Pot_ (Ahorros de DAI)

**Matemática**

* `mul(uint, uint)`, `rmul(uint, uint)`, `add(uint, uint)` y `sub(uint, uint)`: revertirá en caso de escacez o desbordamiento.
* `rpow(uint x, uint n, uint base)`: utilizado para la exponenciación en `drip`, es una función aritmética de punto fijo que eleva `x` a la potencia `n`.

**Autorizaciones**

* `wards`: pueden llamar a funciones protegidas (Administración).

**Almacenamiento**

* `pie`: almacena la _address_ del saldo del `Pot`.
* `Pie`: almacena el saldo total en el `Pot`.
* `dsr`: la `dai savings rate` (tasa de ahorro del DAI). Comienza en `1` (`ONE = 10^27`), pero puede ser actualizado por la Gobernanza.
* `chi`: el acumulador de tasas. Este es el valor, que siempre está creciendo, que decide cuánto `dai` se da cuando se llama a `drip()`.
* `vat`: _address_ que se ajusta a una interfaz `VatLike`. Se establece en el constructor y **no puede ser cambiada**.
* `vow`:  _address_ que se ajusta a una interfaz `VowLike`. No se establece en el constructor. La debe establecer la Gobernanza.
* `rho`: la última vez que se llamó a `drip`.

### DAI (_Stablecoin_)

`name`: La _stablecoin_ DAI

`symbol`: DAI

`version`: 1

`decimals`: 18

`wad`: número entero con 18 decimales (para cantidades básicas, e.j. balances).

`totalSupply`: Suministro Total de DAI

`balanceOf(usr: address)`: Saldo del usuario

`allowance(src: address, dst: address)`: Aprobaciones

`nonces(usr: address)`: Permiso de única vez
