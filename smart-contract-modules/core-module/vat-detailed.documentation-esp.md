# Vat - Documentación Detallada

* **Nombre del Contrato: Vat.sol**
* **Tipo/Ctaegoría:** DSS —> Core System Accounting
* ****[**Diagrama de Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/vat.sol)
* ****[**Etherscan**](https://etherscan.io/address/0x35d1b3f3d7966a1dfe207aa4514c12a259a0492b)

## 1. Introducción (Sumario)

El `Vat` es el motor central de _vaults_ de `dss`. Almacena _vaults_ y realiza un seguimiento de todos los balances de Dai y colaterales asociados. También define las reglas mediante las cuales se pueden manipular las _vaults_ y los balances. Las reglas definidas en `Vat` son inmutables, por lo que, en cierto sentido, las reglas en `Vat` pueden verse como la constitución de `dss`.

![](https://i.imgur.com/ZgHuk9g.jpg)

## 2. Detalles del Contrato

### **Glosario (Vat -** Motor de _Vaults_ **)**

* `gem`: tokens de colaterales.
* `dai`: tokens de _stablecoins_.
* `sin`: _stablecoin_ no respaldado (deuda del sistema, no perteneciente a ninguna `urn`).
* `ilks`: un mapeo de tipos de `Ilk`.
* `Ilk`: un tipo de colateral.
  * `Art`: deuda total normalizada de _stablecoins_.
  * `rate`: multiplicador de deuda de _stablecoins_ (tarifas de establidad acumuladas).
  * `spot`: precio de colaterales con un margen de seguridad, e.j. la _stablecoin_ máxima permitida por unidad de colateral.
  * `line`: el techo de deuda para un tipo específico de colateral.
  * `dust`: el piso de deuda para un tipo específico de colateral.
* `urns`: un mapeo de tipos de `Urn`.
* `Urn`: una _vault_ específica.
  * `ink`: balance de colateral.
  * `art`: deuda pendiente normalizada de _stablecoins_.
* `init`: crea un nuevo tipo de colateral.
* `slip`: modifica el balance del colateral de un usuario.
* `flux`: transfiere colaterales entre usuarios.
* `move`: transfiere _stablecoins_ entre usuarios.
* `grab`: liquida una _vault_.
* `heal`: crea / destruye iguales cantidades de _stablecoin_ y deuda del sistema (`vice`).
* `fold`: modifica el multiplicador de deuda, creando / destruyendo la deuda correspondiente.
* `suck`: acuña _stablecoins_ no respaldadas (contabilizadas con `vice`).
* `Line`: el total de techo de deuda para todos los tipos de colaterales.
* `frob`: modica una _vault_.
  * `lock`: transfiere un colateral en una _vault_.
  * `free`: transfiere un colateral de una _vault_.
  * `draw`: incrementa la deuda de una _vault_, creando Dai.
  * `wipe`: disminuye la deuda de una _vault_, destruyendo Dai.
  * `dink`: cambio en un colateral.
  * `dart`: cambio en la deuda.
* `fork`: para dividir una _vault_ - aprobación binaria o _vaults_ divididas/fusionadas.
  * `dink`: monto de colateral a intercambiar.
  * `dart`: monto de deuda de _stablecoin_ a intercambiar.
* `wish`: chequea si una _address_ tiene permiso para modificar el balance de gem o dai de otra _address_.
  * `hope`: habilita `wish` para un par de _addresses_.
  * `nope`: deshabilita `wish` para un par de _addresses_.

**Nota:** `art` y `Art` representan la deuda normalizada; por e.j. un valor que, cuando se multiplica por la tasa correcta, da la deuda de _stablecoin_ actual y actualizada.

#### **Contabilidad**

* `debt` es la suma de todos los `dai` (la cantidad total de dai emitidos).
* `vice` es la suma de todos los `sin` (la cantidad total de deuda del sistema).
* `Ilk.Art` es la suma de todos los `art` en las `urn`s por ese `Ilk`.
* `debt` es `vice` más la suma de `Ilk.Art * Ilk.rate` a través de todos los `ilks`.

#### **Colateral**

* `gem` siempre puede ser transferida a cualquier _address_ por su dueño.

#### **Dai**

* `dai` solo se puede mover bajo el consentimiento de su dueño.
* `dai` siempre puede ser transferido a cualquier _address_ por su dueño.

## 3. Conceptos y Mecanismos

La _vault_ central, Dai, y el estado de colaterales es guardado en el `Vat`. El contrato del `Vat` no tiene dependencias externas y mantiene las "Invariantes Contables" centrales del Dai. Los principios básicos que se aplican al `Vat` son los siguientes:

1. **El Dai no puede existir sin un colateral:**

* Un `ilk` es un tipo particular de colateral.
* El colateral `gem` es asignado a los usuarios con `slip`.
* El colateral `gem` es transferido entre usuarios con `flux`.

**2. `Urn` es la estructura de la información de una _vault_:**

* Tiene `ink` - colateral grabado
* Tiene `art` - deuda normalizada, grabada

**3. Del mismo modo, un colateral es un `Ilk`:**

* Tiene `Art` - deuda normalizada, grabada
* Tiene `rate` - factor de escalado de deuda (discutido a fondo más abajo)
* Tiene `spot` - precio con margen de seguridad
* Tiene `line` - techo de deuda
* Tiene `dust` - piso de deuda

**Nota:** Arriba, cuando se usa el término "grabado/a", se refiere a "encerrado en una _vault_".

### Gestión de _Vaults_

* Las _vaults_ se administran a través de `frob(i, u, v, w, dink, dart)`, el cual modifica la _vault_ del usuario `u`, usando `gem` del usuario `v` y creando `dai` para el usuario `w`.
*  Las _vaults_ son confiscadas a través de `grab(i, u, v, w, dink, dart)`, el cual modifica la _vault_ del usuario `u`, usando `gem` del usuario `v` y creando `sin` para el usuario `w`. `grab` es el medio por el cual las _vaults_ son liquidadas, transfiriendo la deuda de las _vaults_ al balance `sin` de un usuario.
* `Sin` representa deuda "incautada" o "incobrable" y se puede cancelar con una cantidad igual de Dai usando `heal(uint rad` donde `msg.sender` es usado como la  _address_ para los balances de `dai` y `sin`.
  * **Nota:** Solo el _Vow_ tendrá `sin`, así que, solamente el _Vow_ puede llamar satisfactoriamente al `heal`. Esto se debe a que cada vez que se llaman `grab` y `suck`, la _address_ del _Vow_ se pasa como el destinatario de `sin`. Ten en cuenta que esto depende del diseño y la implementación actual del sistema.
  * **Nota:**  `heal` solo puede ser llamado con un número positivo (uint) y realizará `sub(dai[u])` junto con `sub`, subiendo el `sin`.
* La cantidad de `dai` puede ser transferida entre usuarios por `move`.

### **Actualizaciones de Tasas a Través de `fold(bytes32 ilk, address u, int rate)`**

La `rate` (tasa) de los `ilk` es el factor de conversión entre cualquier deuda normalizada (`art`) girada contra ella y el valor actual de esa deuda con las comisiones devengadas. El parámetro `rate` a `fold` (doblar) es en realidad el cambio en el valor `Ilk.rate`; e.j., una diferencia de factores de escala (nuevo - antiguo). Es un número entero y absoluto y, por lo tanto, los valores actuales de la cuenta pueden aumentar o disminuir. La cantidad `Ilk.Art*rate` se suma al saldo `dai` de la _address_ `u` (lo que representa un aumento o disminución en el excedente del sistema); los saldos de deuda de todas las _vaults_ colateralizadas con el `Ilk` especificado se actualizan implícitamente mediante la adición de `rate` a `Ilk.rate`.

Para más información acerca de Tasas y Estabilización del Sistema, mira la documentación del Módulo de Tasas y el Módulo del Estabilizador del Sistema aquí debajo:

* [Módulo de Tasas](https://docs.makerdao.com/smart-contract-modules/rates-module-esp)
* [Estabilizador del Sistema](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module-esp)

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

Los métodos en el `Vat` están escritos de tal manera para que sean lo más genéricos posible y, como tales, tienen interfaces que pueden ser bastante detalladas. Se debe tener cuidado de no mezclar el orden de los parámetros.

Cualquier módulo que sea autorizado (`auth`) contra el `Vat` tiene accesos completo de raíz y, por ende, puede robar todos los colaterales del sistema. Esto significa que la incorporación de un nuevo tipo de colateral (y adaptadores asociados) acarrean un riesgo considerable.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

#### Error de Código

Un _bug_ (error) en el `Vat` podría ser catastrófico y podría conducir a la pérdida (o bloqueo) de todo el DAI y los Colaterales en el sistema. Se volvería imposible modificar _vaults_ o transferir DAI. Las subastas podrían dejar de funcionar. El Apagado podría fallar.

#### _Feeds_ (Fuentes de alimentación)

`Vat` - Depende de un conjunto de oráculos de confianza que provea datos de precios. Si estos _feeds_ de precios fallasen, sería posible acuñar Dai sin respaldo alguno, o _vaults_ seguras podrían ser liquidadas de manera injusta.

#### Gobernanza

`Vat` - La Gobernanza puede autorizar nuevos módulos contra el `Vat`. Esto les permitiría robar colateral (`slip`) o acuñar Dai sin respaldo (`suck`/adición de tipos de colaterales sin valor). Si fallan las protecciones criptoeconómicas que hacen que hacerlo sea prohibitivamente costoso, el sistema puede quedar vulnerable y abierto para que malos actores drenen el colateral.

#### Adaptadores

`Vat` - Depende de contratos de Adaptadores externos para asegurar que el saldo de los colaterales en el `Vat` representan el saldo real de colaterales externos. Los contratos de Adapatadores están autorizados a modificar los saldos de los colaterales arbitrariamente. Un adaptador de colateral defectuoso podría resultar en la pérdida de todo el colateral en el sistema.