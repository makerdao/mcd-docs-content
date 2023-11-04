# Guía de Configuración del Bot _Keeper_ de Subastas

**Nivel:** Intermedio

**Tiempo Estimado:** 60 minutos

**Audiencia:** Desarrolladores

## Descripción

El Protocolo de Maker, que potencia al DAI Multicolateral (MCD), es un sistema basado en _smart contracts_ (contratos inteligentes) que respalda y estabiliza el valor de DAI a través de una combinación dinámica de _vaults_ (anteriormente conocidas como CDPs), mecanismos de retroalimentación autónoma y actores externos incentivados. Para mantener el sistema en un estado financiero estable, es importante evitar que tanto deudas como excedentes se acumulen más allá de ciertos límites. Aquí es donde entran en juego las Subastas y los _Keepers_ de Subastas. El sistema ha sido diseñado para que hayan tres tipos de Subastas en el sistema: Subastas de Excedentes, Subastas de Deuda y Subastas de Colaterales. Cada subasta es activada como resultado de circunstancias específicas.

Los _Keepers_ de Subastas son actores externos que están incentivados por oportunidades de beneficios para contribuir con sistemas descentralizados. En el contexto del Protocolo de Maker, estos agentes externos son incentivados para automatizar ciertas operaciones alrededor de la _Blockchain_ de Ethereum. Esto incluye:  

* Buscar nuevas oportunidades y comenzar nuevas subastas
* Detectar subastas iniciadas por otros participantes
* Ofertar en subastas, convirtiendo los precios de los tokens en ofertas

Más concretamente, los _Keepers_ participan como postores en las Subastas de Deuda y Colateral cuando las _vaults_ son liquidadas y el  _Keeper_ de Subasta permite la interacción automática con estas subastas MCD. Este proceso se automatiza mediante modelos de oferta específicos que definen el proceso de toma de decisiones, como en qué situaciones presentar una oferta, con qué frecuencia ofertar, qué tan alto ofertar, etc. Ten en cuenta que los modelos de oferta son creados a base de estrategias determinadas individualmente.

### Objetivos de Aprendizaje

El propósito de esta guía es proporcionar un tutorial sobre cómo utilizar `auction-keeper` e interactuar con un despliegue de Kovan de los _smart contracts_ del DAI Multicolateral (MCD). Más específicamente, la guía mostrará cómo configurar y ejecutar un bot _Keeper_ de Subasta tú mismo. Después de pasar por esta guía, lograrás lo siguiente:  

* Aprender sobre _Keepers_ de Subastas y cómo interactúan con el Protocolo de Maker
* Entender modelos de ofertas
* Conseguir que tu propio bot _Keeper_ de Subastas funcione en la red de pruebas de Kovan

### Agenda de la Guía

Esta guía te mostrará cómo utilizar el _Keeper_ de subastas para interactuar con el despliegue de Kovan de los _smart contracts_ MCD. Más específicamente, la guía mostrará cómo pasar a través de los siguientes estadios de configuración y ejecución de un bot _Keeper_ de Subastas:

1. Introducción
2. Modelos de Ofertas
   * Iniciar y detener modelos de ofertas
   * Comunicación con los modelos de ofertas
3. Configuración del Bot _Keeper_ (_Keeper_ de Subasta Flip)
   * Requisitos Previos
   * Instalación
4. Ejecutar tu propio Bot _Keeper_ (Uso)
   * Limitaciones del _Keeper_
5. Contabilidad
   * Obtención de K-DAI MCD
   * Obtención de K-MKR MCD
   * Obtención de Tokens de Colateral MCD
6. Pruebas
7. Soporte

Estamos orgullosos de decir que, dado que el Protocolo de Maker es una plataforma de código abierto, todo el código que hemos creado para ejecutar el bot _Keeper_ es de libre acceso para todos.

## 1. Introducción

