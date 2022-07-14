# Guía de Configuración del _Market Maker Keeper_

**Nivel:** Intermedio

**Tiempo Estimado:** 60 minutos

**Audiencia:** Desarrolladores 

## Agenda de Guía

1. Introducción
2. Requisitos Previos
3. Instalación 
4. Prueba
5. Bandas y Configuración de Bandas
   1. Ejemplo
6. Limitación de Tasa de Pedidos  
7. Lenguaje de Plantilla de Datos  
8. Configuración de Alimentación de Precios
   1. Ejemplo
9. Ejecución de los _Keepers_
   1. Ejemplo \(Keeper Creador de Mercado de Oasis\)
10. Información de Soporte

## 1. Introducción

Esta guía está dedicada a mostrarte cómo crear tu propio _Market Maker Keeper Bot_ así como educar a la comunidad acerca estos Bot _Keepers_ y ayudar tanto a los usuarios como a los desarrolladores a entender el valor de este increíble _software_. Estamos orgullosos de decir que todo el código necesario para tener tu _Market Maker Keeper Bot_ y ejecutarlo, es código abierto.

### Lista de _exchanges_ actuales para las que se pueden construir los _Market Maker Keeper Bots_

* OasisDEX \(`oasis-market-maker-keeper`\)
* EtherDelta \(`etherdelta-market-maker-keeper`\)
* RadarRelay y ERCdEX \(`0x-market-maker-keeper`\)
* Paradex \(`paradex-market-maker-keeper`\)
* DDEX \(`ddex-market-maker-keeper`\)
* Ethfinex \(`ethfinex-market-maker-keeper`\)
* GoPax \(`gopax-market-maker-keeper`\)
* OKEX \(`okex-market-maker-keeper`\)
* TheOcean \(`theocean-market-maker-keeper`\)

## 2. Requisitos Previos

