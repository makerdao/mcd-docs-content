# Tabla de Contenidos

* [Documentos Técnicos de MakerDAO](README-esp.md)

## Comenzando

* [Protocolo de Maker 101](getting-started/maker-protocol-101-esp.md)

## Módulos de _Smart Contracts_ (Contratos Inteligentes)

* [Módulo del DAI](smart-contract-modules/dai-module/README-esp.md)
  * [Dai - Documentación Detallada](smart-contract-modules/dai-module/dai-detailed-documentation-esp.md)
* [Módulo Central](smart-contract-modules/core-module/README-esp.md)
  * [_Vat_ - Documentación Detallada](smart-contract-modules/core-module/vat-detailed-documentation-esp.md)
  * [_Spot_ - Documentación Detallada](smart-contract-modules/core-module/spot-detailed-documentation-esp.md)
* [Módulo de Colaterales](smart-contract-modules/collateral-module/README-esp.md)
  * [_Join_ - Documentación Detallada](smart-contract-modules/collateral-module/join-detailed-documentation-esp.md)
* [Módulo de Liquidaciones 2.0](smart-contract-modules/dog-and-clipper-detailed-documentation-esp.md)
* [Módulo de Estabilizador de Sistema](smart-contract-modules/system-stabilizer-module/README-esp.md)
  * [_Flapper_ - Documentación Detallada](smart-contract-modules/system-stabilizer-module/flap-detailed-documentation-esp.md)
  * [_Flopper_ - Documentación Detallada](smart-contract-modules/system-stabilizer-module/flop-detailed-documentation-esp.md)
  * [_Vow_ - Documentación Detallada](smart-contract-modules/system-stabilizer-module/vow-detailed-documentation-esp.md)
* [Módulo de Oráculos](smart-contract-modules/oracle-module/README-esp.md)
  * [Módulo de Seguridad de Oráculos (OSM) - Documentación Detallada](smart-contract-modules/oracle-module/oracle-security-module-osm-detailed-documentation-esp.md)
  * [_Median_ - Documentación Detallada](smart-contract-modules/oracle-module/median-detailed-documentation-esp.md)
* [Módulo del MKR](smart-contract-modules/mkr-module-esp.md)
* [Módulo de la Gobernanza](smart-contract-modules/governance-module/README-esp.md)
  * [_Spell_ - Documentación Detallada](smart-contract-modules/governance-module/spell-detailed-documentation-esp.md)
  * [_Pause_ - Documentación Detallada](smart-contract-modules/governance-module/pause-detailed-documentation-esp.md)
  * [_Chief_ - Documentación Detallada](smart-contract-modules/governance-module/chief-detailed-documentation-esp.md)
* [Módulo de Tasas](smart-contract-modules/rates-module/README-esp.md)
  * [_Pot_ - Documentación Detallada](smart-contract-modules/rates-module/pot-detailed-documentation-esp.md)
  * [_Jug_ - Documentación Detallada](smart-contract-modules/rates-module/jug-detailed-documentation-esp.md)
* [Módulo de Proxy](smart-contract-modules/proxy-module/README-esp.md)
  * [Acciones de Proxy - Documentación Detallada](smart-contract-modules/proxy-module/proxy-actions-detailed-documentation-esp.md)
  * [Votación de Proxy - Documentación Detallada](smart-contract-modules/proxy-module/vote-proxy-detailed-documentation-esp.md)
  * [Administrador de CDP - Documentación Detallada](smart-contract-modules/proxy-module/cdp-manager-detailed-documentation-esp.md)
  * [Administrador de DSR - Documentación Detallada](smart-contract-modules/proxy-module/dsr-manager-detailed-documentation-esp.md)
* [Módulo de Acuñación Rápida](smart-contract-modules/flash-mint-module-esp.md)
* [Apagado de Emergencia del Protocolo de Maker](smart-contract-modules/shutdown/README-esp.md)
  * [Apagado de Emergencia para Socios](smart-contract-modules/shutdown/emergency-shutdown-for-partners-esp.md)
  * [El Proceso de Apagado de Emergencia para Dai de Colaterales Múltiples (MCD)](smart-contract-modules/shutdown/the-emergency-shutdown-process-for-multi-collateral-dai-mcd-esp.md)
  * [_End_ - Documentación Detallada](smart-contract-modules/shutdown/end-detailed-documentation-esp.md)
  * [ESM - Documentación Detallada](smart-contract-modules/shutdown/emergency-shutdown-module-esp.md)

## Glosario <a href="#other-documentation" id="other-documentation"></a>

* [Glosarios de MCD](other-documentation/system-glossary-esp.md)
* [Anotaciones de _Smart Contract_](other-documentation/smart-contract-annotations-esp.md)

## Registro de Cambios de MCD

