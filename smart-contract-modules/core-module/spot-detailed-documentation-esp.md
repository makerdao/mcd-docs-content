# Spot - Documentación Detallada 

* **Nombre del Contrato:** spot.sol
* **Tipo/Categoría :** DSS —> Modulo Core
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/spot.sol)
* ****[**Etherscan** ](https://etherscan.io/address/0x65c79fcb50ca1594b025960e539ed7a9a6d434a3)

## 1. Introducción 

El vínculo `Spot` entre los `oráculos` y los contratos centrales. Funciona como un contrato de interfaz y solo almacena la lista `ilk` actual.

![](https://github.com/makerdao/mcd-docs-content/blob/master/.gitbook/assets/Screen%20Shot%202019-11-17%20at%202.26.39%20PM.png?raw=true)

## 2. Detalles del Contrato

#### **Matemáticas**

* Todas las operaciones Matemáticas se revertirán en caso de desbordamiento o subdesbordamiento. 

#### **Complejidad**

* Todos los métodos se ejecutan en tiempos constantes  

#### **Variables**

* `ilk` un tipo de colateral determinado.
* `ilk.pip` el contrato que posee el precio actual de un `ilk` determinado.
* `ilk.mat` el ratio de liquidación de un `ilk` determinado.
* `vat` el núcleo del sistema mcd.
* `par` el valor del DAI en el activo de referencia (por ejemplo: 1$ por 1DAI).

#### **Colateral**

* Solo usuarios autorizados pueden actualizar las variables del contrato.

## 3. Mecanismos y Conceptos Claves

### _Poke_ (Picar)

`poke` es la única función no autenticada en `spot`. La función toma un `bytes32` del `ilk` para ser “pokeado”. `poke` llama dos funciones `externas`:
1. `peek` llama al [OSM](https://docs.makerdao.com/smart-contract-modules/oracle-module/oracle-security-module-osm-detailed-documentation) para el `ilk` determinado y toma de vuelta el `val` y el `has` (un booleano que es _false_ si hubo un error en el `osm`). La segunda llamada externa solo ocurre si `has == true`.
2. Cuando se calcula el `spot`, el `par` es crucial para este cálculo, ya que define la relación entre DAI y 1 unidad de valor en el precio. Después, el `val` es dividido por el `par` (para obtener un ratio de `val` a `DAI`) y, luego, el valor resultante es dividido por el `ilk.mat`. Esto nos da el precio `spot` actual para el `ilk` determinado. 
3. A continuación, se llama a `file` después de calcular el `spot`. Esto actualiza el `vat` con el precio de liquidación actual del `ilk` para el que se llamó la función. 

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

Los métodos en el `spotter` son relativamente básicos comparados con la gran mayoría de las otras partes del `dss`. No hay mucho margen para errores de usuario en el método `poke` único y no autorizado. Si se proporciona un `bytes32` incorrecto, la llamada fallará.

Cualquier módulo que se autentique contra el `spot`, tiene acceso total de raíz, y por lo tanto puede, agregar y eliminar qué `ilks` puedan ser "pokeados". Si bien no rompe al sistema por completo, esto puede ser un riesgo considerable.  

## 5. Modos de Fallo

#### **Error de Codificación**

Un error en `spot` probablemente provocaría que los precios de los colaterales no se actualizaran más. En este caso, el sistema necesitaría autorizar un nuevo `spot` que sea capaz de actualizar los precios. En general, esto no se trata de un fracaso catastrófico, ya que solo detendría la fluctuación de los precios durante un período de tiempo.

#### **_Feeds_ (Fuente de Alimentación)**

El `spot` se apoya en un conjunto de oráculos de confianza para proporcionar datos de precios. Si estas fuentes de precios fallaran, podría acuñarse Dai sin respaldo, o que se liquidaran injustamente _Vaults_ seguras. 

#### **Precio _Spot_ se vuelve Obsoleto**

Cuando `poke` no es llamado con la suficiente frecuencia necesaria, el precio `spot` de `Vat` pasa a volverse obsoleto. Esto podría ocurrir por algunas razones, incluyendo la tragedia de los bienes comunes o la colusión minera, pudiendo llevar a resultados negativos como liquidaciones indebidas o que se eviten liquidaciones que debieron haber ocurrido.  