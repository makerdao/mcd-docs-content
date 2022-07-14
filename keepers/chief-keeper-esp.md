# _Keeper_ Jefe

## Introducción 

El `chief-keeper` (_Keeper_ Jefe) monitorea e interactúa con [DSChief](https://github.com/dapphub/ds-chief) y DSSSpells, que es el contrato de votación ejecutiva y un tipo de objeto de propuesta del [Protocolo de Maker](https://github.com/makerdao/dss).   

Su propósito es levantar el `hat` (“sombrero”) en DSChief, así como agilizar acciones ejecutivas. 

Para `lift`(“levantar”) un _spell_ (“hechizo”) debe tener más aprobaciones que el `hat` actual. Las aprobaciones de este _spell_ pueden fluctuar y ser superado por otros _spells_, alguno de los cuales podría ser malicioso. Este _keeper_ "guarda" el `hat` asegurándose de que el _spell_ con mas aprobaciones sea siempre el `hat`. El `chief-keeper` hace esto para maximizar la barrera de entrada \(aprobación\) para `lift` un _spell_ al _hat_, así pues, actúa como un "guardia" contra acciones maliciosas de la gobernanza.  

Mientras esté en funcionamiento, el `chief-keeper`:

* Monitorea cada nuevo bloque para ver si hay un cambio en el estado de las votaciones ejecutivas
*_`lift`s_ ("levanta") el _hat_ ("sombrero") para el _spell_ \(`yay`\) más favorecido \(`approvals[yay]`\)  
* Programa los _spells_ en el GSM llamando `DSSSpell.schedule()`
* Ejecuta _spells_ después de que su `eta` haya trascurrido en el GSM llamando a `DSSSpell.cast()`

#### Revisión 


La siguiente sección asume familiaridad con el [DSChief](https://github.com/dapphub/ds-chief), DSSSpells y [DSPause](https://github.com/dapphub/ds-pause) \(Módulo de Seguridad de la Gobernanza\), así como el proceso dentro de la [Gobernanza de MakerDAO](https://community-development.makerdao.com/governance).   

## Arquitectura 

![texto alternativo](https://github.com/makerdao/chief-keeper/raw/master/operation.jpeg)

`chief-keeper` interactúa directamente con el `DS-Chief` y `DSSSpell`s.

## Funcionamiento 

Este _keeper_ se ejecuta contínuamente y guarda una base de datos local de `yays` \(dirección del _spell_ (“hechizo”)\) y un diccionario `yay:eta` para reducir las lecturas de estado en _chain_ (“cadena”). Si te gustaría crear tu propia base de datos desde cero, primero borra `src/database/db_mainnet.json` antes de ejecutar `bin/chief-keeper`; la consulta inicial puede tardar hasta 15 minutos.      

#### Instalación 

Requisitos Previos:

* [Python v3.6.6](https://www.python.org/downloads/release/python-366/)
* [virtualenv](https://virtualenv.pypa.io/en/latest/)
  * Este proyecto requiere instalar _virtualenv_ si quieres usar las herramientas Python de Maker. Esto ayuda a asegurarse que estás ejecutando la versión correcta de Python y revisa que todos los paquetes _pip_ instalados en el **install.sh** estén en el lugar correcto y tienen la versión correcta.   
  
Para poder clonar el proyecto e instalar los paquetes _third-party_ requeridos debes ejecutar: 

```text
git clone https://github.com/makerdao/chief-keeper.git
cd chief-keeper
git submodule update --init --recursive
./install.sh
```

Si `tinydb` no es visible/instalado a través de `./install.sh`, simplemente ejecuta `pip3 install tinydb` después de los comandos anteriores. 

Para algunos problemas conocidos de Ubuntu y macOS consulta [pymaker](https://github.com/makerdao/pymaker).

#### Ejemplo de Guion _Startup_

Haz un _run-chief-keeper.sh_ para iniciar fácilmente el _chief-keeper_.

```text
#!/bin/bash
/full/path/to/chief-keeper/bin/chief-keeper \
	--rpc-host 'sample.ParityNode.com' \
	--network 'kovan' \
	--eth-from '0xABCAddress' \
	--eth-key 'key_file=/full/path/to/keystoreFile.json,pass_file=/full/path/to/passphrase/file.txt' \
	--chief-deployment-block 14374534
```

## Probando

* Descarga [_docker_ y _docker-compose_](https://www.docker.com/get-started)

Este proyecto usa [pytest](https://docs.pytest.org/en/latest/) para pruebas unitarias. Las pruebas del DAI Multicolateral se llevan a cabo en una _testchain_ (“cadena de pruebas”) local Dockerizada incluida en `tests\config`.

Para poder ejecutar las pruebas, debes instalar las dependencias de desarrollo primero, ejecutando: 

```text
pip3 install -r requirements-dev.txt
```

Puedes ejecutar todas las pruebas con:

```text
./test.sh
```

## Hoja de Ruta

* [ ]  [Estrategia de precios dinámicos del gas](https://github.com/makerdao/market-maker-keeper/blob/master/market_maker_keeper/gas.py)

### Licencia 

Ver expediente [COPYING](https://github.com/makerdao/chief-keeper/blob/master/COPYING).

## Soporte 

Si tienes preguntas acerca de los _Keepers_ Cage, contáctanos el canal [\#keeper](https://chat.makerdao.com/channel/keeper) en [**chat.makerdao.com**](http://chat.makerdao.com/).
