# _Jug_ - Documentación Detallada

* **Nombre del Contrato:** Jug
* **Tipo/Categoría:** DSS —&gt; Rates Module
* [**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* \*\*\*\*[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/jug.sol)
* \*\*\*\*[**Etherscan**](https://etherscan.io/address/0x19c0976f590d67707e62397c87829d896dc0f1f1)

## 1. Introducción

### Sumario

La función principal del _smart contract_ del _Jug_ es el de acumular tarifas de estabilidad para un tipo de colateral en particular cada vez que se llame al método `drip()`. Esto actualiza, de manera efectiva, la deuda acumulada para todas las _vaults_ de ese tipo de colateral así como la deuda acumulada total registrada por el _Vat_ (global) y el monto de excedente de DAI (representado como el monto de DAI perteneciente al [_Vow_](https://www.notion.so/makerdao/Vow-Detailed-Documentation-7f3074dd92514db59efb6f128919b2c5)).

![](https://github.com/makerdao/mcd-docs-content/blob/master/.gitbook/assets/jug.png?raw=true)

## 2. Detalles del Contrato

### Estructuras

`Ilk` : contiene dos valores `uint256` — `duty`, la prima de riesgo específica del colateral, y `rho`, la marca de tiempo de la última actualización de la tarifa.

`VatLike` : contrato simulado para hacer que las interfaces del _Vat_ se puedan llamar desde el código sin una dependencia explícita del contrato del _Vat_ en sí.

### Diseño de Almacenamiento

`wards` : `mapping(address => uint)` que indica que _address_ puede llamar a funciones administrativas.

`ilks` : `mapping (bytes32 => Ilk)` que almacena una estructura de `Ilk` para cada tipo de colateral.

`vat` : un `VatLike` que señala el contrato [*Vat*](https://www.notion.so/makerdao/Vat-Core-Accounting-5314995c98e544c4aaa0ebedb01988ad) del sistema.

`vow` : la `address` del contrato _Vow_.

`base` : un `uint256` que especifica una aplicación de tarifas para todos los tipos de colaterales.

### Métodos Públicos

#### Métodos Administrativos

Estos métodos requieren de `wards[msg.sender] == 1` (es decir, que solo los usuarios autorizados pueden llamarlo).

`rely`/`deny` : agrega o remueve a los usuarios autorizados \(a través de modificaciones en el mapeo de `wards`\)

`init(bytes32)` : comienza una recolección de tarifas de estabilidad para un tipo de colateral en particular.

`file(bytes32, bytes32, uint)` : establece `duty` para un tipo de colateral en particular.

`file(bytes32, data)` : estable el valor de `base`.

`file(bytes32, address)` : estable el valor de `vow`.

#### Métodos de Recolección de Tarifas

`drip(bytes32)` : recolectar tarifas de estabilidad para un tipo de colateral en particular.

## 3. Conceptos y Mecanismos Claves

### `drip`

Cuando es llamado, `drip(bytes32 ilk)` hace una recolección de tarifas de estabilidad para un tipo de colateral en particular (téngase en cuenta que esta es una función pública y puede ser llamada por cualquiera). Esencialmente, el `drip` realiza tres cosas:

1. calcula el cambio en el parámetro de tasa para el tipo de colateral especificado por el `ilk` en función del tiempo transcurrido desde la última actualización y la tasa instantánea actual \(`base + duty`\);
3. llama a `Vat.fold` para actualizar el `rate` de los colaterales, total de la deuda registrada y el excedente del _Vow_;
4. actualiza a `ilks[ilk].rho` para que sea igual a la marca de tiempo.

**El cambio en la tasa es calculado como:**

$$
\Delta rate = (base+duty)^{now-rho} \cdot rate- rate
$$

donde _'now'_ (ahora) representa la hora actual, _'rate'_ es `Vat.ilks[ilk].rate`, _'base'_ es `Jug.base`, _'rho'_ e `Jug.ilks[ilk].rho` y _'duty'_ es `Jug.ilks[ilk].duty`. La función se revierte si algún subcálculo da como resultado un desbordamiento o escasez. Consulta la documentación del _Vat_ para obtener más detalles acerca del `fold`.

### `rpow`

`rpow(uint x, uint n, uint b)`, utilizado para la exponenciación en `drip`, es una función aritmética de número entero que eleva a `x` a la potencia `n`. Se implementa en el ensamblaje de Solidity como un algoritmo de elevación al cuadrado repetido. `x` y el valor devuelto, deben interpretarse como números enteros con factor de escala `b`. Por ejemplo, si `b == 100`, esto especifica que hay dos dígitos decimales de precisión y el valor decimal normal 2.1 se representaría como 210; `rpow(210, 2, 100)` devuelve 441 \(la representación de número entero de dos dígitos decimales de 2,1^2 = 4,41\). En la implementación actual, 10^27 se pasa por `b`, lo que hace que `x` y `rpow` resulten ambos del tipo `ray` en la terminología estándar de número entero MCD. Las invariantes formales de `rpow` incluyen un "sin desbordamiento", así como restricciones en el uso del _gas_.

### Los parámetros solo pueden establecerse por la Gobernanza

_Jug_ almacena algunos parámetros sensibles; en particular, la tasa base y las primas de riesgo específicas de colaterales que determinan la tasa general de la tarifa de estabilidad para cada tipo de colateral. Sus mecanismos de autorización incorporados deben permitir que solo los contratos/actores autorizados de la Gobernanza de MakerDAO establezcan estos valores. Consulta "Modos de fallo" para obtener una descripción de lo que puede salir mal si los parámetros se establecen con valores inseguros.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

### Inicialización Ilk

Se debe llamar a `init(bytes32 ilk)` cuando un nuevo colateral es agregado \(estableciendo `duty` a través de `file()` no es suficiente\) — de lo contrario, `rho` no se inicializará y las tarifas se acumularán en función de la fecha de inicio del 1 de enero de 1970\(comienzo de la época de Unix\).

### Desequilibrio de `base + Ilk.duty` en `drip()`

La llamada a `drip(bytes32 ilk)` agregará la tasa de `base` a la tasa de `Ilk.duty`. La tasa es una tasa compuesta calculada, por lo que `rate(base + duty) != rate(base) + rate(duty)`. Esto significa que si se establece `base`, el `duty` deberá establecerse teniendo en cuenta el factor compuesto existente en la `base`, de lo contrario, el resultado estará fuera de la tolerancia de la tasa. Las actualizaciones del valor `base` requerirán que todos los `ilks` también se actualicen.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

### Tragedia de los Comunes

Si `drip()` se llama con poca frequencia para unos tipos de colaterales (por ejemplo, debido a un bajo uso general del sistema o a tipos de colaterales extremadamente estables que tienen un riesgo de liquidación esencialmente nulo), entonces el sistema fallará en cobrar tarifas de las _vaults_ abiertas y cerradas entre las llamadas a `drip()`. A medida que el sistema logra escalar, esto se vuelve menos preocupante, ya que tanto los *keepers* como los poseedores de MKR tienen un incentivo para llamar regularmente a `drip` (los primeros para activar subastas de liquidación, los segundos para garantizar que el excedente se acumule para disminuir el suministro de MKR); sin embargo, un activo hipotético con una volatilidad muy baja pero una prima de riesgo alta aún podría ver llamadas de `drip` poco frecuentes a escala \(actualmente no hay un ejemplo real de esto; la posibilidad más realista es que la `base` sea grande y eleve las tasas para todos los tipos de colaterales\).

### Configuración Maliciosa o Descuidada de Parámetros

Varios parámetros del _Jug_ pueden ser configurados para dañar al sistema. Mientras que esto puede ocurrir por accidente, la preocupación más grande son los ataque maliciosos, especialmente por una entidad que, de alguna forma, está autorizada a llamar a funciones directamente en los métodos administrativos del _Jug_, pasando por encima de la Gobernanza. Configurar a `duty` (para al menos un ilk) o `base` en valores muy bajos puede conducir a un exceso de suministro de DAI; establecer cualquiera de los dos demasiado alto, puede desencadenar liquidaciones excesivas y, por lo tanto, una pérdida injusta de colaterales. Establecer un valor para `vow` que no sea la verdadera _address_ del Vow puede provocar la pérdida o el robo del excedente.