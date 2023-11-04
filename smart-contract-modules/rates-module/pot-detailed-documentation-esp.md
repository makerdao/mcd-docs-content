# _Pot_ - Documentación Detallada

* **Nombre del Contrato:** pot.sol
* **Tipo/Categoría:** DSS —> Rates Module
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/pot.sol)
* ****[**Etherscan**](https://etherscan.io/address/0x197e90f9fad81970ba7976f33cbd77088e5d7cf7)

## 1. Introducción (Sumario)

_Pot_ es el núcleo de la `Dai Savings Rate` (Tasa de Ahorro del DAI). Permite a los usuarios depositar `dai` y activar la Tasa de Ahorro del DA-I, y ganar ahorros en sus `dai`. El _DSR_ es establecido por la Gobernanza de Maker y, por lo general, será menor que la tarifa de estabilidad básica para seguir siendo sostenible. El propósito de _Pot_ es ofrecer otro incentivo para tener DAI.

![](https://github.com/makerdao/mcd-docs-content/raw/master/.gitbook/assets/Screen%20Shot%202019-11-17%20at%202.19.45%20PM.png)

## 2. Detalles del Contrato

#### Matemática

* `mul(uint, uint)`, `rmul(uint, uint)`, `add(uint, uint)` y `sub(uint, uint)` - se revertirán en caso de desbordamiento o escasez.
* `rpow(uint x, uint n, uint b)`, utilizado para la exponenciación en `drip`, es una función aritmética de número entero que eleva a `x` a la potencia `n`. Se implementa en el ensamblaje de Solidity como un algoritmo de elevación al cuadrado repetido. `x` y el valor devuelto, deben interpretarse como números enteros con factor de escala `b`. Por ejemplo, si `b == 100`, esto especifica que hay dos dígitos decimales de precisión y el valor decimal normal 2.1 se representaría como 210; `rpow(210, 2, 100)` devuelve 441 \(la representación de número entero de dos dígitos decimales de 2,1^2 = 4,41\). En la implementación actual, 10^27 se pasa por `b`, lo que hace que `x` y `rpow` resulten ambos del tipo `ray` en la terminología estándar de número entero MCD. Las invariantes formales de `rpow` incluyen un "sin desbordamiento", así como restricciones en el uso del _gas_.

#### Autorizaciones

* `wards` tiene permitido llamar a funciones protegidas (Administración)

#### Almacenaje

* `pie` - almacena la _address_ del saldo de `Pot`.
* `Pie` - almacena el saldo total en `Pot`.
* `dsr` - la `dai savings rate` (Tasa de Ahorros del DAI). Comienza en `1` (`ONE = 10^27`), pero puede ser actualizado por la Gobernanza.
* `chi` - el acumulador de tasas. Este valor es contínuamente creciente y decide cuánto 'dai' se da cuando se llama a `drip()`.
* `vat` - una _address_ que se ajusta a una interfaz `VatLike`. Se establece durante la construcción y **no se puede cambiar**.
* `vow` - una _address_ que se ajusta a una interfaz `VowLike`. No se establece durante la construcción. Lo debe establecer la Gobernanza.
* `rho` - la última vez que `drip` fue llamado.

Los valores de `dsr` y `vow` pueden ser cambiados en el contrato por una _address_ autorizada (es decir, la Gobernanza de Maker). Los valores de `chi`, `pie`, `Pie` y `rho` son actualizados en el contrato internamente y no pueden ser cambiados manualmente.

## 3. Conceptos y Mecanismos Claves

#### drip()

* Calcula el `chi` más reciente y extrae `dai` del `vow` (incrementando el `sin` del `Vow`).
* Siempre, el usuario debe asegurarse de que `drip` haya sido llamado antes de llamar a la función `exit()`.
* Debe llamarse a `drip` antes de que un usuario use `join` y sea de su interés volver a llamarlo antes de llamar a `exit`, pero no hay una regla establecida para determinar qué tan frecuentemente se puede llamar a `drip`.

#### join(uint wad)

* El parámetro `uint wad` se basa en la cantidad de DAI (ya que `wad` = `dai` / `chi`) que tu quieras `join` (unir) al `pot`. El `wad * chi` debe estar presente en el `vat` ya apropiado por el `msg.sender`.
* El monto de `pie` del `msg.sender` se actualiza para incluir al `wad`.
* El monto total del `Pie` también se actualiza para incluir al `wad`.

#### exit(uint wad)

* `exit()` funciona, esencialmente, como el opuesto a `join()`.
* El parámetro `uint wad` está basado en la cantidad de DAI que tu quieras `exit` (sacar) del `pot`. El `wad * chi` debe estar presente en el `vat` ya apropiado por el `pot` y debe ser menor al saldo de `pie` del `msg.sender`.
* El monto de `pie` del `msg.sender` se actualiza restando el `wad`.
* El monto total del `Pie` también se actualiza restando el `wad`.

#### Administración

Varias firmas de funciones de archivo para administrar `Pot`:

* Estableciendo un nuevo dsr (`file(bytes32, uint256)`)
* Estableciendo un nuevo vow (`file(bytes32, address)`)

### Uso

El uso principal sería, para las `addresses`, el de almacenar su `dai` en el `pot` para acumular interés a largo plazo.

## 4. _Gotchas_ / Preocupaciones de Integración

* El `dsr` es (globalmente) establecido a través del sistema de la Gobernanza. Puede establecerse en cualquier número > 0%. Esto incluye la posibilidad de que se establezca en un número que haga que el DSR se acumule más rápido que las Tarifas de Estabilidad colectivas, acumulando así deuda del sistema y, eventualmente, provocando la acuñación de MKR.
* si `drip()` no ha sido llamado recientemente antes de que una _address_ llame a `exit()`, estos no recibirán el monto total que han ganado durante el período de su depósito.
* Si un usuario desea `join` (unir) o `exit` (sacar) 1 DAI en/del `Pot`, deberá enviar un `wad` igual a `1 / chi` ya que la cantidad movida de su saldo será `1 * chi` (para ver un ejemplo de esto, consulta el [Acciones del Proxy de DSS](https://github.com/makerdao/dss-proxy-actions/blob/master/src/DssProxyActions.sol#L547).

## 5. Modos de Falla e Impactos

#### Error de Código

Un error en el `Pot` puede dirigir al bloqueo de `dai` si la función `exit()` o si las funciones subyacentes `vat.suck()` o `vat.move()` tuvieran errores.

#### Gobernanza

La tasa inicial del `dsr` se puede establecer a través del _Chief_. La Gobernanza será capaz de cambiar el DSR basándose en las reglas que emplea el _DS-Chief_ (el cual incluirá una Pausa para las acciones).

Un riesgo serio es el de si la Gobernanza elige establecer el `dsr` a una tasa extremadamente alta; esto podría provocar que las tasas del sistema se disparen. 
Además, si la Gobernanza permite que el `dsr` supere (significativamente) las tarifas del sistema, provocaría la acumulación de deuda y aumentaría las subastas _Flop_.