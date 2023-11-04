# Dai Multi-Colateral \(MCDs\) CLI

## Instalación

Primero, instala [dapp tools](https://dapp.tools/):

```text
$ curl https://dapp.tools/install | sh
```

Entonces, instala el paquete `mcd`:

```text
$ dapp pkg install mcd
```

####  La siguiente lista detalla todos los comandos disponibles cuando se interactúa con la interfaz de línea de comandos:

```text
MCD - Multi-collateral Dai

Utilización: mcd [<options>] <command> [<args>]
   o: mcd help [<command>]

Comandos:

   bite            Activa la liquidación para una Urn insegura
   bites           'Mordidas' recientes
   cdp             Administración de _CDP_
   dai             Administración de Dai
   debt            Emisión total de Dai
   drip            Activa la acumulación de la tarifa de estabilidad
   flap            Activa las subastas flap
   flips           Vista de las subastas flip y kick-off
   flog            Libera deudas incobrables en cola para la subasta
   flop            Activa las subastas flop
   frob            Administración de urn
   frobs           Frobs recientes
   gem             Administración de colaterales
   help            Imprime la ayuda para el mcd o uno de sus subcomandos
   ilk             Parámetros del ilk (tipo de colateral)
   line            Techo de deuda total
   live            Estado del Live de la bandera
   poke            Actualiza el precio 'spot' para un ilk determinado
   unwrap          Desenrrolla WETH a ETH
   urn             Estado del CDP
   vice            Deuda incobrable total
   vow             Balances del liquidador
   wrap            Enrrolla ETH a WETH
```

### Configuración

MCD está construido en [Seth](https://github.com/dapphub/dapptools/tree/master/src/seth) y utiliza la mismas opciones de configuración de red, las cuales (como en Seth) pueden ser definidas en el archivo de inicialización `~/sethrc`.

Similar a Seth, `mcd` también admite la firma de transacciones con _hardware wallets_ Ledger y puede  correr contra nodos, tanto locales como remotos.

Dado que `mcd` siempre será utilizado contra un despliegue de sistema conocido, los valores predeterminados se pueden cargar siempre que sea posible. En la mayoría de los casos, el único parámetro de configuración requerido es la opción `-C, --chain=<chain>` \(`MCD_CHAIN`\) y la cuenta remitente `-F, --from=<address>` \(`ETH_FROM`\) cuando no se utilice una _testnet_ (red de prueba).

Ejemplo `~/.sethrc`:

```text
#!/usr/bin/env bash
export ETH_FROM=0x4Ffa8667Fe2db498DCb95A322b448eA688Ce430c
export MCD_CHAIN=kovan
```

**Kovan**

Corre contra el último despliegue de Kovan configurando la opción `-C, --chain` a `kovan`. Especifica una cuenta remitente cuando envíes transacciones utilizando la opción `-F, --from`, o a través de la variable env `ETH_FROM`.

```text
$ export ETH_FROM=0x4Ffa8667Fe2db498DCb95A322b448eA688Ce430c
$ mcd --chain=kovan dai join 100
```

**_Testchain_ (cadena de prueba) Remota**

Corre contra los despliegues de la _testchain_ remota configurando la opción `-C, --chain` a la _testchain_ remota. MCD configurará automáticamente la cuenta a través de la API de la _testchain_ de manera que no se necesiten más configuraciones. Para ver la lista de _testchains_ disponibles para correr, puedes utilizar:

```text
$ mcd testnet chains`
```

Entonces, cambia la opción de la _chain_, o la variable env de la _chain_, para apropiarse del ID de la _testchain_.

```text
$ export MCD_CHAIN=12899149080555595289
$ mcd dai join 100
```

**_Testnet_ (red de prueba) Local**

Corre contra una instancia que esté corriendo localmente de [Dapp testnet](https://github.com/dapphub/dapptools/tree/master/src/dapp) donde el sistema ha sido desplegado configurando la opción `C, --chain` a `testnet`. MCD configurará automáticamente pruebas de cuenta para `dapp testnet` de manera que no sea necesaria ninguna otra actualización.

De manera predeterminada, MCD asume que la salida del _script_ de implementación de la _testchain_ está disponible en `~/.dapp/testnet/8545/config/addresses.json`. Las _addresses_ de configuración pueden ser cargadas desde una ubicación diferente configurando la opción `--config` \(`MCD_CONFIG`\).

```text
$ export MCD_CONFIG=~/testchain-deployment-scripts/out/addresses.json
$ mcd -C testnet dai join 100
```

### Ilk

Los *ilks* son tipos de colaterales con con parámetros de riesgo correspondientes que han sido aprobados por el sistema de gobernanza. Utiliza los comandos de `ilks` para ver la lista de links disponibles.

```text
$ mcd ilks
ILK      GEM    DESC

ETH-A    WETH   Ethereum
ETH-B    WETH   Ethereum
REP-A    REP    Augur
```

Cada *ilk* tiene su propio conjunto de parámetros de configuración que pueden ser vistos a través del comando `ilk`. La opción`I, --ilk=<id>` es utilizada para buscar comandos para un *ilk* en particular:

```text
$ mcd --ilk=ETH-A ilk
Art  40.000000000000000000                      Deuda total (DAI)
rate 1.000080370887129123082627939              WETH Tasa de cambio del DAI
spot 99.333333333333333333333333333             WETH precio con alfombra de seguridad (USD)
line 1000.0000000000000000000000000000000000000 Techo de deuda (DAI)
dust 0.0000000000000000000000000000000000000000 Piso de deuda (DAI)
flip 0x9d905effff127a01da3b38124f8da88e766eb8dd Contrato de subasta flip
chop 1.000000000000000000000000000              Pena de liquidación
lump 10000.000000000000000000                   Tamaño de lote de subasta flip
tax  1.000000000782997609082909351              Tarifa de estabilidad
rho  1552802862                                 Marca de tiempo del último goteo
pip  0x98312e16f5b2c0def872a1f7484a8456e5a67a3b Contrato de _feed_ de precios
mat  1.500000000000000000000000000              Ratio de liquidación
```

Los valores individuales de los *ilk* pueden ser regresados al añadir el nombre del parámetro como un argumento en el comando `ilk`:

```text
$ mcd --ilk=ETH-A ilk spot
99.333333333333333333333333333
```

### *Gem*

*Gems* son tokens de colateral. El colateral es incorporado y removido de los adaptadores 'via', que abstraen las diferencias entre varios comportamientos de token. Utiliza `gem [<subcommand>]` para administrar los balances de colateral de cualquier *ilk*.

```text
gem --ilk=<id> symbol             Símbolo del GEM, por ejemplo: WETH
gem --ilk=<id> balance            Imprime los balances de una Urn determinada (predeterminado: ETH_FROM)
gem --ilk=<id> join <wad>         Incorporar colateral a una Urn en particular (predeterminado: ETH_FROM)
gem --ilk=<id> exit <wad> [<guy>] Remueve el colateral de una Urn (predeterminado: ETH_FROM)
```

El comando `join` puede incorporar colateral de una cuenta remitente a cualquier Urn especificada. El comando `exit` puede removar colateral de una Urn especificada, siempre que el remitente controle la llave privada asociada de esa Urn.

De manera predeterminada, `ETH_FROM` es utilizado para determinar qué Urn debería ser acreditada con colateral. Utiliza `U, --urn=<address>` para acreditar opcionalmente una Urn diferente a la establecida predeterminadamente.

```text
$ mcd --ilk=ETH-A --urn=0x123456789abcdef0123456789abcdef012345678 join 100
```

El comando `exit` puede remover colaterales de una Urn específica, siempre que el remitente controle la llave privada asociada con esa Urn. El comando `exit` también puede retirar colateral de una cuenta diferente a `ETH_FROM` pasando la _address_ como un argumento adicional:

```text
$ mcd --ilk=ETH-A exit 100 0xDecaf00000000000000000000000000000000000
```

### *Urn*

*Urns* representa el estado del CDP/vault para cualquier _address_ de *Urn*.

Utiliza el comando `urn` para ver el estado de *Urn* para cualquier *ilk*:

```text
ilk  ETH-A                                      Tipo de colateral
urn  0xC93C178EC17B06bddBa0CC798546161aF9D25e8A Manipulador de la urn
ink  45.000000000000000000                      Colateral Bloqueado (WETH)
art  120.000000000000000000                     Deuda emitida (Dai)
tab  120.000244107582797248312544980            Deuda pendiente (Dai)
rap  0.000244107582797248312544980              Tarifa de estabilidad acumulada (Dai)
-->  37.24                                      Ratio de colateralización

spot 99.333333333333333333333333333             Precio de la manta de seguridad (USD) de WETH
rate 1.000002034229856643749638820              Tasa de Cambio de DAI de WETH
```

De manera predeterminada, `ETH_FROM` es utilizado para determinar qué Urn consultar. Utiliza la opción `U, --urn=<address>` para consultar Urns en otros índices.

**Administración de Urn**

El estado de una *Urn* \(`urn.ink` y `urn.art`\) es administrado a través del comando `frob <dink> <dart>`, donde `dink` y `dart` son cantidades delta por las que `ink` \(colateral bloqueado\) y `art` \(deuda pendiente\) deben ser cambiados. Por ejemplo: para bloqueado 100 WETH y retirar 400 Dai en el *ilk* ETH-A:

```text
$ mcd --ilk=ETH-A frob 100 400
```

Para reducir la deuda pendiente en 200 Dai manteniendo constante la cantidad de colateral bloqueado:

```text
$ mcd --ilk=ETH-A frob -- 0 -200
```

### Dai

Similar a los adaptadores GEM, los adaptadores Dai son utilizados para intercambiar *Vat* Dai por el token Dai de ERC20, el cual puede, entonces, ser utilizado fuera del sistema. Utiliza `dai [<subcommand>]` para administrar los balances de Dai.

```text
dai balance    Imprime los balances para una Urn determinada (predeterminado: ETH_FROM)
dai join <wad> Intercambia DSToken Dai por Vat Dai
dai exit <wad> Intercambia Vat Dai por DSToken Dai
```

Una vez que el Dai ha sido llevado a una Urn, puede ser retirado para ser utilizado fuera del sistema utilizando `dai exit`. El Dai puede ser devuelto para pagar deuda de una Urn a través de `dai join`.

El comando `dai balance` muestra el balance del sistema interno \(vat\) y el balance de token externo \(ext\):

```text
$ mcd dai balance
vat 1030.003120998308631176024235912000000000000000000 Balance de vat
ext 0.000000000000000000 Balance de ERC20
```

Los valores de saldo individuales se pueden recuperar agregando `vat` o `ext` como argumento al comando `balance`:

```text
$ mcd dai balance vat
1030.003120998308631176024235912000000000000000000
```

### Cdp

El comando `cdp` provee compatibilidad con las CDPs administradas a través del Portal de CDP y utiliza el mismo contrato _proxy_ y _front-end_ del [Administrador de CDP](https://github.com/makerdao/dss-cdp-manager). Esto permite que las CDPs sean administradas a través de un identificador íntegro único en lugar de las opciones `I, --ilk` y `U, --urn`.

```text
Utilización: mcd cdp [<id>] [<command>]

Comandos: ls [<owner>]     Lista de Cdp
          count [<owner>]  Conteo de Cdp
          open             Abre una nueva Cdp
          <id> urn         Estado de Cdp
          <id> lock <wad>  Une y bloquea colateral
          <id> free <wad>  Libera y retira colateral
          <id> draw <wad>  Saca y retira dai
          <id> wipe <wad>  Une y borra dai
```

### Ejemplos

Nota: los ejemplos asumen que `ETH_FROM` está fijado a una _address_ controlada por el usuario, y que la variable env `MCD_CHAIN` ha sido fijada a un identificador de cadena válido.

#### 1. Urn Nativa - bloquea 100 ETH y saca 500 Dai

Nota: El sistema no maneja ETH directamente, en su lugar utiliza WETH para representar el colateral ETH. Por cuestiones de conveniencia, los comandos `wrap` y `unwrap` son proveidos por intercambiar ETH a WETH y viceversa.

```text
# i) Envuelve
$ mcd wrap 100
eth  900.000000000000000000
weth 100.000000000000000000

# ii) join del Gem
$ mcd --ilk=ETH-A gem join 100
$ Grant approval to move WETH to the Vat? [Y/n]: Y
vat 100.000000000000000000 Colateral libre (WETH)
ink   0.000000000000000000 Colateral bloqueado (WETH)
ext   0.000000000000000000 Balance de la cuenta externa (WETH)
ext 900.000000000000000000 Balance de la cuenta externa (ETH)

# iii) Bloquea y retira
$ mcd --ilk=ETH-A frob 100 500
ilk  ETH-A                                      Collateral type
urn  0xC93C178EC17B06bddBa0CC798546161aF9D25e8A Manejador de la Urn
ink  100.000000000000000000                     Colateral bloqueado (WETH)
art  500.000000000000000000                     Deuda emitida (Dai)
tab  500.000244107582797248312544980            Deuda pendiente (Dai)
rap  0.000244107582797248312544980              Tarifa de estabilidad acumulada (Dai)
-->  19.86                                      Ratio de colateralización

# iv) Retira Dai
$ mcd dai exit 500
vat 0.000060682318362511884962000000000000000000000 Vat balance
ext 500.000000000000000000 ERC20 balance
```

#### 2. CPD Administrada - bloquea 100 REP y saca 50 Dai

```text
# i) Abre
$ mcd --ilk=REP-A cdp open
mcd-cdp-open: Waiting for transaction receipt...
0x800e5578d3ac4b77b7ada1aba48cf80d0d238d4392d2676d79159eac2c2cdd73
Opened: cdp 19

# ii) Bloquea
$ mcd --ilk=REP-A cdp 19 lock 100
seth-send: Published transaction with 260 bytes of calldata.
seth-send: 0x4d30cb4863ca997d24ff2346c9a92e86648369ce7b4a86ed004c73b8d4ef299a
seth-send: Waiting for transaction receipt...
seth-send: Transaction included in block 333.
ilk  REP-A                                      Collateral type
urn  0x4518c4709a50C915b7996A0e6Dfb38c67248BBcF Manejador de la Urn
ink  100.000000000000000000                     Colateral bloqueado (REP)
art  0.000000000000000000                       Deuda emitida (Dai)
tab  0                                          Deuda pendiente (Dai)
rap  0                                          Tarifa de estabilidad acumulada (Dai)
-->  0                                          Ratio de colateralización

# iii) Retira
$ mcd --ilk=REP-A cdp 19 draw 500
seth-send: Published transaction with 260 bytes of calldata.
seth-send: 0xd5fb7ddf94bb910fbba2af118ecde88a03a13129b2e1979238236afe672781c3
seth-send: Waiting for transaction receipt...
seth-send: Transaction included in block 335.
ilk  REP-A                                      Collateral type
urn  0x4518c4709a50C915b7996A0e6Dfb38c67248BBcF Manejador de la Urn
ink  100.000000000000000000                     Colateral bloqueado (REP)
art  49.999505439113270178                      Deuda emitida (Dai)
tab  50.000000000000000000000000000             Deuda pendiente (Dai)
rap  0.000494560886729822000020743              Tarifa de estabilidad acumulada(Dai)
-->  16.66                                      Ratio de colateralización
```
