# Administrador de Transacciones

El servicio de `transactionManager` (Administrador de Transacciones) es utilizado para registrar el estado de las transacciones a medida que se propaga a través de la _blockchain_.

Los métodos en Dai.js que inicien transacciones son asincrónicos, así que estos devuelven "promesas". Estas promesas pueden ser pasadas como argumentos al administrador de transacciones para configurar devoluciones de llamada cuando las transacciones cambien su estado a `pending` (pendiente), `mined` (minado), `confirmed` (confirmado) or `error`.

```javascript
const txMgr = maker.service('transactionManager');
// instance of transactionManager
const open = maker.service('cdp').openCdp();
// open is a promise--note the absence of `await`
```

Pasa la promesa a `transactionManager.listen` con devolución de llamada como se muestra debajo.

```javascript
txMgr.listen(open, {
  pending: tx => {
    // do something when tx is pending
  },
  mined: tx => {
    // do something when tx is mined
  },
  confirmed: tx => {
    // do something when tx is confirmed       
  },
  error: tx => {
    // do someting when tx fails
  }
});

await txMgr.confirm(open); 
// 'confirmed' callback will fire after 5 blocks
```

Téngase en cuenta que el evento `confirmed` (confirmado) no se activará a menos que `transactionManager.confirm` sea llamada. Esta función asincrónica espera a la cantidad de bloques \(por defecto, 5\) que hayan luego de que la transacción haya sido minada para resolverse. Para cambiar esto de manera global, establece el atributo `confirmedBlockCount` en las [opciones](../maker/#options) de Maker. Para cambiar esto para una sola llamada, pasa el número de bloques a esperar como segundo argumento:

```javascript
await txMgr.confirm(open, 3);
```

## Metadata de Transacciones

Hay funciones, como `lockEth()`, que están compuestas por muchas transacciones internas. Estas se pueden rastrear con mayor precisión accediendo a `tx.metadata` en la devolución de llamada que contiene tanto el `contract` (contrato) como el `method` (método) a partir del cual se crearon las transacciones internas.

## Métodos de Objetos de Transacciones

Un `TransactionObject` también tiene algunos métodos para proporcionar más detalles sobre la transacción:

* `hash` : el _hash_ de la transacción
* `fees()` : monto de ether utilizado/gastado en _gas_
* `timeStamp()` : marca de tiempo de cuándo la transacción fue minada
* `timeStampSubmitted()` : marca de tiempo de cuándo la transacción fue presentada en la red

```javascript
const lock = cdp.lockEth(1);
txMgr.listen(lock, {
  pending: tx => {
    const {contract, method} = tx.metadata;
    if(contract === 'WETH' && method === 'deposit') {
      console.log(tx.hash); // print hash for WETH.deposit
    }
  }
})
```
