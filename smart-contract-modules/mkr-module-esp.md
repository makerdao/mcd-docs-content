# Módulo MKR

* **Nombre del Contrato:** token.sol
* **Tipo/Categoría:** MKR Module
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* ****[**Fuente del Contrato**](https://github.com/dapphub/ds-token/blob/master/src/token.sol)
* ****[**Etherscan**](https://etherscan.io/address/0x9f8f72aa9304c8b593d555f12ef6589cc3a579a2)

## 1. Introducción (Sumario)

El Módulo MKR contiene al token MKR, que es un contrato [Ds-Token](https://github.com/dapphub/ds-token) implementado. Es un token ERC20 que provee una interfaz ERC20 estándar. También contiene una lógica de quemado y acuñamiento autorizado de MKR.

![Interacciones del MKR con el Protocolo de Maker](https://raw.githubusercontent.com/makerdao/mcd-docs-content/master/.gitbook/assets/Screen%20Shot%202019-11-17%20at%202.10.06%20PM.png)

## 2. Detalles del Contrato:

### Glosario (MKR)

* `guy` - _address_ del usuario
* `wad` - una cantidad de tokens, usualmente como un entero fijo con posición decimal 10^18.
* `dst` - refiere a la _address_ de destino.

### Funcionalidades Clave (como se define en el _smart contract_)

`mint` - tokens de crédito en una _address_ mientras aumenta simultáneamente el `totalSupply` (requiere autorización).

`burn` - tokens de débito en una _address_ mientras decrece simultáneamente el `totalSupply` (requiere autorización).

#### **Alias**&#x20;

`push` - transfiere un monto de `msg.sender` a una _address_.

`pull` - transfiere un monto de una _address_ a `msg.sender` (requiere confianza o aprobación).

`move` - transfiere un monto de una _address_ src a una _address_ dst (requiere confianza o aprobación).

**ERC-20 Estándar**

`name` - devuelve el nombre del token - e.j. "MyToken".

`symbol` - símbolo del token.

`decimals` - devuelve el número de decimales que utiliza el token - e.j. `8`, significa dividir el monto del token por `100000000` para obtener la representación del usuario.

`transfer` - transfiere el monto de _\_value_ (valor) de tokens a _address \_to_ (a) y **DEBE** disparar el evento _Transfer_. Esto **DEBERÍA** arrojar si el saldo de la cuenta de la persona que llama a la función no tiene suficientes tokens para gastar.

`transferFrom` - transfiere el monto de _\_value_ (valor) de tokens de _address \_from_ (de) a _address \_to_ (a) y **DEBE** disparar el evento _Transfer_.

`approve` - permite al _\_spender_ (gastador) retirar múltiples veces de su cuenta, hasta el monto de la _\_value_. Si la función es llamada nuevamente, se sobrescribe 
la asignación actual con _\_value_.

`totalSupply` - devuelve el suministro total de tokens.

`balanceOf` - devuelve el balance de la cuenta de otra cuenta con _address \_owner_ (dueño).

`allowance` - devuelve el monto que todavía se le permite al _\_spender_ (gastador) a retirar de _\_owner_ (dueño).

Puedes encontrar más información acerca del Token ERC20 estándar [aquí](https://eips.ethereum.org/EIPS/eip-20).

**Nota importante:** hay una auto-aprobación de `transferFrom` cuando `src == msg.sender`.

## 3. Conceptos y Mecanismos Claves

Junto con que el MKR tiene una interfaz de token ERC20 estándar, también tiene la adición de funciones de acuñamiento y quemado protegidas por DSAuth; aprobación binaria a través de MAX\_UINT; así como alias `push`, `pull` y `move` para operaciones `transferFrom`.

**El token MKR tiene 3 métodos de uso dentro del Protocolo de Maker (para referencia ver la ** [**Presentación 101 del Protocolo de Maker**](https://docs.makerdao.com/maker-protocol-101-esp)**):**

* **Como un token de utilidad:** A medida que las tarifas de estabilidad del DAI ganadas en las _vaults_ se acumulan dentro del Protocolo de Maker, los poseedores de MKR pueden utilizar MKR para votar y permitir que la casa de subastas _Flapper_ venda el excedente de Dai por MKR. Una vez que se completa la subasta, el protocolo de Maker quema el MKR.
* **Como un token de la Gobernanza:** El MKR es utilizado por los poseedores de MKR para votar por la gestión de riesgos y la lógica comercial del Protocolo de Maker. Los tokens son una representación simple del poder de voto. 
* **Como un recurso de recapitalización:** El MKR puede ser acuñado autónomamente por la cada de subastas _Flopper_ y vendido por DAI, el cual es utilizado para recapitalizar al Protocolo de Maker en tiempos de insolvencia.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

* El token MKR es un token ERC-20 que se creó utilizando DSToken. Una diferencia clave para denotar entre el DAI y la mayoría de otros tokens ERC20 populares es que ambos campos utilizan `bytes32` en vez de ser de tipo `string` (texto).

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

* `MKR.stop` - El Apagado de Emergencia no puede ser activado. El MKR en el `chief` (jefe) aun puede votar (`vote`) pero no puede unirse (`join`) o salirse (`exit`).