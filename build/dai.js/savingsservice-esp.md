# Tasa de Ahorro del DAI

Utiliza el servicio `'mcd:savings'` para trabajar con el sistema de Tasa de Ahorro de DAI. En el código, esto se llama [SavingsService](https://github.com/makerdao/dai.js/blob/dev/packages/dai-plugin-mcd/src/SavingsService.js).

```javascript
const service = maker.service('mcd:savings');
```

## Métodos de Instancia

Todo los métodos que se detallan debajo son asincrónicos. `join`, `exit` y `exitAll` utilizan un [contrato  _proxy_](advanced-configuration/using-ds-proxy.md).

### _join_\(monto\)

Deposita la cantidad de DAI especificada en el contrato de la Tasa de Ahorro de DAI \(**DSR** - _Dai Savings Rate_\).

```javascript
await service.join(DAI(1000));
```

### _exit_\(monto\)

Extrae la cantidad de DAI especificada del contrato DSR.

### _exitAll_\(\)

Extrae todo el DAI que posea la cuenta actual del contrato DSR.

### _balance_\(\)

Devuelve el monto de DAI en el contrato DSR que posea la [_address_ actual](advanced-configuration/using-multiple-accounts.md). Estríctamente hablando, este método devuelve el monto de DAI que posee el contrato _proxy_ de la _address_ actual para trabajar con los métodos arriba mencionados.

### *balanceOf*\(_address_\)

Devuelve el monto de DAI en el contrato DSR que posee la _address_ especificada.

### *getTotalDai*\(\)

Obtiene el monto total de DAI en el contrato DSR para todos los usuarios.

### *getYearlyRate*\(\)

Obtiene la tasa de ahorro anual actual.