* [Comunicados Públicos del DAI de Colaterales Múltiples](mcd-changelog/multi-collateral-dai-public-releases-esp.md)

## Seguridad de MCD

* [Seguridad para el Protocolo de Maker](mcd-security/security.makerdao.com-esp.md)

## Construyendo sobre el Protocolo de Maker <a href="#build" id="build"></a>

* [La Librería del Javascript del Dai del Protocolo de Maker](build/dai.js/README-esp.md)
  * [Comenzando](build/dai.js/getting-started-esp.md)
  * [Configuración](build/dai.js/maker/README-esp.md)
    * [Plug-Ins](build/dai.js/maker/plugins-esp.md)
  * [Administrador de _vaults_](build/dai.js/the-mcd-plugin-esp.md)
  * [Tipos de Colateral](build/dai.js/cdptypeservice-esp.md)
  * [Tasas de Ahorro del Dai](build/dai.js/savingsservice-esp.md)
  * [Unidades Monetarias](build/dai.js/currency-units-esp.md)
  * [Datos del Sistema](build/dai.js/systemdataservice-esp.md)
  * [Avanzado](build/dai.js/advanced-configuration/README-esp.md)
    * [Administrador de Transacciones](build/dai.js/advanced-configuration/transactions-esp.md)
    * [DSProxy](build/dai.js/advanced-configuration/using-ds-proxy-esp.md)
    * [Eventos](build/dai.js/advanced-configuration/events-esp.md)
    * [Utilización de Múltiples Cuentas](build/dai.js/advanced-configuration/using-multiple-accounts-esp.md)
    * [Incorporación de un Nuevo Servicio](build/dai.js/advanced-configuration/adding-a-new-service-esp.md)
  * [Sai de Colateral Único](build/dai.js/single-collateral-dai/README-esp.md)
    * [Posición de Deuda Colateralizada](build/dai.js/single-collateral-dai/collateralized-debt-position-esp.md)
    * [Servicio de CDP](build/dai.js/single-collateral-dai/eth-cdp-service-esp.md)
    * [Servicio de Precio](build/dai.js/single-collateral-dai/price-service-esp.md)
    * [Estado del Sistema](build/dai.js/single-collateral-dai/system-status-esp.md)
    * [_Tokens_](build/dai.js/single-collateral-dai/tokens-esp.md)
    * [Conversión de _Tokens_](build/dai.js/single-collateral-dai/token-conversion-esp.md)
    * [Servicio de Intercambio](build/dai.js/single-collateral-dai/exchange-service-esp.md)
* [Pymaker](build/pymaker-esp.md)
* [Tutoriales y Guías para Desarrolladores](build/developer-guides-and-tutorials-esp.md)

## _Keepers_

* [Las Subastas del Protocolo de Maker](keepers/the-auctions-of-the-maker-protocol-esp.md)
* [_Keepers_ de Subastas](keepers/auction-keepers/README-esp.md)
  * [Guía de Configuración de Bot de _Keeper_ de Subastas](keepers/auction-keepers/auction-keeper-bot-setup-guide-esp.md)
* [_Keepers_ del Mercado de Maker](keepers/market-maker-keepers/README-esp.md)
  * [Guía de Configuración del Bot de _Keeper_ del Mercado de Maker](keepers/market-maker-keepers/market-maker-keeper-bot-setup-guide-esp.md)
* [_Cage Keeper_](keepers/cage-keeper-esp.md)
* [_Keeper_ de Arbitraje Simple](keepers/simple-arbitrage-keeper-esp.md)
* [_Keeper_ Jefe](keepers/chief-keeper-esp.md)

## Interfaces de Línea de Comandos <a href="#clis" id="clis"></a>

* [Seth](clis/seth-esp.md)
* [Dai de Colaterales Múltiples (MCD) CLI](clis/mcd-cli-esp.md)
* [Redención de Dai y Colaterales durante un Apagado de Emergencia](clis/dai-and-collateral-redemption-during-emergency-shutdown-esp.md)
* [Apagado de Emergencia (ES) CLI](clis/emergency-shutdown-es-cli-esp.md)

## Misceláneos

* [Sistema de Liquidaciones 1.2 (Obsoleto)](miscellaneous/liquidations-1.2-system-deprecated/README-esp.md)
  * [_Cat_ - Documentación Detallada](miscellaneous/liquidations-1.2-system-deprecated/cat-detailed-documentation-esp.md)
  * [_Flipper_ - Documentación Detallada](miscellaneous/liquidations-1.2-system-deprecated/flipper-detailed-documentation-esp.md)
* [Migración SCD <> MCD](miscellaneous/scd-mcd-migration-detailed-documentation-esp.md)
* [Actualización de la Guía del Dai de Colaterales Múltiples](miscellaneous/upgrading-to-multi-collateral-dai-guide-esp.md)