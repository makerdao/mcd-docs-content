# Apagado de Emergencia \(ES\) CLI

**Nivel**: Intermedio

**Tiempo-Estimado**: 30 minutos

## Descripción

El Apagado de Emergencia \(_ES_ en inglés, _Emergency Shutdown_) es el último recurso para proteger al Protocolo de Maker contra una amenaza seria como, pero no limitado a, ataques de gobernanza, irracionalidad de largo plazo del mercado, hackeo y brechas de seguridad. El Módulo de Apagado de Emergencia (_ESM_ en inglés, _Emergency Shutdown Module_) es responsable por la coordinación del Protocolo de Maker y ubicar apropiadamente los colaterales tanto a los usuarios de _vaults_ como a los _holders_ (poseedores) de tokens. Esta guía define los pasos y procedimientos necesarios para chequear, interactuar y activar el _ESM_.

#### **Objetivos de Aprendizaje:** Ser capaz de Chequear, Depositar y Activar el Apagado de Emergencia.

### Tabla de Contenidos

1. Instalación
2. Configuración de la _address_ del contrato
3. Comandos y Explicaciones
   * Chequeando tu balance de MKR
   * Chequeando y configurando tu aprobación de MKR
   * Chequeando la bandera de live\(\)
   * Chequeando el umbral del _ESM_
   * Depositando una cantidad de prueba de MKR al _ESM_
   * Depositando MKR en el _ESM_
   * Chequeando cuánto MKR hay en el _ESM_
   * Chequeando si el _ESM_ ha sido activado
   * Activando el _ESM_

## 1. Instalación

