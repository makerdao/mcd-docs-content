# Redención de DAI y Colateral durante el Apagado de Emergencia 

**Nivel**: Intermedio

**Tiempo Estimado:** 60 minutos

### Descripción

Esta guía describe cómo los usuarios pueden interactuar con el Protocolo de Maker a través de contratos _proxy_ para redimir DAI y cualquier exceso de colateral si el sistema de Maker ha entrado en un Apagado de Emergencia. Definiremos el proceso de configuración, incluyendo la configuración del contrato _proxy_ , seguido de las llamadas a _seth_ para redimir colateral, como _holder_ de DAI y liberar el exceso de colateral como dueño de una _Vault_. 

### Objetivos de Aprendizaje

Para redimir DAI y/o exceso de colateral en caso de un Apagado de Emergencia 

### Tabla de Contenidos

**Proceso de Configuración**

1. Instalación
2. Configuración de Dirección de Contrato 

**_Holders_ de DAI para Redimir Colateral** 

1. Comprobar los _holdings_ del usuario de DAI  
2. Aprobar un _Proxy_ 
3. Crear _Calldata_ ("datos de llamada")
4. Ejecutar _Calldata_ utilizando el contrato _MYPROXY_
5. Llamar a las funciones _cashETH_ o _cashGEM_
6. Usar _cashETH_
7. Definir _calldata_ para nuestra función
8. Ejecutar _cashETHcalldata_
9. Alternativa del paso (6), utilizando _cashGEM_
10. Definir _calldata_ para nuestra función 
11. Llamar a ejecutar en _MYPROXY_

**Propietarios de _Vaults_ para Redimir el Exceso de Colateral** 

1. Estado del _Holder_ de la _Vault_ 
2. Redimir ETH utilizando la funcion _freeETH_  

    2.1. Establecer _Call Data_ 

    2.2 Ejecutar esta _calldata_
3. Redimir ETH utilizando la función _freeGEM_

3.1 Establecer _Calldata_

3.2 Ejecutar esta _calldata_

**Conclusión**

### Proceso de Configuración  

#### **1. Instalación**

