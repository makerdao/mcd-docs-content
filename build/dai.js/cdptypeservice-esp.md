# Tipos de Colaterales

Utiliza el servicio `'mcd:cdpType'` para buscar parámetros de diferentes tipos de colaterales en el sistema. En el código, esto se llama [CdpTypeService](https://github.com/makerdao/dai.js/tree/dev/packages/dai-plugin-mcd/src/CdpTypeService.js).

```javascript
const service = maker.service('mcd:cdpType');
```

## Propiedades

### _cdpTypes_

Esta es una lista de las [instancias de tipos de colaterales](cdptypeservice.md#collateral-type-instances), que se inicializan en `Maker.create`.

```javascript
service.cdpTypes.forEach(type => console.log(type.ilk));
// ETH-A
// BAT-A
// USDC-A
```

## Métodos de Instancia

### _getCdpType\(\)_

Devuelve la [instancia del tipo de colateral](cdptypeservice.md#collateral-type-instances) para un tipo de moneda en específico y/o el [ilk](cdptypeservice.md#ilk).

```javascript
// esto devolverá un error si más de un tipo es definido para el ETH
const type = service.getCdpType(ETH);

// desambiguar usando el nombre _string_ (cadena) de ilk:
const ethA = service.getCdpType(null, 'ETH-A');
```

## Instancias de Tipos de Colaterales

### Propiedades

#### _ilk_

El nombre del tipo de colateral como _string_ (cadena de caracteres), e.j. "ETH-A".

#### *totalCollateral* (Colateral Total)

El monto total de colateral bloqueado en todas las _vaults_ de este tipo de colateral.

#### *totalDebt* (Deuda Total)

La deuda total en DAI contra todas las _vaults_ de este tipo de colateral.

#### *debtCeiling* (Techo de Deuda)

El techo de deuda para este tipo de colateral.

#### *liquidationRatio* (Ratio de Liquidación)

Las _vaults_ de este tipo se vuelven inseguras \(sujetas a liquidación\) cuando su ratio entre el valor en USD del colateral y el DAI es mayor o igual a este monto. 

#### *price* (Precio)

El precio del USD del token de este tipo de colateral, utilizando la fuente de información de precios más reciente como [ratio de moneda](currency-units.md). \(Ver "Una nota sobre el caché"\).

```javascript
console.log(type.price.toString()); // "9000.01 ETH/USD"
```

#### *liquidationPenalty* (Penalidad de Liquidación)

La penalidad añadida a la cantidad de DAI que se recaudará en la subasta cuando se liquide una _vault_, como un porcentaje. E.j. si la penalidad es del 13%, este valor será de `BigNumber(0.13)`.

#### *annualStabilityFee* (Tarifa de Estabilidad Anual)

La tarifa de estabilidad anual \(prima de riesgo\) de un tipo de colateral, sin incluir la [tasa base](systemdataservice.md#getannualbaserate), como un `BigNumber`(número grande). E.j. si la tasa es de 5%, el valor será de `BigNumber(0.05)`.

### *Caching*

Cuando se crea la instancia de una _vault_, sus datos son precargados en la _blockchain_, permitiendo que las propiedades que se encuentran a continuación se lean sincrónicamente. Estos datos se almacenan en caché en la instancia. Para actualizar estos datos, haz lo siguiente:

```javascript
vault.reset();
await vault.prefetch();
```

Para actualizar los datos de todas las instancias de tipo de colateral a la vez:

```javascript
service.resetAllCdpTypes();
await service.prefetchAllCdpTypes();
```
