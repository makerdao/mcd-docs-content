# Migración SCD <> MCD

* **Nombre del Contrato:** scd-mcd-migration.sol
* **Tipo/Categoría:**  Migración
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki#system-architecture)
* ****[**Fuente del Contrato**](https://github.com/makerdao/scd-mcd-migration/blob/master/src/ScdMcdMigration.sol)
* ****[**Etherscan**](https://etherscan.io/address/0xc73e0383f3aff3215e6f04b0331d58cecf0ab849)

## 1. Introducción (Sumario)

El propósito del contrato de Migración es el de permitir mover SAI y CDPs de un sistema SCD a uno MCD, conviertiéndose así en DAI y _vaults_. También permite a los usuarios mover SAI/DAI en ambas direcciones si desean salir del MCD y volver al SCD.

![](https://github.com/makerdao/mcd-docs-content/raw/master/.gitbook/assets/scd-mcd.png)



![](https://github.com/makerdao/mcd-docs-content/raw/master/.gitbook/assets/scd-mcd2.png)

## 2. Detalles del Contrato

#### Funcionalidades Claves

`swapSaiToDai` - Toma SAI (ERC-20 DAI del Sistema de Colateral Único), devuelve DAI (ERC-20 DAI del Sistema MultiColateral).

`swapDaiToSai` - Toma DAI (ERC-20 DAI del Sistema MultiColateral) devuelve SAI (ERC-20 DAI del Sistema de Colateral Único).

`migrate` - Mueve una _vault_ de SCD a MCD cerrando la SCD y abriendo una MCD correspondiente.

#### Almacenamiento

`tub`: _Address_ del contrato _Tub_ del SCD

`vat`: _Address_ del contrato _Vat_ del MCD

`cdpManager`: Administrador de CDP del MCD

`saiJoin`: Adaptador de colaterales del SAI para MCD

`wethJoin`: Adaptador de colaterales del WETH para MCD

`daiJoin`: Adapatador de unión de DAI para MCD

## 3. Conceptos y Mecanismos Claves

**En general, este contrato tiene dos propósitos principales:**

1. Intercambio bidireccional para SAI y DAI
2. Transferencia unidireccional para _vaults_

#### 1. `constructor` (configuración)

#### 2. `swapSaiToDai`

Los usuarios del sistema actual de DAI desearán moverse al nuevo sistema MCD. Esta función permite a los poseedores de DAI convertir su DAI sin problemas. Los mismos tendrán que aprobar (`approve`) el contrato de migración en el contrato ERC-20 de SAI para poder realizar el `transferFrom` (transferir de). El contrato de migración posee una _vault_ en MCD que toma SAI como colateral y le permite extraer MCD-DAI, lo cual hace y regresa a `msg.sender`.

#### 3. `swapDaiToSai`

En caso de que un usuario deseara volver al SAI, esta función les permite entregar su DAI a cambio de SAI. Esto requiere que el usuario apruebe (`approve`) el contrato de migración en el contrato ERC-20 de DAI para que así pueda realizar `transferFrom` (transferir de) y, luego, unir (`join`) de nuevo el DAI al MCD. Esto salda la "deuda" en su _vault_ SAI-MCD y permite recuperar el "colateral" de SAI y devolverlo a `msg.sender`.

#### 4. `migrate`

Esta función está pensada para ser utilizada en combinación con `MigrationProxyActions`, ya que requiere que el contrato de migración sea propietario de la SCD-Vault (`cup`) y que el contrato de migración tenga suficiente MKR para pagar las tarifas de estabilidad. La función `MigrationProxyActions`, `migrate`, `transferFrom` (transfiere de) del `msg.sender` al contrato de migración para que el contrato de migración tenga suficiente MKR para pagar la tarifa de estabilidad y cerrar la `cup`.

El contrato de migración primero extrae su propio SAI de su contrato MCD y lo usa para pagar la deuda de la `cup` (junto con el MKR que tiene de la acción de _proxy_ para pagar la tarifa). Luego retira el PETH como WETH.

A continuación, el contrato de migración abre una _vault_ utilizando al administrador CDP de MCD y une (`join`) su WETH en su nueva _vault_ y retira suficiente DAI de la nueva _vault_ (y reembolsa su _vault_) para compensar el SAI que extrajo anteriormente en este paso.

Por último, el contrato de migración da la MCD-Vault al `msg.sender`.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

* Cualquier información especial/única acerca del contrato específico.
* Cualquier cosa en la que se pueda confiar, especialmente si no es obvio.
* Fuentes de error del usuario si no se definen explícitamente.

#### `swapSaiToDai`

El monto de `wad` tiene que estar por debajo del techo de deuda para, tanto, el sistema MCD en general como para el tipo de colateral SAI; de lo contrario, `frob` fallará. Esto significa que estos parámetros de la Gobernanza pueden impactar la velocidad de la transición de SAI a DAI.

#### `swapDaiToSai`

El monto de `wad` tiene que estar por debajo del monto de colateral SAI en la _vault_ del contrato de migración. Si un usuario con DAI desea moverse al SAI pero, si ningún usuario del SAI se ha movido todavía al DAI, esto fallará.

#### `migrate`

Debido a que el contrato de migración primero tendrá que extraer SAI de su colateral MCD, el sistema deberá "sembrarse" con SAI en la _vault_ del contrato de migración en una cantidad que exceda la deuda de SAI para la `cup` que se está migrando.

Si un usuario posee tanto `cup` como SAI, debería decidir si tiene sentido:

1. Pagar de regreso el `cup` en el SCD; luego migrar (`migrate`) su `cup` a MCD (esencialmente, transferir la colateral a una nueva _vault_ MCD).
2. Migrar (`migrate`) su `cup` con la deuda en su lugar, luego utilizar `swapSaiToDai` para obtener DAI que luego pueden usar como ERC-20 o pagar su deuda de MCD.

Una consideración adicional, para cerrar o migrar un `cup`, el usuario deberá comprar MKR para pagar la tarifa de estabilidad y poder `exit` del sistema SCD. Sin embargo, una vez en MCD, las nuevas tarifas se devengarán (y deberán pagarse) en DAI. Si el SAI convertido, de un usuario, no cubre su deuda de MCD + tarifa de estabilidad, es posible que tenga que comprar DAI en el mercado abierto.

Antes del cierre de SCD: los usuarios que sacaron una _vault_ en SCD y luego usaron el DAI para comprar algo, tendrán que comprar SAI en el mercado abierto para pagar su deuda de SCD o tendrán que migrar su colateral a MCD.

## 5. Modos de Fallo (Límites de las Condiciones Operativas y Factores de Riesgo Externos)

* **Potencial para error:** Los parámetros de la Gobernaza alrededor del colateral del SAI
  * El ratio de colateralización debe estar establecido en un número muy bajo.
  * Tanto el `ilks["sai"].duty` como `Jug.base` deben estar establecidos en `0` durante el período de migración.
* **Errores de autorización en la unión de SAI**
* Exceso de SAI en el MCD (es decir, más `cup`s se pierden/no son migrados que los SAI que se pierden/no son migrados): resulta en una subasta y, probablemente, en una subasta de MKR para cubrir la deuda incobrable.

**Migración**

* El techo de deuda del SAI en 0.
* El techo de deuda de MCD.ilks\[sai] a SCD.totalDai.
* Que el valor del DSR sea competitivo con Compound para promover la migración.