* Git
* [Python v3.6.6](https://www.python.org/downloads/release/python-366/)
* [virtualenv](https://virtualenv.pypa.io/en/latest/)
  * Este proyecto requiere que se instales _virtualenv_ para utilizar las herramientas de Python de Maker. Esto ayuda a asegurarte de que estás ejecutando la versión correcta de python y revisar que todos los paquetes _pip_ que se instalaron en el [install.sh](http://install.sh/) están en el lugar correcto y tienen la versión correcta.
* [X-code](https://apps.apple.com/ca/app/xcode/id497799835?mt=12) \(para macs\)

## 3. Comenzando \(Instalación\)

**1. Clona el repositorio de `market-maker-keeper` y cambia a su directorio:**

```text
 git clone git@github.com:makerdao/market-maker-keeper.git
 cd market-maker-keeper 
```

**2. Inicializa los submódulos _git_ que traen tanto la biblioteca _pymaker_ como la _pyexchange_:**

```text
git submodule update --init --recursive
```

**3. Configura el _virtual env_ y activarlo:**

```text
 python3 -m venv _virtualenv
 source _virtualenv/bin/activate
```

**4. Revisa que tengas la versión correcta de Python \(Python 3.6.6\) ejecutando:**

```text
 python3 -V
```

**5. Instala los Requerimientos:**

```text
pip3 install $(cat requirements.txt $(find lib -name requirements.txt | sort) | sort | uniq | sed 's/ *== */==/g')
```

* **Nota:** Este comando es \(utilizado en lugar de `pip install -r requirements.txt`\) para recorrer todas las dependencias en el directorio _lib_, para tomar todos los requerimientos necesarios. 

#### Posibles errores que pueden surgir:

* Necesidad de actualizar a la versión **pip** 19.2.2.
  * Ejecuta: `pip install --upgrade pip` para corregirlo.
* Instalación de **jsonnet** \(si se ejecuta macOS Mojave\)

  **Para arreglar, ejecuta lo siguiente:**

  * `xcode-select --install`
  * `open /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg`
  * `pip3 install jsonnet==0.9.5`
  * **Re-run:** `pip3 install $(cat requirements.txt $(find lib -name requirements.txt | sort) | sort | uniq | sed 's/ *== */==/g')`

#### Otros **Potenciales Problemas de Instalación:**

Lee el siguiente documento para estar al tanto de otros problemas conocidos de _Ubuntu_ y _macOS_ \([pymaker](https://github.com/makerdao/pymaker#known-ubuntu-issues)\).

## 4. Probando

Hay cierto valor en ejecutar todas las pruebas unitarias para asegurarse que el `market-maker keeper` ha sido instalado apropiadamente. Después que el repositorio ha sido clonado y la instalación ha sido completada, puedes correr pruebas ejecutando los siguientes comandos. 

**Primeramente, el siguiente comando instalará las bibliotecas requeridas para ejecutar las pruebas unitarias:**

```text
pip3 install -r requirements-dev.txt

```

**Para ejecutar las pruebas unitarias  \(py.test, etc..\), usa el siguiente _script_:**

```text
./test.sh

```

**Ejemplo de _output_:**

```text
===================================== test session starts =====================================
platform darwin -- Python 3.6.6, pytest-3.3.0, py-1.8.0, pluggy-0.6.0
rootdir: /Users/charlesst.louis/market-maker-keeper, inifile:
plugins: timeout-1.2.1, mock-1.6.3, cov-2.5.1, asyncio-0.8.0
collected 97 items                                                                            

tests/test_airswap_market_maker_keeper.py ................                              [ 16%]
tests/test_band.py ......                                                               [ 22%]
tests/test_etherdelta_market_maker_keeper.py ..........................                 [ 49%]
tests/test_feed.py .                                                                    [ 50%]
tests/test_limit.py .......                                                             [ 57%]
tests/test_oasis_market_maker_cancel.py ...                                             [ 60%]
tests/test_oasis_market_maker_keeper.py ...................                             [ 80%]
tests/test_price_feed.py ............                                                   [ 92%]
tests/test_reloadable_config.py .......                                                 [100%]

---------- coverage: platform darwin, python 3.6.6-final-0 -----------
Name                                                    Stmts   Miss  Cover
---------------------------------------------------------------------------
market_maker_keeper/__init__.py                             0      0   100%
market_maker_keeper/airswap_market_maker_keeper.py        252    142    44%
market_maker_keeper/band.py                               260     37    86%
market_maker_keeper/bibox_market_maker_keeper.py           93     93     0%
market_maker_keeper/bitinka_market_maker_keeper.py         96     96     0%
market_maker_keeper/bittrex_market_maker_keeper.py         96     96     0%
market_maker_keeper/coinbase_market_maker_keeper.py       103    103     0%
market_maker_keeper/coinbene_market_maker_keeper.py        95     95     0%
market_maker_keeper/control_feed.py                         7      5    29%
market_maker_keeper/ddex_market_maker_keeper.py           126    126     0%
market_maker_keeper/ercdex_market_maker_keeper.py          12     12     0%
market_maker_keeper/etherdelta_market_maker_keeper.py     193    142    26%
market_maker_keeper/ethfinex_market_maker_keeper.py        94     94     0%
market_maker_keeper/feed.py                                84     46    45%
market_maker_keeper/gas.py                                 20     10    50%
market_maker_keeper/gateio_market_maker_keeper.py         106    106     0%
market_maker_keeper/gopax_market_maker_keeper.py           99     99     0%
market_maker_keeper/hitbtc_market_maker_keeper.py          98     98     0%
market_maker_keeper/idex_market_maker_keeper.py           193    193     0%
market_maker_keeper/imtoken_pricing_server.py              51     51     0%
market_maker_keeper/imtoken_utils.py                       97     97     0%
market_maker_keeper/kucoin_market_maker_keeper.py         108    108     0%
market_maker_keeper/limit.py                               46      0   100%
market_maker_keeper/liquid_market_maker_keeper.py          97     97     0%
market_maker_keeper/mpx_market_maker_keeper.py            137    137     0%
market_maker_keeper/oasis_market_maker_cancel.py           38     22    42%
market_maker_keeper/oasis_market_maker_keeper.py          133     94    29%
market_maker_keeper/okex_market_maker_keeper.py            92     92     0%
market_maker_keeper/order_book.py                         219    188    14%
market_maker_keeper/order_history_reporter.py              38     26    32%
market_maker_keeper/paradex_market_maker_keeper.py        131    131     0%
market_maker_keeper/price_feed.py                         187     86    54%
market_maker_keeper/reloadable_config.py                   67      3    96%
market_maker_keeper/setzer.py                              24     17    29%
market_maker_keeper/spread_feed.py                          7      5    29%
market_maker_keeper/tethfinex_market_maker_keeper.py      149    149     0%
market_maker_keeper/theocean_market_maker_keeper.py       129    129     0%
market_maker_keeper/util.py                                 8      4    50%
market_maker_keeper/zrx_market_maker_keeper.py            177    177     0%
market_maker_keeper/zrxv2_market_maker_keeper.py           26     26     0%
---------------------------------------------------------------------------
TOTAL                                                    3988   3232    19%


================================== 97 passed in 4.04 seconds ==================================

```

## 5. Entendiendo la Configuración de Bandas

El archivo de configuración de Bandas está directamente relacionado a cómo funcionará tu _Market Maker Keeper_. Como se mencionó en la introducción, estos _Keepers_ monitorean y ajustan contínuamente sus posiciones en el libro de órdenes, manteniendo órdenes abiertas de compra y venta en bandas múltiples al mismo tiempo. Para cada banda `buy` y `sell`, los _keepers_ aspiran tener órdenes abiertas para el `minAmount`, al menos. En ambos casos, asegurarán que el precio de órdenes abiertas se mantengan dentro del rango `<minMargin, maxMargin>` del precio actual. Cuando se ejecutan, los _Keepers_ colocan órdenes por las cantidades promedio \(`avgAmount`\) en cada banda utilizando `avgMargin` para calcular el precio de la órden.    

Mientras el precio de las órdenes se mantenga dentro de la(s) banda\(s\) establecidas(s) \(es decir, se encuentra entre el rango `<minMargin,maxMargin>` del precio actual\), los _Keepers_ las mantienen abiertas/en ejecución. Si algunas órdenes salen de la banda, o entran en otra banda adyacente, o quedan fuera de todas las bandas: en el caso de las segundas, se cancelarán inmediatamente; en caso de la primera, los _Keepers_ pueden mantener estas ordenes abiertas siempre que su cantidad esté dentro de los rangos `<minAmount,maxAmount>` para la  banda a la que acaban de entrar. Si está por encima del máximo, algunas de las órdenes abiertas se cancelarán y potencialmente una nueva será creada para traer la cantidad total de vuelta dentro del rango. Si está por debajo del mínimo, se crea una nueva orden por la cantidad restante para que la cantidad total de órdenes en esta banda sea igual a `avgAmount`. Hay algunos _Keepers_, que constantemente usarán gas para cancelar órdenes \(ex: OasisDEX, EtherDelta y 0x\) y crear nuevas \(OasisDEX\) a medida que el precio cambia. El uso del Gas puede limitarse, estableciendo los rangos de margen y de cantidad lo suficientemente amplios, pero también asegurándose que las bandas sean siempre adyacentes entre sí y que sus rangos de cantidad `<min,max>` se superpongan.      

### Formato del Archivo

El archivo de configuración de bandas consiste en dos secciones principales: 

1. _buyBands_
2. _sellBands_

**Nota:** Cada sección es una matriz conteniendo un objeto por cada banda. 

Los campos _`minMargin`_ y _`maxMargin`_ en cada objeto de banda representan el rango de margen \(spread\) de esa banda. Estos rangos no pueden sobreponerse para las bandas del mismo tipo \(_`buy`_ or _`sell`_\) y deberían ser adyacentes entre sí para un mejor rendimiento del _Keeper_ \(donde menos órdenes serán canceladas si las bandas son adyacentes entre sí\). El _`avgMargin`_ representa el margen \(spread\) de órdenes recién creadas dentro de una banda.

#### Glosario

1. _`minAmount`_ - la cantidad mínima para el compromiso del _keeper_ por una banda.  
2. _`avgAmount`_ - la cantidad objetivo para el compromiso del _keeper_ por una banda.
3. _`maxAmount`_ - la cantidad máxima para el compromiso del _keeper_ por una banda.
4. _`dustCutoff`_ - un campo por la mínima cantidad de cada órden creada en cada banda individual \(expresado en comprar _tokens_ para comprar bandas y  en  vender _tokens_ para vender bandas\).
   * Establecer esto a un valor distinto a cero evita que los _Keepers_ creen muchas órdenes pequeñas, que pueden costar mucho gas. Por ejemplo, en el caso de OasisDEX, puede resultar que una orden que es muy pequeña sea rechazada por otras _exchanges_.  

### Configurando tu propio Archivo de Configuración de Bandas: 

#### 1. Creando tu archivo _bands.json_

Para comenzar, toma el ejemplo del archivo de configuración que encontrarás a continuación, cópialo y pégalo a un archivo `.json` dentro del directorio raíz de tu carpeta `market-maker-keeper`. Para que sea fácil usarlo, te recomendamos llamarlo `bands.json`. Este archivo de bandas se configurará con un argumento _command-line_ cuando iniciemos el _Market Maker Keeper_.

**Ejemplo de archivo bands.json que contiene dos bandas:**

Este ejemplo muestra bandas para el par ETH-DAI, donde ETH representa la moneda base y DAI la moneda de cotización: 

```text
{
    "_buyToken": "DAI",
    "buyBands": [
        {
            "minMargin": 0.005,
            "avgMargin": 0.01,
            "maxMargin": 0.02,
            "minAmount": 20.0,
            "avgAmount": 30.0,
            "maxAmount": 40.0,
            "dustCutoff": 0.0
        },
        {
            "minMargin": 0.02,
            "avgMargin": 0.025,
            "maxMargin": 0.03,
            "minAmount": 40.0,
            "avgAmount": 60.0,
            "maxAmount": 80.0,
            "dustCutoff": 0.0
        }
    ],
    "buyLimits": [],

    "_sellToken": "ETH",
    "sellBands": [
        {
            "minMargin": 0.005,
            "avgMargin": 0.01,
            "maxMargin": 0.02,
            "minAmount": 2.5,
            "avgAmount": 5.0,
            "maxAmount": 7.5,
            "dustCutoff": 0.0
        },
        {
            "minMargin": 0.02,
            "avgMargin": 0.025,
            "maxMargin": 0.05,
            "minAmount": 4.0,
            "avgAmount": 6.0,
            "maxAmount": 8.0,
            "dustCutoff": 0.0
        }
    ],
    "sellLimits": []
}
```

Este archivo **bands.json** debería ser lo suficientemente adecuado para que lo pegues y lo corras tal cual esta. Por supuesto, eres libre de configurarlo de la forma que quieras.

**Nota:** dado que este ejemplo estará enfocado en la creación del _Market Maker Keeper_ en Kovan, necesitas asegurarte de que tienes suficiente ETH Kovan \(K-Eth\) para poner en marcha tu _Keeper_. Para recibir ETH Kovan, únete al siguiente Canal Gitter: [https://gitter.im/kovan-testnet/faucet](https://gitter.im/kovan-testnet/faucet) y postea tu dirección ETH desde tu cuenta en Metamask en el chat principal. El grifo Kovan llenará tu _wallet_ con fondos de prueba  \(ten en cuenta que esto podría tomar un par de minutos u horas ya que lo hace el administrador del canal de forma manual\).

#### 2. Estableciendo las Cantidades

Tendrás que configurar esto si vas a operar con pequeñas cantidades para empezar. Esto tendrá que configurarse de tal manera que esas cantidades sean suficientemente significativas en el _exchange_ con el que quieras trabajar \(basado en las reglas del _exchanges_ con el que quieres trabajar. Por ejemplo, sus límites _dust_ específicamente establecidos\).

Como se mencionó anteriormente, hay un parámetro en el archivo de configuración \(_`dustCutoff`\)_ que será usado para determinar la cantidad de comercio `minAmount`. El `dustCutoff` necesitará fijarse **mas alto que/o igual a** la cantidad de comercio `minAmount` del _exchange_ especifico. Esto es para asegurarse de que no se creen operaciones que sean inferiores a los requisitos mínimos de comercio de los _exchanges_. Ten en cuenta que en tu archivo de configuración, también puedes reducir las cantidades requeridas para hacerlo más fácil para ti. Reducir las cantidades de K-ETH será de ayuda para reducir los tiempos de espera, ya que probablemente no querrás esperar largos periodos para obtener suficiente K-ETH para correr tu _Keeper_\).    

#### **Ejemplo de Bandas**

Aquí repasaremos algunos ejemplos de interacciones utilizando el [archivo bands.json](https://www.notion.so/makerdao/Market-Maker-Keeper-Bot-Setup-Guide-c8add6654b154ddca27545f19f6f91af#f7b7143322f8421f93605a09d9cf579f) descrito anteriormente. Estos ejemplos asumen que está denominado en DAI y el precio de DAI es 1 ETH. 

**Usando la Banda 1**

* Si miramos la primera banda de compra, la orden de compra inicial será de 30 DAI \(_`avgAmount`_\) con el precio de -&gt; `price - (price * avgMargin)` -&gt; `0.1 - (0.1 * 0.01)` -&gt; `0.099 ETH per Dai`.
* Si la orden `buy` listada anteriormente \(30 DAI @ 0.099 ETH\) se llena parcialmente \(se compran 15 DAI\), después tendremos \(15 DAI restantes en la orden\). Sin embargo, esta cantidad está por debajo del _`minAmount`_ \(20 DAI\) de la banda, por lo tanto se colocará otra orden completa de 15 DAI en el _exchange_ al mismo precio de 0.099 ETH.     
* En adición a las ordenes `buy`, cuando el _Market Maker Keeper_ se pone en marcha, también se colocarán dos órdenes `sell`.  

**Utilizando la Banda 2**

* Para facilitar la explicación, vamos asumir que estamos vendiendo ETH a 100.00 DAI \(5 ETH @ 101 DAI y 6 ETH @ 102.5 DAI\).
* Ahora imagina una situación donde el precio de ETH de repente cae a 97.50 DAI, empujando las bandas hacia abajo. En este escenario, la segunda banda comenzará a trabajar y será responsable de ambas órdenes `sell`, ya que encajan entre el _`minMargin`_ y _`maxMargin`_ de la segunda banda.

**El _Market Maker Keeper_ restablecerá ahora sus bandas realizando lo siguiente:**

1. Creando una orden en la **Banda 1** \(5 ETH @ 98.475 DAI\) utilizando _`avgMargin`_ y _`avgAmount`_.
2. Cancelar la segunda orden \(5 ETH @ 102.5 DAI\) \(que está ahora en la **Banda 2**\) debido a que se ha violado el _`maxMargin`_ \(cuando `price + (price * maxMargin) = orderPrice` -&gt; `97.5 + (97.5 * 0.05)` -&gt; 102.375 &gt; 102.5\).
3. Mantener la primera orden \(5 ETH @ 101 DAI\), que ahora está en la **Banda 2** porque esta dentro de _`minMargin`_ y _`maxMargin`_ de la **Banda 2**. 
4. Creando una orden en la **Banda 2** \(1 ETH @ 99.937 DAI\) usando _`avgMargin`_ y _`avgAmount`._

**Esto resulta en un total de 3 órdenes:**

* **Banda 1** -&gt; \(5 ETH @ 98.475 DAI\)
* **Banda 2** -&gt; \(5 ETH @ 101 DAI\)
* **Banda 2** -&gt; \(1 ETH @ 99.837 DAI\)

## 6. **Limitación de la Tasa de Orden**

Después, queremos agregar la **Limitación de la Tasa** a los archivos de configuración. Esto se asegurará de que no se produzcan órdenes antigüas constantemente, así como ayudar a gestionar el consumo de gas. Hacemos esto porque queremos que el `period` (período) y la `amount` (cantidad) se establezcan en una cantidad baja cuando empezamos. Esto se hace porque no queremos que los _Market Maker Keeper Bots_ de los nuevos usuarios estén operando frenéticamente todo el tiempo. El objetivo que queremos aquí es configurar nuestros estados iniciales de tal manera que solo sea la colocación de una orden cada 5 minutos mas o menos \(o cualquier cantidad de tiempo que tú decidas\).

Hay dos \(opcionales\) secciones límites \(_`buyLimits`_ y _`sendLimits`_\) que pueden ser utilizadas para limitar la tasa **maxima** de órdenes creadas por los _Market Maker Keepers_. Ambos usan el mismo formato.    

 *Ejemplo de limites de tasas de órdenes*:

```text
"buyLimits": [
    {
        "period": "1h",
        "amount": 50.0
    },
    {
        "period": "1d",
        "amount": 200.0
    }
]
```

* El `period`  define la cantidad de tiempo que el límite debería ser aplicado. 
* La `amount` es la máxima cantidad de órdenes que deberían colocarse durante la cantidad `period` establecida.

En el ejemplo anterior, los _`period`s_ se establecen a 1-hora y 1-dia y las _`amount`s_ son fijadas a 50.0 órdenes y 200.0 órdenes. Esto significa que en el transcurso de una 1-hora, solo 50.0 órdenes pueden colocarse y durante el transcurso de 1-dia, solo se pueden colocar 200.0 ordenes. Las cantidades serán expresadas tanto en términos del token `buy` o el token `sell`, esto dependerá de la sección. Ten en cuenta que el fragmento anterior impone un límite de _50.0_ tokens `buy` dentro de cada ventana de 60-minutos. Adicionalmente, un máximo de _200.0_  tokens `buy` dentro de cada ventana de 24-horas. Ten en cuenta que las unidades de tiempo soportadas son `s`, `m`, `h`, `d` y `w`.      

## 7. **Lenguaje de Plantilla de Datos**

_El_ [_Jsonnet_](https://github.com/google/jsonnet) _lenguaje de plantilla de datos que puede ser utilizado para el archivo de configuración._

En el caso del lenguaje de plantilla de datos, piensa en esto como un lenguaje de pre-procesamiento para analizar el archivo. El propósito del _jsonnet_ es establecer un archivo de configuración de tal manera que puedas hacer que se incremente basado en un precio. Por lo tanto, en adición a la fuente de alimentación de precios, también puedes basar los porcentajes fuera del precio de mercado. Como se puede ver a continuación, hay una cantidad/precio _hardcoded_ y luego las cantidades de abajo que son dependientes del precio.

```text
{
  "_price": 10,

  "_buyToken": "DAI",
  "buyBands": [
    {
      "minMargin": 0.020,
      "avgMargin": 0.050,
      "maxMargin": 0.075,
      "minAmount": 0.05 * $._price,
      "avgAmount": 0.25 * $._price,
      "maxAmount": 0.35 * $._price,
      "dustCutoff": 0.1
    }
  ],

  "_sellToken": "ETH",
  "sellBands": []
}
```

**Nota:** si estás trabajando con un token cuyo precio no fluctúa salvajemente, no necesitas incorporar cualidades relativas para tu cantidad. Normalmente esto es para personas que quieren _Market Maker Keepers_ abiertos por meses en un momento y no quieren preocuparse de tener que cambiar ninguna de sus configuraciones.  

Otra cosa que tener en cuenta acerca de estos archivos es que los  _Market Maker Keepers_ recargan los archivos de configuración automáticamente cuando se detecta un cambio en ellos. Esto lo hace difícil ya que no tienes que reiniciar constantemente tu bot _Keeper_ cuando cambies las configuraciones de tu banda. En resumen, esto funciona tomando periódicamente un _hash_ del archivo de configuración y comparando ese _hash_ con la versión actual. Cuando detecta un cambio en ese archivo _hash_, recargará el archivo configuración y cancelará las órdenes según sea necesario para mantener bandas recién actualizadas. 

## 8. Configuración de Alimentación de  Precio

La fuente de alimentación de precio es una de las más importantes determinando factores de éxito en un _Market Maker Keeper_. Si tienes las bandas configuradas de la forma que deseas, la alimentación de precio ayudará a asegurarse que tus bandas estén fijadas a niveles significativos en relación con el mercado interno. Si tienes bandas anchas y tu estrategia es agregar liquidez para manejar los desequilibrios del mercado, entonces la alimentación de precios no es importante. Sin embargo, a medida que se vayas ajustando las diferencias, la alimentación de precios es un componente crucial para  asegurarte que vas a obtener ganancias en el mercado.  
 
A continuación, tienes una lista de algunas fuentes de alimentación públicas. También puedes usar _sockets_ web, si tienes tu propio alimentador de precio que desees usar. En resumen, funciona cuando cada _Market Maker Keeper_ toma un argumento _command-line_ `--price-feed` que después determina el precio utilizado para el _market-making_.
     
**A día de hoy, estos son los posibles valores de este argumento que hemos enumerado:**

* `fixed:200` - usa un precio fijo, \(`1.56` en este ejemplo\). A continuación puedes ver un ejemplo mas detallado. En _mainnet_, normalmente no utilizarás una cantidad fija, pero es un ejemplo ideal para esta ocasión, ya que no hay alimentadores de precios para Kovan.
* `eth_dai` - usa el precio del alimentador de precio de ETH/USD de GDAX WebSocket.
* `eth_dai-setzer` - usa el promedio de precios ETH/USD de Kraken y Gemini.
* `eth_dai-tub` - usa el alimentador de precio de `Tub` \(solo funciona para _Keepers_ capaces de acceder a un nodo de Ethereum\).
* `dai_eth` - inverso del alimentador de precios `eth_dai`.
* `dai_eth-setzer` - inverso del alimentador de precios `eth_dai-setzer`.
* `dai_eth-tub` - inverso del alimentador de precios `eth_dai-tub`.
* `btc_dai` - utiliza los precios del alimentador de precios de BTC/USD de GDAX WebSocket.
* `dai_btc` - inverso del alimentador de precios del `btc_dai`.
* `ws://...` o `wss://...` - utiliza un alimentador de precios anunciado a través de una conexión WebSocket \(protocolo personalizado\).
 
Adicionalmente, tenemos un alimentador de Precio Uniswap que puede ser usado por _Market Maker Keepers_:  
[https://github.com/makerdao/uniswap-price-feed ](https://github.com/makerdao/uniswap-price-feed).

**Nota:** el argumento _command-line_ `--price-feed` también puede contener una lista separada por comas de diferentes alimentadores de precios. En este caso, si uno de ellos no está disponible, se utilizará el próximo de la lista. Todos los precios listados se ejecutarán constantemente en el fondo, el segundo y los siguientes que están listos para tomar el relevo cuando el primero no esté disponible. **A continuación \(en la sección de _Running Keepers_ \), puedes ver un ejemplo de cómo utilizar una cantidad de precio fija.**  

## 9. Ejecución de los _Market Maker Keepers_

Cada _Market Maker Keeper_ es una herramienta  _command-line_ que toma un argumento _command-line_ genérico \(tales como `--config`, `--price-feed`, `--price-feed-expiry`, `--debug`, etc.\) así como algunos argumentos que son específicos de un _Keeper_ particular \(tales como parámetros de nodos de Ethereum, direcciones, llaves API de intercambio, etc.\). Todos los argumentos _command-line_ aceptados están enumerados en la siguiente sección de ejemplo. También se pueden descubrir tratando de iniciar un _Market Maker Keeper_ con el argumento `--help`.

#### Ejemplo \(_Market Maker Keeper_ de Oasis\)

Para poder ejecutar `oasis-market-maker-keeper`, necesitarás cumplir con los siguientes procesos:

1. Primero, debes desplegar un Nodo de Ethereum \(te recomendamos Parity\).
2. Créate una cuenta en el.
3. Desbloquea permanentemente esa cuenta.
4. Transfiere algunos _tokens_ allí.
5. Por último, puedes ejecutar el _keeper_ \(como se ve a continuación\).

Debes copiar y pegar el siguiente archivo en un nuevo archivo dentro del directorio raíz del repositorio \(`market-maker-keeper`\). Esto debería colocarse en la misma carpeta donde pones el archivo `bands.json`.  

```text
#!/bin/bash

bin/oasis-market-maker-keeper \
    --rpc-host 127.0.0.1 \
    --rpc-port  \
    --rpc-timeout 10 \
    --eth-from [address of your generated Ethereum account] \
		--eth-key ${ACCOUNT_KEY} \
    --tub-address  0x448a5065aebb8e423f0896e6c5d525c040f59af3 \
    --oasis-address  0x14fbca95be7e99c15cc2996c6c9d841e54b79425 \
    --price-feed fixed:200 \
    --buy-token-address [address of the quote token, could be DAI] \
    --sell-token-address [address of the base token, could be WETH] \
    --config [path to the json bands configuration file, e.g bands.json] \
    --smart-gas-price \
    --min-eth-balance 0.001


```

* Asegúrate de recuperar y pegar las direcciones de contrato correctas utilizando el fragmento de arriba.
* `--eth-key ${ACCOUNT_KEY}` - incluye tanto al archivo `.json` \(account.json\) de tu cuenta, como al archivo `.pass` \(ejemplo: account.pass\) que contiene tu contraseña en _plaintext_.
* Si no tienes una cuenta, puedes usar [MyEtherWallet](https://www.myetherwallet.com/) en Kovan y exportar los detalles de la cuenta \(mediante el método de archivo _Keystore_\). Asegúrate de descargar el archivo _.json_ a tu máquina local ya que es lo que necesitarás para configurar la cuenta.

**Lista de Direcciones Kovan requeridas previamente:**

```text
V2_OASIS_SERVER1_ADDRESS= 
V2_OASIS_SERVER1_KEY="key_file=/home/ed/Projects/member-account.json,pass_file=/home/ed/Projects/member-account.pass"
TUB_ADDRESS=0xa71937147b55deb8a530c7229c442fd3f31b7db2 # tub-address
SAI_ADDRESS=0xc4375b7de8af5a38a93548eb8453a498222c4ff2 # buy-token-address
WETH_ADDRESS=0xd0a1e359811322d97991e03f863a0c30c2cf029c # sell-token-address
OASIS_ADDRESS_NEW=0x4a6bc4e803c62081ffebcc8d227b5a87a58f1f8f # oasis-address

```

**Notas Generales:**

* El `OASIS_SERVER1_KEY` es simplemente tu clave privada de la cuenta Kovan \(anota esto en tu archivo de claves de cuentas ETH\) y el archivo de contraseña. Si no lo tienes, crea un archivo con tu contraseña \(en _plaintext_\).
* _ETH From_ es la ubicación de la dirección donde el _market-maker-keeper_ va a obtener los tokens que utiliza para participar y colocar órdenes.
  * **Ejemplo:** Dado que Oasis es un _exchange_ descentralizado \(dex\), está _on-chain_, así que necesitas proporcionar todas las direcciones relevantes al dex. La mayoría de DEX's son así debido a que cuando los estás configurando con un dex necesitas pasar muchas direcciones, mientras que, con un _exchange_ centralizado, generalmente estás dando una llave API, usuario y contraseña. \(a continuación puedes ver un ejemplo de como el proceso difiere de los _exchanges_ centralizados VS los _exchanges_ descentralizados\).
* Actualmente este ejemplo de Oasis es para el **DAI de Un Solo Colateral** \(SCD\), donde configuramos la TUB\_ADDRESS. Sin embargo, a medida que nos movemos a **DAI Multi-Colateral\(MCD\)** la **TUB\_ADDRESS** cambiará por la **PIP\_ADDRESS** para MCD.

#### **Una vez completado, ahora puedes correr tu propio _Market Maker Keeper_. Sigue las siguientes indicaciones para ponerlo en marcha:**

1. Abre tu terminal
2. Ejecuta `chmod +x <el nombre del archivo de tu versión del bin/oasis-market-maker-keeper snippet>` que hemos visto previamente.
3. Ejecuta `./<el nombre de tu versión de bin/oasis-market-maker-keeper snippet>` que hemos visto previamente.
4. ¡Eso es todo, tu _Market Maker Keeper_ debería estar en marcha ahora!

#### _Market Maker Keepers_ en _Exchanges_ Centralizados VS _Exchanges_ Descentralizados

En la situación en la que desées utilizar un _exchange_ centralizado VS un _exchange_ descentralizado, el proceso difiere un poco: 

1. Necesitarás tener una cuenta existente o crear una cuenta \(en el mismo _exchange_\).
2. Obtén el conjunto de claves API con permisos de negociación \(por lo general necesitará ser generado también\).
3. Deposita tokens en tu cuenta en el _exchange_ \(ya que los _keepers_ no manejan depósitos y retiros por sí mismos\).
4. Ejecuta el _Market Maker Keeper_.

## 10. Soporte

**¡Estamos aquí para ayudar!** Cualquier duda o pregunta sobre _market making_ es bienvenida en el canal [\#keeper](https://chat.makerdao.com/channel/keeper) del chat de Maker.