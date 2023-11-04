---
Descripción: El Contrato del Token del DAI
---

# DAI - Documentación Detallada

* **Nombre del Contrato:** dai.sol
* **Tipo/Categoría:** DSS —> Dai Module
* ****[**Diagrama de Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/dai.sol)
* ****[**Etherscan**](https://etherscan.io/address/0x6b175474e89094c44da98b954eedeac495271d0f)

## 1. Introducción (Sumario)

El contrato `Dai` es el contrato de token ERC20 orientado al usuario que mantiene la contabilidad de los saldos externos del Dai. La mayoría de las funciones son estándar para un token con suministro cambiante, pero también presenta la capacidad de emitir aprobaciones para transferencias basadas en mensajes firmados.

![Interacciones del DAI con el Protocolo de Maker](https://github.com/makerdao/mcd-docs-content/blob/master/.gitbook/assets/Screen%20Shot%202019-11-17%20at%202.08.05%20PM.png?raw=true)

## 2. Detalles del Contrato

### DAI (Glosario)

**Funcionalidades Clave (como se encuentran definidas en el _smart contract_)**

`Mint` - Acuñación a una _address_

`Burn` - Quema en una _address_

`Push` - Tranferencia

`Pull` - Tranferir Desde

`Move` - Tranferir Desde

`Approve` - Permite `pull`s y `move`s

`Permit` - Aprobación por firma

**Otros**

`name` - _Stablecoin_ DAI

`symbol` - DAI

`version` - 1

`decimals` - 18

`totalSupply` - Suministro Total de DAI

`balanceOf(usr: address)` - Balance del Usuario

`allowance(src: address, dst: address)` - Aprobaciones

`nonces(usr: address)` - Permiso Único

#### **Unidades**

* `wad` - decimal fijo que cuenta de 18 decimales (para cantidades básica, e.j., saldos).

## 3. Conceptos y Mecanismos Claves

En su mayor parte, `dai.sol` funciona como un token ERC20 típico. Estos tokens ya han sido [muy bien documentados aquí](https://eips.ethereum.org/EIPS/eip-20) y se recomienda leer esa documentación para conocer las funciones principales de un token ERC20.

#### Diferencias de ERC20:

1. En el contrato del DAI,`transferFrom` funciona de una forma ligeramente diferente a la función genérica `transferFrom`. El contrato del DAI permite "aprobaciones ilimitadas". Si el usuario aprueba una _address_ para el valor máximo de uint256, esa _address_ tendrá una aprobación ilimitada hasta que se indique lo contrario.
2. `push`, `pull` y `move` son alias para la llamada de `transferFrom` en forma de `transferFrom(msg.sender, usr, amount)` , `transferFrom(usr, msg.sender, amount)` y `transferFrom(src, dst, amount)` .
3. `permit` es una función de aprobación basada en firmas. Esto permite que un usuario final firme un mensaje que luego puede ser retransmitido por otra de las partes para presentar su aprobación. Esto puede ser útil para solicitudes en las que el usuario final no necesita tener 'ETH'.
   * Para usar esta funcionalidad, la _address_ de un usuario debe firmar un mensaje con el `holder` (titular), `spender` (gastador), `nonce` (permiso único), `expiry` (caducidad) y la cantidad `allowed` (permitida). Esto se puede enviar a `Permit()` para actualizar la aprobación del usuario.
   
## 4. _Gotchas_ (Posibles fuentes de error del usuario)

La asignación ilimitada es una práctica relativamente poco común (aunque cada vez es más común). Esto podría ser algo usado para engañar a un usuario mediante un contrato malicioso para que le dé acceso a todos sus DAI. Esto es preocupante en el caso de los contratos actualizables donde el contrato puede parecer inocente hasta que se actualice a un contrato malicioso.

El DAI también es susceptible a la ya conocida [condición de carrera de ERC20](https://github.com/0xProject/0x-monorepo/issues/850) pero, normalmente, con aprobaciones ilimitadas, esto no debería ser un problema. Recomendamos a todos los usuarios que utilicen el `approval` para un monto específico, que estén atentos a este problema en particular y que sean cautelosos a la hora de autorizar a otros contratos que realicen tranferencias en su nombre.

Hay una ligera desviación en la funcionalidad del `transferFrom` (transferir de): si la `src == msg.sender` (la fuente es igual al mensaje del que lo envía), la función no requiere de `approval` (aprobación) y la trata como si fuera una `transfer` (transferencia) normal del `msg.sender` (mensaje del que lo envía) al `dst`.

#### Funcionalidad de Metatransacción incorporada del DAI

El token del DAI provee aprobaciones _offchain_, lo cual significa que, como dueño de una _address_ de ETH, se puede firmar un permiso (usando la función `permit()`) el cual, básicamente, otorga asignación a otra _address_ de ETH. La _address_ de ETH a la cual se le otorga el permiso puede entonces hacerse cargo de la ejecución de la transferencia pero tiene una asignación.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

* N/A