Para poder interactuar con la _blockchain_ de Ethereum, el usuario necesita instalar _seth_, una herramienta _command line_, como parte del conjunto de herramientas de [Dapp.Tools](https://dapp.tools). [Aquí](https://github.com/makerdao/developerguides/blob/master/devtools/seth/seth-guide-01/seth-guide-01.md) puedes encontrar más información sobre la instalación. Una vez que el usuario ha instalado y configurado **`[seth](<https://dapp.tools/>)`** correctamente para usar la red principal de Ethereum y la dirección que _holdea_ sus MKR, se pueden consultar los balances del contrato, aprobaciones y transferencias. 

#### **2. Configuración de la Dirección del Contrato**

El usuario necesitará las siguientes direcciones de contrato, que se muestran como direcciones de la red principal. El resto de las direcciones de red principal o la red de prueba son accesibles en [changelog.makerdao.com](https://changelog.makerdao.com), y pueden ser verificadas en [Etherscan](https://etherscan.io/token/0x9f8f72aa9304c8b593d555f12ef6589cc3a579a2). De forma similar, en el [End contract](https://github.com/makerdao/dss/blob/master/src/end.sol) y en el [Proxy\_Actions\_End contract](https://github.com/makerdao/dss-proxy-actions/blob/master/src/DssProxyActions.sol#L793), se puede encontrar información adicional sobre los comandos descritos a continuación. Estos deben configurarse de la siguiente manera y pegarse en la terminal, linea por linea:        

```
export DAI=0x6B175474E89094C44Da98b954EedeAC495271d0F
export PROXY_ACTIONS_END=0x069B2fb501b6F16D1F5fE245B16F6993808f1008
export MCD_END=0xaB14d3CE3F733CACB76eC2AbE7d2fcb00c99F3d5
export CDP_MANAGER=0x5ef30b9986345249bc32d8928B7ee64DE9435E39 
export PROXY_REGISTRY=0x4678f0a6958e4D2Bc4F1BAF7Bc52E8F3564f3fE4
export MCD_JOIN_ETH=0x2F0b23f53734252Bda2277357e97e1517d6B042A
export MCD_JOIN_BAT=0x3D0B1912B66114d4096F48A8CEe3A56C231772cA
export MCD_JOIN_DAI=0x9759A6Ac90977b93B58547b4A71c78317f391A28

export MYPROXY=$(seth call $PROXY_REGISTRY 'proxies(address)(address)' $ETH_FROM) 
# Esto crea una address de proxy única llamando al registro de proxy utilizando la adress de Ethereum del usuario.

export ilk=$(seth --to-bytes32 $(seth --from-ascii ETH-A))
export ilkBAT=$(seth --to-bytes32 $(seth --from-ascii BAT-A))
# Aquí hemos definido dos ilk (tipos de colateral) ETH y BAT. 
# El número de tipos de ilk necesitados dependerá de las vaults de ese tipo de colateral que haya abierto el usuario.

export ETH_GAS=4000000
export ETH_GAS_PRICE=2500000000
# Normalmente, los costos de gas son incrementados cuando se trata con contratos de proxy para prevenir un fallo en la transacción.

export cdpId=$(seth --to-dec $(seth call $CDP_MANAGER 'last(address)' $MYPROXY))
# Esta es una llamada al CDP Manager responsable de hacer la ID de CDP de los usuarios. 
# Nota, si el usuario ha creado múltiples vaults, tendrá que tener múltiples IDs de CDP, todas las cuales deben estar referenciadas para retirar colateral.
```

### _Holders_ de DAI para Redimir Colateral  

Hay dos funciones que se deben llamar para poder recuperar el colateral final. El primer paso es `pack` y el segundo paso es `cashETH` o `cashGem` dependiendo de la cantidad sobrante de cada tipo de colateral en el sistema.

Depositar tokens DAI en el sistema puede hacerse utilizando la biblioteca de contratos `PROXY_ACTIONS_END` y la función `pack`. Esta función agrupa eficientemente tres parámetros, incluyendo el adaptador `Dai(join)`, el contracto `end` y la cantidad de tokens DAI que deseas canjear por el colateral permitido de una sola vez.

```
function pack(
        address daiJoin,
        address end,
        uint wad
    ) public {
        daiJoin_join(daiJoin, address(this), wad);
        VatLike vat = DaiJoinLike(daiJoin).vat();
        // Aprueba a end para sacar DAi del balance del proxy en el vat
        if (vat.can(address(this), address(end)) == 0) {
            vat.hope(end);
        }
        EndLike(end).pack(wad);
    }
```

#### 1. Comprobar los _holdings_ de los usuarios de DAI  

El usuario puede revisar su balance de tokens DAI y subsecuentemente guardarlos en el `wad` variable para poder utilizarlo posteriormente en la función _proxy_.

```
export balance=$(seth --from-wei $(seth --to-dec $(seth call $DAI 'balanceOf(address)' $ETH_FROM)))
export wad=$(seth --to-uint256 $(seth --to-wei 13400 eth))
# Arriba, 13400 es un ejemplo de un balance de DAI
```

#### 2. Aprobar un _Proxy_ 

Los usuarios necesitan aprobar `MYPROXY` para poder retirar DAI desde sus _wallets_ utilizando la siguiente función

```
seth send $DAI 'approve(address,uint)' $MYPROXY $(seth --to-uint256 $(mcd --to-hex -1))
```

#### 3. Crear _Calldata_ ("Datos de Llamada")

A continuación es necesario agrupar las definiciones de función y parámetros que el usuario necesita ejecutar. Esto se hace preparando una llamada a la función `MYPROXY`, definida como `calldata.`

```
export calldata=$(seth calldata 'pack(address,address,uint)' $MCD_JOIN_DAI $MCD_END $wad)
.
.
.
# 0x33ef33d6000000000000000000000000fc0b3b61407cdf5f583b5b1e08514e68ecee4a73000000000000000000000000d9026db5ca822d64a6ba18623d0ff2bb07ad162c0000000000000000000000000000000000000000000002d66a5b4bc1da600000
```

#### 4. Ejecutar _calldata_ usando el contrato `MYPROXY` 

El usuario es capaz de llamar la función `execute` y utilizar la función `PROXY_ACTIONS_END.pack()` dentro del entorno de `MYPROXY`. Esto permite al _proxy_ tomar tokens DAI de la _wallet_ de los usuarios en la dirección del _proxy_ y depositarlos en el contrato `end`, donde una cantidad proporcional de colateral puede ser reclamado más adelante. Una vez que el DAI es empaquetado, no puede ser desempaquetado.     

```
seth send $MYPROXY 'execute(address,bytes memory)' $PROXY_ACTIONS_END $calldata
# [ejemplo](<http://ethtx.info/kovan/0x8f4021e46b1a6889ee7045ba3f3fae69dee7ef130dbb447d4cc724771e04bcd6>) transacción mostrando acciones relativas al empaquetado de los Dai del usuario.
```

#### 5. Llamar a las funciones `cashETH` o `cashGEM`

Los usuarios serán capaces de retirar colateral dependiendo del colateral que esté en el VAT al momento del apagado. Por ejemplo, 1 DAI será capaz de reclamar una porción de ETH y de VAT (y cualquier otro colateral aceptado) que cuando se combinen tendrá un valor de 1 USD. Este proceso es completado llamando a `cashETH` o `cashGEM`.

#### 6. **Usando _`cashETH`_**

La siguiente función `cashETH` es referenciada como parte de la función `calldata` y debe ser referenciada [aquí](https://app.gitbook.com/s/-LtJ1VeNJVW-jiKH0xoL/clis/dai-and-collateral-redemption-during-emergency-shutdown).

```
function cashETH(
        address ethJoin,
        address end,
        bytes32 ilk,
        uint wad
    ) public {
        EndLike(end).cash(ilk, wad);
        uint wadC = mul(wad, EndLike(end).fix(ilk)) / RAY;
        // Saca una cantidad de WETH a la address proxy como un token
        GemJoinLike(ethJoin).exit(address(this), wadC);
        // Convierte WETH a ETH
        GemJoinLike(ethJoin).gem().withdraw(wadC);
        // Envía ETH de vuelta a la wallet del usuario
        msg.sender.transfer(wadC);
    }
```

#### 7. Definir _calldata_ para nuestra función 

Después, definimos nuevamente la _calldata_ para nuestra función al agrupar los parámetros `cashETH` mostrados anteriormente.

```
export cashETHcalldata=$(seth calldata 'cashETH(address,address,bytes32,uint)' $MCD_JOIN_ETH $MCD_END $ilk $wad)
```

#### 8. Ejecutar `cashETHcalldata`

Finalmente, ejecutando el  `cashETHcalldata` en la función `execute` del contrato `MYPROXY` del usuario, canjeará ETH por DAI y colocará este ETH en la _wallet_ ETH del usuario. 

```
seth send $MYPROXY 'execute(address,bytes memory)' $PROXY_ACTIONS_END $cashETHcalldata
# [ejemplo](<http://ethtx.info/kovan/0x323ab9cd9817695089aea31eab369fa9f3c9b1a64743ed4c5c1b3ec4d7218cf8>) Transacción exitosa
```

#### 9. Alternativas al paso (6), utilizando **`cashGEM`**

También es posible utilizar la función `cashGEM` para poder canjear diferentes tipos de colateral. En el siguiente ejemplo referenciamos a _gemJoin_ en relación con BAT.

```
function cashGem(
        address gemJoin,
        address end,
        bytes32 ilk,
        uint wad
    ) public {
        EndLike(end).cash(ilk, wad);
        // Saca una cantidad de tokens de la wallet del usuario como un token
        GemJoinLike(gemJoin).exit(msg.sender, mul(wad, EndLike(end).fix(ilk)) / RAY);
    }
```

#### 10. Define _calldata_ para nuestra función

De forma similar a como se hace en el paso (7), el usuario necesita definir la _calldata_ para interactuar con _`cashGEM`_.

```
export cashBATcalldata=$(seth calldata 'cashETH(address,address,bytes32,uint)' $MCD_JOIN_BAT $MCD_END $ilkBAT $wad)
```

#### 11. Llamar a la ejecución en `MYPROXY`

Finalmente, ejecutar el `cashBATcalldata` en la función `execute` del contrato `MYPROXY` del usuario canjeará BAT por DAI y colocará este BAT en la _wallet_ ETH del usuario. 

```
seth send $MYPROXY 'execute(address,bytes memory)' $PROXY_ACTIONS_END $cashBATcalldata
```

### Propietarios de _Vaults_ para Canjear el Exceso de Colateral  

A sí mismo, el dueño de una _vault_ puede usar las funciones de acciones _proxy_  `freeETH` o `freeGEM` para recuperar cualquier exceso de colateral que puedan tener bloqueado en el sistema. 

### 1. Estado del _Holder_ de la _Vault_


Hay que tener en cuenta algunas limitaciones para los _holders_ de _vaults_ . Por ejemplo, si la _vault_ de un usuario esta sub-colateralizada, entonces no tendrán ningún exceso de colateral que reclamar. Asímismo, si la _vault_ del usuario actualmente está en una subasta flip al momento de un Apagado de Emergencia, será necesario para el _holder_ de la _vault_ cancelar la subasta llamando **`skip(ilk, id)`** antes de llamar **`free__()`**.

De forma similar, estas funciones han sido completadas usando llamadas al contrato _proxy_ de Maker. Pueden haber otros escenarios en los que _front ends_ de terceros tales como InstaDApp tienen sus propios proxies, que van a requerir que los usuarios salgan de su _proxy_ para poder utilizar lo siguiente.    

### 2. Canjear ETH usando la funcion`freeETH`    

```
function freeETH(
        address manager,
        address ethJoin,
        address end,
        uint cdp
    ) public {
        uint wad = _free(manager, end, cdp);
        // Saca una cantidad de WETH a la address proxy como un token
        GemJoinLike(ethJoin).exit(address(this), wad);
        // Convierte WETH a ETH
        GemJoinLike(ethJoin).gem().withdraw(wad);
        // Envía ETH de vuelta a la wallet del usuario
        msg.sender.transfer(wad);
    }
```

#### 2.1. Establecer _calldata_

Dependiendo de cuántas _vaults_ tenga el usuario, será necesario repetir este proceso para cada ID de _vault_.

```
export freeETHcalldata=$(seth calldata 'freeETH(address,address,address,uint)' $CDP_MANAGER $MCD_JOIN_ETH $MCD_END $cdpId )
```

#### 2.2. Ejecuta esta _calldata_

Ejecutar el contrato `MYPROXY` redimirá ETH y lo colocará en la dirección del usuario. 

```
seth send $MYPROXY 'execute(address,bytes memory)' $PROXY_ACTIONS_END $freeETHcalldata
```

### 3. Canjear ETH utilizando la función `freeGEM`   

```
function freeGem(
        address manager,
        address gemJoin,
        address end,
        uint cdp
    ) public {
        uint wad = _free(manager, end, cdp);
        // Saca una cantidad de WETH a la address proxy como un token
        GemJoinLike(gemJoin).exit(msg.sender, wad);
    }
```

#### 3.1. Establecer _calldata_ 

Dependiendo de cuántas _vaults_ tenga el usuario, será necesario repetir este proceso para cada ID de _vault_. 

```
export freeBATcalldata=$(seth calldata 'freeETH(address,address,address,uint)' $CDP_MANAGER $MCD_JOIN_BAT $MCD_END $cdpId )
```

#### 3.2. Ejecutar esta _calldata_

La ejecución del contrato `MYPROXY` canjeará BAT (u otros tipos de colateral) y los colocará en la dirección del usuario.  

```
seth send $MYPROXY 'execute(address,bytes memory)' $PROXY_ACTIONS_END $freeBATcalldata
```

### Conclusión

Lo anterior resume cómo canjear DAI y el exceso de colateral de la _vault_ utilizando el _command line_.

En resumen, hemos mostrado cómo comprobar tus _holdings_ de DAI, cómo aprobar un _proxy_ para retirar DAI desde tu _wallet_ y después usar las funciones `cashETH/GEM` para retirar colateral en la _wallet_ ETH del usuario, utilizando el contrato `MYPROXY`. Para dueños de _vaults_, mostramos cómo redimir colateral usando el contrato `MYPROXY` y la función `freeGEM`.   

En el caso de un Apagado de Emergencia, prevemos que seguirá siendo posible vender DAI en el mercado abierto así como hacer uso de _keepers_ de redención incentivados económicamente para satisfacer las necesidades del mercado, tanto para los propietarios de DAI como para los _holders_ de _vaults_.  