Para interactuar con la _blockchain_ de Ethereum, el usuario necesita instalar seth, una herramienta de líneas de comando que parte del conjunto de herramientas de [Dapp.Tools](https://dapp.tools/). También puedes encontrar más información [sobre su instalación aquí](https://github.com/makerdao/developerguides/blob/master/devtools/seth/seth-guide-01/seth-guide-01.md). Una vez que el usuario ha instalado y configurado correctamente [`seth`](https://dapp.tools/) para utilizar la red principal de Ethereum y la _address_ que posee sus tokens MKR, este puede consultar saldos de contratos, aprobaciones y transferencias.

## 2. Configuración de la _Address_ del Contrato

```text
* El usuario requerirá las siguientes `addresses` de contrato; MCD_END y MCD_ESM son accesibles en [Changelog.makerdao.com](https://changelog.makerdao.com) así como la `address` de contrato de Maker, a ser añadida en lugar de MKR_ADR a continuación, la cual puede ser verificada en [Etherscan](https://etherscan.io/token/0x9f8f72aa9304c8b593d555f12ef6589cc3a579a2).
* Las mismas deberían ser configuradas de la siguiente manera:
```

```text
export MCD_END= 0xab14d3ce3f733cacb76ec2abe7d2fcb00c99f3d5
export MCD_ESM= 0x0581a0abe32aae9b5f0f68defab77c6759100085
export MKR_ADR= <MKR ADDRESS from Etherscan.io>
export MY_ADR= <USER ADDRESS>
    
#example values for depositing into the ESM
export TRIAL_AMOUNT=$(seth --to-uint256 $(seth --to-wei 0.1 eth))
export REMAINING_AMOUNT=$(seth --to-uint256 $(seth --to-wei 50000 eth))
```

## 3. Comandos y Explicaciones

#### Chequeando tu Balance de MKR

Antes de depositar tus MKR en el contrato del _ESM_, primero revisa el balance de tu _address_ de MKR:

```text
seth --from-wei $(seth call $MKR_ADR "balanceOf(address)" $MY_ADR | seth --to-dec)
# 100000.000000000000000000 
```

#### Chequeando y Configurando tu Aprobación de MKR

Para ejecutar las funciones del contrato del token MKR es necesario que esas aprobaciones sean fijadas en el token. El primer trabajo es chequear que se le permita al contrato del _ESM_ hacer retiros de tu _address_:

```text
seth call $MKR_ADR "allowance(address,address)" $MY_ADR $MCD_ESM
# 0x0000000000000000000000000000000000000000000000000000000000000000 -> not allowed
```

Si el contrato del _ESM_ no tiene permitido hacer retiros de tu _address_, lo siguiente puede ser utilizado para fijar la asignación en el token MKR. Esto aprobará al _ESM_ para retirar de la _wallet_ del usuario:

```text
seth send $MKR_ADR "approve(address)" $MCD_ESM
```

A continuación, verificamos nuevamente para confirmar que el _ESM_ puede retirar dinero de la cuenta del usuario.

```text
seth call $MKR_ADR "allowance(address,address)" $MY_ADR $MCD_ESM
# 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff -> allowed
```

[Información del contrato de referencia](https://github.com/dapphub/ds-token/blob/cee36a14685b3f93ffa0332853d3fcd943fe96a5/src/token.sol#L36).

#### Chequeando la bandera 'live'

Los contratos Live tienen `live` = 1, que indica que el sistema está funcionando con normalidad. Por lo tanto, cuando se invoca `cage()`, la bandera se fija en 0.

```text
seth call $MCD_END "live()" | seth --to-dec
# 1 -> system is running normally
```

#### Chequeando el umbral del _ESM_

Para consultar el valor `min` (mínimo), puedes llamar a:

```text
seth --from-wei $(seth call $MCD_ESM "min()" | seth --to-dec)
# 50000.000000000000000000
```

#### Depositando una cantidad pequeña \(0.1 MKR\) en el _ESM_

Para depositar una cantidad pequeña de MKR en el contrato del _ESM_ para probar que la función de depósito se esté ejecutando correctamente, se utiliza la función `join` y se especifica una cantidad pequeña.

```text
seth send $MCD_ESM "join(uint256)" $TRIAL_AMOUNT 
```

#### Chequeando cuánto MKR hay en el _ESM_

Para consultar la cantidad total de MKR que ha sido añadida al _ESM_ debemos llamar a la función `Sum()`.

```text
seth --from-wei $(seth call $MCD_ESM "Sum()" | seth --to-dec)
# 50050.000000000000000000
```

#### Chequeando cuánto MKR has incluido al _ESM_

Para consultar cuánto MKR has incluido al ESM, puedes llamar a `sum()` (con minúscula) con la _address_ del usuario como argumento:

```text
seth --from-wei $(seth call $MCD_ESM "sum(address)" $MY_ADR | seth --to-dec)
# 50.000000000000000000
```

#### Depositando (por ejemplo) 50.000 MKR al _ESM_

Para depositar MKR al contrato del _ESM_ se puede utilizar la función `join` y especificar la cantidad

```text
seth send $MCD_ESM "join(uint256)" $REMAINING_AMOUNT 
```

Por favor, especifica la cantidad de MKR que pretendes depositar al _ESM_.

#### Chequeando si el _ESM_ ha sido activado

Para confirmar que el Apagado de Emergencia ha sido activado, la función `fired()` puede ser llamada, la cual regresará un booleano.

```text
seth call $MCD_ESM "fired()" | seth --to-dec
# 0 -> ES has not been triggered
```

#### Activando el _ESM_

Para que el Apagado de Emergencia sea activado, se requiere que `Sum()` sea más grande que `min()`. Solo entonces la función `fire()` puede ser ejecutada de manera exitosa.

```text
seth send $MCD_ESM "fire()"
```

**Nota:** Si la activación del _ESM_ no es exitosa, asegúrate de que el _gas_ esté fijado a un nivel apropiado.

**Nota:** La activación del _ESM_ **no** es tomada a la ligera; para una explicación más detallada sobre sus implicaciones, por favor lee la documentación listada a continuación:

#### Links

* [ESM.sol](https://github.com/makerdao/esm/blob/master/src/ESM.sol)
* [Más Documentación sobre el _ESM_](https://www.notion.so/makerdao/Emergency-Shutdown-Module-3073acf244404f7f98b5e47d2efc7ba9)
* [Documentación sobre *End*](https://www.notion.so/makerdao/End-Detailed-Documentation-1874a49064644c51aa34fb9c303eda90)
