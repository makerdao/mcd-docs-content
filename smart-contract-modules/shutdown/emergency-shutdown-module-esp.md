# ESM - Documentación Detallada

* **Nombre del Contrato:** esm.sol
* **Tipo/Categoría:** Modulo de Apagado de Emergencia
* [**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* [**Fuente del Contrato**](https://github.com/makerdao/esm/blob/master/src/ESM.sol)
* [**Etherscan**](https://etherscan.io/address/0x09e05ff6142f2f9de8b6b65855a1d56b6cfe4c58#code)

## 1. Introducción (Resumen)

El Módulo de Apagado de Emergencia (ESM) es un contrato con la habilidad de llamar `End.cage()` para activar el Apagado del Protocolo Maker.

![](<../../.gitbook/assets/MCD System 2.0.png>)

## 2. Detalles del Contrato

### ESM (Glosario)

**Funcionalidades Clave (definidas en el _smart contract_)**

`rely` - Concede poderes de administrador a una _address_

`deny` - Revoca poderes de administrador a una _address_

`file` - Le permite al administrador actualizar el umbral `min` y la _address_ de `end`

`cage` - Desactiva permanentemente el módulo de apagado

`fire` - Activa el apagado llamando a `End.cage`

`denyProxy` - Siguiendo el patrón de rely/deny de las salas, las llamadas se deniegan en un contrato determinado

`join` - Deposita MKR al módulo de apagado

`burn` - Quema cualquier MKR depositado en el módulo de apagado

**Other**

`gem` - Contrato del token MKR \[_address_]

`wards(admin: address)` - Determina si una _address_ tiene poderes de administrador \[_address_: uint]

`sum(usr: address)` - Balance join de MKR del usuario \[_address_: uint]

`Sum` - MKR total depositado \[uint]

`min` - Cantidad mínima de MKR requerida para `fire` y `denyProxy` \[uint]

`end` - El contrato End \[_address_]

`live` - Determinado si el contrato está 'live' (no 'caged') \[uint]

## 3. Mecanismos y Conceptos Clave

Los poseedores de MKR que deseen activar el Apagado deben `join` (reunir y depositar) MKR dentro del ESM, el cual es quemado inmediatamente. Cuando la variable `Sum` interna del ESM es igual o mayor que el umbral mínimo (`min`), cualquiera puede llamar al método `fire()` del ESM. Este método, a su vez, llama a `End.cage()`, que inicia el proceso de Apagado. 

**El ESM está pensado para ser utilizado en algunos posibles escenarios:**

* Para mitigar Gobernanza maliciosa
* Para evitar que se aprovechen de una vulnerabilidad critica (por ejemplo, una falla que permita que el colateral sea robado)

En el caso de un ataque malicioso de la Gobernanza, los _joiners_ (participantes) no tendrán ninguna expectativa de recuperar sus fondos (ya que necesitarían una mayoría maliciosa para aprobar el voto requerido) y su única opción es establecer un _fork_ (bifurcación) alternativo en el que se recorten los fondos de la mayoría y se restablezcan los suyos.
En otros casos, los poseedores de MKR restantes pueden optar por reembolsar a los que se unieron al ESM, acuñando nuevos tokens.

**Nota:** si la Gobernanza desea desarmar el ESM, solo pueden hacerlo removiendo su autorización de llamar a `end.cage()` antes de que el ESM sea activado.  

## 4. Gotchas (Posibles Fuentes de Error del Usuario)

### Fondos Irrecuperables

Es importante para los usuarios tener en cuenta que ingresar MKR dentro del ESM es irreversible; es decir que se pierde para siempre, independientemente de que el Apagado sea ejecutado exitosamente o no. Aunque si bien es posible que el resto de poseedores de MKR puedan votar y acuñar nuevos tokens para aquellos que los han perdido activando el ESM, no hay una garantía de ello.

### Mal Configuración de los Parámetros 

Los parámetros que gobiernan el ESM son establecidos al momento de la creación y no pueden ser cambiados (sin volver a desplegar el contrato ESM); por lo tanto, hay que tener cuidado de asegurarse que sean correctos y que permitan al contrato funcionar apropiadamente. 

### Teoría de Juego de la Financiación y Activación del ESM

Una entidad que desea activar el ESM pero no posee la suficiente cantidad de MKR para hacerlo de forma independiente debe proceder con cuidado. Esta entidad podría simplemente enviar MKR al ESM para señalar su deseo y esperanza de que otros se unan; esto, sin embargo, es una mala estrategia. La Gobernanza, así sea honesta o maliciosa, se dará cuenta y probablemente proceda a desautorizar el ESM antes de que se alcance el punto de inflexión. Está claro por qué una Gobernanza maliciosa lo haría, pero una Gobernanza honesta podría actuar de forma similar—por ejemplo: para evitar que el sistema sea apagado por _trolls_ o simplemente para mantener un umbral constante para la activación del ESM. (Se esperaría entonces que, tanto una Gobernanza honesta como una Gobernanza caracterizada como maliciosa, reemplacen el ESM). Si la Gobernanza tiene éxito en esto, la entidad simplemente habrá perdido sus MKR sin lograr nada.

Si una entidad con MKR insuficientes desea activar el ESM, es mejor que se coordine con otros ya sea _off-chain_ o, idealmente, a través de un _smart contract_ que sea _trustless_ (sin confianza). Si se usa un _smart contract_, lo mejor sería que empleara criptografía de conocimiento cero y otras técnicas de preservación de la privacidad (como los repetidores de transacciones) para tratar de ocultar la información tanto de la cantidad actual de MKR, así como las direcciones de quienes se han unido.

Si una entidad piensa que otros se unirán antes de que la Gobernanza reaccione (por ejemplo: si el retraso de las acciones de la Gobernanza es muy largo), aún es posible que el envío directo de MKR insuficiente al ESM puede funcionar, pero implica un alto grado de riesgo. La Gobernanza podría incluso confabular con los mineros para evitar llamadas `cage`, etc, si sospecha que se está organizando la activación de un ESM y desean evitarlo.  

## 5. Modos de Falla (Limites en las Condiciones de Funcionamiento y Factores de Riesgo Externos)

### Mal Configuración de la Autorización 

El ESM por sí mismo no tiene un modo de fallo aislado, pero si las otras partes del sistema no tienen las configuraciones de autorización adecuadas (por ejemplo: el contrato Final no autoriza que el ESM llame a `cage()`), entonces el método `fire()` del ESM puede ser incapaz de activar el proceso de Apagado, incluso si se ha comprometido suficiente MKR en el contrato.  