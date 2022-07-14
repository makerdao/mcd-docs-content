# Módulo de Tasas

* **Nombre del Módulo:** Rates Module
* **Tipo/Categoría:** Rates
* [**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* **Fuente del Contrato:**
  * [**_Jug_**](https://github.com/makerdao/dss/blob/master/src/jug.sol)
  * [**_Pot_**](https://github.com/makerdao/dss/blob/master/src/pot.sol)

## Introducción

Una característica fundamental del sistema MCD es acumular tarifas de estabilidad en los saldos de deuda de las _vaults_, así como intereses en los depósitos de Tasa de Ahorro de Dai (DSR - _Dai Savings Rate_).

El mecanismo, utilizado para realizar estas funciones de acumulación, está sujeto a una restricción importante: la acumulación debe ser una operación de tiempo constante con respecto al número de _vaults_ y al números de depósitos de DSR. De lo contrario, los eventos de acumulación serán bastante ineficientes con respecto al _gas_ (y podrían excederse de los límites de bloque de _gas_).

Tanto para las tarifas de estabilidad como para los DSR, la solución es similar: almacenar y actualizar un valor de "tasa acumulativa" global (por colateral para tarifas de estabilidad), el cual puede ser multiplicado por una deuda normalizada o un monto de depósito para obtener el monto total de la deuda o del depósito cuando sea necesario.

** Esto se puede describir más concretamente con notación matemática:**

* Discretizar el tiempo en intervalos de 1-segundo, empezando desde _t_\_0;
* Dejar que (por segundo) la tarifa de estabilidad en momento _t_ tenga un valor de _F\_i_ (esto, generalmente, toma la forma de 1+_x_, donde _x_ es pequeño)
* Dejar que el valor inicial de una tasa acumulativa se denote como _R_\_0
* Dejar que una _vault_ se cree en momento _t\_0_ con deuda _D_\_0 extraiga inmediatamente; la deuda normalizada _A_ (la cual el sistema almacena por _vault_) es calculada como _D_\_0/_R_\_0

**Entonces, la tasa acumulativa **_**R**_** en momento **_**T**_** es dada por:**

$$
R(t) \equiv R_0 \prod_{i=t_0 + 1}^{t} F_i = R_0 \cdot F_{t_0 + 1} \cdot F_{t_0 + 2} \cdots F_{t-1} \cdot F_t
$$

Y la deuda total de una _vault_ en momento _t_ sería:

$$
D(t) \equiv A \cdot R(t) = D_0 \prod_{t=1}^{T} F_i
$$

En el sistema actual, _R_ no es necesariamente actualizado con cada bloque y puede que los valores actuales de _R_ dentro del sistema no tengan el valor exacto que deberían tener. Sin embargo, la diferencia en la práctica debería ser menor dado el ecosistema suficientemente amplio y activo.

Las explicaciones detalladas de ambos mecanismos de acumulación se pueden encontrar debajo.

## Acumulación de Tarifas de Estabilidad

### Descripción General

La acumulación de Tarifas de Estabilidad en el MCD es, en gran medida, una interacción entre dos contratos: el [_Vat_](https://www.notion.so/makerdao/Vat-Core-Accounting-5314995c98e544c4aaa0ebedb01988ad) (el libro contable central del sistema) y el [_Jug_](https://docs.makerdao.com/smart-contract-modules/rates-module/jug-detailed-documentation) (un módulo especializado para la actualización de la tasa de acumulación), con el [_Vow_](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module/vow-detailed-documentation) involucrado solo como la _address_ en la cual se acreditan las tarifas acumuladas.

![](https://i.imgur.com/ZEUQfM2.png)

Para cada tipo de colateral, el _Vat_ almacena una estructura de `ilk` que contiene la tasa de acumulación (`rate`) y la deuda normalizada total asociada con el tipo de colateral (`Art`). _Jug_ almacena la tasa por segundo para cada tipo de colateral como una combinación de valor `base` que se aplica a todos los tipos de colaterales y un valor de deber (`duty`) por colateral. La tasa por segundo dada para un tipo de colateral es la suma de su `duty` particular y la `base` global.

El llamar a la función `Jug.drip(bytes32 ilk)` computa una actualización a las `rate`s de los ilk basado en `duty`, `base` y el tiempo que haya transcurrido desde que `drip` fue llamado por última vez para el ilk (`rho`). Luego, _Jug_ invoca a la función `Vat.fold(bytes32 ilk, address vow, int rate_change)`, la cual:

* agrega `rate_change` a `rate` para un ilk específico
* incrementa el excedente del [_Vow_](https://www.notion.so/makerdao/Vow-Detailed-Documentation-7f3074dd92514db59efb6f128919b2c5) por `Art*rate_change`
* incrementa la deuda total del sistema (es decir, el DAI emitido) por `Art*rate_change`.

Cada _vault_ individual (representada por la estructura de una `Urn` en el _Vat_) almacena un parámetro de "deuda normalizada" llamado `art`. Cada vez que es necesitado por el código, el total de deuda de las _vaults_ (incluidas las tarifas de estabilidad) se pueden calcular como `art*rate` (donde `rate` corresponde a la del tipo de colateral apropiado). Por lo tanto, una actualización de `Ilk.rate` a través de `Jug.drip(bytes32 ilk)` actualiza efectivamente la deuda de todas las _vaults_ colateralizadas con tokens `ilk`.

### Ejemplo con Visualizaciones

Supongamos que al momento 0, se abre una _vault_ y se extraen 20 DAIs de ella. Asumamos que `rate` es 1; esto implica que el `art` almacenado en la `Urn` de las _vaults_ es también de 20. Que la `base` y el `duty` se fijen de tal manera que, luego de 12 años, `art*rate` = 30 (esto corresponde a una estabilidad anual de aproximadamente 3,4366%). Equivalentemente, `rate` = 1,5 luego de 12 años. Asumiendo que la `base + duty` no cambia, el crecimiento de la deuda efectiva puede ilustrarse de la siguiente manera:

![](https://i.imgur.com/JhD8N5c.png)

Ahora supongamos que, a 12 años, se extraen 10 DAI más. El gráfico de Deuda vs Tiempo cambiará y se verá así:

![](https://i.imgur.com/Ek3lVUJ.png)

¿Qué `art` se almacenará en el _Vat_ para reflejar este cambio? (pista: ¡_no_ 30!) Recuerda que `art` se define a partir del requisito de que `art * rate` = deuda de _vault_.  Dado que se sabe que la deuda de la _vault_ es 40 y que la `rate` es 1,5 , podemos resolver para `art`: 40/1,5 \~ 26,67.

El `art` se puede pensar como la "deuda en momento 0" o como "la cantidad de Dai que, si se retira en el momento cero, daría como resultado la deuda total actual". El gráfico que se encuentra debajo muestra esto; la longitud de la barra verde que se extiende hacia arriba desde t = 0 es posterior a la extracción del valor de `art`.

![](https://i.imgur.com/My2c6ag.png)

Es bueno tener en mente algunas de las consecuencias del mecanismos:

* No hay información histórica almacenada de extracciones o limpieza de la deuda de una _vault_.
* No hay información histórica almacenada de cambios de tarifas de estabilidad, solo de `rate`s de acumulación efectivas.
* El valor de `rate` para cada colateral crece perpetuamente (a menos que la tarifa se vuelva negativa en algún punto)

### ¿Quién llama a `drip`?

El sistema depende de los participantes del mercado para llamar a `drip` en lugar de, por ejemplo, llamarla automáticamente a las manipulaciones de las _vaults_. Las siguientes entidades están motivadas para llamar a `drip`:

* Los _keepers_ que buscan liquidar una _vault_ (dada que la acumulación de tarifas de estabilidad pueden empujar a un ratio de la colateralización de una _vault_ a un terreno peligroso, permitiendo a los _keepers_ liquidarla y beneficiarse de la subasta de colaterales resultante).
* Los dueños de _vaults_ que deseen extraer DAI (si no llaman a `drip` antes de realizar la extracción de su _vault_, se les cobrará tarifas a la extracción de DAIs en base a la última vez que se llamó a `drip`; a menos que nadie llame a `drip` antes de que paguen su _vault_, ver debajo).
* Los poseedores de MKR (tienen un interés creado en ver que el sistema funcione bien y la recolección de excedentes en particular es fundamental para el flujo y reflujo del MKR en existencia).

A pesar de la variedad de actores incentivados, es probable que las llamadas a `drip` sean intermitentes debido a los costos del _gas_ y la tragedia de los comunes hasta que se pueda lograr una cierta escala. Por lo tanto, el valor del parámetro `rate`, para un tipo de colateral, puede mostrar el siguiente comportamiento temporal:

![](https://i.imgur.com/QTnVlJE.png)

La deuda extraída y limpiada entre las actualizaciones de `rate` (e.j. entre las llamadas a `drip`) no tendría tarifas de estabilidad evaluadas en ella. Además, dependiendo del momento de las actualizaciones de la tarifa de estabilidad, puede haber pequeñas discrepancias entre el valor real de `rate` y su valor ideal (el valor de si se llamará a `drip` en cada bloque). Para demostrar esto, considera lo siguiente:

* A t = 0, asumamos los siguientes valores:

$$
\text{rate} = 1 \text{ ; total fee} = f
$$

en un bloque con t = 28, `drip` es llamado:

$$
\text{rate} = f^{28}
$$

en un bloque con t = 56, la tarifa es actualizada a un valor nuevo y diferente:

$$
\text{totalfee}  \xrightarrow{} g
$$

en un bloque con t = 70, se llama nuevamente a `drip`; el valor actual de `rate` que se obtiene es:

$$
\text{rate} = f^{28} g^{42}
$$

sin embargo, el `rate` "ideal" (si se llamara a `drip` al principio de cada bloque) sería:

$$
\text{rate}_{ideal} = f^{56}g^{14}
$$

Dependiendo de si _f_ > _g_ o _g_ > _f_, el valor neto de tarifas acreditadas será o muy alto o muy bajo. Se asume que las llamadas a `drip` serán lo suficientemente frecuentes para que dichas inexactitudes sean menores, por lo menos luego de un período de crecimiento inicial. La Gobernanza puede mitigar este comportamiento llamando inmediamente a `drip` antes del cambio de tarifas. De hecho, el código enfatiza que `drip` debe ser llamada antes de la actualización de `duty` pero no impone una restricción similar para `base` (debido a la ineficiencia de iterar sobre todos los tipos de colaterales).

## Tasa de Acumulación de Ahorro de DAI

### Descripción General

La acumulación de DSR es muy similar a la acumulación de tarifas de estabilidad. Se implementa por medio de [_Pot_](https://docs.makerdao.com/smart-contract-modules/rates-module/pot-detailed-documentation), el cual interactúa con el _Vat_ (y, nuevamente, la _address_ del _Vow_ se utiliza para contabilizar el DAI creado). El _Pot_ registra los depósitos normalizados por usuario (`pie[usr]`) y mantiene un parámetro de tasa de interés acumulativo (`chi`). Los actores económicos llaman de manera intermitente a una función de `drip` análoga a la de _Jug_ para desencadenar la acumulación de ahorros.

![](https://i.imgur.com/qpOeSGB.png)

La tasa de ahorro por segundo (or "instantánea") es almacenada en el parámetro `dsr` (análogo a `base+duty`, en el caso de tarifas de estabilidad). El parámetro `chi` en función del tiempo es, por lo tanto (en el caso ideal de que `drip` se llame en cada bloque), dado por: 

$$
\text{chi}(t) \equiv \text{chi}0 \prod{i=t_0 + 1}^{t} \text{dsr}_i
$$

donde chi\_0 es simplemente chi(_t_\_0).

Supongamos que un usuario une _N_ DAI en el _Pot_ al momento _t_\_0. Luego, su saldo interno de ahorros en DAI se presentará como:

$$
\text{pie[usr]} = N / \text{chi}_0
$$

El DAI total que el usuario podrá retirar del _Pot_ al momento _t_ es:

$$
\text{pie[usr]} \cdot \text{chi}(t) = N \prod_{i=t_0 + 1}^{t} \text{dsr}_i
$$

Por lo tanto, vemos que las actualizaciones a `chi` incrementaron, efectivamente, todos los saldos de _Pot_ a la vez, sin tener que iterar sobre todos ellos.

Luego de actualizar `chi`, `Pot.drip` llama a `Vat.suck` con argumentos tales que el DAI adicional creado a partir de esta acumulación de ahorros se acredita al contrato del _Pot_ mientras que el `sin` (deuda no respaldada) del _Vow_ se incrementa por el mismo monto (la deuda global y la deuda no respaldada también aumentan). Para lograr esto de manera eficiente, el _Pot_ realiza un seguimiento de la suma total de todos los valores individuales de `pastel[usr]` en una variable llamada `Pie`.


### Propiedades Notables

Es bueno tener en mente los siguientes puntos al momento de razonar acerca de la acumulación de ahorros (todos tienen análogos en el mecanismo de acumulación de tarifas):

* si `drip` es llamado de manera poco frecuente, el valor instantáneo de `chi` puede diferir del ideal.
* el código requiere que `drip` sea llamado antes de los cambios en el `dsr`, que elimina las desviaciones de `chi` de su valor ideal debido a que tales cambios no coinciden con las llamadas de `drip`.
* `chi` es un valor que aumenta monótonamente a menos que la tasa de ahorro efectiva se vuelva negativa (`dsr` < `ONE`)
* No hay registros almacenados acerca de los depósitos o extracciones de DAI del _Pot_.
* No hay registros almacenados acerca de los cambios al `dsr`.

### ¿Quién llama a `drip`?

Los siguientes actores económicos se encuentran incentivados (o forzados) a llamar a `Pot.drip`:

* cualquier usuario que retire DAI del _Pot_ (de lo contrario, ¡perderán dinero!)
* cualquier usuario que ingrese DAI al _Pot_ — esto no es económicamente racional sino que está obligado por la lógica del _smart contract_ que requiere que se llame a `drip` en el mismo bloque cuando se agrega un nuevo DAI al _Pot_ (de lo contrario, es posible que suceda una explotación económica que drene el excedente del sistema).
* cualquier actor que tenga un motivo para incrementar la deuda del sistema como, por ejemplo, un _keeper_ que quiera activar una subasta _flop_ (subasta de deuda).


## Una Nota sobre la Fijación de Tarifas

Veamos cómo establecer un valor de tasa en la práctica. Supongamos que se desea establecer la DSR en 0,5% anual. Asumamos que la tasa real seguirá a la tasa ideal. Entonces, necesitamos un valor de tasa por segundo _r_ tal que (denotando el número de segundos en un año por _N_):

$$
r^N = 1.005
$$

Se puede usar un cálculo de precisión arbitraria para sacar la raíz _N_-ésima del lado derecho (con _N_ = 31536000 = 365_24_60\*60), para obtener:

$$
r = 1.000000000158153903837946258002097...
$$

El parámetro `dsr` en la implementación del _Pot_ es interpretado como `ray`; es decir, un número de entero de 27 dígitos decimales. Por lo tanto, multiplicamos por 10^27 y eliminamos cualquier cosa después del punto decimal:

$$
\text{dsr} = 1000000000158153903837946258
$$

Luego, `dsr` puede fijarse en un 0,5% anual llamando a:

`Pot.file("dsr", 1000000000158153903837946258)`
