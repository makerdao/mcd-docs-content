# Servicio de Precios

## Resumen

Recupera el `PriceService`(Servicio de Precio) a través de `Maker.service('price')`. El `PriceService` expone la información de los precios de los colaterales y de los tokens de la Gobernanza que es reportado por los oráculos en el sistema de Maker.

```javascript
const price = maker.service('price');
```

## getEthPrice - "Obtener el Precio del ETH"

Obtiene el precio actual en USD del ETH como una [unidad de moneda](https://makerdao.com/documentation/#units) `USD_ETH`.

```javascript
const ethPrice = await price.getEthPrice();
```

## getMkrPrice - "Obtener el Precio del MKR"

Obtiene el precio actual en USD del token MKR de la Gobernanza, como una [unidad de moneda](https://makerdao.com/documentation/#units) `USD_MKR`.

```javascript
const mkrPrice = await price.getMkrPrice();
```

## getPethPrice - "Obtener el Precio del PETH"

Obtiene el precio actual en USD del PETH \(ethereum "pooleado"\), como una [unidad de moneda](https://makerdao.com/documentation/#units) `USD_PETH`.

```javascript
await pethPrice = price.getPethPrice();
```

## setEthPrice - "Establecer el Precio del ETH"

Establece el precio actual en USD del ETH. Esto requiere los permisos necesarios y solo será útil en un ambiente de testeo.

```javascript
await price.setEthPrice(475);
```

## setMkrPrice - "Establecer el Precio del MKR"

Establece el precio actual en USD del token MKR de la Gobernanza. Esto requiere los permisos necesarios y solo será útil en un ambiente de testeo.

```javascript
await price.setMkrPrice(950.00);
```

## getWethToPethRatio - "Obtener WETH al Ratio del PETH"

Devuelve el WETH actual al ratio del PETH.

```javascript
await price.getWethToPethRatio();
```