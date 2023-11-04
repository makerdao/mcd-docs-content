# SAI Unicolateral

## Instalación

La compatibilidad con el DAI Unicolateral en Dai.js se implementa como un [_plugin_ (complemento)](../maker/plugins.md). El _plugin_ SCD (_Single-Colateral DAI_ - DAI Unicolateral) también se encuentra disponible en el paquete [npm](https://www.npmjs.com/package/@makerdao/dai-plugin-scd) y su código fuente se puede encontrar en [Github](https://github.com/makerdao/dai.js/tree/dev/packages/dai-plugin-scd).

`npm install @makerdao/dai-plugin-scd`

```javascript
import { ScdPlugin } from '@makerdao/dai-plugin-scd';
// o
const { ScdPlugin } = require('@makerdao/dai-plugin-scd');
```

## Ejemplos rápidos

El código de debajo crea una _vault_ (antes denominada como "CDP"), bloquea ETH en ella y crea SAI.

```javascript
import Maker from '@makerdao/dai';
import { ScdPlugin } from '@makerdao/dai-plugin-scd';

async function openLockDraw() {
  const maker = await Maker.create("http", {
    plugins: [ScdPlugin],
    privateKey: YOUR_PRIVATE_KEY,
    url: 'https://kovan.infura.io/v3/YOUR_INFURA_PROJECT_ID'
  });

  await maker.authenticate();
  const cdpService = await maker.service('cdp');
  const cdp = await cdpService.openCdp();

  await cdp.lockEth(0.25);
  await cdp.drawSai(50);

  const debt = await cdp.getDebtValue();
  console.log(debt.toString); // '50.00 SAI'
}

openLockDraw();
```

Los servicios y objetos mencionados debajo se utilizan para trabajar con SAIs Unicolaterales.

* [Servicio de CDP/_Vaults_](eth-cdp-service.md)
* [Posición de Deuda Colateralizada](collateralized-debt-position.md)
* [Estado del Sistema](system-status.md)
* [Conversión de Tokens](token-conversion.md)

## openCdp\(\) - "Abrir CDP/_Vault_"

* **Devuelve:** "promesa" \(se resuelve en un nuevo objeto de CDP/_vault_ una vez minado\)

`openCdp()` creará una nueva CDP/_vault_ y, luego, devolverá el objeto de esta, la cual puede ser utilizada para acceder a otras funcionalidades de CDP/_vaults_. La "promesa" se resolverá cuando la transacción sea minada.

```javascript
const cdpService = await maker.service('cdp');
const newCdp = await cdpService.openCdp();
```

## getCdp\(int id\) - "Obtener CDP/_Vault_(ID de número entero)"

* **Devuelve:** "promesa" \(se resuelve en un objeto de CDP/_vault_\)

`getCdp(id)` crea un objeto CDP/_Vault_ para una CDP/_Vault_ ya existente. Luego, el objeto de CDP/_Vault_ puede ser utilizado para interactuar con tu CDP/_Vault_.

```javascript
const cdpService = await maker.service('cdp');
const cdp = await cdpService.getCdp(614);
```

Una vez que tiene una instancia de una CDP/_Vault_, puedes usar los [métodos de instancias de CDP/_vaults_](collateralized-debt-position.md) para leer sobre su estado y realizar acciones.