Los _Keepers_ de Subastas participan en subastas como resultado de eventos de liquidación y así adquieren colateral a precios atractivos. Un `auction-keeper` puede participar en tres tipos diferentes de subastas:  
1. [Subasta de Colateral (`flip`)](https://github.com/makerdao/dss/blob/master/src/flip.sol)
2. [Subasta de Excedentes (`flap`)](https://github.com/makerdao/dss/blob/master/src/flap.sol)
3. [Subasta de Deuda (`flop`)](https://github.com/makerdao/dss/blob/master/src/flop.sol)

Los _Keepers_ de Subastas tienen la capacidad única de conectar _bidding models_ ("modelos de ofertas" o "pujas") externos, que transmiten información al  _Keeper_ sobre cuándo o qué tan alto ofertar (estos tipos de _Keepers_ pueden dejarse ejecutando de forma segura en el fondo). Poco después de que un _Keeper_ de Subastas advierta o inicie una nueva subasta, generará una nueva instancia de un _modelo de ofertas_ y actuará de acuerdo a sus instrucciones específicas. Los modelos de ofertas serán terminados automáticamente por los _Keepers_ de Subastas en el momento que la subasta expire.

**Nota:**

Los _Keepers_ de Subastas automáticamente llamarán `deal` (reclamando una oferta ganadora / liquidando una subasta completa) si la dirección del _Keeper_ gana la subasta.

### Arquitectura del _Keeper_ de Subastas

Como se mencionó anteriormente, los _Keepers_ de Subastas interactúan directamente con los contratos de subastas `Flipper`, `Flapper` y `Flopper` desplegados en la red principal de Ethereum. Todas las decisiones que involucran detalles de precios son delegadas a los _bidding models_ ("modelos de ofertas" o "pujas"). Los modelos de pruebas son simplemente estrategias ejecutables, externas al proceso principal del `auction-keeper` ("_Keepers_ de subastas"). Esto significa que los modelos de ofertas por sí mismos no tienen que saber nada sobre la _blockchain_ de Ethereum ni sus _smart contracts_, ya que pueden ser implementados básicamente en cualquier lenguaje de programación. Sin embargo, necesitan tener la habilidad de leer y escribir documentos JSON, ya que así es como se comunican/intercambian con `auction-keeper`.  Es importante tener en cuenta que como un desarrollador que ejecuta un _Keeper_ de Subastas, es necesario tener un conocimiento básico sobre cómo iniciar y configurar apropiadamente el _Keeper_ de subastas. Por ejemplo, para configurar y ejecutar un _Keeper_ es necesario proporcionar parámetros de inicio como keystore / contraseñas.  Adicionalmente, deberías estar familiarizado con el sistema MCD, ya que el modelo recibirá los detalles de subastas del _keeper_ de subastas en la forma de una mensaje _JSON_ conteniendo llaves como _lot, beg, guy,_ etc.

**Ejemplo de Modelo de Oferta Simple:**

Un modelo de oferta simple podría ser un _script_ de _shell_ que se hace eco de un precio fijo (más detalles a continuación).

### El Propósito de los _Keepers_ de Subastas

**El propósito principal de los _Keepers_ de Subastas:**

* Para descubrir nuevas oportunidades y comenzar nuevas subastas.
* Para monitorear constantemente todas las subastas en curso.
* Para detectar subastas iniciadas por otros participantes.
* Para ofertar en subastas, convirtiendo los precios de _tokens_ en ofertas.  
* Para asegurar que las instancias del _bidding model_ se estén ejecutando para cada tipo de subasta así como asegurar que las instancias coincidan con el estado actual de sus subastas. Esto asegura que los _Keepers_ estén ofertando de acuerdo con las decisiones señaladas por el modelo de ofertas.

Los mecanismos de descubrimiento y monitorización funcionan operando como un bucle, que inicia en cada nuevo bloque y enumera todas las subastas desde `1` a `kicks`. Cuando esto ocurre, incluso cuando el _bidding model_ decide enviar una oferta, no será procesada por el _Keeper_ hasta la nueva iteración de ese bucle. Es importante tener en cuenta que el `auction-keeper` no solo monitorea las subastas existentes y descubre nuevas, sino que también identifican y toman oportunidades de crear nuevas subastas.  

## 2. Modelos de Ofertas

###  Iniciar y Detener modelos de ofertas

Los _Keepers_ de Subastas mantienen una colección de procesos hijos, ya que cada _bidding model_ es su propio proceso dedicado. Nuevos procesos (nuevas instancias del _bidding model_) son generados al ejecutar un comando de acuerdo con el parámetro _command-line_ `--model`. Estos procesos son terminados automáticamente (mediante `SIGKILL`) por el _keeper_ poco después de que sus subastas asociadas expiren. Cada vez que el proceso de _bidding model_ muere, reaparece automáticamente gracias al _Keeper_.    

**Ejemplo:**

```
bin/auction-keeper --model '../my-bidding-model.sh' [...]
```

### **Comunicaciones con** _**bidding models**_

Los _Keepers_ de Subastas se comunican con _bidding models_ mediante su entrada estándar/salida estándar. Una vez que el proceso ha iniciado y cada vez que el estado de la subasta cambia, el _Keeper_ envía un documento _JSON one-line_ a la **entrada estándar** del _bidding model_.

** Un ejemplo de mensaje _JSON_ enviado desde el _keeper_ al modelo que luce como:**

```
{"id": "6", "flapper": " 0xf0afc3108bb8f196cf8d076c8c4877a4c53d4e7c ", "bid": "7.142857142857142857", "lot": "10000.000000000000000000", "beg": "1.050000000000000000", "guy": " 0x00531a10c4fbd906313768d277585292aa7c923a ", "era": 1530530620, "tic": 1530541420, "end": 1531135256, "price": "1400.000000000000000028"}
```

### Glosario (Modelos de Ofertas):

* `id` - identificador de la subasta.
* `flipper` - dirección de Ethereum del contrato `Flipper` (solo para subastas `flip`).
* `flapper` - dirección de Ethereum del contrato `Flapper` (solo para subastas `flap`).
* `flopper` - dirección de Ethereum del contrato `Flopper` (solo para subastas `flop`).
* `bid` - la oferta más alta actualmente (subirá por las subastas `flip` y `flap`).
* `lot` - cantidad siendo subastada actualmente (bajará por las subastas `flip` y `flop`).
* `tab` - valor de la oferta (no debe confundirse con el precio de la oferta) que causará que la subasta entre en la fase `dent` (solo para subastas `flip`).
* `beg` - incremento mínimo del precio (`1.05` significa un incremento mínimo del 5% del precio).
* `guy` - dirección Ethereum del mejor postor actualmente.
* `era` - tiempo en curso (en segundo desde la _UNIX epoch_).
* `tic` - tiempo en que expirará la oferta actual (`None` si aún no hay ofertas).
* `end` - tiempo en que expirará la oferta entera (_end_ es fijado a `0` si la subasta ya no está en vivo).
* `price` - precio actual siendo ofrecido (puede ser `None` si el precio es infinito).

Los _Bidding models_ no deberían asumir nunca que ese mensaje estará fijado solo cuando cambie el estado de la subasta. Está perfectamente bien para el `auction-keeper` enviar periódicamente el mismo mensaje(s) a los _bidding models_.

Al mismo tiempo, el `auction-keeper` lee mensajes _one-line_ del **standard output** del proceso del _bidding model_  y trata de analizarlos como documentos _JSON_. Después extraerá los dos campos siguientes de ese documento:

* `price` - el precio máximo (para subastas `flip` y `flop`) o el precio mínimo (para subastas `flap`) que el modelo está dispuesto a ofertar.
* `gasPrice` (opcional) - el precio del gas en Wei para usar cuando se envía una oferta.  

**Ejemplo de un mensaje enviado desde el Modelo de Oferta al _Keeper_ de Subasta, puede lucir así:**

```
    {"price": "150.0", "gasPrice": 7000000000}
```

En el caso de que se comuniquen los _Keepers_ de Subastas y los Modelos de Ofertas en términos de precios, es el precio del MKR/DAI (para subastas `flap` y `flop`) o el precio del colateral expresado en DAI para las subastas `flip` (por ejemplo, OMG/DAI).

Cualquier mensaje escrito por un Modelo Actual para **stderr** (error estándar) será transmitido por el _Keeper_ de la Subasta a sus registros. Esta es la forma más conveniente de implementar el registro desde los Modelos de Subastas.

## 3. Configuración del Bot _Keeper_ de Subastas (Instalación)

#### Requisitos Previos

* Git
* [Python v3.6.6](https://www.python.org/downloads/release/python-366/)
* [virtualenv](https://virtualenv.pypa.io/en/latest/)
  * Este proyecto requiere instalar _virtualenv_ si deseas usar las herramientas python de Maker. Esto ayuda a asegurarte de que estés ejecutando la versión correcta de python así como revisar que todos los paquetes _pip_ que son instalados en el [install.sh](http://install.sh) estén en el lugar correcto y tengan las versiones correctas. 
* [X-code](https://apps.apple.com/ca/app/xcode/id497799835?mt=12) (para Macs)
* [Docker-Compose](https://docs.docker.com/compose/install/)

### Comenzando

#### Instalación desde el código fuente:

1. **Clona el repositorio `auction-keeper` :**

```
git clone https://github.com/makerdao/auction-keeper.git
```

1. **Cambia en el directorio `auction-keeper` :**

```
cd auction-keeper
```

1. **Instala los paquetes de terceros requeridos:**

```
git submodule update --init --recursive
```

1. **Configura el entorno virtual y actívalo:**

```
python3 -m venv _virtualenv
source _virtualenv/bin/activate
```

**5. Requisitos de Instalación:**

```
pip3 install -r requirements.txt
```

#### Errores Potenciales:

* Necesita actualizar la versión pip a 19.2.2:
  * Arreglalo al ejecutar `pip install --upgrade pip`.

Para otros errores comunes de Ubuntu y macOS consulta el _README_ de [pymaker](https://github.com/makerdao/pymaker).

## 4. Ejecuta tu Bot _Keeper_ 

#### La versión Kovan corre en el [Kovan Release 1.0.2](https://changelog.makerdao.com/releases/kovan/1.0.2/index.html)

Para cambiar la versión que elegiste del lanzamiento Kovan, copia/pega la dirección del contrato que prefieras en `kovan-addresses.json` en `lib/pymaker/config/kovan-addresses.json`

#### 1. Crear tu modelo de ofertas (un ejemplo detallando el modelo de ofertas más simple posible)

El _stdout_ (_output_ estándar) proporciona un precio para el colateral (para subastas `flip`) o MKR (para subastas `flap` y `flop`). El `sleep` bloquea el precio durante un minuto, después del cual _keeper_ reiniciará el modelo de precio y leerá un nuevo precio (considera esto tu intervalo de actualización de precios).

El _bidding model_ más simple que puedes establecer es cuando usas un precio fijo para cada subasta. Por ejemplo:

```
    #!/usr/bin/env bash
    echo "{\"price\": \"150.0\"}" # put your desired fixed price amount here
    sleep 60 # locking the price for a 60 seconds period
```

Una vez que has creado el modelo de ofertas, guárdalo como `model-eth.sh` (o cualquier nombre que consideres apropiado). 

#### 2. Configurando un _Keeper_ de Subastas para una Subasta de Colateral (Flip)  

Las Subastas de Colateral serán los tipos de subastas más comunes para las que la comunidad querrá crear y operar _Keepers_ de Subastas. Esto es debido al hecho de que las Subastas de Colateral ocurrirán más frecuentemente que las subastas Flap y Flop.   

**Ejemplo (_Keeper_ de Subasta Flip):**

* Este ejemplo/proceso asume que el usuario ya tiene un _script_ de _shell_ existente que gestiona su entorno y conecta con la blockchain de Ethereum y que tienes algo de DAI y ETH de Kovan en tu wallet. Si no tienes saldo alguno, revisa la sección de arriba sobre cómo obtenerlo.   

Un ejemplo de cómo establecer tu entorno: `my_environment.sh`

```
SERVER_ETH_RPC_HOST=https://your-ethereum-node
SERVER_ETH_RPC_PORT=8545
ACCOUNT_ADDRESS=0x16Fb96a5f-your-eth-address-70231c8154saf
ACCOUNT_KEY="key_file=/Users/username/Documents/Keeper/accounts/keystore,pass_file=/Users/username/Documents/keeper/accounts/pass"
```

`SERVER_ETH_RPC_HOST` - No debe ser un nodo infura, ya que no proporciona toda la funcionalidad que el _script_ de python necesita.

`ACCOUNT_KEY` - Debe tener la ruta absoluta al archivo _keystore_ y contraseña. Define la ruta como se muestra arriba, ya que el _script_ de Python analizará tanto a los archivos _keystore_ como la contraseña.

```
#!/bin/bash
dir="$(dirname "$0")"

source my_environment.sh  # Set the RPC host, account address, and keys.
source _virtualenv/bin/activate # Run virtual environment

# Allows keepers to bid different prices
MODEL=$1

bin/auction-keeper \
    --rpc-host ${SERVER_ETH_RPC_HOST:?} \
    --rpc-port ${SERVER_ETH_RPC_PORT?:} \
    --rpc-timeout 30 \
    --eth-from ${ACCOUNT_ADDRESS?:} \
    --eth-key ${ACCOUNT_KEY?:} \
    --type flip \
    --ilk ETH-A \
    --from-block 14764534 \
    --vat-dai-target 1000 \
    --model ${dir}/${MODEL} \
    2> >(tee -a -i auction-keeper-flip-ETH-A.log >&2)
```

Una vez finalizada, deberías guardar tu _script_ para correr tu _Keeper_ de Subasta como `flip-eth-a.sh` (o algo similar para identificar que este _Keeper_ de Subastas es para una Subasta Flip). Adicionalmente, asegúrate de verificar que el _script_ de arriba que copiaste+pegaste no genere espacios o caracteres extras al pegar+guardar en tu editor. De lo contrario, notarás un error cuando lo ejecutes más adelante. 

**¡Nota importante sobre Ejecutar los _Keepers_ de Subastas en la _Mainnet_ de Ethereum!**

* Si llegas al punto donde el bot _Keeper_ de subastas no está aceptando la _mainnet_ como un argumento válido, esto es porque no hay parámetros `network`. Para arreglar esto, solo omite ese parámetro  

**Otras Notas:**

* Todos los tipos de Colateral (`ilk`'s) combinan el nombre del token y una letra correspondiente a un conjunto de parámetros de riesgo. Por ejemplo, como puedes ver arriba, el ejemplo usa ETH-A. Ten en cuanta que ETH-A y ETH-B son dos tipos de colaterales diferentes para el mismo token subyacente (WETH) pero tienen diferentes parámetros de riesgos. 
* Para direcciones MCD, simplemente pasamos `--network mainnet|kovan` y recargará los archivos JSON requeridos agrupados dentro de _auction-keeper_ (o _pymaker_).

#### 3. Pasando el modelo de ofertas como un argumento al _script_ _Keeper_  

1. Confirma que tanto tu modelo de ofertas (model-eth.sh) como tu _script_ (flip-eth-a.sh) para ejecutar tu _Keeper_ de Subasta están guardados.
2. El próximo paso es `chmod +x` ambos.
3. Por último, ejecuta `flip-eth-a.sh model-eth.sh` para pasar tu modelo de ofertas en tu _script_ de _Keeper_ de Subastas.  

Ejemplo de un _keeper_ trabajando:\

Después de ejecutar el comando `./flip-eth-a.sh model-eth.sh` verás un _output_ como este: 

```
019-10-31 13:33:08,703 INFO     Keeper connected to RPC connection https://parity0.kovan.makerfoundation.com:8545
2019-10-31 13:33:08,703 INFO     Keeper operating as 0x16Fb96a5fa0427Af0C8F7cF1eB4870231c8154B6
2019-10-31 13:33:09,044 INFO     Executing keeper startup logic
2019-10-31 13:33:09,923 INFO     Sent transaction DSToken('0x1D7e3a1A65a367db1D1D3F51A54aC01a2c4C92ff').approve(address,uint256)('0x9E0d5a6a836a6C323Cf45Eb07Cb40CFc81664eec', 115792089237316195423570985008687907853269984665640564039457584007913129639935) with nonce=1257, gas=125158, gas_price=default (tx_hash=0xc935e3a95e5d0839e703dd69b6cb2d8f9a9d3d5cd34571259e36e771ce2201b7)
2019-10-31 13:33:12,964 INFO     Transaction DSToken('0x1D7e3a1A65a367db1D1D3F51A54aC01a2c4C92ff').approve(address,uint256)('0x9E0d5a6a836a6C323Cf45Eb07Cb40CFc81664eec', 115792089237316195423570985008687907853269984665640564039457584007913129639935) was successful (tx_hash=0xc935e3a95e5d0839e703dd69b6cb2d8f9a9d3d5cd34571259e36e771ce2201b7)
2019-10-31 13:33:13,152 WARNING  Insufficient balance to maintain Dai target; joining 91.319080635247876480 Dai to the Vat
2019-10-31 13:33:13,751 INFO     Sent transaction <pymaker.dss.DaiJoin object at 0x7fa6e91baf28>.join('0x16Fb96a5fa0427Af0C8F7cF1eB4870231c8154B6', 91319080635247876480) with nonce=1258, gas=165404, gas_price=default (tx_hash=0xcce12af8d27f9d6185db4b359b8f3216ee783250a1f3b3921256efabb63e22b0)
2019-10-31 13:33:16,491 INFO     Transaction <pymaker.dss.DaiJoin object at 0x7fa6e91baf28>.join('0x16Fb96a5fa0427Af0C8F7cF1eB4870231c8154B6', 91319080635247876480) was successful (tx_hash=0xcce12af8d27f9d6185db4b359b8f3216ee783250a1f3b3921256efabb63e22b0)
2019-10-31 13:33:16,585 INFO     Dai token balance: 0.000000000000000000, Vat balance: 91.319080635247876480133691494546726938904901298
2019-10-31 13:33:16,586 INFO     Watching for new blocks
2019-10-31 13:33:16,587 INFO     Started 1 timer(s)
```

Ahora el _keeper_ escucha activamente cualquier acción. Si ve una posición subcolateralizada, después intentará ofertar por ella. 

#### Explicación de los Argumentos del _Keeper_ de la Subasta  

Para participar en todas las subastas, debe configurarse un _keeper_ de subastas `flip` para cada tipo de colateral, así como uno para `flap` y otro para `flop`.

1. `--type` - el tipo de _keeper_ de subasta utilizado. En este escenario en particular, se configurará como `flip`.
2. `--ilk` - el tipo de colateral.
3. `--addresses` - _.json_ de todas las direcciones de los contratos MCD así como los tipos de colaterales permitidos/utilizados en el sistema.   
4. `--vat-dai-target` - la cantidad de DAI que el _keeper_ intentará mantener en el _Vat_, para utilizarlo al ofertar. Se reequilibrará al iniciar el _keeper_ y _`deal`ing_ ("negociará") una subasta. 
5. `--model` - el modelo de ofertas que será utilizado para ofertar. 
6. `--from-block` al bloque donde la primera _urn_ fue creada para indicar al _keeper_ el uso de registros publicados por el contrato _vat_ para construir una lista de _urns_ y luego revisar el estado de cada _urn_.

Llama `bin/auction-keeper --help` para una lista completa de argumentos.

### Limitaciones del _Keeper_ de Subastas

* Si una subasta comienza antes de que el _Keeper_ de Subastas haya iniciado, el _Keeper_ no participara en la subasta  hasta que el próximo bloque haya sido acuñado.
* Los _Keepers_ no manejan explícitamente la liquidación global (`End`). Si la liquidación global ocurre mientras una oferta ganadora está pendiente, el _Keeper_ no requerirá un `yank` para reembolsar la oferta. La solución es llamar a `yank` directamente utilizando `seth`. 
* Hay algunas funciones de los _Keepers_ que incurren en tarifas de gas independientemente de si representa una oferta. Esto incluye, pero no está limitada a, las siguientes acciones: 

  * Presentar aprobaciones.
  * Ajustar el balance del excedente a la deuda. 
  * Ofertar un CDP o iniciar una subasta flap o flop, incluso si existen fondos insuficientes para participar en la subasta. 
* El _Keeper_ no revisará modelos de precios hasta que exista una subasta oficialmente. Por lo tanto, procederá a `kick`, `flap` o `flop` en respuesta a las oportunidades independientemente de si tu saldo en DAI o MKR sea suficiente para participar. Esto impone una tarifa de gas que debe ser pagada.    
  * Después de conseguir mas DAI, el _Keeper_ debe ser reiniciado para añadirlo al `Vat`.

## 5. Contabilidad 

Los contratos de Subastas interactúan exclusivamente con DAI (para todos los tipos de subastas) y colateral (para subastas `flip`) en el `Vat`. Hablando más explícitamente:    

* El DAI que es utilizado para ofertar en subastas es retirado del `Vat`.
* El Colateral y DAI excedente ganado al final de la subasta se coloca en el `Vat`.

Por defecto, todo el DAI y colateral dentro de tu cuenta `eth-from` es _`exit`'ed_ del _Vat_ y agregado al saldo de tokens de tu cuenta cuando el _Keeper_ es apagado. Ten en cuenta que esta función puede desactivarse usando los interruptores `keep-dai-in-vat-on-exit` y `keep-gem-in-vat-on-exit`, respectivamente. Se desaconseja el uso de una cuenta `eth-from` con un CDP abierto, ya que la deuda obstaculizará la capacidad de los contratos de subastas para acceder a tus DAI y la capacidad del `auction-keeper` para `exit` DAI del `Vat`.
 
Cuando se ejecutan múltiples _Keepers_ de Subastas utilizando la misma cuenta, el balance de DAI en el `Vat`  se compartirá a través de los _Keepers_. Si se utiliza esta función, debes fijar `--vat-dai-target` al mismo valor para cada _Keeper_, así como un valor lo suficientemente alto para poder cubrir la exposición total deseada.     

**Nota:**

El MKR usado para ofertar en subastas `flap` es directamente retirado de tu balance de tokens. El MKR ganado en la subastas `flop` es directamente depositado en tu balance de tokens. 

### Obteniendo MCD DAI de Kovan, MKR y otros tokens colaterales

#### 1. Obteniendo MCD K-DAI (versión K-MCD 0.2.12)

**Dirección del contrato**: `0xb64964e9c0b658aa7b448cdbddfcdccab26cc584`

1. Accede a tu cuenta en Metamask desde la extensión del _browser_. Agrega o confirma que el token MCD K-DAI personalizado sea añadido a tu lista de tokens.    
   * Esto se hace seleccionando _"Add Token"_ y después añadiendo los detalles bajo la opción _"Custom token"_ ("Token Personalizado").  
2. Cabeza de _Oasis Borrow_ [aquí](https://oasis.app/borrow/?network=kovan).
   * Confirma que de hecho tu estas en la Red Kovan antes de proceder.
3. Conecta tu cuenta de Metamask. 
4. Aprueba la conexión de MetaMask.
5. Debajo del botón _"Overview"_, encuentra y selecciona el botón del signo más para comenzar a configurar tu CDP.
6. Selecciona el tipo de colateral que desee y haga click en _"Continue"_.
   * por ejemplo: ETH-A
7. Deposita tu K-ETH y genera K-DAI seleccionando e introduciendo una cantidad de K-ETH y la cantidad de K-DAI que desea generar. Para proceder, haga click en _"Continue"_.
   * por ejemplo: Deposita 0.5 K-ETH y genera 100 DAI.
8. Haga click en la casilla de verificación para confirmar que has leído y aceptado los **Términos del Servicio**, después haz click en el botón _"Create CDP"_ ("Crear CDP").    
9. Aprueba la transacción en tu extensión MetaMask. 
10. Haz click en el botón _"Exit"_ y espera a que ser cree tu CDP.

Una vez completados todos estos pasos, tendrás el MCD K-DAI generado y estará presente dentro de tu _wallet_. Puedes fácilmente devolver tu DAI o generar mas. 

#### 2. Obteniendo el MCD K-MKR (Versión K-MCD 1.0.2)

**Dirección del Contrato:** `0xaaf64bfcc32d0f15873a02163e7e500671a4ffcd`

Esto requiere estar familiarizado con _Seth_ así como tener la herramienta configurada en tu maquina local. Si no te es familiar, utiliza [esta guía](https://github.com/makerdao/developerguides/blob/master/devtools/seth/seth-guide-01/seth-guide-01.md) para instalarla y configurarla.

**Ejecuta el siguiente comando en _Seth_:**

```
seth send 0xcbd3e165ce589657fefd2d38ad6b6596a1f734f6 'gulp(address)' 0xaaf64bfcc32d0f15873a02163e7e500671a4ffcd
```

**Información de la dirección:**

* La dirección `0x94598157fcf0715c3bc9b4a35450cce82ac57b20` es el grifo que emite 1 MKR por petición.
* La dirección `0xaaf64bfcc32d0f15873a02163e7e500671a4ffcd` es la del token MCD K-MKR. Emitirá 1 MKR.

**Nota Importante:** la dirección del grifo y las direcciones del token suelen cambiar con cada despliegue del _dss_. Las direcciones actuales desplegadas arriba vienen de la **Versión  0.2.12**. Visita [https://changelog.makerdao.com/](https://changelog.makerdao.com) para la versión mas actualizada.  

Por favor refiere esta [guía](https://github.com/makerdao/developerguides/blob/master/mcd/mcd-seth/mcd-seth-01.md#getting-tokens) para obtener tokens de prueba de colateral para Kovan. 

#### 3. Obteniendo Tokens de Colaterales MCD 

## 6. Probando tu _Keeper_

Para ayudar a probar tu _Keeper_ de Subastas, hemos creado una colección de _scripts de python y _shell_ que pueden ser utilizados para probar `auction-keeper`, facilidades de subastas `pymaker` y _smart contracts_ relevantes en `dss`. Para más información sobre cómo probar tu _Keeper_ de Subastas con tu propia _testchain_ ("cadena de prueba") consulta [tests/manual/README](https://github.com/makerdao/auction-keeper/blob/master/tests/manual/README.md).

## 7. Soporte

Cualquier duda o pregunta acerca de los _Keepers_ de Subastas son bienvenidas en el canal [#keeper](https://chat.makerdao.com/channel/keeper) del chat de Maker. 
