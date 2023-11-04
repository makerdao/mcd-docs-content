# Módulo de Colaterales

* **Nombre del Módulo:** Módulo de Colaterales
* **Tipo/Categoría:** DSS —> join.sol, clip.sol
* [**Diagrama de Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* **Fuente del Contrato:**&#x20;
  * [**Join**](https://github.com/makerdao/dss/blob/master/src/join.sol)
  * [**Clip**](https://github.com/makerdao/dss/blob/master/src/clip.sol)****

## 1. Introducción (Sumario)

El Módulo de Colaterales es desplegado para cada `ilk` nuevo (tipo de coletaral) incorporado al `Vat`. Contiene todos los adaptadores y contratos de subasta para un tipo de colateral en específico.

Para más información relacionada con el Módulo de Colaterales puedes leer los siguientes recursos:

* [Subastas y _Keepers_ dentro de MCD 101](https://github.com/makerdao/developerguides/blob/master/keepers/auctions/auctions-101.md)
* [Posteo de Blog: Cómo Hacer Funcionar Tu Propio _Bot Keeper_ de Subastas en MCD](https://blog.makerdao.com/how-to-run-your-own-auction-keeper-bot-in-mcd/)

## 2. Detalles del Módulo

El Módulo de Colaterales contiene 3 componentes centrales que consisten de los contratos `join` y `flip`.

### El Módulo de Colaterales fue construido con los siguientes componentes:

1. [**Documentación de Join**](https://docs.makerdao.com/smart-contract-modules/collateral-module/join-detailed-documentation)
2. Contrato Clipper - ver **** [**Documentación de Liquidaciones 2.0**](https://docs.makerdao.com/smart-contract-modules/dog-and-clipper-detailed-documentation)****

## 3. Mecanismos Principales y Conceptos

#### Sumario de los **Componentes del Módulo de Colaterales**

*   `Join` - adaptadores que son utilizados para depositar/retirar colaterales desbloqueados al `Vat`. Join contiene 3 _smart contracts_:

    1. `GemJoin`
    2. `ETHJoin`
    3. `DaiJoin`.

    Cada uno de los contratos `join` es utilizado específicamente para el tipo de token específico a ser incorporado al `vat`. Debido a este hecho, cada contrato tiene una lógica ligeramente diferente para dar cuenta de los diferentes tipos de token en el sistema.

#### ¿Cómo ayudan, exactamente, los contratos `Join` a operar el sistema MCD?

* `Join` - el propósito de los adaptadores _join_ es el de retener la seguridad del sistema, permitiendo incorporar o remover valor de y hacia el `Vat` sólo a _smart contracts_ de confianza. La ubicación del colateral depositado/bloqueado en _vaults_ está en cada adaptador _join_ respectivo.

## 4. _Gotchas_ (Posibles fuentes de error del usuario)

* Cuando el usuario desea entrar al sistema e interactuar con los contratos `dss`, debe utilizar uno de los contratos `join`.
* Si hubiese un _bug_ de contrato en un contrato `join` y un usuario llamase a `join` por accidente, aún así puede recuperar sus tokens a través de la llamada `exit` correspondiente en dicho contrato `join`.

## 5. Modos de Falla

**Podría haber una actualización de `vat` que requiera que nuevos contratos `join` sean creados**

Si un contrato`gem` fuera a pasar por una actualización de token o ve sus tokens congelados (_freeze_) mientras que el colateral del usuario está en el sistema, podría potencialmente haber un escenario en el cual los usuarios no puedan canjear sus coletarles luego de que el congelamiento o la actualización hayan terminado. Sin embargo, esto parece ser un riesgo menor ya que parecería probable que el token que estuviera pasando por esta actualización, quisiera trabajar junto con la comunidad de Maker para asegurarse de que esto no sea un problema.

**Potenciales Ataques de _Phishing_**

A medida que el sistema MCD evoluciona, veremos muchos más contratos `join`, interfaces de usuarios, etc. Esto acarrea los potenciales problemas de que un usuario vea sus fondos robados por un contrato `join` malicioso que envíe sus tokens a un contrato o _wallet_ externa, en lugar de al `vat`.