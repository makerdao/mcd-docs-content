# Anotaciones de los _Smart Contracts_ (Contratos Inteligentes)

## Introducción 

Entender los diferentes términos utilizados en nuestros _smart contracts_ (contratos inteligentes) puede implicar una inversión de tiempo bastante significativa para desarrolladores que se topan con la base de códigos por primera vez. Debido a este hecho, necesitamos recursos en varios formatos para asegurarnos que puedan entenderlos y empezar lo antes posible. Creemos que complementar la documentación técnica en docs.makerdao.com, resaltar las secciones del código base y anotarlas con comentarios puede sacar a la luz información importante que los desarrolladores necesitan mientras leen los contratos en bruto para entender mejor.

**Ejemplos de cómo la anotaciones pueden ser útiles para los desarrolladores:** 
*Resaltando términos para visualizar sus definiciones.*
  * **Ejemplo:** `ink`, `art`
* Explicar en profundidad parámetros de entrada.
  * **Ejemplo:** Explicar el rol preciso de los usuarios definidos como `gal` o `guy` dentro de parámetros de entrada
* Enlace a secciones de otros contratos inteligentes desde donde típicamente se originan las llamadas a las funciones. 
  * **Ejemplo:** `heal` en `Vat` es llamado desde `Vow`
* Contexto adicional para las prácticas diseñadas específicamente para Maker que otros desarrolladores de contratos inteligentes quizás no estén acostumbrados.   
  * **Ejemplo:** mostrar la firma de registro de eventos producida por el modificador `note` en una función. 
* Ayudar a los desarrolladores a navegar por escenarios que resultan en llamadas a través de múltiples contratos inteligentes.
* **Ejemplo:** _Keepers_ participando en subastas de colateral inician el proceso con `bite` en `Cat`, la deuda incobrable es liquidada en `Vat`, contabilizada en `Vow` y una serie de transacciones en `flip` hasta que reciben el colateral que compraron con descuento.
* Sobreponer la visión operativa del sistema en todas las funciones ayudara a los desarrolladores a construir modelos mentales mas ricos para el Protocolo Maker. 
  * **Ejemplo:** Anotar el modificador `auth` de una función y mencionar los contratos inteligentes autorizados para llamarla.  
Las anotaciones también servirán como una capa de debate abierto y colaborativo que ayuda a los _devs_ a discutir y desarrollar su entendimiento de los _smart contracts_ a través del tiempo.  

## Usar _Hypothesis_

[Hypothesis](https://web.hypothes.is/) es una herramienta web que usamos para anotar el código base alojado en nuestro Github org.

### **Ejemplo**

Échales un vistazo a las anotaciones del _smart contract_ **Vat** [aquí](https://via.hypothes.is/https://github.com/makerdao/dss/blob/master/src/vat.sol)

### Uso

Las anotaciones en _Hypothesis_ se pueden ver expandiendo una barra lateral en la esquina superior derecha de la ventana de tu navegador. 

**Hay tres opciones para ver anotaciones en una página web:**

* Instalar la [extensión de chrome](https://chrome.google.com/webstore/detail/hypothesis-web-pdf-annota/bjfhmglciegochdpefhhlphglcehbmek) desde la tienda web.
* Seguir las instrucciones para configurar un [bookmarklet](https://web.hypothes.is/start/)
* Añadir [https://via.hypothes.is](https://via.hypothes.is/) a una URL.

### **Nuestros Grupos de _Hypothesis_:**

* Grupo restringido de `MakerDAO` - cualquier persona puede leer las anotaciones, pero solo los miembros aprobados pueden colaborar [aquí](https://hypothes.is/groups/zy1LApRW/makerdao).
* Grupo Público de MakerDAO que está abierto para cualquiera que quiera leer y contribuir.

### ¡Créate una cuenta en _Hypothesis_ [aquí](https://hypothes.is/signup) para empezar!