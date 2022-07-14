# Unidades de Monedas

Los métodos que toman valores numéricos como _input_ (entrada) también pueden tomar instancias de clases de token que proporciona la librería. Estos son útiles para manejar la precisión, llevando registro de las unidades y pasando valores de _wei_. La mayoría de los métodos que devuelven valores numéricos, los devuelven envueltos en una de estas clases. Hay dos tipos:

* **unidades de monedas**, que representa un monto de un tipo de moneda.
* **unidades de precios**, conocidas como ratio de moneda, lo que representa una tasa de cambio entre dos monedas.

Las clases que comienzan con `USD` son unidades de precio; e.j. `USD_ETH` representa el precio del ETH en USD. Métodos útiles de instancias:

* **toNumber** (a número): devuelve el valor crudo de JavaScript. Esto puede fallar para números muy grandes.
* **toBigNumber** (a número grande): devuelve el valor crudo como [BigNumber](https://github.com/MikeMcl/bignumber.js).
* **isEqual** (es igual): compara los valores y símbolos de dos instancias diferentes.
* **toString** (a tipo cadena de caracteres): muestra el valor de forma humanamente legible, e.j. "500 USD/ETH".

```javascript
import Maker from '@makerdao/dai';

// DAI Multicolateral

import { ETH, BAT, DAI } from '@makerdao/dai-plugin-mcd';

const maker = await Maker.create(...);
const mgr = maker.service('mcd:cdpManager');

// bloquea BAT en una nueva `vault` y extrae DAI
const vault = await mgr.openLockAndDraw(
  'BAT-A',
  BAT(100),
  DAI(100)
);

// SAI Unicolateral

const {
  MKR,
  SAI,
  ETH,
  WETH,
  PETH,
  USD_ETH,
  USD_MKR,
  USD_SAI
} = Maker;

// Todos estos son idénticos:

// cada método tiene un tipo, por defecto
cdp.lockEth(0.25);
cdp.lockEth('0.25');

// se puede pasar una instancia de unidad de moneda
cdp.lockEth(ETH(0.25));

// las unidades de moneda poseen métodos de conversión convenientes
cdp.lockEth(ETH.wei(250000000000000000));

const eth = ETH(5);
eth.toString() == '5.00 ETH';

const price = USD_ETH(500);
price.toString() == '500.00 USD/ETH';

// la multiplicación maneja unidades
const usd = eth.times(price);
usd.toString() == '2500.00 USD';

// al igual que la división
const eth2 = usd.div(eth);
eth2.isEqual(eth);
```

Si deseas utilizar estas clases de ayuda fuera del Dai.js, lee [@makerdao/currency](https://github.com/makerdao/currency).