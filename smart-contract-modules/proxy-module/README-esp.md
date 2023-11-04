# Módulo Proxy

* **Nombre del Módulo:** Modulo Proxy
* **Tipo/Categoría: Proxy —>** DsrManager.sol, DssCdpManager.sol, VoteProxy.sol y DssProxyActions.sol
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* **Fuente del Contrato:**
  * ****[**DSR Manager**](https://github.com/makerdao/dsr-manager/blob/master/src/DsrManager.sol)****
  * ****[**CDP Manager**](https://github.com/makerdao/dss-cdp-manager/blob/master/src/DssCdpManager.sol)
  * ****[**Vote Proxy**](https://github.com/makerdao/vote-proxy/blob/master/src/VoteProxy.sol)****
  * ****[**Proxy Actions**](https://github.com/makerdao/dss-proxy-actions/blob/master/src/DssProxyActions.sol)

## 1. Introducción (Resumen)

El módulo _Proxy_ fue creado para hacer más conveniente la interacción de los usuarios/desarrolladores con el Protocolo de Maker. Contiene interfaces de contrato, _proxys_ y alias de funciones necesarias para interactuar tanto con la gestión de _vaults_ y _DSR_, como con la Gobernanza de Maker. 

![](../../.gitbook/assets/proxymodulenew.png)

## 2. Detalles del Módulo

### Documentación de los Componentes del Módulo _Proxy_

1. ****[**DSR Manager – Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/proxy-module/dsr-manager-detailed-documentation-esp)****
2. [**CDP Manager - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/proxy-module/cdp-manager-detailed-documentation-esp)
3. [**Vote _Proxy_ - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/proxy-module/vote-proxy-detailed-documentation-esp)
4. [**_Proxy_ Actions - Documentación Detallada**](https://docs.makerdao.com/smart-contract-modules/proxy-module/proxy-actions-detailed-documentation-esp)

## 3. Mecanismos y Conceptos Clave 

#### ¿Por qué estos componentes son importantes para el Sistema Multicolateral del DAI (MCD)?

#### **DSR Manager**

El `DsrManager` proporciona un contrato inteligente fácil de usar que permite a los servicios de proveedores depositar/retirar `dai` del/en el contrato [pot](https://docs.makerdao.com/smart-contract-modules/rates-module/pot-detailed-documentation-esp), para comenzar a ganar la Tasa de Ahorro de DAI en unq _pool_ de DAI en una sola llamada a la función sin la necesidad de un contrato `ds-proxy`. Esto es útil para la integración de _smart contracts_ (contratos inteligentes) que integran la funcionalidad del DSR.

#### **CDP Manager**

El `DssCdpManager` (alias `manager`) fue creado para permitir un proceso formalizado de transferencias de Vaults entre dueños. En resumen, el `manager` trabaja teniendo una envoltura (_wrapper_) `dss` que permite a los usuarios interactuar con sus _vaults_ de una manera fácil, tratándolos como tokens no fungibles (NFTs).

#### **Vote Proxy**

El _VoteProxy_ **facilita la votación _online_ con un almacenamiento de MKR _offline_**. Tener un VoteProxy, permite a los usuarios vincular una _hot wallet_ que pueda atraer y empujar MKR desde la _cold wallet_ correspondiente del _proxy_ y a DS-Chief, donde la votación puede tomar lugar con la _hot wallet_ online.

**Hay dos formas principales de tener/usar este contrato:** 

1. Para apoyar dos diferentes mecanismos de votación. 
2. Para minimizar el tiempo que los dueños de MKR necesitan tener sus _wallets_ online. 

#### **Acciones Proxy**

Las `dss-proxy-actions` fueron diseñadas para utilizarse por el _Ds-Proxy_, que les pertenece individualmente a otros usuarios, empleándolo para interactuar más fácilmente con el Protocolo de Maker. Ten en cuenta que esto no pretende ser utilizado de forma directa (abordaremos el tema más adelante). El contrato _dss-proxy-actions_ fue desarrollado para servir como una biblioteca para los _ds-proxies_ de los usuarios.

**En general, el _ds proxy_ recibe dos parámetros:**

* **Dirección de la biblioteca Proxy**
  * En este caso, la biblioteca _dss proxy actions_.
* **Datos de la llamada**
  * Funciones y parámetros que desées ejecutar.


## 4. Gotchas (Posibles Fuentes de Error del Usuario)

* **DSR Manager**
  * Para los desarrolladores que se quieren integrar con el `DsrManager`, es importante tener en cuenta que los saldos de usuarios en el `pot` serán propiedad del `DsrManager`, que tiene un mapeo interno para determinar los saldos de los usuarios. Consecuentemente, el DAI depositado en DSR puede que no aparezca en soluciones basadas en `ds-proxy` (como [oasis.app/save](https://oasis.app/save))
  * [Aquí](https://docs.makerdao.com/smart-contract-modules/proxy-module/dsr-manager-detailed-documentation-esp) puedes conseguir más información. 
* **CDP Manager**
  * Para desarrolladores que se quieren integrar con el `manager`, necesitarán entender que las acciones de la _vault_ todavía están en el ambiente `urn`. A pesar de esto, el `manager` intenta abstraer el uso de `urn` mediante un `CDPId`. Esto significa que los desarrolladores necesitarán obtener el `urn` (`urn = manager.urns(cdpId)`) para permitir _`join`ing_ (que se una) el colateral de esa _vault_.
  * [Aquí](https://docs.makerdao.com/smart-contract-modules/proxy-module/cdp-manager-detailed-documentation-esp) puedes conseguir más información.
* **Vote Proxy**
  * **Costo de establecer el proxy una sola vez** como un nuevo usuario del contrato proxy, necesitarás configurarlo antes de poder usarlo en futuras votaciones. El precio de la configuración dependerá del precio del gas para ese momento, pero en última instancia hará que la votación sea más fácil y segura para los usuarios. 
  * [Aquí](https://docs.makerdao.com/smart-contract-modules/proxy-module/vote-proxy-detailed-documentation-esp) puedes conseguir más información.
* **Proxy Actions**
  * **Usar el _dss-proxy-actions_ directamente puede resultar en la perdida de control sobre tu _vault_:** si abres una _vault_ nueva mediante el _dss proxy actions_ (centralizado) sin un _ds proxy_ estarías creando una _vault_ que le pertenece al _dss proxy actions_ que cualquiera puede llamar públicamente. Le pertenecería al contrato _dss proxy actions_ y cualquiera podría ejecutar acciones en tu _vault_. Por lo tanto, si utilizas el _dss proxy actions_ directamente puede ser riesgoso.  

  * [Aquí](https://docs.makerdao.com/smart-contract-modules/proxy-module/proxy-actions-detailed-documentation-esp) puedes conseguir más información.

## 5. Modos de Falla (Limites en las Condiciones de Funcionamiento y Factores de Riesgo Externos)
* **CDP Manager**
  * **Problemas Potenciales en torno a la Reorganización de la Cadena**
    * Cundo se ejecuta `open`, se crea una nueva `urn` y se le asigna un `cdpId` para un `owner` (dueño) específico. Si el usuario utiliza `join` (unirse) para agregar colateral al `urn` inmediatamente después que la transacción es acuñada, hay una posibilidad de que ocurra una reorganización de la cadena. Esto podría resultar en que el usuario pierda la propiedad sobre ese par de `cdpId`/`urn`, por lo tanto, perdería su colateral. Sin embargo, el problema solo puede surgir cuando se evita el uso de las [funciones proxy](https://github.com/makerdao/dss-proxy-actions) mediante un [perfil proxy](https://github.com/dapphub/ds-proxy) ya que el usuario `open` (abrirá) el  `cdp` y `join` (unirá) el colateral en la misma transacción.
  * [Aquí](https://docs.makerdao.com/smart-contract-modules/proxy-module/cdp-manager-detailed-documentation#5-failure-modes-bounds-on-operating-conditions-and-external-risk-factors-esp) puedes conseguir más información.
* **Vote Proxy**
  * La pérdida de las llaves privadas tanto para la _hot wallet_ como para la _cold wallet_ te impedirá votar. 
* **Proxy Actions**
  * **Ds proxy es un proxy de propósito general / siempre hay un riesgo cuando se usa un proxy**
    * En término de modos de falla, esto significa que tu puedes ejecutar un proxy de acciones maliciosas, así como una acción directa que podría potencialmente enviar tus ETH a una dirección aleatoria. Para ser precavido, deberías revisar los datos de llamadas de tus _wallets_ y/o auditar lo que hace tu _wallet_, ya que se podrían presentar usuarios con algunos datos de llamadas aleatorias no deseadas y ejecutar acciones no deseadas.