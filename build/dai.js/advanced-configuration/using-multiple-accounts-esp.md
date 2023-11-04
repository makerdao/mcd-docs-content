# Utilizando Múltiples Cuentas

## Resumen

Dai.js soporta el uso de múltiples cuentas \(es decir, llaves privadas\) con una única instancia de Maker. Las cuentas pueden ser especificadas en las opciones para `Maker.create` o con el método `addAccount`.

Llama a `useAccount` para alternar a usar una cuenta por su nombre, o `useAccountWithAddress` para alternar a usar una cuenta por su _address_ y las llamadas posteriores utilizarán esa cuenta como firmante de la transacción.

Cuando se crea la instancia de Maker por primera vez, si existe, utilizará la cuenta llamada `default` (por defecto), caso contrario utilizará la primer cuenta que esté en la lista.

```javascript
const maker = await Maker.create({
  url: 'http://localhost:2000',
  accounts: {
    other: {type: privateKey, key: someOtherKey},
    default: {type: privateKey, key: myKey}
  }
});

await maker.addAccount('yetAnother', {type: privateKey, key: thirdKey});

const cdp1 = await maker.openCdp(); // owned by "default"

maker.useAccount('other');
const cdp2 = await maker.openCdp(); // owned by "other"

maker.useAccount('yetAnother');
const cdp3 = await maker.openCdp(); // owned by "yetAnother"

await maker.addAccount({type: privateKey, key: fourthAccount.key}); // the name argument is optional
maker.useAccountWithAddress(fourthAccount.address);
const cdp4 = await maker.openCdp(); //owned by the fourth account
```

Puedes chequear la cuenta actual con `currentAccount` y `currentAddress`:

```javascript
> maker.currentAccount()
{ name: 'other', type: 'privateKey', address: '0xfff...' }
> maker.currentAddress()
'0xfff...'
```

## Tipos de Cuentas

Además del tipo de cuenta `privateKey`, hay otros dos tipos que se encuentran integrados:

* `provider`: Obtiene la primer cuenta del proveedor \(e.j. el valor de `getAccounts`\).
* `browser`: Obtiene la primer cuenta del proveedor en el navegador \(e.j. MetaMask\), incluso si la instancia de Maker está configurada para utilizar un proveedor diferente.

```javascript
const maker = await Maker.create({
  url: 'http://localhost:2000',
  accounts: {
    // this will be the first account from the provider at
    // localhost:2000
    first: {type: 'provider'},

    // this will be the current account in MetaMask, and it
    // will send its transactions through MetaMask
    second: {type: 'browser'},

    // this account will send its transactions through the
    // provider at localhost:2000
    third: {type: 'privateKey', key: myPrivateKey}
  }
})
```

## _Wallets_ de Hardware

Los _plugins_ pueden agregar tipos de cuenta adicionales. Actualmente hay dos _plugins_ de este tipo para el soporte de _wallet_ de hardware:

* [dai-plugin-trezor-web](https://github.com/makerdao/dai-plugin-trezor-web)
* [dai-plugin-ledger-web](https://github.com/makerdao/dai-plugin-ledger-web)

```javascript
import TrezorPlugin from '@makerdao/dai-plugin-trezor-web';
import LedgerPlugin from '@makerdao/dai-plugin-ledger-web';

const maker = await Maker.create({
  plugins: [
    TrezorPlugin,
    LedgerPlugin,
  ],
  accounts: {
    // default derivation path is "44'/60'/0'/0/0"
    myTrezor: {type: 'trezor', path: derivationPath1},
    myLedger: {type: 'ledger', path: derivationPath2}
  }
});
```

## **Demo** - "Modelo de Demostración"

Instala la [aplicación de _demo_ de múltiples cuentas](https://github.com/makerdao/integration-examples/tree/master/accounts) para ver esta funcionalidad en acción.
