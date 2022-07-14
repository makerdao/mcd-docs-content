# *Plugins*

Dai.js soporta _plugins_, lo que le permite a un desarrollador agregar funcionalidades (hardware de soporte de *wallet*, soporte de intercambios, etc.) para necesidades específicas sin incrementar el tamaño ni la lista de dependecias de la librería central.

### *Plugins* Disponibles

1. [*Plugin* de Trezor](https://github.com/makerdao/dai-plugin-trezor-web) para utilizar Trezor con dai.js en un entorno de navegador.
2. [*Plugin* de Ledger](https://github.com/makerdao/dai-plugin-ledger-web) para utilizar Ledger en un entorno de navegador.
3. [*Plugin* de la Gobernanza](https://github.com/makerdao/dai-plugin-governance) para trabajar con contratos de la Gobernanza.
4. [*Plugin* Instantáneo de eth2dai](https://app.gitbook.com/s/-LtJ1VeNJVW-jiKH0xoL/build/dai.js/maker/dai-plugin-eth2dai-instant) para comercios atómicos en el OTC (Oasis) de Maker.
5. [*Plugin* de OTC de Maker](https://github.com/makerdao/dai-plugin-maker-otc) para interactuar con el contrato OTC de Maker (Oasis).
6. [*Plugin* de MCD](../the-mcd-plugin.md) para interactuar con los contratos del DAI Multicolateral.
7. [*Plugin* de SCD](../single-collateral-dai/) para interactuar con los contratos del DAI Unicolateral.

### Construyendo tu propio *plugin*

Echa un vistazo a la [Plantilla de *Plugin* de Dai](https://github.com/makerdao/dai-plugin-template).