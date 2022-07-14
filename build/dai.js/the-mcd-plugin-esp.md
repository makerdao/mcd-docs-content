# Administrador de _Vaults_

El administrador de _vaults_ trabaja con _vaults_ que posee el contato [_CdpManager_](https://github.com/makerdao/dss-cdp-manager/), el cual también es utilizado por "Oasis Borrow". Este contrato intermediario permite el uso de IDs con enteros incrementales para las _vaults_, familiar para los usuarios del SAI Unicolateral, así como otras conveniencias.

En el código, esto se llama [CdpManager](https://github.com/makerdao/dai.js/blob/dev/packages/dai-plugin-mcd/src/CdpManager.js).

```javascript
const mgr = maker.service('mcd:cdpManager');
```

## Métodos de Intancias

Los métodos mencionados debajo son todos asincrónicos.

### getCdpIds\(\) - "Obtener los IDs de _Vaults_"

Devuelve una matriz que describe a las _vaults_ que posee la _address_ especificada. Ten en cuenta que si las _vaults_ se crearon en "Oasis Borrow", se debe utilizar la _address_ del contrato *proxy*.

```javascript
const proxyAddress = await maker.service('proxy').currentProxy();
const data = await mgr.getCdpIds(proxyAddress);
const { id, ilk } = data[0];
// e.g. id = 5, ilk = 'ETH-A'
```

### getCdp\(\) - "Obtener _Vault_"

Obtener una _vault_ por su ID numérico. Devuelve la [instancia de la _vault_](the-mcd-plugin.md#vault-instances).

```javascript
const vault = await mgr.getCdp(111);
```

### open\(\) - "Abrir"

Abre una _vault_ con el [tipo de colateral](cdptypeservice.md) especificado. Creará un [*proxy*](advanced-configuration/using-ds-proxy.md) si todavía no existe uno. Devuelve la [instancia de una _vault_](the-mcd-plugin.md#vault-instances). Trabaja con el [administrador de transacciones](advanced-configuration/transactions.md).

```javascript
const txMgr = maker.service('transactionManager');
const open = await mgr.open('ETH-A');
txMgr.listen(open, {
  pending: tx => console.log('tx pending: ' + tx.hash)
});
const vault = await open;
```

### openLockAndDraw\(\) - "Abrir, Bloquear y Crear"

Abre una _vault_ nueva, luego bloquea y/o crea en una única transacción. Creará un [*proxy*](advanced-configuration/using-ds-proxy.md) si todavía no existe uno. Devuelve la [instancia de una _vault_](the-mcd-plugin.md#vault-instances).

```javascript
const vault = await mgr.openLockAndDraw(
  'BAT-A', 
  BAT(1000),
  DAI(100)
);
```

## Instancia de las _vaults_

En el código, esto se llama [ManagedCdp](https://github.com/makerdao/dai.js/blob/dev/packages/dai-plugin-mcd/src/ManagedCdp.js).

### Propiedades

**Un comentario sobre el caché:** Cuando se crea la instancia de una _vault_, sus datos se obtienen previamente de la _blockchain_, lo que permite que las propiedades, que se describen debajo, se lean de forma sincrónica. Estos datos se almacenan en caché en la instancia. Para actualizar estos datos, haz lo siguiente:

```javascript
vault.reset();
await vault.prefetch();
```

#### *collateralAmount* - "Monto del Colateral"

El monto bloqueado de tokens del colateral, como una [unidad de moneda](currency-units.md). 

#### *collateralValue* - "Valor del Colateral"

El valor en USD del colateral bloqueado, dado el precio actual según la fuente de precios, como una [unidad de moneda](currency-units.md).

#### *debtValue* - "Valor de Deuda"

El monto del DAI creado, como una [unidad de moneda](currency-units.md).

#### *liquidationPrice* - "Precio de Liquidación"

El precio en USD del colateral al que la _vault_ se vuelve insegura.

#### *isSafe* - "Es Seguro"

Si la _vault_ es actualmente segura o no.

### Métodos de Instancias

Todos los métodos que se mencionan debajo son asincrónicos y trabajan con el [administrador de transacciones](advanced-configuration/transactions.md). Los argumentos de cantidad deben ser [unidades de moneda](currency-units.md), e.j.:

```javascript
import { ETH, DAI } from '@makerdao/dai-plugin-mcd';

await vault.lockAndDraw(ETH(2), DAI(20));
```

#### *lockCollateral*\(monto\) - "Bloquear Colateral"

Deposita el monto de colateral especificado.

#### *drawDai*\(monto\) - "Crear DAI"

Genera el monto de DAI especificado.

#### *lockAndDraw*\(*lockAmount*, *drawAmount*\) - "Bloquear y Crear (Monto Bloqueado, Monto Creado)"

Deposita un poco de colateral y genera algo DAI en una única transacción.

#### *wipeDai*\(monto\) - "Borrar DAI"

Paga el monto de DAI especificado.

#### *wipeAll*\(\) - "Borrar Todo"

Paga toda la deuda. Este método asegura que no permanezcan los montos `dust`.

#### *freeCollateral*\(monto\) - "Liberar Colateral"

Extrae el monto de colateral especificado.

#### *wipeAndFree*\(*wipeAmount*, *freeAmount*\) - "Borrar y Liberar (Monto Borrado, Monto Liberado)"

Paga un poco de la deuda y extrae algo de colateral en una única transacción.

#### *wipeAllAndFree*\(*freeAmount*\) - "Borrar Todo y Liberar (Monto Liberado)"

Paga toda la deuda, asegurando que no permanezcan los montos `dust`, y extrae el monto de colateral especificado en una única transacción.

#### *give*\(*address*\) - "Dar"

Transfiere la titularidad de la _vault_ a `address`. Téngase en cuenta que, si el nuevo dueño desea utilizar esta _vault_ con "Oasis Borrow", esta deberá ser transferida a su _proxy_ con [giveToProxy](the-mcd-plugin.md#givetoproxy-address) en vez de `give()`.

#### *giveToProxy*\(*address*\) - "Dar a *Proxy*"

Busca el contrato _proxy_ que posee la `address` y transfiere la titularidad de esta _vault_ a ese _proxy.
