# Configuración

## Maker.create
Puedes configurar el comportamiento del Dai.js al pasar diferentes argumentos al `Maker.create`. El primer argumento será el nombre de uno de los preseteados y el segundo es un objeto de las opciones.

### Preseteados
- `'browser'`
    - Utiliza este preseteado cuando utilices la librería en un entorno de navegador. Este intentará conectarse utilizando `window.ethereum` o `window.web3`.
- `'http'`
    - Conecta a un nodo de JSON-RPC. Requiere una `url` para ser establecida en las opciones.
- `'test'`
    - Utiliza un nodo local (e.j. Ganache) que corra en `http://127.0.0.1:2000` y firma transacciones utilizando llaves administradas por nodos.

```javascript
const makerBrowser = await Maker.create('browser');

const makerHttp = await Maker.create('http', {
  url: 'https://kovan.infura.io/v3/YOUR_INFURA_PROJECT_ID'
});

const makerTest = await Maker.create('test');
```

### Opciones
- `privateKey`
    - Opcional. La *private key* (llave privada) utilizada para firmar transacciones. Si esto es omitido, se utilizará la primer cuenta del proveedor de Ethereum que se encuentre disponible. Solo se puede utilizar con el preseteado `'http'`.
    - Si esto es omitido y el proveedor no tiene una cuenta disponible, el objeto `maker` comenzará en [modo de solo lectura](https://github.com/makerdao/mcd-docs-content/blob/master/build/dai.js/maker/#read-only-mode).
- `url`
    - La URL del nodo a conectarse. Solo se puede utilizar con el preseteado `'http'`.
- `web3.transactionSettings`
    - Un objeto que contiene las opciones de transacciones a ser aplicadas a todas las transacciones enviadas a la librería.
    - Valor por defecto: `{ gasLimit: 4000000 }`
- `web3.confirmedBlockCount`
    - Número de bloques en cola luego de que una transacción haya sido minada al llamar a la función `confirm` (confirmar). Lee [Transacciones](https://github.com/makerdao/mcd-docs-content/blob/master/build/dai.js/advanced-configuration/transactions.md) para más información.
    - Valor por defecto: 5
- `web3.inject`
    - Para usuarios avanzados. En vez de utilizar el `HttpProvider` (proveedor de http) preestablecido, con esto puedes inyectar tu propia instancia personalizada de un proveedor Web3.
- `log`
    - Establécelo en `false` (falso) para reducir la verbosidad del registro.
- `autoAuthenticate`
    - Establécelo en `false` (falso) para crear la instancia de Maker sin haberte conectado aún. De ser así, debes correr `await maker.authenticate()` antes de usar cualquier otro método.

```javascript
// It doesn't necessarily make sense to set all these
// options at the same time (e.g. `url` and `inject`),
// this is just meant to illustrate the shape of the
// options object.
const maker = await Maker.create('http', {
  privateKey: YOUR_PRIVATE_KEY, // '0xabc...'
  url: 'http://some-ethereum-rpc-node.net',
  web3: {
    statusTimerDelay: 2000,
    confirmedBlockCount: 8
    transactionSettings: {
      gasPrice: 12000000000
    },
    inject: someProviderInstance
  },
  log: false,
  autoAuthenticate: false
});
```

## Métodos de Instancias

### service() - "Servicio"
- Devuelve: objeto del servicio

Devuelve una instancia del servicio que fue incluída en la instancia de `maker`.

```javascript
const accountsService = maker.service('accounts');
```

## Servicios

El _plugin_ del MCD define muchos de los servicios que se utilizan para trabajar con el DAI Multicolateral. Lee [Comenzando](https://github.com/makerdao/mcd-docs-content/blob/master/build/dai.js/getting-started.md) para ver cómo agregar el _plugin_.

- `'mcd:cdpManager'`: para trabajar con _Vaults_.
- `'mcd:cdpType'`: para leer parámetros y datos en vivo (totales y precios) para los tipos de colaterales.
- `'mcd:savings'`: para trabajar con la Tasa de Ahorro del DAI.
- `'mcd:systemData'`: para leer parámetros de todo el sistema.

## Modo Solo Lectura
Como ya se había mencionado, la instancia `Maker` puede ser utilizada en el modo de solo lectura si solo necesitas leer los datos de la _blockchain_ sin necesidad de firmar transacciones. Simplemente omite la opción `privateKey` (llave privada).

Puedes comenzar con el modo de solo lectura y, aún así, [agregar una cuenta](https://github.com/makerdao/mcd-docs-content/blob/master/build/dai.js/advanced-configuration/using-multiple-accounts.md) con la habilidad de firmar transacciones más adelante.