# _Keeper_ del _Cage_ (o `cage-keeper)`

## Introducción&#x20;

El `cage-keeper` es utilizado para facilitar el [Apagado de Emergencia](https://blog.makerdao.com/introduction-to-emergency-shutdown-in-multi-collateral-dai/) del [Protocolo de Maker](https://github.com/makerdao/dss). El Apagado de Emergencia es un proceso complejo y determinista que requiere la interacción de todos los tipos de usuarios: dueños de _vaults_, holders de DAI, _Keepers_ de redención, gobernadores de MKR y otras partes interesadas del Protocolo de Maker. Una visión general de alto nivel sería la siguiente: 

1. **Sistema _Caged_ ("Enjaulado")** - El Módulo de Seguridad de Emergencia [(ESM)](https://github.com/makerdao/esm) llama la función `End.cage()`, que congela el precio del USD para cada tipo de colateral así como muchas partes del sistema.
2. **Periodo de Procesamiento** - Después, los dueños de _vaults_  interactúan con _End_ para liquidar sus _vaults_ y retirar el exceso de colateral. Las subastas se dejan concluir o se retiran antes de la redención de DAI.
3. **Redención de DAI** - Después de transcurrido el período de procesamiento `End.wait` se supone que la liquidación de _vaults_ y todo el proceso de generación de DAI (subastas) han concluido. A este punto, los holders de DAI pueden empezar a reclamar una cantidad proporcional para cada tipo de colateral a una tasa fija. 

Para evitar una _race-condition_ ("condición de carrera") para los holders de DAI durante el Paso 3, es imperativo que cualquier _vault_ que tenga un ratio de colateralización de menos del 100% en el Paso 1 sea procesada durante el Paso 2. El dueño de una _vault  underwater_ no recibiría un exceso de colateral, por lo que carecen de incentivos para `skim` sus posiciones en el contrato `End`. Por lo tanto, es responsabilidad de una de las partes interesadas de MakerDAO (holders de MKR, grandes holders de DAI, etc) asegurar que el sistema facilite una fase de redención de DAI sin una variable de tiempo. El `cage-keeper` es una herramienta para ayudar a las partes interesadas a cargar con esta responsabilidad.   

### Requisitos Previos 

La siguiente sección asume que estás familiarizad@ con el Apagado de Emergencia. Algunos buenos lugares para empezar son el Módulo de Apagado de Emergencia en la Sección 3 y Sección 4 del [Protocolo Maker 101](https://docs.makerdao.com/maker-protocol-101-esp) así como una [descripción técnica](https://docs.makerdao.com/smart-contract-modules/shutdown-esp) más completa. Las Funciones mencionadas provienen de la implementación contenida por el contrato `End`, que esta [localizado aquí](https://github.com/makerdao/dss/blob/master/src/end.sol). 

**Para ser consistente con la terminología técnica del Protocolo para el resto de esta descripción:**

* `urn` = Vault
* `ilk` = Tipo de Colateral

## Arquitectura

![](<../.gitbook/assets/cage2 (1).png>)

El `cage-keeper` interactúa directamente con los contratos `End`, `Flopper` y `Flapper`.

El objetivo central del `cage-keeper` es procesar todas las `urns` subcolateralizadas. Este paso contable se lleva a cabo dentro de `End.skim()` y dado que está rodeado por otros pasos importantes/requeridos en el Apagado de Emergencia, una primera iteración de este _Keeper_ ayudará a llamar la mayoría de las otras funciones publicas dentro del contrato `End`.  

Como se puede ver en el diagrama de flujo, el _keeper_ revisa si el sistema ha sido _caged_ ("enjaulado") antes de intentar `skim` todas las _underwater urns_ y `skip` todas las funciones _flip_. Después que se ha facilitado el período de procesamiento y se ha alcanzado el tiempo de espera `End.wait`, hará la transición del sistema a la fase de redención de DAI del Apagado de Emergencia llamando a `End.thaw()` y `End.flow()`. Esta primera iteración de este _Keeper_ es ingenua, ya que asume que es el único _Keeper_ e intenta dar cuenta de todas las _urns_, _ilks_ y subastas. Debido a esto, es importante que la dirección del _Keeper_ tenga suficiente ETH para cubrir los costos de gas que implica enviar numerosas transacciones. Cualquier transacción que intenta llamar una función que ya ha sido invocada por otro _Keeper_/usuario simplemente fallará.

## Operación

Este _Keeper_ puede ejecutarse de forma contínua en una maquina local/virtual o se puede ejecutar cuando el operador se vuelve consciente del Apagado de Emergencia. Un ejemplo del _script_ de la _startup_ se muestra a continuación. La dirección _Keeper_ de Ethereum debería tener suficiente ETH para cubrir los costos de gas y es una función del estado del protocolo al momento del apagado (es decir mas _urns_ para `skim`, significa que se requiere mas ETH para cubrir los costos de gas). Cuando un nuevo tipo de colateral es agregado al protocolo, el operador debe sacar la ultima versión del _Keeper_, que incluiría contratos asociados con los tipos de colateral antes mencionados. 

Después que el `cage-keeper` facilita el período de procesamiento, se puede apagar hasta que casi alcance `End.wait`. Después, en ese punto, el operador pasaría el argumento `--previous-cage` durante el inicio del _Keeper_ para poder saltarse la función que soporta al período de procesamiento.  

## Instalación

Este proyecto utiliza _Python 3.6.2_.

Para poder clonar el proyecto e instalar los paquetes de terceros requeridos, debes ejecutar:

```
git clone https://github.com/makerdao/cage-keeper.git
cd cage-keeper
git submodule update --init --recursive
./install.sh
```

Para algunos problemas conocidos de Ubuntu y macOS consulta el LÉEME de [pymaker](https://github.com/makerdao/pymaker).

### _Script_ Ejemplo de _Startup_

Haz un _run-cage-keeper.sh_ para girar fácilmente el _cage-keeper_.

```
#!/bin/bash
/full/path/to/cage-keeper/bin/cage-keeper \
	--rpc-host 'sample.ParityNode.com' \
	--network 'kovan' \
	--eth-from '0xABCAddress' \
	--eth-key 'key_file=/full/path/to/keystoreFile.json,pass_file=/full/path/to/passphrase/file.txt' \
	--vat-deployment-block 14374534
```

## Pruebas

**Requisitos Previos:**

* descargar [docker y docker-compose](https://www.docker.com/get-started)

Este proyecto utiliza [pytest](https://docs.pytest.org/en/latest/) por unidad de prueba. Las pruebas del DAI Multi-colateral se llevan a cabo en un cadena de prueba local Dockerizada incluida en `tests\config`.
 
**Para poder ser capaz de ejecutar pruebas, primero debes instalar dependencias de desarrollo ejecutando:**

```
pip3 install -r requirements-dev.txt
```

**Después puedes ejecutar todas las pruebas con**:

```
./test.sh
```

## Licencia 

Ver el archivo [COPYING](https://github.com/makerdao/auction-keeper/blob/master/COPYING)

## Soporte

Si tienes preguntas sobre los _Keepers_ del _Cage_, contáctanos en el canal [#keeper](https://chat.makerdao.com/channel/keeper) en [**chat.makerdao.com**](http://chat.makerdao.com).