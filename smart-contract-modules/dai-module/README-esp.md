<!-- ---
descripción: El contrato del token DAI y todos los adaptadores DaiJoin.
--- -->

# Módulo del DAI

* **Nombre del Módulo:** Módulo del DAI
* **Tipo/Categoría: Proxy —>** Dai.sol y DaiJoin.sol
* ****[**Diagrama de Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* **Fuente del Contrato:**
  * ****[**dai**](https://github.com/makerdao/dss/blob/master/src/dai.sol)****
  * ****[**daiJoin**](https://github.com/makerdao/dss/blob/2318555e87b1798e322feaab36265a6e20d637be/src/join.sol#L100)****

## 1. Introducción (Resumen)

El origen del DAI fue diseñado para representar cualquier _token_ que el sistema principal considere equivalente en valor a su unidad de deuda interna. Por lo tanto, el **Módulo del DAI** contiene el contrato del _token_ del DAI y todos los adaptadores de adaptadores DaiJoin.

## 2. Detalles del Módulo

### Glosario (DAI)

**Funcionalidades Clave(como se encuentran definidas en el _smart contract_)**

`Mint` - Acuñación a una _address_

`Burn` - Quema en una _address_

`Push` - Tranferencia

`Pull` - Tranferir Desde

`Move` - Tranferir Desde

`Approve` - Permite `pull`s y `move`s

`Permit` - Aprobación por firma

**Otros**

`name` - _Stablecoin_ DAI

`symbol` - DAI

`version` - 1

`decimals` - 18

`totalSupply` - Suministro Total de DAI

`balanceOf(usr: address)` - Balance del Usuario

`allowance(src: address, dst: address)` - Aprobaciones

`nonces(usr: address)` - Permiso Único

### Glosario (Join)

* `vat` - almacenamiento de la _address_ del IVA (_Vat_)
* `ilk` - id del Ilk por el cual un `GemJoin` es creado
* `gem` - la _address_ del `ilk` para transferir
* `dai` - la _address_ del _token_ del `dai`
* `one` - una unidad de 10^27 utilizada para hacer cálculos en `DaiJoin`

### Documentación de los Componentes Principales del Módulo

1. ****[**Dai - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/dai-module/dai-detailed-documentation-esp)****
2. [**Documentación de DaiJoin** ](https://docs.makerdao.com/smart-contract-modules/collateral-module/join-detailed-documentation#3-key-mechanisms-and-concepts-esp)(Se referencia en "_Join_ - Documentación Detallada")

## 3. Mecanismos y Conceptos Claves

#### ¿Por qué son estos componentes importantes para el sistema Multi-Colateral del DAI (MCD)?

El contrato `Dai` es el usuario que enfrenta el contrato ERC20 manteniendo la contabilidad de los balances externos de Dai. La mayoría de las funciones son estándar para un token con suministro de carácter cambiante, pero también presenta la capacidad de emitir aprobaciones para transferencias basadas en mensajes firmados.

`Join` consiste de tres _smart contracts_, lo cual, uno de ellos, es el contrato de DaiJoin. Cada contrato de unión es creado específicamente para permitir la unión del tipo de _token_ utilizado con el `vat`. Debido a esto, cada contrato de unión tiene una lógica ligeramente diferente para dar cuenta de los diferentes tipos de _tokens_ dentro del sistema. El contrato DaiJoin permite a los usuarios retirar su Dai del sistema en un token ERC20 estándar.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

* El `DAI` también es susceptible a la ya conocida [condición de carrera de ERC20](https://github.com/0xProject/0x-monorepo/issues/850) pero, normalmente, con aprobaciones ilimitadas, esto no debería ser un problema. Recomendamos a todos los usuarios, que utilicen el `approval` para un monto específico, que estén atentos a este problema en particular y que sean cautelosos a la hora de autorizar a otros contratos que realicen tranferencias en su nombre.
* Hay fuentes limitadas de error de usuarios en el sistema de contratos de "Join" debido a la funcionalidad limitada del sistema. A menos que se produzca un error en el contrato, si un usuario llama a la función `join` por accidente, siempre podría recuperar sus tokens a través de la llamada al `exit` correspondiente en el contrato de unión dado. El problema principal a tener en cuenta aquí sería un ataque de _phishing_ bien ejecutado. A medida que el sistema evoluciona y potencialmente se crean más contratos de unión, o se crean más interfaces de usuario, existe la posibilidad de que un usuario pierda sus fondos mediante un contrato de unión malicioso que en realidad no envía tokens al `vat`, sino a otros contratos o _wallets_.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

**Potencialmente, podría haber una actualización del `vat` que requiriese la creación de nuevos contratos de `join`.**

Si un contrato `gem` fuera a pasar por una actualización de _token_ o congelara los _tokens_ mientras el colateral de un usuario estaba en el sistema, podría haber un escenario en el que los usuarios no pudieran redimir su colateral luego de que haya finalizado la congelación o la actualización. Sin embargo, esto parece ser un riesgo menor ya que parecería probable que el token que estuviera pasando por esta actualización quisiera trabajar junto con la comunidad de Maker para asegurarse de que esto no sea un problema.