# Pausa - Documentación Detallada

* **Nombre del Contrato:** pause.sol
* **Tipo/Categoría:** Módulo de la Gobernanza
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* ****[**Fuente del Contrato**](https://github.com/dapphub/ds-pause/blob/master/src/pause.sol)

## 1. Introducción (Sumario)

El `ds-pause` es un _proxy_ basado en una _delegatecall_ (llamada de delegados) con un retraso forzoso. Esto permite a los usuarios autorizados programar llamadas de función que solo se pueden ejecutar una vez que haya transcurrido un período de espera predeterminado. El atributo de retraso configurable establece el tiempo de espera mínimo que se utilizará durante la gobernanza del sistema.

![](https://i.imgur.com/cJ2NslE.png)

## 2. Detalles del Contrato:

#### Funcionalidades Claves (como se definen en el _smart contract_)

**Planes** Un plan describe una única operación _delegatecall_ y una marca de tiempo unix, `eta`, antes de la cual no puede ser ejecutada.

**Un plan consiste de:**

* `usr`: _address_ a la cual se le realizará la _delegatecall_
* `tag`: el _codehash_ esperado de usr
* `fax`: _calldata_ (información de la llamada) a usar
* `eta`: primera vez posible de ejecución (como segundos, desde la época de Unix)

Es importante remarcar que cada plan tiene un id único, definido como un "keccack256" (abi.encode(usr, tag, fax, eta)).

#### **Operaciones**

Los planes pueden ser manipulados de la siguientes formas:

* `plot`: programar un plan
* `exec`: ejecutar un plan programado
* `drop`: cancelar un plan programado

El contrato `pause` (pausa) contiene al contrato `DSPauseProxy` para permitir que el plan se ejecute en un contexto de almacenamiento aislado para proteger a la pausa de la modificación maliciosa del almacenamiento durante la ejecución del plan.

## 3. Conceptos y Mecanismos Claves

El `ds-pause` fue diseñado para ser utilizado como un componente en el sistema de la Gobernanza del Protocolo de Maker para darle el tiempo necesario a las partes afectadas para responder ante decisiones. Si aquellos que se encuentran afectados por las decisiones de la Gobernanza tienen, por ejemplo, derechos de salida o de veto, entonces la pausa puede servir como un control efectivo del poder de la Gobernanza.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

#### **Identidad y Confianza**

Para proteger el almacenamiento interno de la *pausa* de escrituras maliciosas durante la ejecución del plan, se realiza una operación _delegatecall_ en un contrato separado con un contexto de almacenamiento aislado (DSPauseProxy) donde cada *pausa* tiene su propio _proxy_ individual.

Esto significa que los planes se ejecutan con la identidad del `proxy`. Por lo tanto, al integrar la *pausa* en algún esquema de autenticación, querrás confiar en el _proxy_ de la *pausa* y no en la *pausa* en sí.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

**La ruptura de cualquiera de los siguientes se clasificará como un problema crítico:**

**General**

* No hay manera de eludir el retraso.
* El código ejecutado por el _delegatecall_ no puede, directamente, modificar el almacenamiento de la *pausa*.
* La *pausa* siempre retendrá la titularidad de su _proxy_.

**Administrativo**

* La autoridad, el dueño y el retraso solo pueden ser cambiados si un usuario autorizado traza un plan para hacerlo.

**Planeación**

* Un plan solo puede ser planeado/trazado si su "eta" es posterior a "block.timestamp + delay".
* Un plan solo puede ser planeado/trazado por usuarios autorizados.

**Ejecución**

* Un plan solo puede ser ejecutado si este fue previamente planeado/trazado. 
* Un plan solo puede ser ejecutado una vez que su "eta" haya pasado.
* Un plan solo puede ser ejecutado si su _tag_ coincide con el "extcodehash(usr)"."
* Un plan solo puede ser ejecutado por única vez.
* Un plan puede ser ejecutado por cualquiera.

**Eliminación**

* Un plan solo puede ser eliminado por usuarios autorizados.

#### Otros Modos de Falla

`DSPause.delay` - cuando el retraso de la pausa es establecido en su valor máximo, la Gobernanza no puede volver a modificar el sistema.

`DSPause.delay` - cuando el retraso de la pausa es establecido en su valor mínimo, es más fácil pasar acciones maliciosas de la Gobernanza.