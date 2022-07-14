# Módulo Central

* **Nombre del Módulo:** Módulo Central de _Vault_
* **Tipo/Categoría:** Módulo Central de más robustos—> ( Vat.sol, Spot.sol )
* [**Diagrama de Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* **Fuente del Contrato:**
  * ****[**Vat**](https://github.com/makerdao/dss/blob/master/src/vat.sol)****
  * ****[**Spot**](https://github.com/makerdao/dss/blob/master/src/spot.sol)****

## 1. Introducción (Sumario)

El **Módulo Central de _Vault_** es crucial para el sistema dado que contiene el estado entero del Protocolo de Maker y controla los mecanismos centrales del sistema mientras esté en el estado operativo normal esperado.

## 2. Detalles del Módulo

### Documentación de Componentes del Módulo 

1. [**Vat - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/core-module/vat-detailed-documentation-esp)
2. [**Spot - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/core-module/spot-detailed-documentation-esp)

## 3. Mecanismos Principales y Conceptos

* `Vat` - La _Vault_ central, el Dai y el estado del colateral son guardados en el `Vat`. El contrato `Vat` no tiene dependencias externas y mantiene las "Invariantes Contables" centrales del Dai.
* `Spot` - `poke` es la única función no-autenticada en `spot`. La función toma un `bytes32` de `ilk` para ser _"pokeado"_. `poke` llama dos funciones `external`: `peek` y `file`.

## 4.0 _Gotchas_ (Posibles fuentes de error del usuario)

* Los métodos en el `Vat` están escritos para ser tan genéricos como sea posible y, como tales, tienen interfaces que pueden ser bastante detalladas. Se debe tener cuidado de no mezclar el orden de los parámetros. Cualquier módulo sobre el cual se utiliza `auth` contra el `Vat` tiene acceso completo a la raíz y puede, por ende, robar todo el colateral en el sistema. Esto significa que la incorporación de un nuevo tipo de colateral (y su adaptador asociado) conlleva un riesgo considerable.
* Cuando el `Cat` es actualizado, hay varias referencias al mismo que deben ser actualizadas al momento (`End`, `Vat.rely`, `Vow.rely`). También debe apoyarse en `End`, el `pause.proxy()` del Sistema. Puedes leer más [aquí](https://docs.makerdao.com/smart-contract-modules/core-module/cat-detailed-documentation#4-gotchas-potential-source-of-user-error-esp).
* Los métodos en el `spotter` son relativamente básicos comparados con la mayoría de las otras porciones del `dss`. No hay mucho espacio para errores de los usuarios en el único método no autenticado, `poke`. Si un `bytes32` incorrecto es suministrado, la llamada fallará. Cualquier módulo que esté autenticado contra el `spot` tiene acceso total a la raíz y puede, por ende, incorporar y remover qué `ilks` pueden ser _"pokeados"_. Si bien esto no puede romper el sistema, podría causar un riesgo considerable.
* 
## 5. Modos de Falla (Limitados a Condiciones de Operación y Factores de Riesgo Externos)

### Errores de Código

* `Vat` - Un _bug_ en el `Vat` podría ser catastrófico y podría redundar en la pérdida (o bloqueo) de todo el Dai y los Colaterales en el sistema. Podría volverse imposible modificar _Vaults_ o transferir Dai. Las Subastas dejarían de funcionar. El Apagado podría fallar.
* `Spot` - Un _bug_ en el `spot` podría resultar en que, los precios de los colaterales, no sean actualizados nunca más. En este caso, el sistema necesitará autorizar a un nuevo `spot` que podrá, luego, actualizar los precios. En general, esto no es una falla catastrófica, ya que solo detendría todas las fluctuaciones de precios durante un período.

### _Feeds_

* `Vat` - Depende de un conjunto de oráculos de confianza que provea datos de precios. Si estos _feeds_ de precios fallasen, sería posible acuñar Dai sin respaldo alguno, o _vaults_ seguras podrían ser liquidadas de manera injusta.
* `Spot` - Depende de un conjunto de oráculos de confianza que provea datos de precios. Si estos _feeds_ de precios fallasen, sería posible acuñar Dai sin respaldo alguno, o _vaults_ seguras podrían ser liquidadas de manera injusta.

### Gobernanza

* `Vat` - La Gobernanza puede autorizar nuevos módulos contra el `Vat`. Esto les permitiría robar colateral (`slip`) o acuñar Dai sin respaldo (`suck`/adición de tipos de colaterales sin valor). Si fallan las protecciones criptoeconómicas que hacen que hacerlo sea prohibitivamente costoso, el sistema puede quedar vulnerable y abierto para que malos actores drenen el colateral.