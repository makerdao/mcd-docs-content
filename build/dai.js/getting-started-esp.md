# Empezando

## **Instalación**

Instala el paquete con el npm en tu terminal:

`npm install @makerdao/dai`

Una vez instalado, importa el módulo en tu proyecto como se muestra debajo.

```javascript
import Maker from '@makerdao/dai';
// or
const Maker = require('@makerdao/dai');
```

La compatibilidad con el DAI Multicolateral en Dai.js se implementa como un [_plugin_ (complemento)](maker/plugins.md). Esto puede cambiar en el futuro. El _plugin_ MCD también está disponible como paquete [npm](https://www.npmjs.com/package/@makerdao/dai-plugin-mcd) y su código fuente se puede encontrar en [Github](https://github.com/makerdao/dai.js/tree/dev/packages/dai-plugin-mcd).

`npm install @makerdao/dai-plugin-mcd`

```javascript
import { McdPlugin } from '@makerdao/dai-plugin-mcd';
// or
const { McdPlugin } = require('@makerdao/dai-plugin-mcd');
```

\(Nótese el `.default` (por defecto) al final de la línea cuando se usa `require` (requerir).\)

**UMD**

Esta librería también puede utilizarse como [módulo UMD](https://github.com/umdjs/umd), el cual puedes contruír con `npm run build:frontend`.

```javascript
<script src="./dai.js" />

<script>
// once the script loads, window.Maker is available
</script>
```

## **Ejemplos rápidos**

### Buscar información acerca de una _vault_

Este código utiliza [`getCdpIds`](the-mcd-plugin.md#getcdpids) para buscar una _vault_ que fue creada en la interfaz de [Oasis Borrow](https://oasis.app/borrow). Ya que este código solo está leyendo datos, no creando transacciones, no es necesario proveer una llave pública o conectar una _wallet_.

```javascript
// tú provees estos datos
const infuraKey = 'your-infura-api-key';
const ownerAddress = '0xf00...';

const maker = await Maker.create('http', {
  plugins: [McdPlugin],
  url: `https://mainnet.infura.io/v3/${infuraKey}`
});

const manager = maker.service('mcd:cdpManager');
const proxyAddress = maker.service('proxy').getProxyAddress(ownerAddress);
const data = await manager.getCdpIds(proxyAddress); // devuelve una lista de objetos { id, ilk }
const vault = await manager.getCdp(data[0].id);

console.log([
  vault.collateralAmount, // monto de tokens de colaterales
  vault.collateralValue,  // valor en USD, usando los valores de la fuente de precios actual
  vault.debtValue,        // monto de deuda de DAI
  vault.collateralizationRatio, // _collateralValue_ (valor del colateral) / deuda
  vault.liquidationPrice  // la _vault_ se vuelve insegura a este precio
].map(x => x.toString());
```

### Crear una _vault_

El código que aparece debajo abre una _vault_, bloquea ETH en ella y crea DAI de ella. 

Ya que este código envía transacciones, requiere de una cuenta que pueda firmar transacciones. La forma más fácil de realizar esto es proveyendo la opción de configuración de `privateKey` (llave privada), como se muestra debajo, pero también puedes conectar una Metamask, u otros proveedores basados en el navegador, o conectarse a _wallets_ de hardware.

```javascript
import Maker from '@makerdao/dai';
import { McdPlugin, ETH, DAI } from '@makerdao/dai-plugin-mcd';

// you provide these values
const infuraKey = 'your-infura-api-key';
const myPrivateKey = 'your-private-key';

const maker = await Maker.create('http', {
  plugins: [McdPlugin],
  url: `https://mainnet.infura.io/v3/${infuraKey}`,
  privateKey: myPrivateKey
});

// verifica que la llave privada haya sido leída correctamente
console.log(maker.currentAddress());

// asegúrate de que la cuenta actual posee un contrato _proxy_;
// crea uno, de ser necesario. El contrato _proxy_ es utilizado para realizar
// múltiples transacciones en una única operación.
await maker.service('proxy').ensureProxy();

// utiliza el servicio de "vault manager" (administrador de _vault_) para trabajar con las _vaults_
const manager = maker.service('mcd:cdpManager');
  
// ETH-A es el nombre del tipo de colateral; en el futuro,
// pueden existir múltiples tipos de colaterales para un token con
// diferentes parámetros de riesgo
const vault = await manager.openLockAndDraw(
  'ETH-A', 
  ETH(50), 
  DAI(1000)
);

console.log(vault.id);
console.log(vault.debtValue); // '1000.00 DAI'
```

En la próxima sección, aprende más acerca de cómo configurar la instancia de Maker con `Maker.create`. O pega un salto para aprender acerca de las [cuentas](advanced-configuration/using-multiple-accounts.md), [_vaults_](the-mcd-plugin.md), [_proxies_](advanced-configuration/using-ds-proxy.md) y [unidades de monedas](currency-units.md).

## Ejemplos de Integración

Para mayores ejemplos acerca de la integración de esta librería, revisa el [repositorio](https://github.com/makerdao/integration-examples) y el [plantilla de reacción del DAI](https://github.com/ethanbennett/dai-react-template).