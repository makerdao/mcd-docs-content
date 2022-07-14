# Seth

## Introducción a Seth

Seth es una herramienta de línea de comandos simple pero poderosa, creada para interactuar con la _blockchain_ de Ethereum. Es parte del conjunto de herramientas [Dapp.Tools](https://dapp.tools/) junto con otras herramientas para Ethereum. Sus dos funcionalidades principales, dentro de otras, son realizar llamadas \(consultas a la _blockchain_ de Ethereum - una operación de “lectura”\) y enviar transacciones \(escribir dentro de la *blockchain*, cambiar su estado\). También proporciona conversión entre datos, con el formato propio y específico de Ethereum, y los formatos de datos habituales más comúnmente conocidos.

### Empezando

En la próxima sección repasaremos la instalación y configuración de Seth. Estos pasos solo funcionan en sistemas basados en Unix \(es decir, Linux y macOS\); sin embargo, en Windows, puedes probar con un emulador como [cmder](http://cmder.net/) o [cygwin](https://www.cygwin.com/), el [subsistema de Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) en Windows 10, una máquina virtual o un _container_.

### Instalación

Seth se puede instalar como parte del paquete de Dapp Tools, que es una colección de herramientas de _blockchain_ creada con la filosofía de Unix en mente. La forma más conveniente de hacer esto es instalar Dapp Tools con el script de una línea proporcionado en la [página web](https://dapp.tools/). Aquí está cómo hacerlo:

De la página de Dapp Tools:

> Si lo estás corriendo en GNU/Linux o macOS, puedes aprovechar nuestro instalador todo en uno.

`$ curl https://dapp.tools/install | sh`

Este script descarga el administrador de paquetes Nix, configura el caché binario con Cachix e instala las herramientas que más utilizamos.

### Instalación Manual

Si tienes problemas utilizando el script de arriba, puedes probar con la instalación manual
en el sitio web antes mencionado. En este momento, puedes realizar una instalación manual ejecutando los siguientes scripts:

```text
$ curl https://nixos.org/nix/install | sh
$ . "$HOME/.nix-profile/etc/profile.d/nix.sh"
$ nix-env -if https://github.com/cachix/cachix/tarball/master --substituters https://cachix.cachix.org --trusted-public-keys cachix.cachix.org-1:eWNHQldwUO7G2VkjpnjDbWwy4KQ/HNxht7H4SSoMckM=  
$ cachix use dapp  
$ git clone --recursive [https://github.com/dapphub/dapptools](https://github.com/dapphub/dapptools) $HOME/.dapp/dapptools  
$ nix-env -f $HOME/.dapp/dapptools -iA dapp seth solc hevm ethsign
```

Para probar si las herramientas fueron instaladas correctamente, chequea la versión actual de Seth con el siguiente comando:

`$ seth --version`

Si Seth fue instalado correctamente, el comando debería devolver lo siguiente:

`seth 0.7.0`

Al momento de escribir esto, seth 0.7.0 es la última versión disponible sin embargo, en un futuro, el númeor de la versión puede variar.

#### Errores y una nota acerca de macOSX Mojave

Si el comando detallado arriba no funciona o tuviste problemas al tratar de instalarlo, puede que sea debido a Mac OSX Mojave, ya que hemos experimentado varios errores con las herramientas Nix y Cachix que no funcionan en este sistema operativo, específicamente debido a un error por múltiples usuarios. Si tiene múltiples usuarios en tu Mac y experimentas errores corriendo esta guía, [este documento](https://github.com/makerdao/developerguides/blob/master/devtools/seth/seth-guide-01/seth-guide-01.md#) puede ayudarte a resolver tus problemas. Si este documento no soluciona el problema, estás más que bienvenido a pedir ayuda en el canal \#help en [chat.makerdao.com](https://chat.makerdao.com/).

### Establecimiento y configuración de variables

Se puede configurar Seth con varibales de entorno u opciones de línea de comando. Las variables de entorno, generalemnte, pueden utilizarse de dos formas: puedes guardarlas en un lugar específico en un archivo de configuración llamado `.sethrc`, como tu carpeta local o puedes establecerlas solo para la sesión actual de terminal. En esta guía, utilizaremos variables de entorno con este último enfoque por motivos de simplicidad; sin embargo, para facilitar su uso en el futuro, te recomendamos que guardes las variables en la carpeta de tu proyecto. Sigue [este ejemplo](https://github.com/dapphub/dapptools/tree/master/src/seth#example-sethrc-file) para hacerlo.

#### Usando una red privada local

Puedes configurar rápidamente una red privada local con la herramienta Dapp, que también creará cuentas y sus archivos de almacenamiento de claves con cadenas vacías para contraseñas, de forma predeterminada. La configuración de una red local puede ser útil cuando se desarrollan dapps para implementar y probar la funcionalidad de manera rápida y fácil sin la necesidad de adquirir la prueba de red de Ether para la implementación por contrato.

Abre una nueva terminal, ejecuta el siguiente comando y déjalo corriendo en segundo plano durante este turorial:

`$ dapp testnet`

Copia la _address_ de tu cuenta de la salida del comando.

Luego, en otra terminal, vamos a crear un archivo de contraseña vacío:

`$ touch pass`

y creamos variables de entorno:

```text
$ export ETH_PASSWORD=$PWD/pass   
$ export ETH_KEYSTORE=~/.dapp/testnet/8545/keystore   
$ export ETH_FROM=<your ethereum account address>
```

#### Utilizando Kovan

Seth puede conectar con la red de prueba de Kovan Ethereum testnet a través de un nodo remoto predeterminado proporcionado por Infura. Esta es la forma más conveniente de hacerlo. Puedes crear una cuenta nueva tanto como utilizar una ya creada en la red de prueba. Si decides utilizar una ya existente, solo debes cambiar el parámetro cadena:

`$ export SETH_CHAIN=kovan`

Si decides crear una nueva cuenta, el método más simple de realizarlo sería usando la opción de "crear una nueva _wallet_" en MEW: [https://www.myetherwallet.com/](https://www.myetherwallet.com/). Es posible utilizar Parity o Geth para crear una nueva cuenta o puedes utilizar el archivo de almacenaje de llaves para una cuenta en Parity o Geth. También necesitarás guardar las contraseña de tu archivo de alamcenaje de llaves en un archivo de texto (¡nunca utilices este archivo de almacenaje de llaves para ETH real!; guardar la contraseña para tu archivo de llaves en un archivo de texto, puede ser muy inseguro si fuera una cuenta real. ¡Esto también aplica para la cuenta de la red de prueba!).

Luego, debes establecer las mismas variables:

```text
$ export ETH_KEYSTORE=<path to your keystore folder>
$ export ETH_PASSWORD=<path and filename to the text file containing the password for your account e.g: /home/one1up/MakerDAO/415pass >
$ export ETH_FROM=<your ethereum account address> 
$ export SETH_CHAIN=kovan
```

Necesitarás ETH de Kovan para el *gas*, puedes conseguir algunos siguiendo esta guía: [https://github.com/kovan-testnet/faucet](https://github.com/kovan-testnet/faucet)

### Operaciones de Seth

Para las primeras dos operaciones, puedes utilizar tu red de prueba como la rede de prueba de Kovan. ¡Prueba ambas, si lo deseas!

#### `seth balance` - Chequear los saldos de ETH

Verificar los saldos de ETH es bastante sencillo. Puede realizarse con el subcomando `balance` y, luego, especificando la _address_ como parámetro:

`$ seth balance $ETH_FROM`

#### `seth send` - Enviar ETH

Vamos a enviar a Kovan o a una red privada de ETH a una *address*. Puedes elegir cualquier *address* válida; en este ejemplo usaremos la [_address_ de donaciones de la Fundación de Ethereum](https://www.ethereum.org/donate):

`$ seth send --value 0.1` [`0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359`](https://etherscan.io/address/0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359)

Tras la ejecución, deberías ver algo como lo siguiente:

```text
seth-send: warning: `ETH_GAS' not set; using default gas amount
seth-send: Published transaction with 0 bytes of calldata.
seth-send: 0x000000…
seth-send: Waiting for transaction receipt.......
seth-send: Transaction included in block xxxxxx.
```

Esto indica que la transacción fue exitosa.

#### `seth call` - Leer el almacenamiento de contratos

Dado que no tenemos ningún contrato implementado en nuestra red privada, **utilicemos Kovan de ahora en adelante**. Usemos uno de los contratos más simples posibles: un contrato de token ERC-20. En este ejemplo, vamos a utilizar un token colateral de prueba \(COL1\). Puedes guardar su _address_ en una variable con el siguiente comando:

`$ export COL1=0x911eb92e02477a4e0698790f4d858e09dc39468a`

Puedes leer el resultado, de una función pública de un contrato, mediante el subcomando `call`, la _address_ del contrato y el nombre de la función.

Veamos la el número de decimales de este token:

`seth call $COL1 'decimals()'`

La salida/resultado de este comando es:

`0x0000000000000000000000000000000000000000000000000000000000000012`

No dejes que esto te engañe. Seth consulta los datos del contrato en un nivel bajo y devuelve el valor en hexadecimal, tal como se representa en el contrato, pero puedes convertirlo usando:

`$ seth --to-dec $(seth call $COL1 'decimals()')`

La salida/resultado de este comando es:

`18`

#### Enviando transacciones de contrato con `seth send`

Puedes enviar transacciones a un contrato agregando unos parámetro extra al mismo comando `send`. Igual que con `call`, necesitas especificar la _address_ del contrato y la función que estás llamando. Obtengamos algunos tokens COL1 de un *faucet* previamente configurado:

`$ export FAUCET=0xe8121d250973229e7988ffa1e9330b420666113a`

`$ seth send $FAUCET ‘gulp(address)’ $COL1`

#### Utilizando parámetros de funciones

Ahora puedes ver los saldos de tus COL1. Esta vez, tendrás que presentar un parámetro para el método `balanceOf` ("balance de") del contrato ERC-20. Puede hacer esto definiendo primero el tipo que la función toma entre paréntesis y luego colocando el parámetro de entrada luego del método:

`$ seth --to-dec $(seth call $COL1 'balanceOf(address)' $ETH_FROM)`

La salida/resultado de este comando es:

`500000000000000000000`

¡Ese es un gran número!. La razón para esto es que el contrato almacena los saldos en unidades *wei* \(10^-18\), por lo que debemos convertirlo para obtener el número actual de COL1 que tenemos:

`$ seth --from-wei $(seth --to-dec $(seth call $COL1 'balanceOf(address)' $ETH_FROM)) eth`

La salida/resultado de este comando es:

`50.000000000000000000`

#### `seth block` - Recuperando de información del bloque

Con los bloques de Seth, somos capaces de cnsultar cualquier información sobre el bloque de Ethereum. Aquí puedes ver la utilización de la opción `help` ("ayuda") --> `$ seth block --help`:

```text
Usage: seth block [-j|--json] <block> [<field>] 
Print a table of information about <block>.
If <field> is given, print only the value of that field.
```

Como cualquier otro comando de Seth, este comando depende de las llamadas del RPC JSON de Ethereum, que son parte de cualquier interfaz de usuario/cliente de Ethereum. Aquí puedes indagar en la documentación correspondiente \([https://github.com/ethereum/wiki/wiki/JSON-RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC)\) para aprender más acerca de esto.

Lo que puede resultar útil es el hecho de que, en lugar de un número de bloque, también podemos usar el más antiguo, el más reciente o el pendiente. Entonces, si quisiéramos consultar el límite de *gas* del bloque actual \(hemos probado esto con Seth configurado para la red de prueba de Kovan\) podemos hacer lo siguiente:

`$ seth block latest gasLimit`

La salida/resultado de este comando es:

`8000000`

#### `seth estimate` - Estimando el costo del *gas* de una transacción

`seth estimate` puede traer la estimación de la utiolización del _gas_ de una transacción. La sintáxis es parecida a la de `seth send`, pero `seth estimate` no enviará la transacción.

Cuando quieras enviar una transacción a una función de contrato con Seth, debes proporcionar la firma de la función y los parámetros en orden, luego de la firma. La firma de la función de transferencia de ERC-20, tiene el siguiente aspecto en Solidity:

`transfer(address _to, uint256 _value) public returns (bool success)`

La parte de la firma que Seth necesita de esto es `transfer\(*address*, uint\)` y, los parámetros, son la _address_ del destinatario y la cantidad. El contrato debe recibir el monto en representación hexadecimal del número en unidades *wei*, razón por la cual necesitamos esas conversiones.

Ahora, para estimar la utilización del _gas_ de una transferencia de token ERC-20, ejecutamos lo siguiente:

`$ seth estimate $COL1 'transfer(address, uint)'` [`0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359`](https://etherscan.io/address/0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359) `$(seth --to-uint256 $(seth --to-wei 0.1 ether))`

La salida/resultado de este comando es:

`37240`

#### `seth receipt` y `seth tx`

Con `seth receipt` y `seth tx`, podemos consultar todos los detalles imaginables sobre una transacción. Ambos toman un _hash_ de transacción \(`tx`\) como parámetro de entrada. La principal diferencia entre los dos es que el `receipt` ("recibo"), que contiene los resultados de la transacción, solo se construye luego de que se extrae la transacción; mientras que, la salida de `seth tx`, solo contiene los parámetros básicos de la transacción antes de que surta efecto.

Por ejemplo, puedes probarlos ejecutando, primero, una transacción para tener un *hash* de transacción:

`$ seth send $COL1 'transfer(address, uint)'` [ `0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359`](https://etherscan.io/address/0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359) `$(seth --to-uint256 $(seth --to-wei 0.1 ether))`

La salida/resultado de este comando es:

```text
seth-send: Published transaction with 68 bytes of calldata.
seth-send: 0x58ba3980775741aecaf8435646a003bff3395d7d4e00c8f7a32ad1fa0ce64e01
seth-send: Waiting for transaction receipt....
seth-send: Transaction included in block 9704345.
```

Ahora, puedes probar con los comandos antes mencionados \(utiliza tu propio *hash* `tx` del `tx` previo\):

`$ seth receipt 0x58ba3980775741aecaf8435646a003bff3395d7d4e00c8f7a32ad1fa0ce64e01`

`$ seth tx 0x58ba3980775741aecaf8435646a003bff3395d7d4e00c8f7a32ad1fa0ce64e01`

Ambos generan una salida bastante larga, pero podemos filtrar cada consulta con un parámetro adicional opcional. Por ejemplo, veamos qué tan precisa fue nuestra estimación anterior para el consumo de *gas* \(era perfectamente precisa\):

`$ seth receipt 0x58ba3980775741aecaf8435646a003bff3395d7d4e00c8f7a32ad1fa0ce64e01 gasUsed`

La salida/resultado de este comando es:

`37240`

### Recursos Adicionales

Esta guía fue escrita basada en la documentación oficial de Seth en el repositorio de Github. Puedes encontrar información adicional allí: [https://github.com/dapphub/dapptools/tree/master/src/seth](https://github.com/dapphub/dapptools/tree/master/src/seth)

### Errores Conocidos

* Errores con MacOS Mojave