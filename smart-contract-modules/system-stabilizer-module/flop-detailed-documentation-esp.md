# _Flopper_ - Documentación Detallada

* **Nombre del Contrato:** flop.sol
* **Tipo/Categoría:** DSS —> System Stabilizer Module
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/flop.sol)
* ****[**Etherscan**](https://etherscan.io/address/0xa41b6ef151e06da0e34b009b86e828308986736d#code)****

## 1. Introducción (Sumario)

**Sumario:** Las Subastas de Deuda se utilizan para recapitalizar el sistema subastando MKR por un monto fijo de DAI. En este proceso, los postores compiten ofreciendo aceptar cantidades decrecientes de MKR por el DAI que terminarán pagando.

![Interacciones de la _Flop_ con el Protocolo de Maker](https://github.com/makerdao/mcd-docs-content/raw/master/.gitbook/assets/Screen%20Shot%202019-11-17%20at%202.15.41%20PM.png)

## 2. Detalles del Contrato

#### _Flopper_ (Glosario)

* `flop`: subasta de deuda (cubriendo la deuda al inflar el MKR y venderlo por _stablecoins_)
* `lot`: cantidad para la subasta / _gems_ en venta (MKR)
* `guy`: mejor postor (_address_)
* `gal`: destinataria de los ingresos de la subasta / recibe los ingresos de dai (este es el contrato de _Vow_)
* `ttl`: tiempo de vida de la oferta (duración máxima de oferta / tiempo de vida de una única oferta)
* `beg`: decrecimiento mínimo de oferta
* `pad`: incremento del tamaño de `lot` durante el `tick` (por defecto al 50%)
* `tau`: duración máxima de la subasta
* `end`: cuándo va a finalizar la subasta / duración máxima de la subasta
* `kick`: activa una subasta / realiza una nueva `bid` en MKR para la subasta
* `dent`: realiza una oferta, disminuyendo el tamaño del `lot` (presentando una `bid` fija de DAI con tamaño de `lot` decreciente)
* `deal`: reclamo de una apuesta ganadora / resolver una subasta finalizada
* `vat` - la _address_ del Vat
* `gem`- Token MKR (_address_)
* `kicks` - recuento total de subastas, usado para rastrear los `id`s de la subasta
* `live` - bandera de jaula
* `wards [usr: address]`, `rely`/`deny`/`auth` - mecanismos de autorización
* `Bid` - estado específico de la subasta {`bid`, `lot`, `guy`, `tic`, `end`}
* `bid` - monto de la oferta en DAI / DAI pagado
* `tic` - caducidad de la oferta
* `tick` - restablecela subasta

#### **Parámetros establecidos por la Gobernanza**

* Los votantes de la Gobernanza de Maker determinan el límite de deuda. La Subasta de Deuda se activa cuando la deuda de DAI del sistema supera ese límite.
* La Gobernanza de Maker establece el `Vow.dump` el cual determina el `lot` inicial para una subasta, así como el `pad` que determina cuánto puede incrementar el `lot` en el `tick`.
* Los contratos que son autorizados (`auth`) para llamar a la función `kick()` (deben ser solo `Vow`) y `file()` para cambiar `beg`, `ttl`, `tau` (deben ser solo contratos de la Gobernanza).

**Nota Informativa:** `cage` establece la subasta _Flop_ para que ya no esté activa y el `yank` se usa durante el Asentamiento Global para devolver una oferta al postor, ya que `dent` y `deal` ya no pueden ser llamadas.

## 3. Conceptos y Mecanismos Claves

El proceso de las Subastas _Flop_ comienzan cuando los votantes de la Gobernanza de Maker determinan el límite de deuda del sistema ([`Vow.sump`](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module/vow-detailed-documentation#auctions-esp)). Entonces, cuando la deuda en DAI del sistema supera dicho límite, se activan las Subastas de Deuda.

Para determinar si el sistema tiene deuda neta, el excedente, las tarifas de estabilidad devengadas y la deuda deben estar conciliadas. Cualquier usuario puede realizar esto al enviar la transacción `heal` al contrato del sistema llamado [Vow.sol](https://github.com/makerdao/dss/blob/master/src/vow.sol). Siempre que haya suficiente deuda (es decir, deuda después de la curación (`heal`) > `Vow.sump`), cualquier usuario puede enviar una transacción `Vow.flop` para activar una subasta de deuda.

`Flop` es una subasta reversa, donde los _keepers_ ofertan por qué tan poco MKR están dispuestos a aceptar por la cantidad fija de DAI que deben pagar en la resolución de la subasta. Los postores competirán con cantidades de `lot`s de MKR decrecientes por una cantidad fija de `bid` (oferta) de DAI. Una vez que se lanza (`kick`), la `bid` es establecida en el tamaño de ofertas de la subasta _flop_ (`Vow.sump`) y el `lot` se establece en un número lo suficientemente grande (`Vow.dump`). La subasta terminará cuando la última duración de oferta (`ttl`) haya concluido **O** cuando la duración de la subasta (`tau`) ha sido alcanzada. El proceso de pago comienza cuando se realiza la primer oferta. La primer oferta pagará la deuda del sistema y, cada oferta subsiguiente, pagará al postor anterior (que ya no está ganando). Cuando la subasta finaliza, el proceso termina limpiando la oferta y acuñando MKR para el postor ganador.

Si la subasta caduca sin haber recibido ninguna oferta, cualquiera puede reiniciar la subasta llamando a la función `tick(uint auction_id)`. Esto hará dos cosas:

1. Reestablece `bids[id].end` a `now + tau`
2. Reestablece `bids[id].lot` a `bids[id].lot * pad / ONE`

![Un diagrama que detalla las interacciones que tiene un usuario con _Flopper_ y _Vow_](https://i.imgur.com/ZWQVXVy.png)

#### **Requerimientos al momento de ofertar durante la subasta**

Durante una subasta, la cantidad de `lot` decrecerá por un porcentaje con cada nuevo `dent`, decreciendo el `lot` por el `beg` por la misma `bid` de DAI. Por ejemplo, el `beg` puede estar establecido en 5%, significando que el postor actual tiene un `lot` de 10 (MKR) para una `bid` de 100 (DAI); entonces, la próxima oferta, debe superar como mucho un `lot` de 9,5 (MKR) por una `bid` de 100 (DAI).

#### **Ofertando**

Cuando una oferta es derrotada por otro postor, el saldo de DAI del nuevo ganador es usado para reembolsar al postor ganador anterior. Una vez realizada la oferta, estas *no pueden* cancelarse.

#### Ejemplo de **flujo de ofertas**:

1. El _Vow_ activa (`kick`) una nueva subasta _Flop_.
2. El Postor 1 realiza una oferta que decrece el tamaño del `lot` por el `beg` del monto inicial. El saldo de DAI en el _Vat_ del Postor 1, descrece por la `bid` y el saldo de DAI en el _Vat_ del _Vow_ se incrementa por la `bid`.
3. El Postor 2 realiza una oferta que decrece el `lot` del Postor 1 por el `beg`. El saldo de DAI en el _Vat_ del Postor 2 decrece por la `bid` y el saldo de DAI en el _Vat_ del Postor 1 se incrementa por la `bid` (reembolsando al Postor 1 por la oferta no ganadora).
4. El Postor 1 realiza una oferta que decrece el `lot` del Postor 2 por el `beg`. El DAI del Postor 1 es igual a `Vat.dai[bidder1]` menos la `bid`; El DAI del Postor 2 es igual a `Vat.dai[bidder2]` más la `bid`.
5. El Postor 2 (y el resto de postores que se encuentran en la subasta) decide que ya no vale la pena seguir aceptando `lot`s tan bajos, por lo que dejan de ofertar. Una vez que el `Bid.tic` expira, el Postor 1 llama a `deal` y nuevos tokens MKR son acuñados a su _address_ (`MKR token contract.balances(Bidder1)` = `MKR.balances(Bidder1)` + `lot`).

**Nota:** Durante una subasta _Flop_, el `beg` es el monto mínimo de decrecimiento. En `dent`, la nueva oferta tiene que tener `lot` \* `beg`, que es menor o igual a su tamaño de `lot` actual. Dado que la teoría de la subasta _Flop_ es que la oferta de un postor es tomar cada vez menos tokens MKR (`lot`) por la misma cantidad de DAI (`bid`), entonces el `beg` es la cantidad por la cual cada oferta debería disminuirse.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

#### **_Keepers_**

En el contexto de que se esté dirigiendo un _keeper_ (puedes encontrar más información [aquí](https://github.com/makerdao/developerguides/tree/master/keepers)), para poder realizar ofertas dentro de una subasta, un modo de falla principal podría ocurrir cuando un _keeper_ especifica un precio no rentable para MKR.

* Este modo de falla se debe a que no hay nada que el sistema pueda hacer si un usuario desea pagar una suma significativamente superior a la del mercado por un token en una subasta (esto aplica para todos los tipos de subastas, `flip`, `flop` y `flap`).
* En el caso de una _Flop_, ya que el monto de DAI se encuentra fijo para la subasta, esto significa que el riesgo para un _keeper_ sea de realizar una oferta "ganadora" que pague el monto de DAI ofertado pero que no reciba ningún MKR a cambio (`lot` == 0). Las ejecuciones subsiguientes a esta mala estrategia se encontrarán limitadas por el monto de DAI (no MKR) en el balance de su _Vat_.
* 
## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

1. `Flopper` tiene el potencial de emitir una cantidad excesivamente grande de MKR y, a pesar de los esfuerzos de mitigación (la inclusión de los parámetros `dump` y `pad`), si la Gobernanza no configura correctamente `dump`, la gran emisión de MKR podría ocurrir igualmente.
2. Ve la [documentación del Módulo Estabilizador del Sistema](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module-esp)