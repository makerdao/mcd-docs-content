# _Cat_ - Documentación Detallada

* **Nombre del Contrato:** cat.sol
* **Tipo/Categoría:** DSS
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/cat.sol)
* ****[**Etherscan**](https://etherscan.io/address/0x78f2c2af65126834c51822f56be0d7469d7a523e)

## 1. Introducción (Sumario)

`Cat` es el agente de liquidación del sistema; habilita a los _keepers_ a marcar posiciones como inseguras y mandarlas a subasta.

![](https://i.imgur.com/2pWFpFx.png)

## 2. Detalle del Contrato

### Glosario (_Cat_)

#### **Matemática**

* `mul(uint, uint)`/`rmul(uint, uint)` - revertirá en caso de desbordamiento o de no alcanzar.
* `bite(bytes32, address)` revertirá si `lot` o `art` son mayores o iguales a 2^255.
* `bite` no dejará una _vault_ con deuda y sin colateral.

#### **Autorizaciones**

* `wards` tiene permitido llamar a funciones protegidas (Administración y `cage()`).

#### **Almacenamiento**

* `ilks`: almacena la estructura de `Ilk`.
  * `Ilk`: es la estructura con la _address_ del contrato de la subasta de colaterales (`flip`), la penalización por la liquidación de ese colateral (`chop`) y el máximo de colateral que puede ser subastado de una vez (`dunk`).
* `live`: debe ser `1` para el `Cat` a `bite`. (lee `cage` en mecanismos)
* `box`: el límite de deuda y tarifas de penalidad para una subasta. \[RAD]
* `dunk`: (_"debt chunk"_, pedazo de deuda) monto de deuda más la tarifa de penalidad por subasta, en DAI. \[RAD]
* `vat`: _address_ que se ajusta a una interfaz `VatLike` (lee [la documentación de _`vat`_](https://docs.makerdao.com/smart-contract-modules/core-module/vat-detailed-documentation-esp) para más información). Se establece durante el constructor y **no puede ser cambiada**.
* `vow`: _address_ que se ajusta a una interfaz `VowLike` (lee [la documentación de _`vow`_](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module/vow-detailed-documentation-esp) para más información).

Aquí, los valores de todos los parámetros (excepto `vat`) pueden ser cambiados por una _address_ confiable (`rely`). Por ejemplo, el módulo `End` debe ser autenticado (`auth`) para permitir que llame a `cage()` y actualice `live` (en vivo) de 1 a 0. La Gobernanza (a través de una _address_ autorizada) debería poder agregar tipos de colaterales a `Cat`, actualizar sus parámetros y cambiar el `vow`.

#### **Inseguro**

`bite` puede ser llamado en cualquier momento pero solo será exitoso si la _vault_ es insegura. Una _vault_ es insegura cuando su colateral bloqueado (`ink`), multiplicado por el precio de liquidación de su colateral (`spot`), es menor que su deuda (`art` multiplicada por la tarifa de la `rate` - tasa - del colateral). El precio de liquidación es el precio reportado por los oráculos, escalado por el índice de liquidación de colateral.

#### **Eventos**

* `Bite`: emitido cuando un `bite(bytes32, address)` es ejecutado exitosamente. Contiene:
  * `ilk`: Colateral
  * `urn`: _Address_ de una _vault_
  * `ink`: ve `lot` en `bite`
  * `art`: ve `art` en `bite`
  * `tab`: ve `tab` en `bite`
  * `flip`: _address_ del contrato de la subasta
  * `id`: ID de la subasta en el `Flipper`

## 3. Conceptos y Mecanismos Claves

#### **cage()**

* `auth` (autorización)
* Establece `live` a 0 (previene `bite`). Lee [la documentación de `End`](https://docs.makerdao.com/smart-contract-modules/shutdown/end-detailed-documentation-esp) para más información.
* Una vez que `live=0` no puede volver a establecerse a 1.

#### **bite(bytes32 ilk, address urn)**

* El parámetro `bytes32 ilk` representa el tipo de colateral que está siendo "mordido".
* `address urn` es la _address_ que identifica a una _vault_ que está siendo "mordida".
* Devuelve `uint id` el cual es el ID de la nueva subasta en el `Flipper`.
* `bite` chequea si la _vault_ está en una posición insegura y, si lo está, comienza una subasta _Flip_ por una proporción del colateral para cubrir una parte de la deuda.

Lo siguiente es una explicación, línea por línea, de qué es lo que hace el `bite`:

```
function bite(bytes32 ilk, address urn) public returns (uint id) {
  // Toma el `rate`, `spot` y `dust` del `ilk` en el `vat`.
  (,uint256 rate,uint256 spot,,uint256 dust) = vat.ilks(ilk);
  // Toma el `ink` y `art` de la `urn` del `Vat`.
  (uint256 ink, uint256 art) = vat.urns(ilk, urn);

  // Asegura que `End` no haya sucedido
  require(live == 1);
  // Requiere que la `vault` sea insegura (ve la definición arriba).
  require(spot > 0 && mul(ink, spot) < mul(art, rate), "Cat/not-unsafe");

	// Carga la data de `ilk` en la memoria como una optimización.
  Ilk memory milk = ilks[ilk];
  // Declara una variable que será asignada en el siguiente ámbito
  uint256 dart;

	// Define un nuevo ámbito, esto evita un error apilado demasiado profundo en Solidity
  {
		// Calcula el espacio disponible en la caja
    uint256 room = sub(box, litter);

    // Testea si el espacio restante en la caja de gato (`litter`) está sucia (`dust`).
    require(litter < box && room >= dust, "Cat/liquidation-limit-hit");

		// Establece el monto de deuda que debe ser cubierta por una subasta.
    // (más pequeño entre el monto de la deuda normalizada, el tamaño máximo de la porción de la deuda o el espacio en la caja)
    // dividido por la tasa, dividido por la tarifa de penalidad.
    dart = min(art, mul(min(milk.dunk, room), WAD) / rate / milk.chop);
  }

	// Toma el mínimo del balance del colateral
  // Monto del colateral representado por la deuda que debe ser cubierta
  uint256 dink = min(ink, mul(ink, dart) / art);

	// Previene subasta sin colateral
  require(dart >  0      && dink >  0     , "Cat/null-auction");
  // Protege contra un desbordamiento de `int` cuando se está convirtiendo de `uint` a `int`
  require(dart <= 2**255 && dink <= 2**255, "Cat/overflow"    );

	// Llamado de esta forma, vat.grab podrá:
  // - Remover el `dink` y el `dart` de la _vault_ "mordida" (`urn`)
  // - Agrega colateral (`dink`) al `gem` del `Cat.
  // - Agrega la deuda (`dart`) a la deuda del `Vow` (`vat.sin[vow]`)
  // - Incrementa el total de DAI no respaldado (`vice`) en el sistema
  // Esto puede dejar al CDP es una estado sucio (`dust`)
  vat.grab(
    ilk, urn, address(this), address(vow), -int256(dink), -int256(dart)
  );
  // Agrega la deuda a la cola de espera de deuda en el `Vow` (`Vow.Sin` y `Vow.sin[now]`)
  vow.fess(mul(dart, rate));

  { // Evite apilar demasiado
    // Este cálculo desbordará si `dart*rate` excede ~10^14,
    // es decir, el `dunk` máximo es de aproximadamente 100 trillones de DAI.
    // Multiplica la tasa acumulada por la deuda normalizada cubierta
    // para obtener la `tab` de la deuda total (deuda + tarifa de estabilidad + penalización por liquidación) para la subasta.
    uint256 tab = mul(mul(dart, rate), milk.chop) / WAD;
    // Actualiza el monto en la caja de gato (`litter`)
    litter = add(litter, tab);

		// Llama a `kick` en el contrato `Flipper` del colateral
    // `tab` es el total de deuda que se mandará a subasta
    // `gal`: `address(vow)` establece el `Vow` como el recipiente de ingresos de DAI para esta subasta
    // `bid`: 0 indica que esta es la oferta inicial
    // Esto mueve al colateral del `gem` del `Cat` al `gem` del `Flipper` en el `Vat`.
    id = Kicker(milk.flip).kick({
        urn: urn,
        gal: address(vow),
        tab: tab,
        lot: dink,
        bid: 0
    });
  }

// Emite un evento sobre la "mordida" (`bite`) para notificar a los actores (por ejemplo, `keepers`) sobre la nueva subasta
  emit Bite(ilk, urn, dink, dart, mul(dart, rate), milk.flip, id);
}
```

#### **Administración**

Varias firmas de funciones de archivo para administrar `Cat`:

* Estableciendo un nuevo _vow_ (`file(bytes32, address)`)
* Estableciendo un nuevo colateral (`file(bytes32, bytes32, address)`)
* Estableciento penalidad o un tamaño de `dunk` para colaterales (`file(bytes32, bytes32, uint)`)

### **Uso**

El uso principal será que los _keepers_ llamen  a `bite` (mordida) en una _vault_ que crean que no es segura para comenzar el proceso de subasta.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

* Cuando `Cat` es actualizado, hay múltiples referencias a él que deben ser actualizadas al mismo tiempo (`End`, `Vat.rely`, `Vow.rely`). Debe confiar en `End`, el `pause.proxy()` del sistema.
* La actualización del `Vat` requerirá un nuevo `Cat`.
* El `Cat` almacena cada penalidad de liquidación de `Ilk` y el tamaño máximo de la subasta.
* Cada `ilk` iniciará con el `file` (archivo) para su `Flipper`; sin embargo, las subastas no pueden iniciar hasta que el `file` sea llamado para establecer `chop` y `dunk`. Sin estas subastas, por 0 `gems` o 0 DAI, estos se crearían llamando a `bite` en una _vault_ insegura.
* `bite` necesita ser llamado _n_ cantidad de veces donde `n = urn.ink / ilks[ilk].dunk` si `n > 1`. Esto permite la posibilidad de que la _vault_ se vuelva segura entre llamadas a `bite`, ya sea a través de un mayor colateral (en valor o cantidad) o una deuda reducida.
* Llamar a `bite` devuelve el `id`  de la subasta y emite un evento con el `id`. Esto es necesario para ofertar en el `Flipper` en la subasta.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

#### **Errores de codificación**

Un error en `Cat` podría provocar la pérdida (o bloqueo) de DAI y colateral al asignarlo a una _address_ que no puede recuperarlo (es decir, una _address_ del _Vow_ incorrecta o un _Flipper_ programado incorrectamente). El principal modo de falla de codificación de `Cat` es si tiene un error que hace que las subastas dejen de funcionar. Esto requerirá actualizar el sistema a un contrato `Cat` corregido. Si hay un error en `Cat` que se revierte en `cage`, entonces el Apagado podría fallar (hasta que se inicie un `Cat` correcto).

#### **Alimentación**

`Cat` se basa indirectamente en las fuentes de precios, ya que busca el seguimiento de los precios de los colaterales (`spot`) de `Vat` para determinar la seguridad de una _vault_. Si este sistema falla, podría provocar el robo de colaterales (`spot` demasiado bajo) o DAI sin respaldo (`spot` incorrectamente alto).

#### **Gobernanza**

La Gobernanza puede autorizar y configurar nuevos colaterales para `Cat`. Esto podría conducir a una mala configuración o ineficiencias en el sistema. Una mala configuración podría hacer que `Cat` no funcione correctamente o no funcione en absoluto. Por ejemplo, si un `Ilk.dunk` está configurado para ser mayor que 2\*\*255, podría permitir que las _vaults_ muy grandes sean imposibles de `bite` (morder).

Las ineficiencias en el `dunk` o `chop` podrían afectar las subastas Por ejemplo, un `dunk` demasiado grande o demasiado pequeño, podría desincentivar la participación de los _keepers_ en las subastas. Un `chop`, que sea demasiado pequeño, no desincentivará lo suficiente a las _vaults_ riesgosas y, si es demasiado grande, podría conducir a que se convierta en deuda incobrable. En este [documento de _grindeo_ de Subastas](https://github.com/livnev/auction-grinding/blob/master/grinding.pdf) se describe más detalladamente cómo los parámetros podrían provocar ataques al sistema.

#### **_Flipper_**

`Cat` se basa en un contrato `Flipper` externo para ejecutar la subasta y mueve el colateral del `Cat` a los contratos `Flipper` en el `Vat`. Un contrato de subasta de colateral defectuoso podría resultar en la pérdida de colateral o DAI o subastas que no funcionen.