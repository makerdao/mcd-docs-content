# _Keeper_ de Arbitraje Simple

## Visión General

Normalmente, los _Keepers_ en forma de _bots_ automatizados son actores externos que participan en la [creación de mercado](https://github.com/makerdao/market-maker-keeper), [subastas](https://github.com/makerdao/auction-keeper), [arbitraje de mercado](https://github.com/makerdao/arbitrage-keeper) y el mantenimiento del sistema dentro del Protocolo de Maker y del gran ecosistema de Ethereum. Bajo un supuesto puramente económico, solamente están incentivados por las ganancias o intereses creados. Sin embargo, su actividad proporciona servicios indirectos como el aumento de la liquidez y la consistencia de precios a través de varios mercados financieros. Esta guía presenta un _Keeper_ de arbitraje simple y su estructura, que puede utilizarse fuera de la caja para participar en mercados muy volátiles. El propósito de esta guía es motivar al usuario a mejorar sus funcionalidades para participar en mercados menos volátiles, llevando a un ecosistema financiero eficiente. 

### Objetivos de Aprendizaje

Al leer esta guía, ganarás:
* Un mejor entendimiento del Arbitraje, cómo se logra y el rol que juegan los mercados financieros. 
* Una visión general del _Keeper_ de Arbitraje Simple y la estructura común a través del marco de los _Keepers_ de Maker.
* Información sobre el funcionamiento del _Keeper_ de arbitraje simple y formas de mejorar su código. 

### Requisitos Previos 

* Al menos Python 3.6.2 y algo de experiencia en python 
* Linux, macOS, o Cygwin (en Windows)
* Alguna experiencia general Desarrollando en Ethereum (gestión de cuentas, interacción de contratos, _hosting_ de nodos)

### Guía 

#### Introducción al Arbitraje

El arbitraje es el proceso de comprar simultáneamente un activo en una _exchange_ y vender un activo similar, si no idéntico, en otra _exchange_ a un precio mayor. Conceptualmente, esta estrategia de negociación puede dividirse en dos tramos, el primero de compra y el segundo de venta. A medida que las marcas de tiempo de estos tramos convergen, el riesgo de exposición del activo se aproxima a cero. Por ejemplo, si ambas negociaciones no son ejecutadas en perfecta sincronía, hay una posibilidad de que la segunda negociación sea completada por otro actor, obligando al arbitrista a mantener un activo hasta que surja otra oportunidad.  

Con el avance de la tecnología, esta estrategia de negociación normalmente es automatizada y puede detectar, así como beneficiarse, de desviaciones de precios en cuestión de segundos. Las oportunidades de arbitraje existen debido a las [ineficiencias del mercado](https://www.investopedia.com/terms/i/inefficientmarket.asp), así como la escasez de liquidez temporal. Como resultado, el arbitraje proporciona un mecanismo para asegurar que los precios en una _exchange_/mercado no se desvíen mucho del [valor justo](https://www.investopedia.com/terms/f/fairvalue.asp). [1](https://www.investopedia.com/terms/a/arbitrage.asp)

#### Estructura del Proceso del Sistema  

![](../.gitbook/assets/arb.jpeg)

**Funcionamiento del _Keeper_ de Arbitraje Simple**

El [simple-arbitrage-keeper ("_Keeper_ de Arbitraje Simple")](https://github.com/makerdao/simple-arbitrage-keeper) es un _Keeper_ de arbitraje para OasisDex y Uniswap. Dada una ganancia mínima por operación y un tamaño máximo de operación, definidos por el usuario y en unidades de `entry_token`, el _Keeper_ buscará oportunidades de arbitraje.  Cuando se ejecuta, monitorea el precio `arb_token/entry_token` en ambos _exchanges_ y cuando es detectada una discrepancia, ejecuta una operación atómica de múltiples pasos en una sola transacción de Ethereum. La operación de múltiples pasos se compone de dos fases: la primera fase es la compra de `arb_token` con el `entry_token` en la `start_exchange`, mientras que la segunda fase es la venta del `arb_token` con el `entry_token` en la `end_exchange`. Los tipos de `entry_token` y `arb_token` normalmente son tipo estable y volátil respectivamente; ya que las oportunidades de arbitraje pueden escasear, el _Keeper_ a veces descansa en estados de _default_, holdeando el `entry_token` por períodos extendidos de tiempo mientras busca nuevas oportunidades. La exposición a un solo _token_ estable es especialmente importante en este estado de _default_. Si cualquier fase de la operación fracasa, toda la operación se revierte, manteniendo así el atributo de "riesgo-libre" del arbitraje verdadero.  

Hereda una estructura del sistema que es consistente con otros _Keepers_ en el Marco del _Keeper_ de Maker. Aparte de las típicas bibliotecas de los Proyectos de Python, se basa en [pymaker](https://github.com/makerdao/pymaker) y [pyexchange](https://github.com/makerdao/pyexchange), ambas siendo APIs de Python áltamente aplicables para los Contratos del Protocolo de Maker y _exchanges_ de criptomonedas, respectivamente. Como se puede ver en el diagrama de flujo, el ‘Lifecycle box’ es una [clase de utilidad pymaker](https://github.com/makerdao/pymaker/blob/master/pymaker/lifecycle.py#L38) que ayuda a definir el ciclo de vida apropiado de un _keeper_; consiste en la _startup_ del _keeper_, temporizadores y/o subscripciones a eventos Web3, y una fase de apagado del _keeper_. La caja ‘Process block’ se ejecuta cuando el nodo del _keeper_ recibe un nuevo bloque. Generalmente, contiene la lógica de consultar el estado de la _blockchain_, evaluar cualquier oportunidad de ganancias y, cuando es aplicable, publicar una transacción.  

**El paso para cada proceso en el _Keeper_ de Arbitraje Simple es de la siguiente forma:**
1. Entrada del _Keeper_ -> entrar en la clase _SimpleArbitrageKeeper_
2. _Startup_ -> hacer todas las aprobaciones necesarias entre _tokens_, _exchanges_ y _tx-manager_
3. Evaluar el Mercado -> extraer datos de precios de la API REST de Oasis y contratos de intercambio Uniswap 
4. ¿Oportunidad de Ganancia Encontrada? -> revisa si existen oportunidades de beneficio con un máximo compromiso y una mínima ganancia 
5. Ejecutar una operación de múltiples pasos -> con el uso de _TxManager_, la compra de un activo en el _exchange_ A y vender en el _exchange_ B con una transacción de Ethereum.  

#### Documentación

La información general de alto nivel y los comentarios para cada clase/método están en el [repositorio GitHub del _Keeper_ de Arbitraje Simple](https://github.com/makerdao/simple-arbitrage-keeper).

#### _Startup_ en Kovan

El uso del bot se puede encontrar en la [sección de Uso](https://github.com/makerdao/simple-arbitrage-keeper#usage) del LÉEME. Aquí hay algunos pasos de preparación de la ejecución en Kovan: 

Para operar el _Keeper_, llama al script `/bin/simple-arbitrage-keeper` con los argumentos requeridos. Esto agrega los módulos _pyexchange_ y _pymaker_ a la ruta python e inicia la clase _SimpleArbitrageKeeper_. Es conveniente escribir un guion _shell_ que pueda ser fácilmente re ejecutado después de los cambios de argumentos, pero antes de empezar, preparemos nuestra dirección _Keeper_ de ethereum, desplegar nuestro _TxManager_, encontrar la dirección de intercambio Uniswap y la dirección _MatchingMarket_.  

**Instalación del _Keeper_**

Este proyecto usa Python 3.6.2. Para poder clonar el proyecto e instalar paquetes de terceros requeridos, ejecuta:

```
$ git clone https://github.com/makerdao/simple-arbitrage-keeper.git
$ cd simple-arbitrage-keeper
$ git submodule update --init --recursive
$ pip3 install -r requirements.txt
```
Para algunos problemas conocidos de Ubuntu y macOS consulta el [LÉEME de pymaker](https://github.com/makerdao/pymaker).

**Instalar _DappHub Toolkit_**

Si estás ejecutando Linux o macOS puedes tomar ventaja de nuestro instalador todo en uno.

```
$ curl https://dapp.tools/install | sh
```

Si estás teniendo problemas con la instalación, dirígete a la sección ['Instalación de Manual'](http://dapp.tools)

**Dirección Ethereum del _Keeper_**

Tu _Keeper_ publicará transacciones a través de esta dirección. Tu dirección de Ethereum necesita ser accesible por [seth](https://github.com/dapphub/dapptools/tree/master/src/seth), para poder desplegar _TxManager_ a través de [dapp.tools](http://dapp.tools). Además, cuando el _Keeper_ es implementado y está en funcionamiento, esta dirección debe contener algo de ETH para los costos de transacción, así como una cantidad de _entry-tokens_ ("tokens de entrada") igual al compromiso máximo fijado por el Usuario. Sigue [estas instrucciones](https://github.com/dapphub/dapptools/tree/master/src/seth#key-management-and-signing) para asegurar que _seth_ pueda ver tus archivos _keystore_ y _passphrase_.  

Si quieres comenzar desde cero, [instala _geth_](https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Mac) y usa `geth account new`, para crear una nueva cuenta, establecer una _passphrase_ ("frase de contraseña") y colocarla en un archivo `<address>.txt` en el directorio `~/Library/Ethereum/passphrase/`; el archivo _keystore_ correspondiente puede encontrarse en el directorio `~/Library/Ethereum/keystore/`. Tanto los archivos de _keystore_ y _passphrase_ se pasarán posteriormente en el _Keeper_ como un argumento. Haz una nota de tu dirección _Keeper_ de ethereum, la localización de tu archivo _keystore_ y la localización de tu archivo de frase de contraseña.  

**Despliegue del _TxManager_**

Un [TxManager](https://github.com/makerdao/tx-manager) debe ser desplegado y enlazado con la dirección _Keeper_ de Ethereum para agrupar múltiples llamadas de contratos en una sola transacción de Ethereum; esto es lo que llamamos transacción atómica, donde todo el conjunto de la llamada es exitoso o el estado no se mantiene afectado. Una vez instalado _DappHub Toolkit_, ejecuta los siguientes comandos para desplegar _TxManager_: 

```
$ git clone https://github.com/makerdao/tx-manager.git
$ cd tx-manager
$ dapp update
$ dapp --use solc:0.4.25 build --extract

$ export SETH_CHAIN=kovan
$ export ETH_FROM=<your Keeper Ethereum Address e.g. 0xABC>
$ export ETH_GAS=2000000
$ dapp create TxManager --password /path/to/passphraseOfAddress.txt

```

El _output_ de `dapp create` es la dirección de tu _TxManager_

**Despliega/Encuentra un nodo de paridad**

Necesitas acceso a un _host_ JSON-RPC que esté conectado a un nodo de paridad. Desafortunadamente, las APIs de Infura no están conectadas en los nodos de paridad, así que necesitarás encontrar acceso a un _endpoint_ remoto (un ejemplo es [https://rivet.cloud/](https://rivet.cloud)) o ejecutar un nodo en tu máquina local. Un ejemplo de un nodo remoto podría ser [https://kovan.sampleparitynode.com:8545](https://kovan.sampleparitynode.com:8545).
 
**Direcciones de Intercambio _Uniswap_**

Necesitarás encontrar la dirección de intercambio _Uniswap_ correspondiente al _entry-token_ ("token de entrada") así como al _arb-token_. Para hacerlo, llama `getExchange(TokenAddress)` al Contrato de fábrica _Uniswap_ para consultar la dirección de intercambio. [Puedes hacerlo aquí](https://kovan.etherscan.io/address/0xD3E51Ef092B2845f10401a0159B2B96e8B6c3D30#readContract). Por ejemplo, puedes usar la dirección del token Kovan WETH y llamar `getExchange(0xd0A1E359811322d97991E03f863a0C30C2cF029C)`

**_Script_ de _Shell_**

Como hemos mencionado antes, hagamos un _script_ de _shell_ para que sea fácil ejecutar este Bot. Por ejemplo, estamos usando _Kovan Sai_ para el _entry-token_  ya que no nos someteremos a la exposición del precio del token _arb_ entre operaciones. Además, para pruebas de _Kovan_, estaremos utilizando el par WETH/DAI, puesto que la liquidez es escasa en las versiones de _Kovan_ de intercambio _MatchingMarket_ y _Uniswap_. Cualquier error `NoneType` es debido a la falta de liquidez en cualquiera de los _exchanges_ y puede ser resuelto una vez que la liquidez es agregada. Como ya hemos mencionado, los argumentos de ganancia mínima y máximo compromiso definen cómo funcionará el _Keeper_. El Máximo Compromiso se refiere a la cantidad de ganancia obtenida de la ejecución del arbitraje; ambos argumentos son dominados en _entry-tokens_ y en _unidades de ether_ (en el próximo _script_, 1 SAI de ganancia mínima sería 1 SAI). Por ejemplo, el siguiente _script_ comerciará con un máximo compromiso de 10 SAI, pero solo si el valor retornado desde la operación resulta en al menos 11 SAI. 

Crea un archivo llamado de la siguiente forma, inserta los argumentos relevantes para tu entorno, hazlo ejecutable y córrelo. 


```
$ vim run-simple-keeper-kovan.sh
```

En tu archivo run-simple-keeper-kovan.sh:

```
#!/bin/bash
/full/path/to/githubClone/simple-arbitrage-keeper/bin/simple-arbitrage-keeper \
	--rpc-host 'kovan.sampleparitynode.com' \
	--eth-from '0xABC' \
	--eth-key 'key_file=/path/to/keystore.file,pass_file=/path/to/passphrase.txt' \
	--uniswap-entry-exchange '0x47D4Af3BBaEC0dE4dba5F44ae8Ed2761977D32d6' \
	--uniswap-arb-exchange '0x1D79BcC198281C5F9B52bf24F671437BaDd3a688' \
	--oasis-address '0x4A6bC4e803c62081ffEbCc8d227B5a87a58f1F8F' \
	--oasis-api-endpoint 'https://kovan-api.oasisdex.com' \
	--tx-manager '0xABC' \
	--entry-token '0xC4375B7De8af5a38a93548eb8453a498222C4fF2' \
	--arb-token '0xd0A1E359811322d97991E03f863a0C30C2cF029C' \
	--arb-token-name 'WETH' \
	--min-profit 1 \
	--max-engagement 10 \
```

Haz tu _Keeper_ ejecutable y córrelo

```
$ chmod +x run-simple-keeper-kovan.sh
$ ./run-simple-keeper-kovan.sh

```

**_Startup_ Inicial**

Durante una _startup_ inicial, se enviarán varias transacciones aprobadas al _tx-manager_ y cambios importantes. Cuando se complete, verás un mensaje: _“Watching for new blocks”_ (Buscando nuevos bloques) que indica que el bot está en bucle a través del método `lifecycle.process_block()`; siguiendo ese mensaje y con cada nuevo bloque, un mensaje de "Best Trade regardless of profit" (El mejor comercio independientemente de la ganancia) actúa como un "heartbeat" (latido) para el _Keeper_.  

```
Kentons-Macbook:scripts kentonprescott$ ./run-simple-arbitrage-keeper-mainnet-XYZ-XYZ.sh
2019-10-19 17:21:57,463 INFO     Keeper connected to RPC connection https://...
2019-10-19 17:21:57,463 INFO     Keeper operating as 0xABCD
2019-10-19 17:21:58,523 INFO     Executing keeper startup logic
2019-10-19 17:22:04,820 INFO     Watching for new blocks
2019-10-19 17:22:13,983 INFO     Best trade regardless of profit/min-profit: -1.243077085275947008 DAI from Uniswap to Oasis
```

**Operación Nominal**

Después de la fase de aprobación inicial, el _startup_ del _Keeper_ se verá como el siguiente _output_ y proseguirá a imprimir mensajes "Best Trade ... "  (Mejor comercio) con cada nuevo bloque presenciado. Cuando el _Keeper_ publica una transacción, toda la información relevante estará registrada e impresa en la consola. Finalmente, cuando quieras apagar el _Keeper_ simplemente presiona CTRL-C y se apagará correctamente.\

### _Troubleshooting_

Durante la operación del _Keeper_, si se muestra el siguiente error, solo presiona CTRL-C y re ejecuta: 

```
ValueError: {'code': -32010, 'message': 'Transaction with the same hash was already imported.'}
```

### Siguientes Pasos

#### Mejoras 

Aquí hay algunas sugerencias para mejorar la usabilidad, versatilidad y rentabilidad de este _Keeper_. Esperamos que todos jueguen con los parámetros, ejecuten al _Keeper_ para su propio beneficio y eventualmente actualicen el bot para satisfacer su tolerancia al riesgo y su apetito de ganancias. 

* Desarrolla unidades de prueba para cada método 
* Usa el Contrato de Fabricación _Uniswap_ para consultar la dirección _uniswap-entry-exchange_ y _uniswap-arb-exchange_ en lugar de requerirlo como un argumento
* En lugar de implementar el precio del gas por _default_, implementar una [estrategia de precios dinámicos del gas](https://github.com/makerdao/market-maker-keeper/blob/master/market\_maker\_keeper/gas.py#L24)
* Aumenta el alcance 
  * Supervisa más que un par 
  * Supervisa más que dos _exchanges_ descentralizados
  * Implementa apoyo para _exchanges_ centralizados
  * Aumenta el número intermediario de _arb tokens_ 
    * (es decir DAI → WETH → BAT → DAI)
    * [https://math.stackexchange.com/a/94420](https://math.stackexchange.com/a/94420)
    * [https://www.dailycodingproblem.com/blog/how-to-find-arbitrage-opportunities-in-python/](https://www.dailycodingproblem.com/blog/how-to-find-arbitrage-opportunities-in-python/)
* Mejora la eficiencia de _TxManager_
  * ¿Realmente necesitamos enviar todo nuestro saldo token al contrato durante cada transacción atómica? 
 * Lee el estado del contrato _MatchingMarket (OasisDex)_ en lugar de usar la API REST de Oasis  
  * Para ahorrar gas, [obtener todas las ordenes activas](https://github.com/makerdao/pymaker/blob/master/pymaker/oasis.py#L613) y [tomar ordenes especificas](https://github.com/makerdao/pymaker/blob/master/pymaker/oasis.py#L428) por ID en lugar de usar `MatchingMarket.offer(...)` y  el motor de emparejamiento _on-chain_
* Envía actualizaciones comerciales a través de una API Python de texto/email  

### Recursos

* [https://github.com/makerdao/market-maker-keeper](https://github.com/makerdao/market-maker-keeper)
* [https://github.com/makerdao/arbitrage-keeper](https://github.com/makerdao/arbitrage-keeper)
* [https://github.com/makerdao/tx-manager](https://github.com/makerdao/tx-manager)
* [https://github.com/makerdao/pymaker](https://github.com/makerdao/pymaker)
* [https://github.com/makerdao/pyexchange](https://github.com/makerdao/pyexchange)
