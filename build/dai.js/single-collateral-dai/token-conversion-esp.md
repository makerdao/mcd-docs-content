# Conversión de Tokens

Obtén el servicio de conversión de tokens en maker.service \('conversion'\).

```javascript
const conversionService = maker.service('conversion');
```
El servicio de conversión de tokens ofrece funciones para conevrtir entre ETH, WETH y PETH, manejando las asignaciones en caso de ser necesario.

## convertEthToWeth - "Convertir ETH a WETH"

```javascript
return await conversionService.convertEthToWeth(ETH(10));
```

* Parámetros: monto de ETH a convertir
* Devuelve: "promesa" \(se resuelve a `transactionObject` una vez minado\)

**Note:** esto es lo mismo que `weth.deposit`

`convertEthToWeth` deposita ETH en un contrato WETH.

## convertWethToPeth - "Convertir WETH a PETH"

```javascript
return await conversionService.convertWethToPeth(WETH(10));
```

* Parámetros: monto de WETH a convertir
* Devuelve: "promesa" \(se resuelve a `transactionObject` una vez minado\)

`convertWethToPeth` une a WETH en PETH, dando primero una asignación de ser necesario.

**Nota:** Este proceso no es atómico si la asignación del token necesita ser establecida, por lo que es posible que una de las transacciones sea exitosa, pero no ambas. Lee [DsProxy](../advanced-configuration/using-ds-proxy.md) para ejecutar múltiples transacciones atómicamente. También, [*peth.join*](tokens.md#join-peth-only) puede ser llamado en su lugar si no deseas que la asignación se establezca primero de forma automática.

## convertEthToPeth - "Convertir ETH a PETH"

```javascript
return await conversionService.convertEthToPeth(ETH(10));
```

* Parámetros: monto de ETH a convertir
* Devuelve: "promesa" \(se resuelve a `transactionObject` una vez minado\)

`convertEthToPeth` espera a convertEthToWeth, luego llama a convertWethToPeth

**Nota:** Este proceso no es atómico si la asignación del token necesita ser establecida, por lo que es posible que una de las transacciones sea exitosa, pero no ambas. Lee [DsProxy](../advanced-configuration/using-ds-proxy.md) para ejecutar múltiples transacciones atómicamente.

## convertWethToEth - "Convertir WETH a ETH"

```javascript
return await conversionService.convertconvertWethToEth(WETH(10));
```

* Parámetros: monto de WETH a convertir
* Devuelve: "promesa" \(se resuelve a `transactionObject` una vez minado\)

`convertWethToEth` retira ETH del contrato WETH

**Nota:** esto es lo mismo que `weth.withdraw`

## convertPethToWeth - "Convertir PETH a WETH"

```javascript
return await conversionService.convertPethToWeth(PETH(10));
```

* Parámetros: monto de PETH a convertir
* Devuelve: "promesa" \(se resuelve a `transactionObject` una vez minado\)

`convertPethToWeth` saca PETH a WETH, dando primero una asignación de ser necesario.

**Nota:** Este proceso no es atómico si la asignación del token necesita ser establecida, por lo que es posible que una de las transacciones sea exitosa, pero no ambas. Lee [DsProxy](../advanced-configuration/using-ds-proxy.md) para ejecutar múltiples transacciones atómicamente. También, *peth.exit* puede ser llamado en su lugar si no deseas que la asignación se establezca primero de forma automática.

## convertPethToEth - "Convertir PETH a ETH"

```javascript
return await conversionService.convertPethToEth(PETH(10));
```

* Parámetros: monto de PETH a convertir
* Devuelve: "promesa" \(se resuelve a `transactionObject` una vez minado\)

`convertPethToEth` espera a convertPethToWeth, luego llama a convertWethToEth

**Nota:** Este proceso no es atómico si la asignación del token necesita ser establecida, por lo que es posible que una de las transacciones sea exitosa, pero no ambas. Lee [DsProxy](../advanced-configuration/using-ds-proxy.md) para ejecutar múltiples transacciones atómicamente.