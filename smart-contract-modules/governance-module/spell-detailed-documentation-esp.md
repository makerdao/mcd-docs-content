# _Spell_ - Documentación Detallada

* **Nombre del Contrato:** spell.sol
* **Tipo/Categoría:** Governance Module
* \*\*\*\*[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* \*\*\*\*[**Fuente del Contrato**](https://github.com/dapphub/ds-spell/blob/master/src/spell.sol)

## 1. Introducción (Sumario)

Un `DS-Spell` es un objeto sin propietario que realiza, por única vez, una acción o una serie de acciones atómicas (múltiples transacciones). Esto se puede considerar como un `DSProxy` único, sin propietario (sin mezcla de `DSAuth`, no es un `DSThing`).

Esta primitiva es útil para expresar objetos que hacen acciones que no deberían depender del "remitente", como la actualización a un sistema de contratos que necesian permiso de raíz. Por convención, usualmente es lo que se utiliza para cambiar los parámetros del sistema MCD o SCD (donde se le da autorización a través de la votación en [ds-chief](https://github.com/dapphub/ds-chief/blob/master/src/chief.sol)).

![](https://i.imgur.com/cJ2NslE.png)

## 2. Detalles del Contrato
 El contrato `spell.sol` contiene dos contratos principales: DSSPELL y DSSpellBook. DSSPELL es el contrato central que, con las instrucciones de llamada establecidas en el constructor, puede realizar la acción de una sola vez. DSSpellBook es un contrato de fábrica diseñado para facilitar la creación de DSSPELL.

### Glosario (_Spell_)

* `whom` - es la _address_ que el _spell_ tiene como objetivo, usualmente SAI_MOM en SCD.
* `mana` - es la cantidad de ETH que se está enviando, el cual en _spells_ usualmente es 0.
* `data` - memoria de _bytes_ del _calldata_ (llamada de datos).
* `done` - indica que el _spell_ ha sido llamado con éxito.

## 3. Conceptos y Mecanismos Claves

* `hat` - Un _spell_ se vuelve efectivo como _hat_ cuando alguien llama a la función _lift_. Esto solo es posible cuando el _spell_ en cuestión tiene más MKR votados a favor que el _hat_ actual.
* `cast` - Una vez que el _spell_ se convierte en _hat_, se puede lanzar (`cast`) y sus nuevas variables se volverán efectivas como parte del sistema activo de Maker. Vale la pena señalar que un hechizo (_spell_) solo puede ser lanzado (`cast`) una vez.
* `lift` - El proceso por el cual un nuevo _spell_ reemplaza la propuesta anterior.

**Nota:** `hat` y `lift` tienen que ver más con `ds-chief` que con `ds-spell` pero es importante mencionarlos aquí para dar contexto.

#### **Acciones Inmutables**

`whom`, `mana` y `data` se encuentran establecidos en el constructor, por lo tanto, la acción que debe realizar un _spell_ no se puede cambiar después de que se haya implementado el contrato.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

El _spell_ solo se marcará como "realizado" si la LLAMADA que realiza es exitosa, significando que no se revirtió ni terminó en una condición excepcional. Por el contrario, los contratos que utilizan valores devueltos en lugar de excepciones para indicar errores podrían llamarse con éxito sin tener el efecto que uno podría desear. La "aprobación" de _spells_ para tomar medidas en un sistema después de que se implementa el _spell_, generalmente, requiere que el sistema use el manejo de errores basado en excepciones para evitar problemas.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

* `spell` - Un _spell_ debe permanecer sin ser utilizado/lanzado (`cast`) si no consigue la cantidad de MKR necesarios para ser aprobado. Si esto ocurre, el _spell_ puede permanecer disponible como un futuro objetivo si suficiente MKR es votado a su favor.
* `lift` - A pesar de que los _spells_ no pueden ser utilizados/lanzados (`cast`) por segunda vez, estos pueden ser levantados (`lift`) más de una vez para convertirse en el _hat_ si quedan suficientes votos en MKR en esa propuesta. Los parámetros de la propuesta no entrarán en vigencia, sin embargo, cualquier _spell_ adicional deberá tener más de esa cantidad de MKR votado a favor para convertirse en el nuevo _hat_. Chequea el [posteo del foro](https://forum.makerdao.com/t/an-explanation-of-continuous-voting-and-the-peculiarities-of-the-7-26-executive-stability-fee-vote/193) para ver una descripción de esto, habiendo ocurrido ya una vez.
* `cast` - Si, cuando `cast` es llamado, la acción de única vez del _spell_ falla, `done` no se activa y _spell_ queda disponible para ser utilizado/lanzado (`cast`).