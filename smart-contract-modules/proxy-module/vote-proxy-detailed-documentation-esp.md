# _Proxy_ de Votación - Documentación Detallada

* **Nombre de Contrato:** VoteProxy.sol
* **Tipo/Categoría:** Módulo Proxy
* \*\*\*\*[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* \*\*\*\*[**Fuente de Contrato**](https://github.com/makerdao/vote-proxy/blob/master/src/VoteProxy.sol)

## 1. Introducción (Sumario)

El contrato de `VoteProxy` (_Proxy_ de Votación) permite votar a los usuarios de MKR con una _wallet_ tanto fría como caliente (_Cold_ y _Hot_) usando una identidad de votación _proxy_ en vez de interactuar directamente con el _chief_ (jefe). Además de admitir dos mecanismos de votación diferentes, el *proxy* de votación también minimiza el tiempo que los poseedores de MKR necesitan para tener su/s _wallet/s_ en línea.

![](https://i.imgur.com/cJ2NslE.png)

## 2. Detalles del Contrato

### _Proxy_ de Votación (Glosario)

* `approvals`: Un mapeo de las _addresses_ de los candidatos a su peso de `uint`.
* `slate` - Un mapeo de `bytes32` a matrices de `addresses`. Representa conjuntos de candidatos. Se otorgan votos ponderados en las listas.
* `votes`: Un mapeo de las _addresses_ de los votantes a las listas por las que han votado.
* `GOV`: `DSToken` utilizado para votar.
* `IOU`: `DSToken` emitido a cambio de bloquear tokens `GOV`.

## 3. Conceptos y Mecanismos Claves

El contrato de `VoteProxy` permite, a los poseedores de MKR, votar con el peso total de MKR que poseen tanto para las votaciones ejecutivas como para las de Gobernanza. Como se mencionó arriba, este proceso también reduce el riesgo que tienen los usuario de MKR al votar con una _wallet_ fría (_Cold_). Esto se hace al permitir que el poseedor de MKR designe una "_wallet_ caliente" (_Hot_) que se usa para transferir MKR al _proxy_ y solo se puede usar para votar en las votaciones Ejecutivas y de la Gobernanza. La "_wallet_ caliente" se puede utilizar para bloquear MKR en el sistema de votación y devolverlo a su _wallet_ fría.

### Funcionalidades Claves (como se definen en el _smart contract_)

**`auth`** - Chequea que el remitente sea una _wallet_ fría o caliente.

**`lock`** - Cobra al usuario tokens MKR `wad`, emite una cantidad igual de tokens IOU al `VoteProxy` y agrega peso `wad` a los candidatos en la lista seleccionada por el usuario.

**`free`** - Cobra al usuario tokens IOU `wad`, emite una cantidad igual de tokens MKR al usuario y reduce el peso `wad` a los candidatos en la lista seleccionada por el usuario.

**`vote`** - Guarda un conjunto de *addresses* ordenadas como una lista, mueve el peso del votante de su lista actual a la nueva y devuelve el identificador de dicha lista.

**`vote(bytes32 slate)`** - Quita el peso del votante de su actual lista y lo agrega a la lista especificada.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

* **Costo Único de Configuración del _Proxy_**
  * Como un nuevo usuario de contrato del _proxy_, tendrás que configurarlo antes de puedas utilizarlo en futuras votaciones. El precio de la configuración dependerá del precio del _gas_ en Ethereum en ese momento, pero esto hará que el votación sea más segura y fácil para los usuarios.
* Cualquier MKR que haya sido movido/transferido del _proxy_ de votación de un usuario durante la votacíón de encuestas, será removido/sustraído de cualquier encuesta existente por la cual, el usuario, ha votado. Para que tu voto cuente, tiene que asegurarte de que el MKR se encuentre en la _wallet_ cuando la encuesta finalice.
* **Nota:** Para los usuario que no deseen usar el `VoteProxy`, ahora pueden votar directamente con una sola _wallet_; depositando, directamente, en el _Chief_ y, luego, votando con su _wallet_.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

* La pérdida de llaves privadas para ambas _wallets_, tanto frías como calientes, te impedirá votar.