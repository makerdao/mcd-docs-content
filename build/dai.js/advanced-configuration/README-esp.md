# Avanzado

Para anular la _address_ de uno de los contratos utilizados por Dai.js o un _plugin_, puedes pasar la opción `smartContract.addressOverrides`. Debes conocer la _key_ (llave) del contrato en el archivo de las _addresses_ para poder anularla.

[Lista de _addresses_ de la red principal](https://github.com/makerdao/dai.js/blob/dev/packages/dai/contracts/addresses/mainnet.json)

```javascript
const service = Maker.create('test' {
  smartContract: {
    addressOverrides: {
      PROXY_REGISTRY: '0xYourAddress'
    }
  } 
});
```