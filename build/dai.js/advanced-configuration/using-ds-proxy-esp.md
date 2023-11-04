# *DSProxy*

```javascript
const service = maker.service('proxy');
```

## Resumen

El `DSProxyService` (Servicio de DSProxy) incluye todas las funcionalidades necesarias para interactuar con ambos contratos del _proxy_ que son utilizados en los productos de Maker: ***proxies* de perfil** y ***proxies* de reenvío**.

Los *proxies* de reenvío son contratos simples que agregan llamadas a funciones en el cuerpo de un solo método. Son utilizados en el [Portal de CDP/_vault_](https://github.com/makerdao/sai-proxy) y [Oasis Directo](https://github.com/makerdao/oasis-direct-proxy) para permitir a los usuarios ejecutar múltiples transacciones atómicamente, lo cual es seguro y más fácil de utilizar que implementar varios pasos como transacciones discretas.

```text
// "proxy" de reenvío

function lockAndDraw(address tub_, bytes32 cup, uint wad) public payable {
  lock(tub_, cup);
  draw(tub_, cup, wad);
}
```

Los *proxies* de reenvío pretenden ser lo más simple posible, por lo que carecen de algunas características que podrían ser importantes si se van a utilizar como interfaces para una lógica de _smart contracts_ más compleja. Este problema se puede resolver utilizando *proxies* de perfil  \(es decir, copias de [*DSProxy*](https://github.com/dapphub/ds-proxy)\) para ejecutar las funcionalidades definidas en los _proxies_ de reenvío.

La primera vez que una cuenta es utilizada para interactuar con cualquier aplicación de Maker, se le solicitará al usuario que implemente un *proxy* de perfil. Esta copia del _DSProxy_ puede ser utilizada con cualquier producto, incluyendo dai.js, a través de un [registro de *proxy*](https://github.com/makerdao/proxy-registry/tree/master) universal. Luego, los datos de llamada de cualquier función en el *proxy* de reenvío se pueden pasar al método `execute()` (ejecutar) de *DSProxy*, que ejecuta el código proporcionado en el contexto del *proxy* de perfil.

```javascript
// Llamando al "proxy" de reenvío con dai.js

function lockAndDraw(tubContractAddress, cdpId, daiAmount, ethAmount) {
  const saiProxy = maker.service('smartContract').getContractByName('SAI_PROXY');

  return saiProxy.lockAndDraw(
    tubContractAddress,
    cdpId,
    daiAmount,
    {
      value: ethAmount,
      dsProxy: true
    }
  );
}
```

Esto hace posible que las asignaciones de tokens de los usuarios persistan de una aplicación de Maker a otra, y permite a los usuarios [recuperar cualquier fondo](https://proxy-recover-funds.surge.sh/) enviado por error a la dirección del *proxy*. Muchas de las funciones en `DSProxyService` solo serán relevantes para `power users` (usuarios con poder). Todo lo que es estrictamente requerido para generar de manera automática los datos de llamada de una función y encontrar el *proxy* de perfil correcto es la inclusión de `{ dsProxy: true }` en el objeto de opciones para cualquier transacción, siempre que el usuario ya haya implementado un *proxy* de perfil. Si eso no es seguro, también puede ser necesario consultar el registro para determinar si un usuario ya posee un *proxy* y crear (`build`) uno si no lo tiene.

## currentProxy\(\) - "*Proxy* Actual"

* **Parámetros:** Ninguno
* **Devuelve:** promesa \(se resuelve en la _address_ **o** `null` - nulo\)

Si la `currentAccount` (cuenta actual) \(según `Web3Service`\) ya ha implementado un *DSProxy*, `currentProxy()` (_proxy_ actual) devolverá su _address_. Si no, devuelve `null`. Esta se actualizará de forma automática en el evento de que la cuenta activa sea cambiada. Esta función deberá ser utilizada para chequear si un usuario tiene un _proxy_ antes de que intente crear uno.

```javascript
async function getProxy() {
  return maker.service('proxy').currentProxy();
}
```

## build\(\) - "Construir"

* **Parámetros:** Ninguno
* **Devuelve:** `TransactionObject` (Objeto de la Transacción)

`build` (construir) implementará una copia del _DSProxy_ perteneciente a la cuenta actual. **Esta transacción se revertirá si la cuenta actual ya tiene un _proxy_ de perfil**. Por defecto, `build()` finaliza luego de que la transacción haya sido minada.

```javascript
async function buildProxy() {
  const proxyService = maker.service('proxy');
  if (!proxyService.currentProxy()) {
    return proxyService.build();
  }
}
```

## ensureProxy\(\) - "Asegurar _Proxy_"

Esta función de conveniencia devolverá un *proxy* existente o creará uno.

```javascript
const proxyAddress = await maker.service('proxy').ensureProxy();
```

## getProxyAddress\(\) - "Obtener la _Address_ del _Proxy_"

* **Parámetros:** *Address* \(opcional\)
* **Devuelve:** promesa \(se resuelve en la _address_ del contrato\)

`getProxyAddress` consultará el registro del *proxy* para la _address_ de *proxy* de perfil asociada con una cuenta determinada. Si no se proporciona una _address_ como parámetro, la función devolverá la _address_ del *proxy* perteneciente a la `currentAccount` (cuenta actual).

```javascript
const proxy = await maker.service('proxy').getProxyAddress('0x...');
```

## getOwner\(\) - "Obtener Dueño"

* **Parámetros:** *Address*
* **Devuelve:** promesa \(se resuelve en la _address_\)

`getOwner` consultará el registro del *proxy* para el dueño de una instancia proporcionada del *DSProxy*.

```javascript
const owner = await maker.service('proxy').getOwner('0x...');
```

## setOwner\(\) - "Establecer Dueño"

* **Parámetros:** *Address* de un nuevo dueño, *address* del DSProxy \(opcional\)
* **Devuelve:** `TransactionObject` (Objeto de la Transacción)

`setOwner` se puede usar para otorgar un *proxy* de perfil a un nuevo propietario. Se debe especificar la _address_ de la cuenta del destinatario, pero la _address_ del DSProxy se establecerá de forma predeterminada en `currentProxy` (_proxy_ actual) si se excluye el segundo parámetro.

```javascript
await maker.service('proxy').setOwner(newOwner, proxyAddress);
```