# Módulo de Oráculo

* **Nombre del Módulo:**  Módulo de Oráculo 
* **Tipo/Categoría:** Oráculos —> OSM.sol & Median.sol
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)****
* **Fuente del Contrato:**
  * ****[**Median**](https://github.com/makerdao/median/blob/master/src/median.sol)****
  * ****[**OSM**](https://github.com/makerdao/osm/blob/master/src/osm.sol)****

## 1. Introducción (Resumen)

Se despliega un módulo de oráculo para cada tipo de colateral, alimentándolo con los datos de precios para el tipo de colateral correspondiente al `Vat`. El módulo de oráculo introduce la lista blanca de direcciones, que les permite emitir actualizaciones de precios _off-chain_, que luego se introducen en `median` antes de ser llevado dentro del `OSM`. El _`Spot`'ter_ luego procederá a leer el `OSM` y actuará como un enlace entre los `oracles` (oráculos) y `dss`.   

## 2. Detalles del Módulo 

El Módulo de Oráculo tiene 2 componentes principales que consisten en los contratos `Median` y `OSM`.

### Documentación de los Componentes del Módulo de Oráculo

* ****[**Documentación _Median_**](https://docs.makerdao.com/smart-contract-modules/oracle-module/median-detailed-documentation-esp)****
* ****[**Documentación OSM**](https://docs.makerdao.com/smart-contract-modules/oracle-module/oracle-security-module-osm-detailed-documentation-esp)****

## 3. Mecanismos y Conceptos Clave 

![Diagrama de Integración (Crédito: Presentación MCD-101, por Kenton Prescott)](../../.gitbook/assets/oracles2.png)

#### Resumen de los **Componentes del Módulo de Oráculo**

* **`Median`** proporciona un precio de referencia confiable para Maker. En resumen, funciona manteniendo una lista blanca de contratos de alimentación de precios que están autorizados de postear actualizaciones de precios. Cada vez que se recibe una nueva lista de precios, se calcula la _median_ de estos y se utiliza para actualizar el valor almacenado. La _Median_ tiene una lógica de permisos que la habilita para agregar y remover direcciones de alimentación de precios de la lista blanca, que se controlan mediante la Gobernanza. La lógica de permisos capacita a la Gobernanza para fijar otros parámetros y controlar el comportamiento de _Median_ (por ejemplo, el parámetro `bar` es el número mínimo de precios necesarios para aceptar un nuevo valor de la _median_).    
* El **`OSM`** (nombrado por el acrónimo en inglés "Oracle Security Module", Módulo de Seguridad de Oráculos) asegura que los nuevos valores de precios propagados desde los Oráculos no sean tomados por el sistema hasta que haya pasado un retraso especifico. Los Valores son leídos desde un contrato [DSValue](https://github.com/dapphub/ds-value) designado (o cualquier contrato que implemente las interfaces `read()` y `peek()`) mediante el método `poke()`; los métodos `read()` y `peek()` darán el valor actual de la fuentes de precios y otros contratos deben estar en la lista blanca para poder llamarlos. Un contrato OMS solo puede leer de una sola fuente de precios, así que en la práctica se debe desplegar un contrato OMS por cada tipo de colateral.

## 4. Gotchas (Posibles fuentes de error del usuario)

#### **Relación entre el OSM y la _Median_:**

* Puedes leer directamente desde _median_ y, a cambio, obtendrías un precio más acorde en tiempo real. Sin embargo, esto depende de la cadencia de la actualización (llamadas a _poke_).  
* El OSM es similar, pero tiene un retraso de 1 hora. Tiene el mismo proceso de la lectura (listas blancas, _auth_, _read_ y _peek_) que _median_. La forma en que el OSM trabaja, es que no se puede actualizar directamente, pero se puede `poke` para ir y leerlo desde algo que también tiene la misma estructura (el método `peek`, en este caso, es la _median_ pero puedes configurarlo para leer desde cualquier cosa que se ajuste a la misma interfaz).
* Cada vez que el OSM lee desde una fuente, pone en cola el valor leido para la siguiente hora o la siguiente propiedad `hop`, que es fijado a 1 hora (pero puede ser cualquier cosa). Cuando es `poke`'d (¨pokeado¨) lee el valor de _median_ y guarda el valor. Luego, el valor previo se convierte en eso, así que siempre estará desfasado durante una hora. Después de trascurrida la hora, cuando es `poke`'d (¨pokeado¨), el valor guardado se convertirá en el valor actual y cualquier valor que esté en _median_ se convertirá en el valor futuro para la siguiente hora. 
* `spot` - si es `poke`d con un ilk (ejemplo: ETH), leerá el OSM y, si el precio es válido, se actualizará.  

#### Relación con el _`Spot`'ter_:

* En relación con el `Spot`, el Módulo de Oráculo maneja cómo los precios del mercado son registrados en la _blockchain_. El _`Spot`ter_ opera como el contrato de interfaz que los actores externos pueden utilizar para recuperar el precio de mercado actual del Módulo de Oráculo por el tipo de colateral específico. A cambio, el `Vat` lee el precio de mercado del _`spot`ter_. 

## 5. Modos de Falla (Limites en las Condiciones de Funcionamiento y Factores de Riesgo Externos)

* `Median` - actualmente no hay forma de desactivar el oráculo (fallo o retorno falso) si todos los oráculos vienen juntos y firman un precio de cero. Esto resultaría en que el precio sea inválido y retornaría falso en `peek`, diciendo que no nos confiemos del valor.    

* `OSM`
  * `poke()` no es llamado prontamente, permitiendo que los precios maliciosos suban rápidamente.
  *  Ataques de Autorización y Mal Configuraciones.
  * Puedes leer más [aquí](https://docs.makerdao.com/smart-contract-modules/oracle-module/oracle-security-module-osm-detailed-documentation-esp#5-failure-modes-bounds-on-operating-conditions-and-external-risk-factors).