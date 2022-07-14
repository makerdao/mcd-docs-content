# _Median_ - Documentación Detallada

* **Nombre del Contrato:** median.sol
* **Tipo/Categoría:** Módulo de Oráculos
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/median/blob/master/src/median.sol)

## 1. Introducción (Resumen)

El _median_ ("número medio") proporciona el precio de referencia confiable de Maker. En resumen, funciona manteniendo una _whitelist_ ("lista blanca") de contratos de alimentación de precios autorizados a publicar actualizaciones de precios. Cada vez que una nueva lista de precios es recibida, el _median_ de estos es calculado y utilizado para actualizar el valor almacenado. El _median_ tiene una lógica de permisos que es la que permite agregar y retirar direcciones de alimentación de precios de la _whitelist_ (lista blanca) que son controladas mediante la Gobernanza. La lógica de permisos permite a la Gobernanza fijar otros parámetros que controlen el comportamiento del _median_, por ejemplo, el parámetro `bar` es el número mínimo de precios necesarios para aceptar un nuevo valor de _median_.  

**Diagrama general de alto nivel de los componentes que se involucran e interactúan con la _median_ :**

![](../../.gitbook/assets/oracles.png)

**Nota:** Todas las flechas sin etiquetas son llamadas de Gobernanza.

## 2. Detalles del Contrato

### _Median_ (Glosario)

#### Funcionalidades Claves (como están definidas en el contrato inteligente)

* `read` - Obtiene un precio distinto a cero o falla.
* `peek` - Obtiene el precio y la validez.
* `poke` - Actualiza precios de los proveedores de la _whitelist_.  
* `lift`- Agrega una dirección a la _whitelist_ de los escritores.  
* `drop` - Elimina una dirección de la _whitelist_ de los escritores.  
* `setBar` - Establece el `bar`.
* `kiss` - Agrega una dirección a la _whitelist_ de los lectores. 
* `diss` - Elimina una dirección de la _whitelist_ de los lectores. 

**Nota:** `read` devuelve el `value` o falla si es inválido y `peek` devuelve el `value` y si el `value` es válido o no.

#### Otro

* `wards(usr: address)` - mecanismos de autenticación.
* `orcl(usr: address)` - `val` _whitelist_ de escritores / firmantes de los precios (añadidos a la _whitelist_ mediante la Gobernanza / las partes autorizadas).
* `bud(usr: address)` - `val` _whitelist_ de escritores.
* `val` - el precio (privado) debe ser leído con `read()` o `peek()`.
* `age` - la marca de tiempo del bloque de la última actualización del precio `val`. 
* `wat` - el tipo de oráculo de precios (ejemplo: ETHUSD) / nos dice qué tipo de activo es.  
* `bar` - el quórum mínimo de escritores para `poke` / número mínimo de mensajes válidos que necesita tener para actualizar el precio.

## 3. Mecanismos y Conceptos Clave

Como se mencionó anteriormente, _´median´_ es el _smart contract_ que proporciona el precio de referencia confiable de Maker. La autorización **(auth)** es un componente clave incluido en el mecanismo de este contrato y sus interacciones. Por ejemplo, el precio (`val`) no es público (intencionalmente) porque la intención es solo leerlo desde las dos funciones `read` y `peek`, que están en la _whitelist_. Esto significa que es necesario estar autorizado, lo que se completa a través del `bud`. El `bud` se modifica para que las autoridades de la _whitelist_ puedan leerlo _on-chain_ (con permiso), mientras que todo lo que esta _off-chain_ es público.

El método `poke` no está bajo ningún tipo de _`auth`_. Esto significa que cualquier persona puede llamarlo. Esto fue diseñado con el propósito de hacer que los _keepers_ llamen esta función e interactúen con las Subastas. La única forma de modificar su estado es si llamas y envías los datos válidos. Por ejemplo, digamos que este oráculo necesita 15 fuentes diferentes; esto significa que necesitaríamos enviar 15 firmas diferentes. Entonces procederá a cotejar cada uno de ellos y validar que quien ha enviado los datos ha sido _`auth`_ ("autorizado") a hacerlo. En el caso de ser un oráculo autorizado, revisará si firmo el mensaje con una marca de tiempo que es más grande que la anterior. Esto está hecho con el propósito de asegurar que esto no es un mensaje viejo. El próximo paso es comprobar los valores de los pedidos, esto requiere que envíes todo en una matriz que está formateada en orden ascendente. Si no se envía en el orden correcto (ascendente), el _median_ no se calcula correctamente. Esto se debe a que si asumes que los precios están ordenados, solo tomará el valor del medio, el cual puede no ser suficiente o no funcionar. Para comprobar la unicidad, hemos implementado el uso de un filtro _`bloom`_. En resumen, un filtro _`bloom`_ es una estructura de datos diseñada para decirnos de forma rápida y con eficiencia de memoria, qué elemento está presente en un conjunto. Esta utilización del filtro _bloom_ ayuda con la optimización. Para agregar firmantes a la _whitelist_, los primeros dos caracteres de sus direcciones (el primer `byte`) tiene que ser único. Por ejemplo, digamos que tienes 15 firmantes de precios diferentes, ninguno de los primeros dos caracteres de sus direcciones puede ser el mismo. Esto ayuda a filtrar que todos los 15 firmantes sean diferentes.

