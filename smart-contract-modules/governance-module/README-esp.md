# Módulo de la Gobernanza

* **Nombre del Módulo:** Módulo de la Gobernanza
* **Tipo/Categoría:** Gobernanza —&gt; Chief.sol, Pause.sol, Spell.sol
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ******Fuente de los Contratos:**
  * [**_Chief_**](https://github.com/dapphub/ds-chief/blob/master/src/chief.sol)
  * [**_Pause_**](https://github.com/dapphub/ds-pause/blob/master/src/pause.sol)
  * [**_Spell_**](https://github.com/dapphub/ds-spell/blob/master/src/spell.sol)

## 1. Introducción (Sumario)

El Módulo de la Gobernanza contiene los contratos que facilitan la votación en MKR, la ejecución de propuestas y las seguridades de votación del Protocolo de Maker.

## 2. Detalles del Módulo

El Módulo de la Gobernanza tiene 3 componentes principales que consisten de los contratos `Chief` (jefe), `Pause` (pausa) y `Spell` (hechizo).

### Documentación de los Componentes del Módulo de la Gobernanza

* [**_Chief_ - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/governance-module/chief-detailed-esp)
* [**_Pause_ - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/governance-module/pause-detailed-documentation-esp)
* [**_Spell_ - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/governance-module/spell-detailed-documentation-esp)

## 3. Conceptos y Mecanismos Clave

#### Resumen de los **Componentes del Módulo** de la Gobernanza

* `Chief` - El _smart contract_ (contrato inteligente) `Ds-Chief` proporciona un método para elegir un contrato **"chief"** (jefe) a través de un sistema de votación de aprobación. Esto se puede combinar con otro contrato, como `DSAuthority`, para elegir un conjunto de reglas para un sistema de _smart contract_. 
* `Pause` - `ds-pause` es un _proxy_ basado en una _delegatecall_ (llamada de delegados) con un retraso forzoso. Esto permite a los usuarios autorizados programar llamadas de función que solo se pueden ejecutar una vez que haya transcurrido un período de espera predeterminado. El atributo de retraso configurable establece el tiempo de espera mínimo que se utilizará durante la gobernanza del sistema.
* `Spell` - Un `DS-Spell` es un objeto sin propietario que realiza, por única vez, una acción o una serie de acciones atómicas (múltiples transacciones). Esto se puede considerar como un `DSProxy` único, sin propietario (sin mezcla de `DSAuth`, no es un `DSThing`).

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

* `Chief`
  * En general, cuando nos referimos al **"chief"**, puede ser tanto como _addresses_ como gente que representa contratos. Es decir, `ds-chief` puede trabajar igual de bien tanto como un método para seleccionar el código para la ejecución como para realizar proceso políticos.
  * **Token IOU:** El propósito del token IOU es el de permitir el encadenamiento de los contratos de la Gobernanza. En otras palabras, esto te permite tener una cantidad de `DSChief`, `DSPrism` u otros contratos similares que usen el mismo token de la Gobernanza mediante la aceptación del token IOU del contrato `DSChief` antes de que sea un token de la Gobernanza.
  * **Votación de Aprobación:** Este tipo de votación es cuando cada votante selecciona qué candidatos aprueban, siendo elegidos los candidatos _n_ "más aprobados". Cada votante puede emitir hasta _n_ + _k_ votos, donde _k_ es igual a un número entero positivo distinto a cero. Lee más [aquí](https://docs.makerdao.com/smart-contract-modules/governance-module/jefe-detailed-documentation#approval-voting).
  * **Implementaciones:** Si estás escribiendo una interfaz de _front-end_ para este _smart-contract_, por favor, ten en cuenta que los parámetros de la _address_\[\] que se pasan a las funciones `etch` y `vote` deben ser conjuntos ordenados por bytes. Lee más [aquí](https://docs.makerdao.com/smart-contract-modules/governance-module/chief-detailed-documentation-esp#implementaciones).
* `Pause`
  * **Identidad y Confianza:** Para proteger el almacenamiento interno de la *pausa* de escrituras maliciosas durante la ejecución del plan, se realiza una operación _delegatecall_ en un contrato separado con un contexto de almacenamiento aislado (DSPauseProxy) donde cada *pausa* tiene su propio _proxy_ individual. Esto significa que los planes se ejecutan con la identidad del `proxy`. Por lo tanto, al integrar la *pausa* en algún esquema de autenticación, querrás confiar en el _proxy_ de la *pausa* y no en la *pausa* en sí.
* `Spell`
  * El _spell_ solo se marcará como "realizado" si la LLAMADA que realiza es exitosa, significando que no se revirtió ni terminó en una condición excepcional. Por el contrario, los contratos que usan valores devueltos en lugar de excepciones para indicar errores podrían llamarse con éxito sin tener el efecto que uno podría desear. La "aprobación" de _spells_ para tomar medidas en un sistema después de que se implementa el _spell_, generalmente, requiere que el sistema use el manejo de errores basado en excepciones para evitar problemas.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

* `Chief`
  * **Usuarios de MKR que mueven sus votos de un _spell_ a otro:** Uno de los mayores y potenciales errores ocurre cuando la gente mueve su voto de un _spell_ a otro. Esto abre una brecha/período de tiempo en donde se requiere de un pequeño monto de MKR se necesita para levantar un _hat_ al azar.
* `Pause`
  * No hay manera de eludir el retraso.
  * El código ejecutado por el _delegatecall_ no puede, directamente, modificar el almacenamiento de la *pausa*.
  * La *pausa* siempre retendrá la titularidad de su _proxy_.
  * Lee más [aquí](https://docs.makerdao.com/smart-contract-modules/governance-module/pause-detailed-documentation-esp#5-failure-modes-bounds-on-operating-conditions-and-external-risk-factors).
* `Spell`
  * El principal modo de falla del `spell` surge cuando hay una instancia del _spell_ que permanece sin ser utilizado cuando tiene una cantidad de votos de MKR que luego se convierte en un objetivo.