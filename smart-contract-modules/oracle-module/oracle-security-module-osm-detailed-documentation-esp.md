# Módulo de Seguridad del Oráculo \(OSM\) – Documentación Detallada 

* **Nombre del Contrato:** OSM
* **Tipo/Categoría:** Oráculos – Modulo de Alimentación de Precios 
* \*\*\*\*[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* \*\*\*\*[**Fuente del Contrato**](https://github.com/makerdao/osm/blob/master/src/osm.sol)

## 1. Introducción 

### Resumen

El OSM \(acrónimo de "Oracle Security Module"\ o Módulo de Seguridad del Oráculo) asegura que los nuevos valores de precios difundidos por los Oráculos no son tomados por el sistema hasta que un retraso específico haya pasado. Los valores se leen desde el contrato [DSValue](https://github.com/dapphub/ds-value) designado \(o cualquier contrato que tenga las interfaces `read()` y `peek()` \) mediante el método `poke()`; los métodos `read()` y `peek()` darán el valor actual de la alimentación de precios y otros contratos deben estar en la _whitelist_ (lista blanca) para poder llamarlos. Un contrato OSM solo puede leer desde un sola fuente de alimentación de precios, así que en la práctica se debe implementar un contrato OSM por tipo de colateral.   

![](../../.gitbook/assets/osm.png)

## 2. Detalles del Contrato - Glosario \(OSM\)

### Diseño de Almacenamiento

* `stopped`: bandera \(`uint256`\) que deshabilita las actualizaciones de alimentación de precios si es distinta a cero .
* `src`: `address` (“dirección”) del DSValue del que leerá el OSM .
* `ONE_HOUR`: 3600 segundos \(`uint16(3600)`\).
* `hop`: tiempo de retraso entre las llamadas `poke` \(`uint16`\); por defecto es `ONE_HOUR` (“una hora”).
* `zzz`: hora de la última actualización \(redondeando hacia abajo al múltiplo más cercano de `hop`\).
* `cur`: estructura `Feed` que contiene el valor del precio actual.
* `nxt`: estructura `Feed` que contiene el próximo valor del precio.
* `bud`: mapeo de `address` a `uint256`; lectores de fuentes de alimentación de _whitelists_.

### Métodos Públicos 

#### Métodos Administrativos 

Estas funciones solo se pueden llamar por direcciones autorizadas \(es decir, direcciones `usr` tales que `wards[usr] == 1`\).

* `rely`/`deny`: agregar o remover usuarios autorizados \(mediante modificaciones en el mapeo de `wards` \).
* `stop()`/`start()`: alternar si la alimentación de precios puede ser actualizada \(cambiando el valor de `stopped`\).
* `change(address)`: cambiar la fuente de datos \(estableciendo `src`\).
* `step(uint16)`: cambiar el intervalo entre las actualizaciones de precios \(estableciendo `hop`\).  
* `void()`: similar a `stop`, excepto que también establece `cur` y `nxt` a una estructura `Feed` con valores cero.
* `kiss(address)`/`diss(address)`: agregar/remover consumidores de fuentes de alimentación de precios \(mediante modificaciones al mapeo de `buds`\) 

#### Métodos de Lecturas de Alimentadores  

Estos solo se pueden llamar por las direcciones de la _whitelist_ \(es decir direcciones `usr` tales que `buds[usr] == 1`\):

* `peek()`: devuelve el valor de alimentación actual y un booleano indicando si es válido.  
* `peep()`: devuelve el próximo valor de alimentación \(es decir, el que se convertirá en el valor actual en la próxima llamada `poke()` \) y un booleano indicando si es válido.
* `read()`: devuelve el valor de alimentación actual, se revierte si no fue establecido por un mecanismo válido.

#### Métodos de Actualización de Fuentes de Alimentación 

* `poke()` : actualiza el valor de alimentación actual y lee el próximo .

Estructura `Feed`: una estructura con 2 miembros `uint128`, `val` y `has`. Utilizados para almacenar la _data_ de los alimentadores de precios.

## 3. Mecanismos y Conceptos Clave
 
El mecanismo central del OSM consiste en alimentar periódicamente un precio retrasado en el sistema MCD por un tipo particular de colateral. Para que esto funcione correctamente, un actor externo debe llamar al método `poke()` regularmente para actualizar el precio actual y leer el próximo precio. El contrato rastrea la hora de la última llamada a `poke()` en la variable `zzz` \(redondeando hacia abajo al múltiplo más cercano de `hop`; para más discusiones al respecto, puedes revisar [Modos de Falla](https://docs.makerdao.com/smart-contract-modules/oracle-module/oracle-security-module-osm-detailed-documentation-esp#5-failure-modes-bounds-on-operating-conditions-and-external-risk-factors) \), y no permitirá que `poke()` sea llamado de nuevo hasta que `block.timestamp` sea al menos `zzz+hop`. Los valores se leen desde un contrato DSValue designado \(su dirección está almacenada en `src`\). El propósito de este mecanismo de actualización retrasado
es el de asegurar que hay tiempo para detectar y reaccionar ante un ataque de Oráculo \(es decir configurar el precio de un colateral a cero\). Respuestas a estos incluyen llamar `stop()` o `void()`, o activar el Apagado de Emergencia.       

Otros contratos, si son añadidos a la _whitelist_, pueden inspeccionar el valor de `cur` mediante los métodos `peek()` y `read()` \(`peek()` retorna un booleano adicional que indica si el valor ha sido realmente establecido; `read()` se revierte si el valor no ha sido establecido\). El próximo valor puede ser inspeccionado mediante `peep()`.     
El contrato usa un esquema de autorización de doble nivel: direcciones mapeadas a 1 en `wards` pueden comenzar y parar, establecer el `src`, llamar a `void()`, y agregar nuevos lectores; direcciones mapeadas a 1 en `buds` pueden llamar a `peek()`, `peep()` y `read()`. 

## 4. Gotchas \(Posibles Fuentes de Error del Usuario \)

### Confundir `peek()` con `peep()` \(o viceversa\)

Los nombres de estos métodos difieren por un solo carácter y en el uso lingüístico actual, ambos "peek" y "peep" tienen esencialmente el mismo significado. Esto hace que sea fácil que los desarrolladores los confundan y llamen al equivocado. Naturalmente, las consecuencias de tales errores dependen del contexto, pero por ejemplo: podría invalidar completamente el propósito del OSM si se llama a `peep()` en vez del `peek()` que se debió utilizar. Una mnemotécnica para ayudar a distinguirlos es: “la 'k' viene primero que la 'p' en el alfabeto inglés, el valor que devuelve `peek()` viene primero que el valor que devuelve `peep()` en orden cronológico”. O: "`peek()` devuelve el valor a**K**tual".

## 5. Modos de Fallo \(Limites en las Condiciones de Funcionamiento y Factores de Riesgo Externos\)

#### `poke()` no es llamado con prontitud , permitiendo que precios maliciosos sean adoptados rápidamente 

Por diversas razones, siempre se llama a `poke()` tan pronto cuando incrementa `block.timestamp / hop`, independientemente de cuándo ocurrió la última llamada a `poke()` \(porque `zzz` es redondeado hacia abajo al múltiplo más cercano de `hop`\). De hecho, esto significa que el contrato no garantiza que un intervalo de tiempo de al menos `hop` segundos haya pasado desde la última llamada `poke()` antes de la siguiente; mas bien esto se garantiza \(aproximadamente\) si la última llamada `poke()` ocurre poco después del incremento previo de `block.timestamp / hop`. Así pues, un valor de precio malicioso puede ser reconocido por el sistema en un tiempo mucho menor que `hop`.

**Esta fue una decisión de diseño deliberada. Los argumentos que lo favorecieron, a grandes rasgos, son:**

*Proporcionar un tiempo previsible en el que los poseedores de MKR deben comprobar si hay evidencia de ataques de oráculo \(en la práctica, `hop` es de 1 hora, así que las revisiones deben llevarse en la parte superior de la hora \)*
* Permitir que todos los OSMs sean _poked_ confiablemente al mismo tiempo y en una sola transacción. 
El hecho de que `poke` sea publica (y por lo tanto cualquier persona la pueda llamar) ayuda a mitigar preocupaciones, aunque no las elimina. Por ejemplo, la congestión de red podría evitar que cualquiera llame exitosamente a `poke()` durante cierto periodo de tiempo. Si un poseedor de MKR observa que `poke` no ha sido llamado prontamente, **las acciones que pueden tomar incluyen:**   

1. Llamar a `poke()`  ellos mismos y decidir si el próximo valor es malicioso o no.
2. Llamar a `stop()` o `void()` \(la primera solo si `nxt` es malicioso; la segunda si el valor malicioso ya está en `cur`\).
3. Activar el Apagado de Emergencia \(si la integridad del sistema en general ya ha sido comprometida o si se cree que el oráculo\(s\) rebelde no puede ser reparado en un periodo de tiempo razonable\). 

En el futuro, la lógica del contrato se puede ajustar para mitigar aún más esta situación \(ejemplo: **solo** permitiendo llamadas `poke()` en una ventana de tiempo corta en cada periodo `hop`).    

### Ataques de Autorización y Malas Configuraciones

Individuos autorizados o contratos pueden tomar varias acciones perjudiciales, ya sea maliciosamente o accidentalmente: 

* Revocando el acceso de los contratos principales a los métodos de lectura de valores, causando caos ya que los precios fallan en actualizarse     
* Revocando completamente todos los accesos al contrato 
* Cambiando `src` a un contrato malicioso o a algo que carezca de interfaz `peek()`, causando que las transacciones que _`poke()`_ (“pokeen”) el OSM afectado se reviertan 
* Llamando funciones disruptivas como `stop` y `void` inapropiadamente

La única solución a estos problemas es la diligencia y el cuidado respecto al `wards` del OSM.