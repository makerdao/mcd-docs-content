# Tokens

Obtén el objeto token a través de la función `getToken(tokenSymbol)` en el *tokenService* (servicio de token).

Los tokens que pueden ser utilizados en `getToken()` son: SAI, MKR, WETH, PETH, ETH.

Esta lista puede ser obtenida con `tokenService.getTokens()`. Esta función devuelve una representación en _string_ (elemento cadena) del símbolo del token, e.j. 'SAI', que también puede ser utilizado en `getToken`.

Cuando el _plugin_ del DAI Multicolateral se encuentra en uso, `getToken('DAI')` devolverá un objeto de token para el DAI.

```javascript
const tokenService = maker.service('token');
const sai = tokenService.getToken('SAI');
const weth = tokenService.getToken('WETH');
const peth = tokenService.getToken('PETH');
```
La mayoría de los métodos que se describen debajo pueden ser llamados en cualquier objeto de token. `deposit` y `withdraw` son solo para WETH; `join` y `exit` son solo para PETH.
## allowance - "Asignación"

```javascript
const allowance = await dai.allowance('0x...owner', '0x...spender');
```

* Parámetros:
* `tokenOwner` - *address* del dueño del token
* `spender` - *address* que utilizará/gastará el token
* Devuelve: "promesa" \(se resuelve a la asignación del token\)

`allowance` devuelve una [unidad de moneda](https://makerdao.com/documentation/#units) que representa la asignación del token.

## balance - "Balance/Saldo"

```javascript
const balance = await dai.balance();
```

* Parámetros: ninguno
* Devuelve: "promesa" \(se resuelve en el balance/saldo de la cuenta actual\)

`balance` devuelve una [unidad de moneda](https://makerdao.com/documentation/#units) que representa el balance del token en la cuenta actual.

## balanceOf - "Balance/Saldo de"

```javascript
const balanceOf = await dai.balanceOf('0x...f00');
```

* Parámetros: *address* a ser revisada
* Devuelve: "promesa" \(se resuelve en el balance/saldo de la _address_\)

`balanceOf` devuelve una [unidad de moneda](https://makerdao.com/documentation/#units) que representa el balance de token de la _address_ suministrada.


## totalSupply - "Suministro Total"

```javascript
const totalSupply = await dai.totalSupply();
```

* Parámetros: ninguno
* Devuelve: "promesa" \(se resuelve en el suministro total de tokens\)

`totalSupply` devuelve una [unidad de moneda](https://makerdao.com/documentation/#units) que representa el suministro total de tokens.

## approve - "Aprobar"

```javascript
return await dai.approve('0x...f00', DAI(10));
```

* Parámetros:
* *spender* - *address* que utilizará/gastará el token
* *amount* - monto de token a ser aprobado
* Devuelve: "promesa" \(se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) una vez minado\)

`approve` (aprueba), a la _address_ que utilizará/gastará tokens, a gastar hasta el `amount` (monto) de los tokens del `msg.sender`.

## approveUnlimited - "Aprobar Ilimitadamente"

```javascript
return await dai.approveUnlimited('0x...f00');
```

* Parámetros: *address* que utilizará/gastará el token
* Devuelve: "promesa" \(se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) una vez minado\)

`approveUnlimited` (aprueba) a la _address_ que utilizará/gastará tokens a gastar el monto máximo de los tokens del `msg.sender`.

## transfer - "Transferir"

```javascript
return await dai.transfer('0x...f00', DAI(10));
```

* Parámetros:
* *to* - _address_ a la que se enviará
* *amount* - monto de tokens a ser enviado
* Devuelve: "promesa" \(se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) una vez minado\)

`transfer` transfiere el `amount` (monto) de tokens a la _address_ de `to`.

## transferFrom - "Transferir de"

```javascript
return await dai.transferFrom('0x...fr0m', '0x...t0', DAI(10));
```

* Parámetros:
* *from* - _address_ desde la que se enviará
* *to* - _address_ a la que se enviará
* *amount* - monto de tokens a ser enviado
* Devuelve: "promesa" \(se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) una vez minado\)

`transferFrom()` transfiere el `amount` (monto) de tokens desde la _address_ del `from` a la *address* del `to`. La transacción fallará si el `msg.sender` no tiene la asignación para transferir el monto de tokens de la _address_ del `from`.

## deposit \(solo WETH\) - "Depositar"

```javascript
return await weth.deposit(ETH(10));
```

* Parámetros: monto de ETH a depositar
* Devuelve: "promesa" \(se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) una vez minado\)

`deposit` convierte el `amount` (monto) de ETH al `amount` (monto) de WETH.

## withdraw \(solo WETH\) - "Retirar"

```javascript
return await weth.withdraw(WETH(10));
```

* Parámetros: monto de WETH a ser retirado
* Devuelve: "promesa" \(se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) una vez minado\)

`withdraw` convierte el `amount` (monto) de WETH en el `amount` (monto) de ETH.

## join \(solo PETH\) - "Unir"

```javascript
return await peth.join(WETH(10));
```

* Parámetros: monto de WETH a unirse
* Devuelve: "promesa" \(se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) una vez minado\)

`join` convierte el `amount` (monto) de WETH a PETH, al [Ratio de WETH a PETH](https://makerdao.com/documentation/#getwethtopethratio).

## exit \(solo PETH\) - "Escapar"

```javascript
return await peth.exit(PETH(10));
```

* Parámetros: monto de PETH a ser liberado
* Devuelve: "promesa" \(se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) una vez minado\)

`withdraw` convierte el `amount` (monto) de PETH en WETH, al [Ratio de WETH a PETH](https://makerdao.com/documentation/#getwethtopethratio).
