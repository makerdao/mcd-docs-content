# Actualización de la Guía del DAI Multi-Colateral  

### **Resumen**&#x20;

**Nivel:** Intermedio

**Tiempo Estimado:** 30 minutos

**Audiencia:** Equipos técnicos y comerciales con socios y _holders_ de DAI 

## Introducción

El DAI Multi-Colateral trae consigo un montón de nuevas y emocionantes características, tales como el respaldo para nuevos tipos de colaterales de _vaults_. Para poder respaldar la nueva funcionalidad, todo el núcleo de _smart contracts_ de Maker ha sido reescrito. Las nuevas direcciones de los _smart contracts_ y ABIs pueden encontrarse aquí:  [https://changelog.makerdao.com/releases/mainnet/1.0.0/](https://changelog.makerdao.com/releases/mainnet/1.0.0/)

Por lo tanto, los usuarios y socios que interactúen con el DAI de Un Solo Colateral (SCD o SAI) deben migrar sus tokens DAI de Un Solo Colateral (SAI) a tokens DAI Multi-Colateral (actual DAI) y CDPs al nuevo sistema. Adicionalmente, compañías o proyectos integrados con SAI y CDPs deben actualizar sus bases de código para apuntar a nuevos _smart contracts_ y refactorizar su código para soportar las funciones actualizadas.   

Esta guía se enfocará en la migración de DAI y CDP con una visión general de alto nivel sobre el proceso de actualización para diferentes actores en el ecosistema Maker.  

Los pasos necesarios para migrar del DAI de Un Solo Colateral (SDC) al DAI Multi-Colateral (MCD) difieren dependiendo de tu plataforma y del caso de uso de DAI, así que esta guía se divide en secciones para diferentes tipos de usuarios y socios.   

### Nota importante sobre las convenciones de Nomenclatura 

En esta guía nos referimos al sistema del DAI de Un Solo Colateral como **SCD** y al DAI Multi-Colateral como **MCD**. Nos referimos al token DAI de un solo Colateral (el antiguo DAI) como **SAI**, y al nuevo token DAI Multi-Colateral como **DAI**.   

### Objetivo de Aprendizaje

* Conocer cómo es el funcionamiento de la migración a MCD 
* Las mejores practicas de migración para diferentes usuarios y socios 
* Dónde encontrar guías sobre escenarios de migración específicos 

### Requisitos Previos 

* Conocimientos Básicos de MakerDAO: el sistema DAI y/o de _vaults_. [Consulta  la guía MCD 101, especialmente las secciones 1 y 2.](https://github.com/makerdao/developerguides/tree/master/mcd/mcd-101)

### Secciones 

* [Escenarios de Migración de Usuarios y Socios](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#user-and-partner-migration-scenarios)
  * [Como _holder_ de Sai](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-sai-holder)
  * [Como propietario de un CDP de SCD](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-scd-cdp-owner)
  * [Como _Exchange_ Centralizado o _Wallet_ de Custodia](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-centralized-exchange-or-custodial-wallet)
  * [Como una _Exchange_ Decentralizado](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-decentralized-exchange)
  * [Como una _Wallet_  de No-Custodia](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-non-custodial-wallet)
  * [Como un _Keeper_](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-keeper)
  * [Como un _Market Maker_](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-market-maker)
  * [Como un Integrador de CDP](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-cdp-integrator)
  * [Como un Protocolo de Préstamos](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-lending-protocol)
  * [Como una Dapp](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-a-dapp)
  * [Como otro tipo de socio no mencionado anteriormente](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#as-another-partner-type-not-mentioned-above)
* [App de Migración](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#migration-app)
* [Contrato de Migración](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#migration-contract)
  * [Funcionalidad](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#functionality)
  * [Actualización de Sai a Dai](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#upgrading-dai)
  * [Cambio de Dai por Sai](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#swapping-back-to-sai)
  * [Migración de CDPs](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#migration-of-cdp)

## Escenarios de Migración de Usuarios y Socios

En la siguiente sección se describe un proceso de migración recomendado para diferentes actores en el ecosistema Maker.  

### Como _Holder_ de SAI 

**Tu tienes el control de tu llave privada**

Si tienes tu SAI en una _wallet_ donde controlas tus llaves privadas, dirígete a [migrate.makerdao.com](https://migrate.makerdao.com) y sigue las siguientes instrucciones para actualizar tu SAI a DAI y opcionalmente activar el _smart contracts_ de la Tasa de Ahorro DAI, que te permite obtener ahorros.  

**La siguiente figura describe el flujo de migración:**

![](https://camo.githubusercontent.com/4f1cca40bea0e24e56013c13f64f3f1f3911e129/68747470733a2f2f6c68362e676f6f676c6575736572636f6e74656e742e636f6d2f4d3430365a5f4d6c71414252325279396d355f67774b3850574e50783973554173334e62617a7748377076765768386753495078687a476c4d587876436b3137766f7842796152422d6351654b5a664a33447644385f4f34626947725a4c376541384c794a42377248315a6b7161756c784a58696a6a4e4f4743744d775673415749786f756a555a)

**No controlas tu llave privada**

Si tu SAI es depositado en un _exchange_ o _wallet_ centralizado o es bloqueado en un _smart contract_ dApp, puedes seguir las instrucciones que estás plataformas proporcionan o retirar el SAI y completar la actualización tu mismo en [migrate.makerdao.com](https://migrate.makerdao.com).

Con MCD puedes depositar tu DAI en el _smart contract_ de la Tasa de Ahorro DAI que te hará ganar ahorros anuales acumulados. Encuentra mas información en makerdao.com.

### Como propietario de CDP de SDC 

Como propietario de un CDP de SCD puedes mover tu CDP al núcleo del CDP del MCD a través de la App de Migración en [migrate.makerdao.com](https://migrate.makerdao.com). El siguiente diagrama muestra el flujo para la migración del CDP.  

![](https://camo.githubusercontent.com/24123aa333aa2f506c72d48345fc478e9a8095bc/68747470733a2f2f6c68342e676f6f676c6575736572636f6e74656e742e636f6d2f556f47593738685f545078654d3868327132616f4a6862774b55324b41394d59486577364c397943324766486b6d664c6d397952556b464e707278774b667876614a41495366514472635549555764724268695251793764334e575670573951734157493843706d684a61685250595030386f4c505865576677526f6e41624364546c3933726f73)

También puedes optar por cerrar manualmente tu CDP pagando de vuelta tu deuda, canjeando tu Ether y utilizando tu colateral canjeado para abrir un nuevo CDP de MCD. 

Si tienes un CDP de SCD grande, es posible que el contrato de migración no tenga suficiente liquidez SAI para llevar a cabo la migración. En ese caso, siéntete libre de contactar [integrate@makerdao.com](mailto:integrate@makerdao.com) para obtener ayuda. Puedes leer más sobre migración en la sección del [Contrato de Migración](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#migration-contract) que podrás encontrar más adelante en esta guía.

**Notas sobre Instadapp**

Si has creado tu CDP a través del servicio de Instadapp, necesitas retirar la propiedad del CDP del servicio de vuelta. Para hacer esto, necesitas dirigirte a la [pagina _exit_](https://instadapp.io/exit/) y hacer clic en _Withdraw_  ("retirar") tu CDP en la pestaña de _Debt Positions_ ("posiciones de deuda"). Esto te dará la custodia del CDP, que lo hará visible en [migrate.makerdao.com](https://migrate.makerdao.com) donde serás capaz de llevar a cabo la migración del CDP. 

**Notas sobre _MyEtherWallet_ ("mi _wallet_ de ether")**

Si has creado tu CDP en _MyEtherWallet_ entonces podrás migrar tu CDP utilizando la App de Migración en [migrate.makerdao.com](https://migrate.makerdao.com). (Sin embargo, si la llave privada utilizada con _MyEtherWallet_ está almacenada en un archivo local u otro formato no compatible, primero debes importar tu llave a una _wallet_ con soporte para Web3).

Una vez actualizada, puedes utilizar la Tasa de Ahorro de DAI bloqueando tu DAI en el _smart contract_ de la Tasa de Ahorro DAI y recibir los ahorros acumulados. Encuentra mas información en makerdao.com.  

### Como un _Exchange_ Centralizado o _Wallet_ de Custodia  

Como un _Exchange_ Centralizado o _Wallet_ de Custodia 

Te recomendamos cumplir con los siguientes pasos para actualizar a MDC:

* El 18 de Noviembre: cambiar el nombre del DAI de Un Solo Colateral a "SAI". Esto se está coordinando con todos los socios de Maker y sirve para evitar que los usuarios depositen el token equivocado en tu sistema.   
* El 2 de Diciembre: llevar a cabo la actualización de los balances de los usuarios. 
* Informa a tus usuarios lo antes posible sobre las fechas. Para los usuarios que quieran retrasar su actualización, esto les permite optar por retirar SAI desde tu _exchange_ antes de la fecha.  
* Proceso Propuesto para la actualización del 2 de Diciembre: 
  * Congelar los depósitos/retiros de SAI 
  * Utiliza el contrato/la App de migración para actualizar todos los _holdings_ de SAI a DAI. (Consulta más detalles en las secciones de la [App de Migración](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#migration-app)/[contrato](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#migration-contract) que puedes encontrar a continuación).
  * Apunta la base de código de la nueva dirección del contrato del token DAI. El nuevo token se despliega en [0x6b175474e89094c44da98b954eedeac495271d0f](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f) - utiliza los [logos actualizados que se encuentran aqui](https://www.notion.so/makerdao/Maker-Brand-ac517c82ff9a43089d0db5bb2ee045a4) para el nuevo token DAI.
  * Cambia el nombre del listado/token a ¨DAI¨
  * Descongela los depósitos/retiros de DAI.
* Informa a los usuarios sobre la [Tasa de Ahorro de Dai](https://blog.makerdao.com/an-update-on-the-dai-savings-rate-in-multi-collateral-dai/), que le permite a los _holders_ de DAI obtener ahorros. 
  * Opcional: Escoge uno de los siguientes:
    * Integrar la Tasa de Ahorro DAI y distribuir los ingresos entre tus usuarios.
    * Integrar la Tasa de Ahorro DAI en tu _exchange_ y mantener los ahorros acumulados en tu propio balance.  

![](https://camo.githubusercontent.com/4bd8e6d1f3408f8f2401b02f1023a875b1765a42/68747470733a2f2f6c68332e676f6f676c6575736572636f6e74656e742e636f6d2f725458636d355f42434b4b44565859566336765830356f61744256484c73596a53696d383447666c6847706754595453574b714e704a3042754d44642d4b56364f53736e424335467661366b364c4874644a516f66664b6d34575139326e37705a455030754c792d496a6a44446b68393241697769305558546a726c67642d37766f56684156536b)

Este enfoque dará como resultado el siguiente recorrido para el usuario de la _exchange_/_wallet_:

![](https://camo.githubusercontent.com/db991bc0866a753cc83146217d2fe85b5f8d2154/68747470733a2f2f6c68352e676f6f676c6575736572636f6e74656e742e636f6d2f6f6159547a427374565a4f6b74574369536250337163756c774b666c42635a365332643357654a756637324771776d7149496b425a6e7a447444726f6c7a6474647879356568484e47797877596c345a32367038454a7242584c3646594551644f4b4369512d6f496a396f61765863796250516d6c5f623437677258674751496a6371576e6d3067)

### Como un _Exchange_ Descentralizado

Te recomendamos cumplir con los siguientes pasos para actualizar a MDC:

* El 18 de Noviembre: cambiar el nombre del DAI de Un Solo Colateral a "SAI" y _ticker_ "SAI". Esto se coordina con todos los socios de Maker y sirve para evitar que los usuarios intenten depositar el token equivocado en tu sistema. 
* Selecciona una fecha entre el 18-25 de Noviembre para listar el DAI Multi-Colateral. El nuevo token se despliega en [0x6b175474e89094c44da98b954eedeac495271d0f](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f) - usa los [logos actualizados que se encuentran aquí](https://www.notion.so/makerdao/Maker-Brand-ac517c82ff9a43089d0db5bb2ee045a4) para el nuevo token DAI. El logo para SAI debe mantener el diamante amarillo.   
* En la fecha de tu propio listado de DAI: agrega soporte para el nuevo token DAI en tu plataforma. El nuevo token DAI debe llamarse DAI y tener el _ticker_ "DAI". Desactiva el comercio de SAI en tu UI _frontend_, pero permite a los usuarios cancelar órdenes y retirar balances.     
* Informa a los usuarios que pueden canjear SAI por DAI en migrate.makerdao.com
  * Opcional: Proporciona una UI en tu propia interfaz para la migración de tokens a través del contrato de migración.
* Informa a los usuarios sobre la Tasa de Ahorro de DAI, que permite a los _holders_ de DAI obtener ahorro.
  * Opcional: construye una UI que facilite el uso del servicio de la Tasa de Ahorro DAI para tus usuarios en tu _exchange_, donde los usuarios se quedarían con los propios ahorros acumulados.
  * Opcional: dirige a los usuarios a [oasis.app](https://oasis.app) para activar la Tasa de Ahorro DAI. 

### Como _Wallet_ de No-Custodia 

Si eres creador de una _wallet_ que permite a los usuarios tener el control de sus llaves privadas te recomendamos hacer lo siguiente:

* El 18 de Noviembre: cambiar el nombre del DAI de Un Solo Colateral a "SAI".
* Selecciona una fecha entre el 18-25 de Noviembre para ejecutar la actualización de soporte DAI Multi-Colateral, que debe ser listado como "DAI". El nuevo token es desplegado en [0x6b175474e89094c44da98b954eedeac495271d0f](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f) para el nuevo token DAI. El logo para SAI debe mantener el diamante amarillo.
* Informa a tus usuarios lo antes posible sobre los calendarios para tu propia actualización a MCD. 
* Apoya los balances de SAI y DAI durante un período hasta que la demanda de DAI disminuya. 
* Informa a tus usuarios que podrían intercambiar SAI por DAI en [migrate.makerdao.com](https://migrate.makerdao.com).
  * Opcional: Proporciona una UI en tu propia interfaz para la migración de tokens a través del contrato de migración.
* Informa a los usuarios sobre la Tasa de Ahorro de DAI, que permite a los _holders_ de DAI obtener ahorro.
  * Opcional: Crea una UI donde los usuarios puedan activar la Tasa de Ahorro DAI.
  * Opcional: dirige a los usuarios a [oasis.app](https://oasis.app) para activar la Tasa de Ahorro DAI. 
* Opcional: implementar el pago del costo de gas de las transacciones DAI en nombre de tus usuarios.

### Como un _Keeper_

* Familiarízate con las actualizaciones de _Keepers_ y Subastas en MCD con [esta guía](https://github.com/makerdao/developerguides/blob/master/keepers/auctions/auctions-101.md).
* Actualizar
* Esperamos lanzar una biblioteca Python para trabajar con subastas. Esta será la forma recomendada de ofertar en Subastas.

Alternativamente, si estás dispuesto a hacer algo de trabajo adicional y trabajar con una interfaz de nivel menor, puedes interactuar con los contratos de Subasta directamente [flip](https://github.com/makerdao/dss/blob/master/src/flip.sol), [flap](https://github.com/makerdao/dss/blob/master/src/flap.sol), [flop](https://github.com/makerdao/dss/blob/master/src/flop.sol)). Ten en cuenta que futuros tipos de colateral pueden venir con formatos de subastas personalizados. Más información estará disponible antes del lanzamiento. 

### Como un _Market Maker_

* Te animamos a participar como _Market Maker_ en el DAI Multi-Colateral tan pronto como tus socios de _exchange_ añadan soporte para ello.  
* Si tus socios de _exchange_ mantienen su lista de SAI al mismo tiempo que su lista de DAI, te animamos a _market make_ en ambos tokens durante el tiempo de vida restante de SAI.   
* Si tus socios de _exchange_ utilizaran un _ticker_ diferente para DAI y para SAI, deberías actualizar tus herramientas en consecuencia. 

### Como Integrador de CDP

**Servicio de CDP de Custodia**

* El 18 de Noviembre: cambiar el nombre del DAI de Un Solo Colateral a "SAI".
* Selecciona una fecha entre el 18-25 de Noviembre para ejecutar la actualización a MCD.
* Informa a tus usuarios lo antes posible sobre la fecha. 
* En la fecha elegida: 
  * Congela el acceso al servicio de CDP para tus usuarios. 
  * lanza la actualización de tu servicio que soporte el nuevo núcleo CDP. [Las direcciones del contrato inteligente y ABIs se pueden encontrar aquí.](https://changelog.makerdao.com/releases/mainnet/1.0.0/) 
    * Si estás utilizando _Dai.js_ para tu integración CDP, consulta “[Usando _Dai.js_](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#using-dai.js)” más abajo para saber cómo actualizar tu implementación a MCD. 
    * Si te has integrado directamente con los _smart contracts_ de CDP, consulta “[La integración directa con _smart contracts_](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#direct-integration-with-smart-contracts)” más abajo para saber cómo actualizar tu implementación a MCD.
  * Migrar todos los CDPs a MCD. Consulta la sección de "App de Migración" más abajo.  
  * Listar el token DAI Multi-Colateral como "Dai" 
  * Descongelar el acceso al servicio de CDP 
* Opcional: Implementar soporte para los tipos de colateral añadidos en MCD  
* Si es relevante para tu servicio, informa a tus usuarios sobre la Tasa de Ahorro de DAI 
  * Opcional: Implementar UI para bloquear DAI en el _smart contract_ de la Tasa de Ahorro DAI.  

**Servicio CDP de No-Custodia**

* El 18 de Noviembre: cambiar el nombre del DAI de Un Solo Colateral a "SAI".
* Selecciona una fecha entre el 18-25 de Noviembre para ejecutar la actualización a MCD.
* Informa a tus usuarios lo antes posible sobre las líneas de tiempo para tu propia actualización a MCD. 
* Informa a tus usuarios sobre el MCD y el proceso de migración de CDPs.
* En la fecha de lanzamiento seleccionada: 
  * Lanza la actualización de tu servicio que soporte el nuevo núcleo CDP.  
    * Si estás utilizando _Dai.js_ para tu integración CDP, consulta “[Utilizando _Dai.js_](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#using-dai.js)” más abajo para saber cómo actualizar tu implementación a MCD.
    * Si te has integrado directamente con los _smart contracts_ CDP, consulta “[La integración directa con _smart contracts_](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/upgrading-to-multi-collateral-dai.md#direct-integration-with-smart-contracts)” más abajo para saber cómo actualizar tu implementación a MCD.
  * Lista el token DAI Multi-Colateral como "Dai" 
* Escoge uno de los siguientes:
  * Opción A: dirige a tus usuarios a [migrate.makerdao.com](https://migrate.makerdao.com) en la fecha de lanzamiento del MCD para la migración de CDP en su panel de control CDP. También consulta la sección de App de Migración mas abajo.   
  * Opción B: Crea tu propio UI para la migración, creando un _frontend_ para interactuar con el contrato de migración (consulta más abajo la sección sobre el Contrato de Migración). 

### **Actualiza tu implementación de integración CDP** 

#### **Utilizando _Dai.js_**

* Si has integrado CDPs utilizando la [biblioteca _Dai.js_](https://github.com/makerdao/dai.js), asegúrate de actualizar la biblioteca a la ultima versión.
* Actualiza tu código base para soportar la funcionalidad del [_plugin_ MCD](https://github.com/makerdao/dai.js/tree/dev/packages/dai-plugin-mcd). En el momento del lanzamiento de este _plugin_ se incluirá por defecto en la biblioteca _Dai.js_.  
* Opcional: Ayuda a tus usuarios a migrar su CDP a MCD
  * Opción A: Dirige a tus usuarios a [migrate.makerdao.com](https://migrate.makerdao.com) si tu app es compatible con Web3. 
  * Opción B: Implementa tu propia UI de migración en tu app, conectándose al contrato de migración descrito en la sección de abajo. 
  * Opción C: Si tu app no es compatible con migrate.makerdao.com, puedes guiar a tus usuarios sobre cómo exportar su CDP desde tu app a una _wallet_ compatible.  
* Opcional: Implementar el soporte para la nueva funcionalidad de MCD
  * Agregar soporte para nuevos tipos de colaterales.  
  * Agregar soporte para la Tasa de Ahorro de DAI.

### **Integración directa con _smart contracts_**

* Si te has integrado directamente con los contratos inteligentes, debes agregar soporte  para los nuevos _smart contracts_ del núcleo Maker. Dado que los _smart contracts_ han sido completamente reescritos, muchas funciones de llamada han sido cambiadas.   
* Familiarízate con la [nueva implementación de MCD](https://github.com/makerdao/dss)
  * [Puedes encontrar una introducción al sistema aquí](https://github.com/makerdao/developerguides/blob/master/mcd/mcd-101/mcd-101.md)
* Implementar soporte para los _smart contracts_ MCD 
  * [Consulta esta guía sobre cómo interactuar con el gestor CDP](https://github.com/makerdao/developerguides/blob/master/mcd/mcd-seth/mcd-seth-01.md)
* Apunta el código base a los nuevos [_smart contracts_ MCD](https://changelog.makerdao.com)

### Como un Protocolo de Prestamos  

**Servicio de custodia**

* El 18 de noviembre: Cambia el nombre del DAI de Un Solo Colateral a "Sai".
* Selecciona una fecha entre el 18-25 de Noviembre para ejecutar la actualización a MCD.
* Informa a tus usuarios lo antes posible sobre la fecha. 
* En la fecha escogida:
  * Deja de prestar (depósitos) y tomar prestado (retiros) de SAI   
  * Lista el token DAI Multi-Colateral como "DAI". El nuevo token es desplegado en [0x6b175474e89094c44da98b954eedeac495271d0f](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f) - utiliza los [logos actualizados que se encuentran aquí](https://www.notion.so/makerdao/Maker-Brand-ac517c82ff9a43089d0db5bb2ee045a4) para el nuevo token DAI.
  * Abrir para prestar (depósitos) y pedir prestado (retiros) DAI. 
  * Para los préstamos pendientes en SAI, elige una de las siguientes opciones: 
    * Aceptar la devolución de préstamos en SAI
    * Migrar continuamente las devoluciones de antiguas posiciones de SAI a DAI tu mismo. 
    * Informa a tus usuarios que ya no puedes devolver "SAI", pero que deben migrar su SAI a DAI a través de migrate.makerdao.com antes de devolver el préstamo.   

**Servicios de No-Custodia**

* El 18 de noviembre: Cambia el nombre del DAI de Un Solo Colateral a "Sai".
* Selecciona una fecha entre el 18-25 de Noviembre para ejecutar la actualización a MCD.
* Informa a tus usuarios lo antes posible sobre las fechas para tu propia actualización a MCD.
* Informa a los usuarios sobre las posibles fechas para el apagado de SCD.
* En el momento del lanzamiento:
  * Lista el token DAI Multi-Colateral como "DAI". El nuevo token es desplegado en [0x6b175474e89094c44da98b954eedeac495271d0f](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f) - utiliza los [logos actualizados que se encuentran aquí](https://www.notion.so/makerdao/Maker-Brand-ac517c82ff9a43089d0db5bb2ee045a4) para el nuevo token DAI.
  * Lanza el soporte para los préstamos en DAI.
  * Pausa la creación de prestamos en SAI.
  * Dirige a los usuarios a [migrate.makerdao.com](https://migrate.makerdao.com) para la migración a SAI. 
  * Deja correr los préstamos existentes en SAI hasta que caduquen o sean pagados de vuelta. 
* Opcional:
  *  Crea una UI para que  los usuarios migren sus balances de SAI a DAI. 

### Como una Dapp

* El 18 de noviembre: Cambia el nombre del DAI de Un Solo Colateral a "Sai"
* Selecciona una fecha entre el 18-25 de Noviembre para ejecutar la actualización a MCD.
* Informa a tus usuarios lo antes posible sobre las fechas para tu propia actualización a MCD.
* En la fecha escogida:
  * Lista el token DAI Multi-Colateral como "DAI". El nuevo token es desplegado en [0x6b175474e89094c44da98b954eedeac495271d0f](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f) - utiliza los [logos actualizados que se encuentran aquí](https://www.notion.so/makerdao/Maker-Brand-ac517c82ff9a43089d0db5bb2ee045a4) para el nuevo token DAI.
  * Actualiza la base del código para soportar el uso del nuevo token DAI en el lanzamiento.  
  * Opcional: Implementa el pago del costo del gas de las transacciones de DAI en DAI.
* Si tienes un producto que usa SAI: 
  * Apaga la funcionalidad de SAI en una fecha limite, bien comunicado con suficiente antelación a tus usuarios.
* Informa a tus usuarios sobre la posible confusión entre SAI y DAI. 
* Informa a tus usuarios que pueden migrar de SAI a DAI en [migrate.makerdao.com](https://migrate.makerdao.com)
  * Opcional: crea una UI para llevar a cabo la migración desde SAI a DAI. 

### Como otro tipo de socio no mencionado antes 

Ponte en contacto con [integrate@makerdao.com](mailto:integrate@makerdao.com) y estaremos felices de discutir tu escenario de migración. 

## App de Migración

Tras el lanzamiento del MCD, la App de Migración en [migrate.makerdao.com](https://migrate.makerdao.com) te permitirá realizar la migración de DAI y CDP a través de una UI web intuitiva en solo unos cuantos clics. Al iniciar sesión con tu _wallet_ favorita, la app escaneara tu _wallet_ en búsqueda de migraciones recomendadas y las mostrará en la UI (como se ve en la imagen de abajo). Está previsto que esta función de escaneo de migraciones reciba un apoyo continuo, asegurando que los usuarios siempre utilicen una versión actualizada de la plataforma Maker.  

![](https://camo.githubusercontent.com/5b22fab66572ce94685d96c28dfa34ef0c348ed2/68747470733a2f2f6c68342e676f6f676c6575736572636f6e74656e742e636f6d2f346c44634533443439584b746c724c532d614143444b3073307638336d3447347a77705a726d575a4c364c53326b3844726a44705946452d7957316e78342d72643871615878504a684c5a6e636a6d4e6c7a65436b316f6474704a796e4e527a48336579434f316a6d665033563639624c444e6151794d4b344c74786f494d30374266646b323465)

_Pagina de Aterrizaje que te mostrará posibles migraciones para la _wallet_ conectada._ 

![](https://camo.githubusercontent.com/553d08d5f144a4bd70ed6e23fbcdc7133b525ba1/68747470733a2f2f6c68332e676f6f676c6575736572636f6e74656e742e636f6d2f425244646738574232517a7952735f393267473035734b444763716d734b5a5a7657526470514a6d4637786d695366376a79306f5a7138775537786d4c365834396763545646454b6e33746576655f556e72705a796e46633038304e78546c6d43564632534a5673666d666e5931346a376f6a52524f5858726e59646d793458552d744a36754233)

_Wizard para la migración de Sai a Dai._

![](https://camo.githubusercontent.com/f26216bad050d8ca4c746bba0ddcd91d8976c5b4/68747470733a2f2f6c68342e676f6f676c6575736572636f6e74656e742e636f6d2f5f5a324c594f45396c7346754267776955504f546b6d72564b7870545536745a624c5356517663702d4c5256393576486f7a5545562d76365a5267434367496a693048584241554e49336f73386568515a416374463135794b467341444c76735a75616c436f6d6938444e32767658584d364e68356a436779636c4475694f70764133586e417071)

_Wizard para la migración de un CDP SCD a un CDP MCD._

La App de Migración utiliza un contrato _proxy_ para llevar a cabo la migración de CDP. En consecuencia, la app solo puede utilizarse para CDPs que hayan sido creados a través del contrato _proxy_ Maker. Esto ocurre automáticamente si ya has abierto tu CDP en [cdp.makerdao.com](https://cdp.makerdao.com).

Si ya has creado CDPs utilizando servicios de terceros que no usan _proxies_ de Maker para interactuar con el núcleo CDP, el contrato de migración puede no funcionar. En su lugar, puedes llevar a cabo tu propia migración manual, simplemente cerrando tu CDP SCD y moviendo el ETH a un CDP MCD 

### Contrato de Migración 

La funcionalidad de la App de Migración descrita en la sección anterior es manejada por un Contrato de Migración que se desplegará en el lanzamiento de MCD para poder soportar una transición suave desde el DAI de un Solo Colateral al DAI Multi-Colateral. El contrato hará que el canje de tokens DAI de un Solo Colateral (**Sai**) a DAI Multi-Colateral (**Dai**) y la migración de CDPs al nuevo motor CDP de MCD sea una tarea fácil. Esta sección describirá cómo funciona el contrato para poder ayudar a los super usuarios y socios a prepararse para la migración a MCD.     

#### Funcionalidad 

Los _smart contracts_ de migración son de código abierto y se pueden encontrar aquí: [https://github.com/makerdao/scd-mcd-migration](https://github.com/makerdao/scd-mcd-migration)

En la carpeta `src`, se puede encontrar el código fuente del _smart contract_. El contrato principal es el `ScdMcdMigration.sol` que contiene la funcionalidad de migración con la que interactúas.     

Contiene tres funciones principales: 

* `swapSaiToDai` - una función que actualiza de Sai a Dai 
* `swapDaiToSai` - una función que te permite intercambiar tu Dai de vuelta a Sai 
* `migrate` - una función que te permite migrar tu CDP SDC a MCD. 

Las siguientes secciones profundizarán en estas llamadas a funciones. La App de Migración presentará esta funcionalidad en una IU fácil de usar, por lo que un usuario regular no tendrá que lidiar con estas llamadas de funciones directamente. Sin embargo, nos sumergiremos en las siguientes secciones para diseccionar cómo funciona la migración y esbozar el proceso para los usuarios avanzados o socios, que quieran llevar a cabo la migración por sí mismos.      

### Actualización de DAI 

Para poder actualizar tu SAI a MCD, debes usar la función [swapSaiToDai](https://github.com/makerdao/scd-mcd-migration/blob/master/src/ScdMcdMigration.sol#L59) en el contrato de migración. Primero debes aprobar que el contrato de migración pueda transferir tokens SAI desde tu cuenta. Después estás listo para invocar el _swap_, llamando a la función, especificando la cantidad de SAI que quieres actualizar a DAI. Después que la llamada a la función sea acuñada, el DAI actualizado es enviado a la dirección de Ethereum que inicia la actualización. [Aquí puedes encontrar]](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/cli-mcd-migration.md) un recorrido detallado utilizando herramientas CLI para llevar a acabo estas funciones.

Desde la perspectiva del usuario, esta función simplemente actualiza una cantidad de SAI a DAI. 

Detrás de escenas, los tokens SAI depositados son utilizados para crear un CDP colectivo en MCD para todos los usuarios que migran, cuyo DAI es acuñado. El contrato de migración tomará los tokens SAI de tu cuenta y los depositará en el adaptador del token SAI, que permite al motor CDP _Vat_ utilizar los tokens para acciones de CDP. El contrato de migración entonces invocará al _Vat_ para bloquear SAI y emitir DAI al adaptador DAI. El contrato de migración saldrá entonces de los tokens DAI del adaptador DAI, que es llevado a cabo invocando una función de acuñación en el contrato token DAI, que generará nuevo DAI para la dirección Ethereum del originador. La razón por la que la migración de SAI a DAI utiliza el núcleo CDP (_vat_) del nuevo sistema es porque este es el único componente que tiene la autoridad de acuñar nuevo DAI. El proceso y las llamadas a la función se describen en el siguiente diagrama.       

El siguiente diagrama describe qué ocurre al migrar 10 SAI a 10 DAI. 

![](https://camo.githubusercontent.com/f175b27976e1dfb08a53b494905b716a32a7e7f0/68747470733a2f2f6c68342e676f6f676c6575736572636f6e74656e742e636f6d2f516c4f47653433525a4d70664a364548323248334c3750534a4e4c424753737a586c4232396b476f5342582d7176685f7141594e374366462d77732d686f69505134636b546f2d70684a766d34574a7347326e73545f744a585f446c6e434361766645577a4464544e5938793079536841464a4331735155654a526b426659674c6369795750476c)

![](https://camo.githubusercontent.com/f70e39df18a538952509d56d92eb67838542d2f2/68747470733a2f2f6c68332e676f6f676c6575736572636f6e74656e742e636f6d2f776a435041396e367739335634414f52587546504b3952685258786c673059692d365a337a66386b364967425730535450693645796e4a39532d4150535a37747368537879755a344d4a79564f3461476a344e6e61704155755147506b4f4747675979576f484442543855536168617a5947427452774b796c69432d6866396c457376476b324433)

### _Swapping_ de vuelta a SAI

El contrato de migración también permite a los usuarios "volver atrás" al intercambiar DAI para SAI, utilizando la función [_swapDaiToSai_](https://github.com/makerdao/scd-mcd-migration/blob/master/src/ScdMcdMigration.sol#L75). En este caso, la operación CDP esta reservada, ya que el DAI es pagado de vuelta al sistema y SAI es liberado, al igual que el reembolso de un CDP normal, excepto que sin coste de tasa de estabilidad.   

Sin embargo, esta operación requiere un excedente de SAI que ya esta depositado en el contrato de migración. Por lo tanto, debe haber al menos una cantidad de SAI depositada en el contrato que sea equivalente a la cantidad de DAI que quieres canjear. 

Esta llamada a la función es muy similar a la anterior, excepto que esta vez el DAI es depositado en el CDP y se libera el colateral SAI. Esto requiere que apruebes que el contrato de migración pueda transferir DAI desde tu _wallet_ y después invocar la función _swapDaiToSai_ con la cantidad específica de DAI que quieres canjear. Puedes consultar [esta guía](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/cli-mcd-migration.md) para ver en detalle como se llaman a las funciones.  

### Migración de CDP

El contrato de migración también permite que los usuarios migren sus CDPs desde el núcleo SCD al núcleo MCD. Esto se hace a través de la función [_migrate_](https://github.com/makerdao/scd-mcd-migration/blob/master/src/ScdMcdMigration.sol#L90). La función esencialmente trata  de cerrar tu CDP, utilizando el exceso de SAI depositado en el contrato de migración (por usuarios que han actualizado SAI a DAI) para pagar tu deuda pendiente de CDP. Para poder hacerlo, necesitas transferir el control del CDP al contrato de migración. Entonces el contrato de migración pagará de vuelta la deuda utilizando el SAI depositado en el contrato, canjeará el colateral ETH, creará un nuevo CDP en el sistema MCD, bloqueará el colateral ETH y pagará de vuelta la deuda utilizando el DAI generado, resultando en una deuda CDP equivalente  en MCD.    

Sin embargo, para poder cerrar el CDP, se debe pagar una tarifa de estabilidad en MKR, por lo que necesitas conceder al contrato de migración la aprobación para gastar MKR de tu cuenta y así poder llevar a cabo la migración.

El contrato de migración utiliza un contrato _proxy_ para llevar a cabo todos los pasos anteriores de una sola vez. Consecuentemente, el contrato solo puede ser utilizado por CDPs que han sido creadas a través de un contrato _proxy_ de Maker. Esto ocurre automáticamente si has abierto tu CDP en [cdp.makerdao.com](https://cdp.makerdao.com). Por lo tanto, debes utilizar el contrato [_MigrationProxyActions.sol_](https://github.com/makerdao/scd-mcd-migration/blob/master/src/MigrationProxyActions.sol) para llevar a cabo la  [llamada a la función _migrate_](https://github.com/makerdao/scd-mcd-migration/blob/master/src/MigrationProxyActions.sol#L38).

Si has creado CDPs utilizando servicios de terceros que no utilizan _proxies_ de Maker para interactuar con el núcleo CDP, puede que el contrato de migración no funcione. En su lugar, puedes realizar tu propia migración manual simplemente cerrando tu CDP SCD y moviendo ETH a un CDP MCD.    

Para migrar tu CDP, también dependes del exceso de liquidez de SAI en el contrato de migración usado para cerrar tu CDP. Si tienes un CDP con una deuda de 10,000 SAI que quieras migrar, debe haber al menos 10,000 SAI depositados en el CDP MCD de SAI que es propiedad del contrato de migración (de los usuarios que actualizan SAI a DAI) para llevar a cabo la migración CDP. La migración no puede llevarse a cabo parcialmente, por lo que toda la deuda del CDP debe ser cubierta por la liquidez de SAI en el contrato para llevar a cabo la migración. Si tienes un gran CDP y te preocupas por la migración, no dudes en ponerte en contacto con el equipo de integraciones en [integrate@makerdao.com](mailto:integrate@makerdao.com)

[Lee más sobre las llamadas de funciones para migrar un CDP aquí](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/cli-mcd-migration.md#migrating-cdps)

### En Resumen

En esta guía, te hemos presentado los pasos para actualizarte a DAI Multi-Colateral. Te hemos proporcionado directrices para los diferentes tipos de plataformas  que utilizan DAI y para los _holders_ regulares de DAI. Más detalles estarán disponibles en el futuro.  

#### Solución de Problemas

Si tienes algún problema respecto al proceso de actualización, no dudes en ponerte en contacto con nosotros. 

* Contacta al equipo de integraciones - [integrate@makerdao.com](mailto:integrate@makerdao.com)
* Rocket chat - canal #dev

#### Siguientes Pasos

Creemos que después que termines esta guía puedes disfrutar las siguientes:

* Aprende sobre nuestro progreso alrededor del lanzamiento del [MCD](https://blog.makerdao.com/multi-collateral-dai-milestones-roadmap/).

#### Recursos

**Información:**

* [Blog post: El camino hacia el lanzamiento de la _Mainnet_](https://blog.makerdao.com/the-road-to-mainnet-release/)

**Otras Guías:**

* [Introducción y Visión General del DAI Multi-Colateral : MCD101](https://github.com/makerdao/developerguides/blob/master/mcd/mcd-101/mcd-101.md)
* [Utilización de MCD-CLI para crear y cerrar un CDP MCD en Kovan](https://github.com/makerdao/developerguides/blob/master/mcd/mcd-cli/mcd-cli-guide-01/mcd-cli-guide-01.md)
* [Utilización de _Seth_ para crear y cerrar un CDP MCD en Kovan](https://github.com/makerdao/developerguides/blob/master/mcd/mcd-seth/mcd-seth-01.md)
* [Utilización de _Seth_ para la migración MCD](https://github.com/makerdao/developerguides/blob/master/mcd/upgrading-to-multi-collateral-dai/cli-mcd-migration.md)
* [Agregar un nuevo tipo de colateral a DCS - Kovan](https://github.com/makerdao/developerguides/blob/master/mcd/add-collateral-type-testnet/add-collateral-type-testnet.md)

**Código fuente/wiki:**

* [Código del DAI Multi-Colateral + wiki](https://github.com/makerdao/dss)