A continuación, están las funciones `lift`. Estas funciones nos dicen quién puede firmar mensajes. Se pueden enviar múltiples mensajes o puede ser solo uno, pero se colocan en el oráculo autorizado. Sin embargo, actualmente no hay nada que evite que alguien _`lift`'ing_ ("levante") dos firmantes de precios que empiecen con la misma dirección. Esto es algo respecto a lo que, por ejemplo, la Gobernanza necesita estar atenta. (revisa un ejemplo de cómo se ve una propuesta de Gobernanza en este caso, en la sección **Gotchas** ) 

Debido al diseño de mecanismo de funcionamiento de los oráculos, el **quórum** tiene que ser un número impar. Si es un número par, no funcionará. Esto fue diseñado como una optimización (`val = uint128(val_[val_.length >> 1]);`); este fragmento de código describe cómo funciona, que es tomando la matriz de valores (todos los precios que cada uno de los firmantes de los precios informó, ordenados de 200 a 215) y luego tomar el que está en el medio. Esto se hace tomando la longitud de la matriz (15) y desplazándola hacia la derecha en 1 (que es lo mismo que dividirlo por 2). Esto termina siendo 7.5 y después el EVM lo baja a 7. Si fuéramos a aceptar números pares, sería menos eficiente. Esto plantea el problema de que deberías tener un balance definido entre cuántos requieres y cuántos firmantes tienes en realidad. Por ejemplo, digamos que el oráculo necesita 15 firmas, necesitas al menos 17-18 firmantes porque si requieres 15, solo tienes 15 y uno de ellos se cae, no tienes manera de modificar el precio, así que siempre deberías tener un poquito más. Sin embargo, no deberías tener demasiados, ya que podría comprometer la operación.  

## 4. Gotchas

#### **Oráculos de Emergencia**

* Pueden cerrar la fuente de alimentación de precios, pero no pueden abrirla nuevamente. Para que pueda funcionar de nuevo la fuente de alimentación de precios debe intervenir la Gobernanza. 

#### **Congelación de Precios**

* Si anulas el módulo Ethereum de oráculos, la idea es que no puedas interactuar con ninguna _vault_ que dependa de ese _ilk_.  
  * **Ejemplo:** Apagado ETHUSD (aún puedes agregar colateral y pagar la deuda - aumenta la seguridad) pero no puedes hacer nada que aumente el riesgo (disminuye la seguridad - elimina colateral, genera DAI, etc.) porque el sistema no sabría si estás subcolateralizado o no.  

#### **Los Oráculos Requieren Mucho Mantenimiento**

* Necesitan mantener a todos los relés funcionando. 
* La comunidad necesitaría autovigilarse (observando cada firmante del precio, etc.) si alguno de ellos tiene que ser reemplazado. Necesitarían asegurarse que se les llama constantemente cada hora (por cada hora, una transacción es enviada al OSM, lo que significa que unas pocas transacciones aún han sido enviadas a la _median_ para actualizarla también. Además, tendría que haber una transacción enviada al `spotter`, ya que el _DSS_ opera en un método tipo _pool_ (no actualiza el sistema/escribe en él, dile que lo lea del OSM).

#### **No hay nada que evite que sean _`lift`'ed_ (_levantados_) dos firmantes de precios que comiencen con la misma dirección** 

* La única cosa que esto evita es que no puedas tener más de 256 oráculos, pero no esperamos tener tantos, así que es un límite difícil. Sin embargo, la Gobernanza necesita asegurarse que quien sea que esté votando a cualquiera que ya haya votado antes, sea con los mismos dos primeros caracteres.
* Un ejemplo de cómo se vería una propuesta de la Gobernanza en este caso: 
  * Estamos añadiendo un nuevo oráculo y proponiendo (la Gobernanza) una lista de firmantes (que han sido utilizados en el pasado); ya tenemos un oráculo, pero queremos agregar algunos nuevos (ejemplo Dharma o dydx). Diríamos que ellos quieren ser firmantes de precios, así que estas son sus direcciones y queremos _lift_ ("levantar") esas dos direcciones. Ellos votarían por eso, nosotros necesitaríamos mantener una lista de todas las direcciones existentes y ellos necesitarían crear una dirección que no entre en conflicto con las que ya existen.  

## 5. Modos de Falla (Límites en las Condiciones de Funcionamiento y Factores de Riesgo Externos)
* Por diseño, actualmente no hay manera de desactivar el oráculo (falla o devuelve _false_) si todos los oráculos se juntan y firman un precio de cero. Esto resultaría en que el precio sea inválido y devolvería _false_ en `peek`, diciéndonos que no confiemos en el valor. 
* Actualmente, estamos investigando (módulo ETH de oráculos) que eso invalidaría el precio, pero no hay forma de hacerlo en el _median_ hoy. Esto se debe a la separación de preocupaciones que el DSS no lee directamente desde el _median_, lee desde el OSM. Pero esto puede terminar cambiando. 
