# Servicio de CDP / *Vault* 

## Resumen

Esta página aplica, solamente, a DAI Unicolateral.

Recupera el servicio ETH de CDP/_vault_ a través de Maker.service\('cdp'\). El servicio ETH de CDP/_vault_ expone información de parámetros de riesgo para el tipo Ether de CDP/_vault_ \(en Dai unicolateral, este es el único tipo CDP/_vault_\).

```javascript
const service = maker.service('cdp');
```

## getLiquidationRatio - "Obtener el Ratio de Liquidación"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" \(se resuelve en el ratio de liquidación\)

`getLiquidationRatio()` devuelve una representación decimal del ratio de liquidación, e.j. 1,5.

```javascript
const ratio = await service.getLiquidationRatio();
```

## getLiquidationPenalty - "Obtener la Penalidad de Liquidación"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" \(se resuelve en la penalidad por liquidación\)

`getLiquidationPenalty()` devuelve una representación decimal de la penalidad por liquidación, e.j. 0,13.

```javascript
const penalty = await service.getLiquidationPenalty();
```

## getAnnualGovernanceFee - "Obtener la Tarifa Anual de la Gobernanza"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" \(se resuelve en la tarifa anual de la Gobernanza\)

`getAnnualGovernanceFee()` devuelve una representación decimal de la tarifa anual de la Gobernanza, e.j. 0,005.

```javascript
const fee = await service.getAnnualGovernanceFee();
```

**Nota:** normalmente, se refiere a esto como `Tarifa de Estabilidad` aunque, técnicamente, la `Tarifa de Estabilidad` es la tarifa que se paga en DAI y la `Tarifa de la Gobernanza` es la tarifa que se paga en MKR. Pero, ya que las tarifas en DAI Unicolateral solo son pagadas en MKR y las tarifas en DAI Multicolateral en DAI, la tarifa en DAI Unicolateral, a menudo, se denomina como `Tarifa de Estabilidad` para ser coherente con el término que se utilizará en el DAI Multicolateral y para evitar confundir indebidamente a los usuarios habituales.