# Eventos

## Resumen

El evento de _pipeline_ (tuberías) permite a los desarrolladores crear, de manera fácil, aplicaciones en tiempo real, permitiéndoles escuchar cambios importantes de estados y eventos del ciclo de vida.

## _Wildcards_ - "Comodines"

* Un nombre de evento pasado a cualquier método de emisor de eventos, puede contener un _wildcard_ (el carácter `*`\). Un *wildcard* puede aparecer como `foo/*`, `foo/bar/*` o, simplemente, `*`.
* `*` coincide con un nivel más abajo.

e.j. `price/*` coincidirá tanto `price/USD_ETH` como `price/MKR_USD` pero no lo hará con `price/MKR_USD/foo`.

* `**` coincide con todos los niveles que se encuentren por debajo.

e.j. `price/**` coincidirá con `price/USD_ETH`, `price/MKR_USD` y `price/MKR_USD/foo`.

**Objetos de Eventos**

Los eventos desencadenados recibirán el objeto que se muestre a la derecha.

* `<event_type>` - el nombre del evento
* `<event_payload>` - el nuevo estado de la data enviado con el evento
* `<event_sequence_number>` - un índice secuencialmente creciente
* `<latest_block_when_emitted>` - el bloque actual en el momento de la emisión

```javascript
{
    type: <event_type>,
    payload: <event_payload>, /* if applicable */
    index: <event_sequence_number>,
    block: <latest_block_when_emitted>
}
```

## Objeto de Maker

### Precio

| Nombre del Evento | Objeto |
| :--- | :--- |
| `price/ETH_USD` | { precio } |
| `price/MKR_USD` | { precio } |
| `price/WETH_PETH` | { ratio } |

```javascript
maker.on('price/ETH_USD', eventObj => {
    const { price } = eventObj.payload;
    console.log('ETH price changed to', price);
})
```

### Web3

| Nombre del Evento | Objeto |
| :--- | :--- |
| `web3/INITIALIZED` | { proveedor: { tipo, url } } |
| `web3/CONNECTED` | { api, red, nodo } |
| `web3/AUTHENTICATED` | { cuenta } |
| `web3/DEAUTHENTICATED` | { } |
| `web3/DISCONNECTED` | { } |

```javascript
maker.on('web3/AUTHENTICATED', eventObj => {
    const { account } = eventObj.payload;
    console.log('web3 authenticated with account', account);
})
```

### Objeto de CDP/_Vault_

| Nombre del Evento | Objeto |
| :--- | :--- |
| `COLLATERAL` | { USD, ETH } |
| `DEBT` | { dai } |

```javascript
cdp.on('DEBT', eventObj => {
    const { dai } = eventObj.payload;
    console.log('Your cdp now has a dai debt of', dai);
})
```
