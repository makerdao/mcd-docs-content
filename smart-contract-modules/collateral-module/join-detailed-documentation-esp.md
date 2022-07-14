# Join - Documentación Detallada 

* **Nombre del Contrato:** join.sol
* **Tipo/Categoría:** DSS —> Modulo Adaptador de Token 
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/join.sol)
* **Etherscan**
  * ****[**Join Dai** ](https://etherscan.io/address/0x9759a6ac90977b93b58547b4a71c78317f391a28)
  * ****[**Join Eth**](https://etherscan.io/address/0x2f0b23f53734252bda2277357e97e1517d6b042a)
  * ****[**Join Bat**](https://etherscan.io/address/0x3d0b1912b66114d4096f48a8cee3a56c231772ca)

## 1. Introducción (Resumen)

Join consiste en tres contratos inteligentes: `GemJoin`, `ETHJoin`, y `DaiJoin:` 

`GemJoin` - permite depositar tokens ERC20 estándar para su uso en el sistema.

 `ETHJoin` - permite que Ether nativo sea utilizado en el sistema. 
 
`DaiJoin` - permite a los usuarios retirar su Dai del sistema en un token ERC20 estándar.  

Cada contrato `join` es creado específicamente para permitir que un tipo de token determinado pueda unirse al `vat`. Debido a esto, cada contrato `join` tiene una lógica ligeramente diferente para tener en cuenta los distintos tipos de tokens dentro del sistema. 

![Las Interacciones de Join con el Protocolo de Maker](<../../.gitbook/assets/Screen Shot 2019-11-17 at 2.05.06 PM.png>)

## 2. Detalles del Contrato:

### Glosario (Join)

* `vat` - almacenamiento de la dirección de `Vat`.
* `ilk` - ID del _Ilk_ para el que un `GemJoin` fue creado. 
* `gem` - la dirección del `ilk` por transferir.
* `dai` - la dirección del token `dai`.
* `one` - un uint 10^27 usado para matemáticas en `DaiJoin`.
* `live` - una bandera de acceso para el adaptador `join`.
* `dec` - decimales para Gem.
 
Cada contrato `join` tiene 4 funciones públicas: constructor, `join`, `exit` y `cage`. El constructor es utilizado en la iniciación del contrato y establece las variables principales de ese contrato `join`. `Join` y `exit` hacen honor a sus nombres. `Join` provee un mecanismo para que los usuarios puedan agregar el token determinado al `vat`. Cada variante tiene una lógica ligeramente distinta, pero generalmente se reduce una `transferencia` y a una llamada de función en el `vat`.  `Exit` es muy similar, pero permite que el usuario pueda retirar el token deseado del `vat`. `Cage` permite vaciar el adaptador (permite que los tokens salgan, pero no que entren). 

## 3. Mecanismos y Conceptos Clave

El contrato `GemJoin` sirve a un propósito muy específico y singular, que está relativamente abstraído del resto del sistema central de _smart contracts_. Cuando un usuario desea entrar en el sistema e interactuar con los contratos `dss`, debe llamar a `exit` para dejar el sistema y tomar sus tokens. Cuando el `GemJoin` es _"`cage`'d"_ (enjaulado) por una dirección _"`auth`'ed"_ (autorizada), puede _`exit`_ (salir) del colateral del `vat` pero ya no puede utilizar _`join`_ para unirse a un nuevo colateral.    

Los balances de usuario para colaterales de tokens agregados al sistema mediante `join` son contabilizados en el `Vat` como `Gem` de acuerdo al tipo de colateral `Ilk`, hasta que se conviertan en tokens de colateral bloqueados (`ink`) para que el usuario pueda retirar DAI.

El contrato `DaiJoin` tiene un propósito similar. Dirige el intercambio de DAI que es rastreado en el `Vat` y el DAI ERC-20 que es rastreado por `Dai.sol`. Después que un usuario retira DAI en contra sus colaterales, tendrán un saldo en `Vat.dai`. Este saldo en DAI puede ser _"`exit`' ed"_ (sacado) del Vat, usando el contrato `DaiJoin` que mantiene el saldo de `Vat.dai` y la acuñación de DAI ERC-2O. Cuando un usuario quiere mover sus DAI nuevamente al sistema contable `Vat` (para pagar deuda, participar en subastas, utilizar el _DSR_, etc.) deberá llamar a `DaiJoin.join`. Al llamar a `DaiJoin.join` se utiliza`burn` para quemar el DAI ERC-20 y se transfiere `Vat.dai` del saldo de `DaiJoin` a la cuenta del usuario en el `Vat`. En operaciones normales del sistema, el `Dai.totalSupply` debe ser igual al saldo de `Vat.dai(DaiJoin)`. Cuando el contrato `DaiJoin` es _"`cage`'d"_ (enjaulado) por una dirección _"`auth`'ed"_ (autorizada), puede mover DAI de vuelta al `vat` pero este DAI no puede _`exit`_ del Vat.    

## 4. Gotchas (Fuente Potencial de Errores de Usuario)

 La fuente principal de errores de usuario con el contrato `Join` es que los usuarios nunca deberían `transferir` tokens directamente a los contratos, **se deben** usar las funciones `join` o no podrán recuperar sus tokens.  
Hay fuentes limitadas de errores de usuario en el sistema de contrato `join` debido a la funcionalidad limitada del sistema. Salvo que haya un error en el contrato, si un usuario llama `join` por accidente podría obtener sus tokens de vuelta a través de la correspondiente llamada a `exit` en el contrato `join` determinado. 
 
El principal problema al que estar atento sería un ataque _phishing_ bien elaborado. Como el sistema evoluciona y posiblemente sean creados más contratos `join` o más interfaces de usuarios, hay una posibilidad de que los fondos de un usuario sean robados por un contrato `join` malicioso, que en realidad no envíe tokens al `vat`, si no a otro contrato u otra _wallet_.    

## 5. Modos de Falla (Limites en las Condiciones de Operación y Factores de Riesgo Externos) 

**Podría haber una actualización de `vat` que requiera la creación de un nuevo contrato `join`** 

Si un contrato `gem` pasara por una actualización de token o tuviera los tokens congelados mientras el colateral del usuario estuviera en el sistema, podría haber un escenario en el que los usuarios no pudieran cambiar su colateral una vez terminada la congelación o actualización. Este escenario presenta poco riesgo, porque el token que pasara por esta actualización probablemente querrá trabajar con la comunidad Maker para asegurarse de que esto no sea un problema.    