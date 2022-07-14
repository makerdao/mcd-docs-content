# Posición de Deuda Colateralizada

> 💡 Esta página solo aplica al SAI Unicolateral.

Luego de abrir [un nuevo CDP/_vault_ u obteniendo uno ya existente](https://hackmd.io/9eZE8u2RQVa2muW4_AyROg), puedes utilizar el objeto `cdp` obtenido para llamar funciones en él.

## Propiedades

### **`id`**

Este es el ID del objeto CDP. Puedes utilizar este ID en [Maker.getCdp](https://app.gitbook.com/s/-LtJ1VeNJVW-jiKH0xoL/build/dai.js/single-collateral-dai/Maker#getcdp).

## Métodos

### getDebtValue - "Obtener el Valor de Deuda"

* **Parámetros:** [Unidad de moneda](https://app.gitbook.com/s/-LtJ1VeNJVW-jiKH0xoL/build/dai.js/single-collateral-dai/Currency-units) (opcional)
* **Devuelve:** "promesa" (se resuelve el monto de la deuda pendiente)

`cdp.getDebtValue()` devuelve el monto de la deuda que fue prestada contra el colateral en el CDP. Por defecto, devuelve el monto de DAI como una [unidad de moneda](https://makerdao.com/documentation/#units), pero puede devolver el equivalente en USD si el primer argumento es `Maker.USD`.

```javascript
const daiDebt = await cdp.getDebtValue();
const usdDebt = await cdp.getDebtValue(Maker.USD);
```

### getGovernanceFee - "Obtener la Tarifa de la Gobernanza"

* **Parámetros:** [Unidad de moneda](https://app.gitbook.com/s/-LtJ1VeNJVW-jiKH0xoL/build/dai.js/single-collateral-dai/Currency-units) (opcional)
* **Devuelve:** "promesa" (se resuelve en el valor de la tarifa acumulada en USD de la Gobernanza)

`cdp.getGovernanceFee()` devuelve el valor de la tarifa acumulada de la Gobernanza. Por defecto, devuelve el monto en MKR como una [unidad de moneda](https://makerdao.com/documentation/#units), pero puede devolver el equivalente en USD si el primer argumento es `Maker.USD`.

**Nota:** normalmente, se refiere a esto como `Tarifa de Estabilidad` aunque, técnicamente, la `Tarifa de Estabilidad` es la tarifa que se paga en DAI y la `Tarifa de la Gobernanza` es la tarifa que se paga en MKR. Pero, ya que las tarifas en DAI Unicolateral solo son pagadas en MKR y las tarifas en DAI Multicolateral en DAI, la tarifa en DAI Unicolateral, a menudo, se denomina como `Tarifa de Estabilidad` para ser coherente con el término que se utilizará en el DAI Multicolateral y para evitar confundir indebidamente a los usuarios habituales.

```javascript
const mkrFee = await cdp.getGovernanceFee();
const usdFee = await cdp.getGovernanceFee(Maker.USD);
```

### getCollateralizationRatio - "Obtener Ratio de Colateralización"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" (se resuelve al ratio de colateralización)

`cdp.getCollateralizationRatio()` devuelve el valor en USD del colateral en el CDP/_vault_ dividido por el valor en USD de la deuda de DAI del CDP/_vault_, e.j. `2.5`.

```javascript
const ratio = await cdp.getCollateralizationRatio();
```

### getLiquidationPrice - "Obtener el Precio de Liquidación"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" (se resuelve al precio de liquidación)

`cdp.getLiquidationPrice()` devuelve el precio de Ether en USD que causa que el CDP/la _vault_ se vuelva inseguro (capaz de ser liquidado). Devuelve un [precio unitario](https://makerdao.com/documentation/#units) `USD_ETH`.

```javascript
    const ratio = await cdp.getLiquidationPrice();
```

### getCollateralValue - "Obtener el Valor del Colateral"

* **Parámetros:** [Unidad de moneda](https://app.gitbook.com/s/-LtJ1VeNJVW-jiKH0xoL/build/dai.js/single-collateral-dai/Currency-units) (opcional)
* **Devuelve:** "promesa" (se resuelve al monto del colateral)

`cdp.getCollateralValue()` devuelve el valor del colateral en el CDP/la _vault_. Por defecto, devuelve el monto del ETH como una [unidad de moneda](https://makerdao.com/documentation/#units), pero puede devolver el equivalente en PETH o USD dependiendo del primer argumento.

```javascript
const ethCollateral = await cdp.getCollateralValue();
const pethCollateral = await cdp.getCollateralValue(Maker.PETH);
const usdCollateral = await cdp.getCollateralValue(Maker.USD);
```

### isSafe - "Es Seguro"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" (se resuelve en un booleano)

`cdp.isSafe()` devuelve `true` si el CDP/la _vault_ es segura, eso es si, el valor en USD del colateral es mayor o igual al valor en USD de su deuda multiplicada por el ratio de liquidación.

```javascript
const ratio = await cdp.isSafe();
```

### enoughMkrToWipe - "MKR Suficiente para ser borrado"

* **Parámetros:** monto de SAI a ser borrado
* **Devuelve:** "promesa" (se resuelve en un booleano)

`cdp.enoughMkrToWipe(dai)` devuelve `true` si la cuenta actual tiene MKR suficiente para "borrar" el monto específico de SAI del CDP/ la _vault_.

```javascript
const enoughMkrToWipe = await cdp.enoughMkrToWipe(10000000000000000000, SAI.wei);
```

### lockEth - "ETH bloqueado"

* **Parámetros:** monto a ser bloqueado en el CDP/ la _vault_, en unidades definidas por el servicio de precios.
* **Devuelve:** "promesa" (se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) (objeto de transacción) una vez minado)

`cdp.lockEth(eth)` abstrae las conversiones de token necesarias para bloquear el colateral en un CDP/la _vault_. Primero convierte el ETH a WETH, luego convierte el WETH a PETH, luego bloquea el PETH en el CDP/la _vault_.

**Nota:** este proceso no es atómico, por lo que es posible que algunas de las transacciones tengan éxito, pero no las tres. Lee [Usando DsProxy](https://makerdao.com/documentation/#proxy-service) para ejecutar múltiples transacciones atómicamente.

```javascript
return await cdp.lockEth(10000000000000000000, ETH.wei);
// or equivalently
return await cdp.lockEth(100, ETH);
```

### drawSai - "SAI a retirar"

* **Parámetros:** monto a retirar (en SAI, como elemento cadena)
* **Devuelve:** "promesa" (se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) (objeto de transacción) una vez minado)

`cdp.drawSai(sai)` extrae el monto especificado en SAI como un préstamo contra el colateral en el CDP/la _vault_. Como tal, fallará si el CDP/la _vault_ no tiene suficiente PETH bloqueado para permanecer colateralizado al menos en un 150%.

```javascript
return await cdp.drawSai(10000000000000000000, SAI.wei);
// or equivalently
return await cdp.drawSai(100, SAI);
```

### wipeSai - "Borrar SAI"

* **Parámetros:** monto a ser pagado (en SAI, como elemento cadena)
* **Devuelve:** "promesa" (se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) (objeto de transacción) una vez minado)

`cdp.wipeSai(sai)` envía SAI de nuevo al CDP/la _vault_ para pagar parte (o la totalidad) de su deuda pendiente.

**Nota:** Los CDPs/las _vaults_ acumulan deuda en MKR de la Gobernanza durante su ciclo de vida. Esto debe ser saldado cuando se "borre" la deuda en SAI y, por lo tanto, se debe adquirir MKR antes de llamar a este método.

```javascript
return await cdp.wipeSai(10000000000000000000, SAI.wei);
// or equivalently
return await cdp.wipeSai(100, SAI);
```

### freePeth - "Liberar PETH"

* **Parámetros:** monto de colateral de PETH a ser liberado del CDP/ la _vault_, en unidades definidas por el servicio de precios.
* **Devuelve:** "promesa" (se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) (objeto de transacción) una vez minado)

`cdp.freePeth(peth)` extrae el monto especificado de PETH y lo devuelve a la _address_ del dueño. Como tal, el contrato solo le permitirá liberar PETH bloqueado en exceso del 150% de la deuda pendiente del CDP/ la _vault_.

```javascript
return await cdp.freePeth(100, PETH);
// or equivalently
return await cdp.freePeth(10000000000000000000, PETH.wei);
```

### give - "Dar"

* **Parámetros:** _address_ de Ethereum (cadena)
* **Devuelve:** "promesa" (se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) (objeto de transacción) una vez minado)

`cdp.give(address)` transfiere la titularidad del CDP/_vault_ del actual dueño a la _address_ que proveas como argumento.

```javascript
return await cdp.give('0x046ce6b8ecb159645d3a605051ee37ba93b6efcc');
```

### shut - "Cerrar"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" (se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) (objeto de transacción) una vez minado)

`cdp.shut()` "borra" el SAI restante, libera todo el colateral y elimina el CDP/_vault_. Esto fallará si el que llamada a la función no tiene el DAI suficiente para saldar la deuda de SAI completa y MKR suficiente para pagar por todas las tarifas de estabilidad acumuladas.

```javascript
return await cdp.shut();
```

### **bite** - "Mordida"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" (se resuelve a [`transactionObject`](https://makerdao.com/documentation/#transactions) (objeto de transacción) una vez minado)

`cdp.bite()` iniciará el proceso de liquidación de un CDP/_vault_ subcolateralizada.

```javascript
return await cdp.bite();
```
