# Estado del Sistema

## Resumen

Para acceder a la información del estado del sistema, recupera el Servicio CDP/_vault_ de ETH a través de Maker.service\('cdp'\).

```javascript
const service = maker.service('cdp');
```

## getSystemCollateralization - "Obtener la Colateralización del Sistema"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" \(se resuelve en el ratio de colateralización del sistema\)

`getSystemCollateralization()` devuelve el ratio de colateralización para todo el sistema, e.j. 2,75.

```javascript
const systemRatio = await service.getSystemCollateralization();
```

## getTargetPrice - "Obtener el Precio Objetivo"

* **Parámetros:** ninguno
* **Devuelve:** "promesa" \(se resuelve en el precio objetivo\)

`getTargetPrice()` devuelve el precio objetivo del SAI en USD, eso es, el valor al que el SAI está vinculado que, históricamente, ha sido 1. Este devuelve una [unidad de precio](system-status.md#units) `USD_SAI`.

```javascript
const targetPrice = await service.getTargetPrice();
```
