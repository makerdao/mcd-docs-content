# Datos del Sistema

Utiliza el servicio `mcd:systemData` para buscar parámetros de todo el sistema. En el código, esto se llama [SystemDataService](https://github.com/makerdao/dai.js/blob/dev/packages/dai-plugin-mcd/src/SystemDataService.js).

```javascript
const service = maker.service('mcd:systemData');
```

## Métodos de Instancia

### *getAnnualBaseRate*\(\) - "Obtener la Tasa Anual Base"

Devuelve la tasa base aplicada a todos los tipos de colateral, además de sus primas de riesgo individuales.

```javascript
const base = await service.getAnnualBaseRate();
```

### *getSystemWideDebtCeiling*\(\) - "Obtener el Techo de Deuda de todo el Sistema"

Devuelve el techo de deuda para todo el sistema.

```javascript
const line = await service.getSystemWideDebtCeiling();
```

### *isGlobalSettlementInvoked*\(\) - "¿Se ha invocado el/la Asentamiento/Liquidación Global?"

Devuelve un booleano que es `TRUE` si el Apagado de Emergencia ha sido activado.

```javascript
const dead = await service.isGlobalSettlementInvoked();
```