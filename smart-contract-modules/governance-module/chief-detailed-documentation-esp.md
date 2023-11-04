# _Chief_ - Documentación Detallada

* **Nombre del Contrato:** chief.sol
* **Tipo/Categoría:** Módulo de la Gobernanza
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* ****[**Fuente del Contrato**](https://github.com/dapphub/ds-chief/blob/master/src/chief.sol)
* ****[**Etherscan**](https://etherscan.io/address/0x0a3f6849f78076aefaDf113F5BED87720274dDC0#code)

## 1. Introducción (Sumario)

El _smart contract_ `Ds-Chief` provee un método para elegir un contrato _"chief"_ a través de un sistema de votación de aprobación. Este puede estar combinado con otro contrato, como `DSAuthority`, para elegir un conjunto de reglas para un sistema de _smart contract_.

En resumen, los votantes bloquearán sus tokens de votación para darle más peso a su voto en el sistema. Luego, la votación se realiza mediante [votación de aprobación](https://en.wikipedia.org/wiki/Approval\_voting) continua, donde los votantes reciben tokens IOU cuando bloquean sus tokens de votación, lo cual es útil para los mecanismos secundarios de la Gobernanza. Los tokens IOU no se pueden cambiar por tokens bloqueados, excepto por personas que realmente hayan bloqueado fondos en el contrato mismo, y solo hasta la cantidad que hayan bloqueado. Los tokens IOU no se pueden cambiar por tokens bloqueados, excepto por personas que realmente hayan bloqueado fondos en el contrato mismo y solo hasta la cantidad que hayan bloqueado.

![Las interacciones del contrato de la Gobernanza con el sistema](https://i.imgur.com/cJ2NslE.png)

## 2. Detalles del Contrato

### Glosario (_Chief_)

**`DSChiefApprovals` provee las siguientes propiedades públicas:**

* `slate` - Un mapeo de `bytes32` a matrices de `addresses`. Representa conjuntos de candidatos. Se otorgan votos ponderados en las listas.
* `votes`: Un mapeo de las _addresses_ de los votantes a las listas por las que han votado.
* `approvals`: Un mapeo de las _addresses_ de los candidatos a su peso de `uint`.
* `deposits`: Un mapeo de _addresses_ votantes al número `uint` de tokens bloqueados.
* `GOV`: `DSToken` utilizado para votar.
* `IOU`: `DSToken` emitido a cambio de bloquear tokens `GOV`.
* `hat`: Contiene la _address_ del _"chief"_ actual.
* `MAX_YAYS`: El número máximo de candidatos que puede tener una lista.

La mayoría de las funciones están decoradas con el modificador `note` (nota) de [ds-note](https://dapp.tools/dappsys/ds-note.html), significando que activará un evento estandarizado cuando se lo llame. Además, también se provee un evento personalizado:

* `Etch(bytes32 indexed slate)`: Se activa cuando una lista es creada.

## 3. Conceptos y Mecanismos Claves

**Hay dos contratos en ds-chief:**

1. DSChiefApprovals
2. DSChief, que hedera de DSChiefApprovals.

#### Funcionalidades Claves (como se definen en el _smart contract_)

#### DSChiefApprovals

#### **`DSChiefApprovals(DSToken GOV_, DSToken IOU_, uint MAX_YAYS_)`**

* El constructor. Establece `GOV`, `IOU` y `MAX_YAYS`.

#### **`lock(uint wad)`**

* Carga el `wad` del usuario con tokens `GOV`, emite la misma cantidad en tokens `IOU` al usuario y agrega peso `wad` de los candidatos en las listas seleccionadas por los usuarios. Activa un evento `LogLock`.

#### **`free(uint wad)`**

* Carga el `wad` del usuario con tokens `IOU`, emite la misma cantidad en tokens `GOV` al usuario y resta peso `wad` de los candidatos en las listas seleccionadas por los usuarios. Activa un evento `LogFree`.


#### **`etch(address[] yays) devuelve (bytes32 slate)`**

* Guarda una serie de _addresses_ ordenadas como una `slate` (lista) y devuelve un identificador único para la misma.

#### **`vote(address[] yays) devuelve (bytes32 slate)`**

* Guarda una serie de _addresses_ ordenadas como una `slate`(lista), mueve el peso del votante de la lista actual a la nueva y devuelve el identificador de la lista.

#### **`vote(bytes32 slate)`**

* Remueve peso del votante de su lista actual y se agrega a la lista especificada.

#### **`lift(address whom)`**

* Chequea la _address_ dada y la promociona a `s/chief/hat` si esta tiene más peso que la actual `s/chief/hat`.

#### DSChief

`DSChief` es una combinación de `DSRoles` del paquete `ds-roles` y `DSChiefApprovals`. Se puede utilizar junto con `ds-auth` (como un objeto de autorización) para regir los sistemas de _smart contracts_.

#### Funciones Públicas

#### **`DSChief(DSToken GOV_, DSToken IOU_, uint MAX_YAYS_)`**

* El constructor. Establece `GOV`, `IOU` y `MAX_YAYS`.

#### **`setOwner(address owner_)`**

* Revierte la transacción. Anulado de `DSAuth`.

#### **`setAuthority(DSAuthority authority_)`**

* Revierte la transacción. Anulado de `DSAuth`.

#### **`isUserRoot(address who) devuelve constantemente un (bool)`**

* Devuelve `true` (verdadero) si la _address_ actual es la _chief_.

#### **`setRootUser(address who, bool enabled)`**

* Revierte la transacción. Anulado de `DSRoles`.

#### **`DSRoles`**

Para saber acerca de las características heredadas, lee [ds-roles](https://dapp.tools/dappsys/ds-roles.html).

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

En general, cuando nos referimos al **"hat"** (sombrero), este puede ser cualquier _address_, ya sea un contrato de un solo uso, como _ds-spell_, un contrato de usos múltiples o la _wallet_ de una persona. Por lo tanto, _ds-chief_ puede funcionar bien como método para seleccionar código para ejecución, así como para realizar procesos políticos.

**Ejemplo:**

El `ds-chief` podría utilizarse como un sistema de votación ponderado por token que rige otro conjunto de _smart contracts_ que use `ds-auth` con `ds-roles`. En un escenario como este, los "candidatos" consistirían en contratos que cambian el estado del _smart contract_ establecido bajo la Gobernanza. Si se elige un contrato de este tipo como "hat", se le otorgarán todos los permisos para ejecutar cualquier cambio que sea necesario. El `ds-chief` también podría usarse dentro de un contrato de este tipo junto con un contrato _proxy_, como el `ds-proxy` o un sistema de resolución de nombres, como ENS, con el fin de votar en nuevas versiones de contratos.

#### Entendiendo al Token IOU

El propósito del token IOU es el de permitir el encadenamiento de los contratos de la Gobernanza. En otras palabras, esto te permite tener una cantidad de `DSChief`, `DSPrism` u otros contratos similares que usen el mismo token de la Gobernanza mediante la aceptación del token IOU del contrato `DSChief` antes de que sea un token de la Gobernanza.

**Ejemplo:**

Digamos que hay tres contrados `DSChief` (chiefA, chiefB y chiefC) y un `chiefA.GOV` que es el token MKR. Si establecemos `chiefB.GOV` a `chiefA.IOU` y `chiefC.GOV` a `chiefB.IOU`, esto permite que los tres contratos se ejecuten utilizando un grupo común de MKR.

#### **Votación de Aprobación**

Este tipo de votación es cuando cada votante selecciona qué candidatos aprueban, siendo elegidos los candidatos _n_ "más aprobados". Cada votante puede emitir hasta _n_ + _k_ votos, donde _k_ es igual a un número entero positivo distinto a cero. De esta manera, los votantes pueden mover su aprobación de un candidato a otro sin antes tener que retirar el apoyo del candidato que va a ser reemplazado. Sin esto, mover la aprobación a un nuevo candidato podría resultar en que un candidato "menos aprobado" pase, momentáneamente, al grupo de candidatos electos. **Nota:** En el caso de `ds-chief`, _n_ es igual a 1.

Además, el `ds-chief` pesa los votos de acuerdo a la cantidad del token de votación que se ha elegido para bloquear en el contrato `DSChief` o `DSChiefApprovals`. Es importante tener en cuenta que el token de votación utilizado en una implementación de `ds-chief`, debe especificarse en el momento de la implementación y, luego, no puede cambiarse .

#### Implementaciones

Si estás escribiendo una interfaz de _front-end_ para este _smart-contract_, por favor, ten en cuenta que los parámetros de la _address_\[\] que se pasan a las funciones `etch` y `vote` deben ser conjuntos ordenados por bytes.

**Ejemplo:**

Es válido usar `[0x0, 0x1, 0x2, ...]` pero no lo es `[0x1, 0x0, ...]`, ni `[0x0, 0x0, 0x1, ...]`. Esta restricción de orden permite que el contrato asegure, de manera económica, que los votantes no puedan multiplicar sus pesos al incluir al mismo candidato en su lista varias veces.

## 5. 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

* **Los usuarios de MKR que muevan sus votos de un _spell a otro**
  * Uno de los mayores y potenciales errores ocurre cuando la gente mueve su voto de un _spell_ a otro. Esto abre una brecha/período de tiempo en donde se requiere de un pequeño monto de MKR se necesita para levantar un _hat_ al azar.
* **_Lift_ no es llamado en _spells_ que tienen más MKR que el _hat_ actual**
  * La única forma de que un _spell_ obtenga un _hat_, es si se llama a `lift` (levantar) en él. Entonces, más allá de que un _spell_ gane mucho más MKR que un _hat_, si nunca se llama a `lift` en él, el _hat_ permanecerá en un _spell_ que ya no posee la mayor cantidad de MKR. Esto podría bajar la vara para la cantidad de MKR necesario para aprobar algo, haciendo al sistema potencialmente menos seguro.
* **_Spells_ extraviados sin fecha de caducidad**
  * Dada a la contínua naturaleza de la votación, un _spell_ permanecerá activo en el sistema por más que no haya sido aprobado como una propuesta de la Gobernanza. Esto significa que los poseedores de MKR pueden continuar votando al candidato y, en tiempos de una participación baja de votantes, existe la posibilidad de que se genere un modo de falla al votar por un candidato innesperado y/o viejo. Esto ilustra por qué es importante una mayor participación de los votantes y también que suma a la estabilidad del sistema que haya una mayor cantidad de MKR en la propuesta actual de la Gobernanza.
* **Estados inseguros al migrar a un nuevo contrato _chief_**
  * Al migrar a un nuevo _chief_, las autoridades deben transferirse al nuevo contrato y revocarse del anterior. Esto presenta un pequeño problema de coordinación ya que el nuevo contrato debe poseer el MKR necesario en su `hat` para estar seguro contra ataques a la Gobernanza, mientras que, los votantes que promulgan el cambio, deben tener suficiente MKR en el antiguo _chief_ para aprobar la propuesta.
