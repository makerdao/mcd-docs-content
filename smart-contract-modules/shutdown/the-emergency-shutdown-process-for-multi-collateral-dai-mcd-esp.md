# Proceso de Apagado para el DAI Multicolateral (MCD) 

## **Introducción al Proceso de Apagado de Emergencia**

El Protocolo de Maker, que potencia al DAI Multicolateral (DAI), respalda y estabiliza el valor de DAI al precio objetivo de 1 Dólar Estadounidense, lo que se traduce a una paridad o vinculación suave de 1:1 con el Dólar Estadounidense. El mecanismo de estabilización es manejado a través de un sistema de _smart contracts_ (contratos inteligentes) autónomos, una combinación dinámica de _vaults_ y actores externos apropiadamente incentivados, como lo son los _Keepers_.  

Los [_Keepers_](https://docs.makerdao.com/keepers/auction-keepers-esp) juegan un rol crítico en mantener la salud del sistema y la estabilidad de DAI. En marzo del 2020, la Fundación Maker [señaló](https://blog.makerdao.com/recent-market-activity-and-next-steps/) la necesidad de un ecosistema de _Keepers_ más desarrollado. Un aumento en la participación de los _Keepers_ en última instancia mejoraría la salud y el funcionamiento del Protocolo de Maker. Para más información sobre cómo obtener y ejecutar un _Keeper_, consulta esta [guía](https://docs.makerdao.com/keepers/auction-keepers/auction-keeper-bot-setup-guide-esp). 

El Protocolo de Maker tiene un procedimiento de **Apagado de  Emergencia (ES)** que puede ser activado como último recurso para proteger al sistema y a sus usuarios de una amenaza seria o facilitar una actualización del Protocolo.

## **Visión General del Proceso de Apagado de Emergencia**

![](https://lh6.googleusercontent.com/tHGq8IsHybndRILRSF\_pkAObTOPrcylSnBdN4DC7LDudq2EeH0K8Q9qEZgrQG-ozBtjwCWZtWPBbp-\_tSnlP75nXRWcSMb4FzZsjCZvZBAPavGkJcsaoYDwIehDqE\_6tlqp8KH3Y)

### **Resumen**

El Apagado de Emergencia esta pensado para activarse en caso de una actualización del sistema o de emergencias serias, tal como la irracionalidad a largo plazo del mercado, un _hack_ o una brecha de seguridad. Cuando es activado, el Apagado de Emergencia detiene y apaga al Protocolo de Maker mientras se asegura que todos los usuarios, tanto los _holders_ de DAI y los usuarios de _Vaults_ reciben el valor neto de los activos a los que tienen derecho según la lógica del _smart contract_ del Protocolo. Sin embargo, el valor del Colateral que los _holders_ de DAI pueden canjear puede variar dependiendo del excedente del sistema (superávit) o déficit en el momento del Apagado de Emergencia. Es, por lo tanto, posible que los _holders_ de DAI recibirán menos o más de 1 USD de colateral por 1 DAI.

El proceso de inicialización del Apagado de Emergencia es descentralizado y controlado por los votantes de MKR, quienes pueden activarlo al depositar MKR dentro del [Módulo de Apagado de Emergencia (ESM)](https://docs.makerdao.com/smart-contract-modules/emergency-shutdown-module-esp), un contrato con la capacidad de llamar [**End.cage**](https://github.com/makerdao/dss/blob/44330065999621834b08de1edf3b962f6bbd74c6/src/end.sol#L264), o mediante un Voto Ejecutivo. En el caso del método del Módulo de Apagado de Emergencia, 100,000 MKR deben ser depositados en el Módulo de Apagado de Emergencia para activar el Apagado de Emergencia. Adicionalmente, una vez que los usuarios depositan MKR en el Módulo de Apagado de Emergencia, este es inmediatamente quemado.  Ya sea a través del Módulo de Apagado de Emergencia o el método del Voto Ejecutivo, cuando se llama a _cage_, esta debe alterar el comportamiento de casi cualquier componente del sistema, así como actuar bajo una variedad de posibles escenarios de subcolateralización.  

### **Propiedades de Implementación del Apagado de Emergencia**

* **Condición no-carrera del DAI**: Cada _holder_ de DAI será capaz de canjear la misma cantidad relativa de colateral proporcional a sus _holdings_ de DAI, independientemente del momento en que interactúan con el contrato. 
* **Paridad de _Vault_**: los dueños de _vaults_ tienen prioridad, permitiéndoles retirar su exceso de Colateral antes que los _holders_ de DAI sean capaces de acceder al Colateral.  
  * En el momento del apagado de Emergencia, tipos de colateral enteros o el Protocolo Maker pueden estar subcolateralizados, que es cuando el valor de la deuda excede el valor del Colateral ("capital negativo"). Por lo tanto, el valor del colateral que los _holders_ de DAI pueden canjear, puede variar dependiendo del excedente del sistema (superávit) o déficit en el momento del Apagado de Emergencia. Por lo tanto, es posible que los _holders_ de DAI reciban mas o menos 1 USD de Colateral por 1 DAI.
* **Redención Inmediata de la _Vault_**: después que inicia el Apagado de Emergencia, los dueños de _Vaults_ pueden liberar su colateral inmediatamente, siempre que ejecuten todas las llamadas de contrato atómicamente.
* **No hay cálculos _off-chain_**:  el sistema no necesita que la autoridad _cage_ proporcione ningún valor calculado _off-chain_ (es decir, puede confiar completamente en la ultima fuente de alimentación de precios del OSM). 

* **Asistencia del _Buffer Vow_**: después de que el Apagado de Emergencia sea iniciado, cualquier excedente o deuda incobrable en el _buffer_ actúa como una recompensa o penalización distribuida a pro-rata para todos los _holders_ de DAI. Por ejemplo, si el 10% de la deuda total del sistema está en forma de excedente neto en el _Vow_, después los _holders_ de DAI reciben 10% más de Colateral.   

## **Redención de DAI y Colateral durante el Apagado de Emergencia**

### **Proceso de Apagado de Emergencia para Dueños de  _Vaults_**  

Los dueños de _vaults_ pueden recuperar el exceso de colateral de sus _vaults_ inmediatamente después de la inicialización del Apagado de Emergencia. Pueden hacer esto mediante los _frontends_ de _vaults_ tales como [Oasis Borrow](https://oasis.app/borrow), que tienen implementado el soporte de Apagado de Emergencia o mediante herramientas _command-line_.  

### **Proceso de Apagado de Emergencia para los _Holders_ de DAI**

Los _holders_ de DAI pueden, después de un periodo de espera determinado (para su procesamiento) por los votantes de MKR, intercambiar sus DAI por una pro-rata proporcional relativa de todos los tipos de Colateral en el sistema. La cantidad de Colateral que puede ser reclamada durante este periodo es determinada por los Oráculos de Maker en el momento que el Apagado de Emergencia es activado. Es importante tener en cuenta que los _holders_ de DAI siempre recibirán la misma cantidad pro-rata relativa de Colateral del sistema, tanto si sus reclamos están entre los primeros o los últimos en ser procesados. La Fundación de Maker inicialmente ofrecerá una página web con este propósito para hacer que el proceso sea más fácil para los _holders_ de DAI.  

### **Porque el Apagado de Emergencia Prioriza a los Dueños de _Vaults_ sobre los _Holders_ de DAI**

_La priorización de los Dueños de _Vaults_ sobre los _Holders_ de DAI durante el Apagado de Emergencia puede desglosarse en tres puntos principales:_&#x20;

1. Las _vaults_ sobrecolateralizadas no subvencionan el Protocolo de Maker para las _vaults_ subcolateralizadas durante el funcionamiento actual del sistema, por lo que es coherente que el Apagado de Emergencia tenga el mismo comportamiento. La principal diferencia es que el posible _haircut_ es transferido desde los _holders_ de MKR a los _holders_ de DAI, ya que no se pueden hacer suposiciones acerca del valor del MKR después de un apagado.    
2. &#x20;Dar prioridad a los propietarios de _vaults_ para recuperar su exceso de Colateral (si su _vault_ no está subcolateralizada) incentivándolos a mantener la sobrecolateralización. Esto es importante debido a que el incentivo se mantiene incluso si un Apagado de Emergencia parece probable, que en última instancia hace que el protocolo sea más resiliente. 
3. El Apagado de Emergencia no renuncia a las Tarifas de Estabilidad acumuladas antes del Apagado de Emergencia. Los dueños de _vaults_ pueden aceptar tarifas más altas si saben que están protegidos de los niveles de colateralización de otros, potencialmente resultando en un excedente más alto durante los escenarios de Apagados de Emergencias así como permitir un mayor DSR durante el funcionamiento normal. 

## **Liquidación de la Subasta Durante el Apagado de Emergencia**

Hay un tiempo de retraso en el Apagado de Emergencia  que está determinado por la Gobernanza. El retraso debe expirar antes que tome lugar cualquier intercambio de DAI por Colateral. La orientación general es que el retraso debería ser lo suficientemente largo para asegurar que todas las subastas terminen o sean saltadas; pero, no hay garantía de esto en el código. Es importante destacar, que cualquiera puede cancelar una subasta de colateral (Flip) en cualquier momento, mientras que las subastas de excedentes (Flap) y las subastas de deuda (Flop) son congeladas por la llamada a **Cage** inicial. Tanto las subastas Flap como las subastas Flop pueden ser llamadas para devolver las ofertas al ultimo postor. 

Ten en cuenta que la cancelación de subastas no es un proceso inmediato, ya que los participantes del ecosistema deben cancelar todas las subastas de colateral en curso para apropiarse del Colateral y  devolverlo al _pool_ de colateral. Esto permite un procesamiento más rápido de las subastas a expensas de más llamadas de procesamiento. En cuanto a las subastas de excedentes y deudas, también deben ser llamadas. Si nadie llama estas funciones, las subastas no serán canceladas.  

## **Intenciones del Apagado de  Emergencia**&#x20;

**El Apagado de Emergencia puede adoptar dos** formas principales. Por un lado, el Apagado de Emergencia puede ser activado y el sistema se da por terminado sin un plan futuro de redistribución. Esto permite a los usuarios reclamar el exceso de Colateral o reclamar el Colateral desde su DAI.

**Por otro lado, el Apagado de Emergencia puede iniciarse con un Escenario de Redespliegue.** Esta situación puede surgir cuando el sistema ha sido activado en el caso de un Apagado de Emergencia. Aún así, los _holders_ de tokens MKR, o terceros, han decidido volver a desplegar el sistema con los cambios necesarios para volver a ponerlo en marcha. Esto permitirá a los usuarios abrir nuevas _vaults_ y tener un nuevo token DAI mientras reclaman los Colaterales del antigüo sistema.  

## **Cómo el Apagado de Emergencia Afecta a los Usuarios**&#x20;

Durante un Apagado de Emergencia, cada una de las partes interesadas del Ecosistema de Maker deberían actuar en consecuencia: &#x20;

### **_Holders_ de DAI**

Si tu _wallet_ tiene una interfaz viable para reclamar Colateral o migrar tu DAI, o tiene un navegador Dapp incorporado, puedes utilizar el [portal de migración](https://migrate.makerdao.com) para reclamar y/o trasladar. Si tu _wallet_ **no** soporta la funcionalidad anterior, debes transferir tu DAI a una nueva _wallet_ que permita la funcionalidad antes de proceder a utilizar el [portal de migración](https://migrate.makerdao.com).

### **Propietarios de _Vaults_**

Si usas [Oasis.app/borrow](https://oasis.app/borrow) para gestionar tu _vault_, procede al [portal de migración](https://migrate.makerdao.com) y sigue el proceso de canje de emergencia descrito.   

Si eres usuario de una interfaz de terceros, tales como [DefiSaver](http://defisaver.com) o [InstaDapp](https://instadapp.io), verifica que tengan una Interfaz de Apagado de Emergencia incorporada antes de proceder. De ser así, usa su interfaz para reclamar el exceso de Colateral o migrar a un sistema recién desplegado. Si el proveedor externo no tiene el proceso de reembolso incorporado, transfiérelo al [portal de migración](https://migrate.makerdao.com) si es posible.

### **_Holders_ de MKR**

Los _holders_ de MKR pueden votar en encuestas y votaciones ejecutivas relacionadas con la activación del proceso de Apagado de Emergencia. Esto se hace en el _frontend_  del Módulo de Apagado de Emergencia (ESM) o directamente a través del [ES CLI](https://docs.makerdao.com/clis/emergency-shutdown-es-cli-esp). Adicionalmente, los _holders_ de MKR también pueden votar en relación con una futura redistribución del Protocolo de Maker en el [Portal de Gobernanza](http://vote.makerdao.com).

### **Exchange Centralizada o _Wallet_ de Custodia**

En el caso de un Apagado de Emergencia, los proveedores de servicios pueden seguir las acciones recomendadas a continuación.

**Procedimiento Recomendado**

* Alertar a los usuarios sobre la situación actual y proporcionarles orientación sobre las acciones correctas a tomar. Dependiendo del escenario del Apagado de Emergencia, ya sea Apagado o redistribución, aconséjeles que actúen en consecuencia.
* Dar a los usuarios las opciones de retirar sus DAI/MKR de la _exchange_ o informarles que el _exchange_/_wallet_ manejará el proceso de Apagado de Emergencia en su nombre.

**Escenario: Apagado**

* Escoge una de las siguientes opciones:
  * **Opción 1:** dejar que los usuarios retiren DAI y MKR de las plataformas y después guiarlos al [portal de migración](https://migrate.makerdao.com) para el proceso de redención.
  * **Opción 2:** reclamar el DAI equivalente en Colateral en nombre de los usuarios, utilizando el [portal de migración](https://migrate.makerdao.com).
  * Elije una de las siguientes: 
    * Distribuir el colateral a los usuarios.
    * Obtener la dirección de retiro de los usuarios para los tipos de colaterales no respaldados en el _exchange_.
    *Conservar el Colateral (para venderlo, por ejemplo) y actualizar los balances internos de Fiat para reflejar la cantidad que les corresponde.

**Escenario: Redistribución**

* Migrar los _holdings_ de DAI al nuevo token de DAI en nombre de los usuarios, utilizando el [portal de migración](https://migrate.makerdao.com).
* Alternativamente, llevar a cabo la migración de salida interactuando directamente con los contratos de migración utilizando las herramientas CLI. Consulta [esta guía](https://github.com/makerdao/developerguides/blob/master/governance/Collateral%20Redemption%20during%20Emergency%20Shutdown.md).
* Si procede, migra los _holdings_ de tokens MKR en nombre de los usuarios usando el [portal de migración](https://migrate.makerdao.com)
* Actualiza la(s) dirección(es) del token en tu sistema.  

### **_Wallet_ No Custodiada**

En caso de un Apagado de Emergencia, los proveedores de _wallets_ no custodiadas deben alertar a tu base de usuarios acerca del Apagado de Emergencia y proporcionar enlaces públicos para más información. Pueden seguir la recomendación de procedimientos listados a continuación en el caso de un Apagado de Emergencia. 

#### **Procedimiento Recomendado**

* **Escenario: Apagado**
  * Redirigir a los usuarios al [portal de migración](https://migrate.makerdao.com) para reclamar su equivalente DAI en Colateral o crear una interfaz para manejar el proceso localmente.  
* **Escenario: Redistribución**
  * Informar a los usuarios para que migren sus DAI en el [portal de migración](https://migrate.makerdao.com) o crear una interfaz interna para manejar el proceso localmente.   
  * Agregar apoyo destacado para el nuevo token(s).

### **_Exchanges_ Descentralizados (DEXs)**

Como un _exchange_ descentralizado, puedes informar a los usuarios con un _banner_ acerca del estado actual del Protocolo de Maker y dirigirlos hacia los canales de comunicación relevantes para obtener más información. **Puedes elegir una de las siguientes dos opciones para permitir a tus usuarios llevar a cabo el proceso de canje de Apagado de Emergencia:**
* Dirigirlos al [portal de migración](https://migrate.makerdao.com), donde pueden iniciar el proceso de reclamación por sus DAI. 
* Construir una interfaz para manejar el proceso del Apagado de Emergencia en tu plataforma, informar a tus usuarios y hacer que actúen en consecuencia.  

#### **Procedimiento Recomendado:**

* **Escenario: Apagado**
  * Informar a los usuarios para que reclamen el valor equivalente de DAI en Colateral en el [portal de migración](https://migrate.makerdao.com) o crear una interfaz para manejar el proceso localmente. 
* **Escenario: Redistribución**
  * Informar a los usuarios para que migren sus DAI al nuevo DAI (y MKR, si procede) en el [portal de migración](https://migrate.makerdao.com) o crear una interfaz para manejar el proceso en tu plataforma. 
  * Agregar nuevo(s) token(s) al _exchange_.

### **Navegadores _Dapp_**

Como un navegador _dapp_, por favor asegúrate de alertar a tu base de usuarios acerca del Apagado de Emergencia y proporcionales enlaces para mas información (es decir, [blog.makerdao.com](http://blog.makerdao.com) o [makerdao.com](http://makerdao.com)). En el caso de que sea un Apagado de Emergencia del Sistema o una Redistribución del Sistema después de que se active el Apagado Emergencia, redirija a sus usuarios al [portal de migración](https://migrate.makerdao.com) para que reclamen su Colateral.     

### **Integradores de _Vaults_**

Como integrador de _vaults_, es muy importante que te integres con los contratos del Protocolo de Maker (mas específicamente, _end.sol_). Esta integración crucial te permitirá crear rápidamente una lógica reactiva que se encargara del proceso de post - Apagado de Emergencia de tus usuarios. Si eres un servicio de custodia, tales como _exchanges_ centralizados, por favor informa a tus usuarios por adelantado sobre tu plan de manejar el evento del Apagado de Emergencia. Puedes seguir los procedimientos recomendados que se indican a continuación en caso de un Apagado de Emergencia.    

**Procedimiento Recomendado**

* **Escenario: Apagado**
  * Reclamar los fondos de los usuarios a través del [portal de migracion](https://migrate.makerdao.com) o interactuando directamente con los contratos de migración y hacerlos disponibles en sus cuentas.
* **Escenario: Redistribución**
  * Migrar los fondos de los usuarios a un nuevo sistema redistribuido utilizando el [portal de migración](https://migrate.makerdao.com) o interactuando directamente con los contratos de migración. 

Como un **integrador de _vault_ no-custodiada**, por favor asegúrate de integrarte con los contratos del Protocolo de Maker (_end.sol_). Esto te permite ser notificado en el momento exacto en que se ha activado el Apagado. De otra manera, se sugiere que informe a sus usuarios sobre cómo pueden liberar el Colateral en las _vaults_. Esto puede hacerse en la UI del integrador de la _vault_ no-custodiada o puedes dirigirlos a [Oasis.app/borrow](https://oasis.app/borrow), si los usuarios necesitan migrar sus _vaults_. Si decides usar tus propios servicios, necesitarás una UI que permita a los usuarios retirar sus _vaults_ de un contrato _proxy_ para que aparezca en el [portal de migración](https://migrate.makerdao.com). Dirige a tus usuarios allí. Alternativamente, puedes crear una interfaz que ayude a los usuarios a migrar sus DAI en caso de una redistribución, o permitir a los usuarios reclamar su Colateral en caso de un único escenario de apagado.  

### **Aplicaciones Descentralizadas (Dapps)**

Se sugiere que las Dapps se integren con los contratos del Protocolo de Maker (_end.sol_), que efectivamente proporcionan un sistema de notificaciones que muestra si el Apagado de Emergencia ha sido activado. En términos de preparación, cuando el Apagado de Emergencia es activado, ten listo lo siguiente para tus usuarios:    

* Una interfaz UI que alerte e informe a los usuarios sobre el evento. 
* Si tu Dapp utiliza un _proxy_, tendrás que permitir que los usuarios salgan del _proxy_ para poder usar la app/el portal de migración.&#x20;
* Proporcionar canales comunicación oficiales para mas información, así como un enlace al [portal de migracion](https://migrate.makerdao.com) para la redención de DAI y _Vault_.   

#### **Servicios de Custodia**

Si controlas el acceso a los contratos inteligentes que respaldan tu Dapp, es recomendable que le permitas a tus usuarios recuperar DAI o acceder a sus _Vaults_ desde sus _wallets_ personales, así como dirigirlos al [portal de migracion](https://migrate.makerdao.com) para el proceso de redención del Apagado de Emergencia. Alternativamente, puedes reclamar colateral DAI o reclamar el exceso de Colateral de las _Vaults_ en nombre de tus usuarios en el [portal de migracion](https://migrate.makerdao.com), y proceder a distribuirlo  entre tus usuarios, asegurándote que  ellos lo puedan recuperar exitosamente.  

#### **Servicios de No-Custodia**

Si no controlas los contratos inteligentes que respaldan tu Dapp directamente, entonces puedes dirigir a tus usuarios al [portal de migracion](https://migrate.makerdao.com) para el canje de DAI y _Vault_. Alternativamente, puedes crear una interfaz que permita a los usuarios reclamar el equivalente de DAI en Colateral o reclamar el exceso de Colateral de las _VaultS_ en caso de un apagado del sistema. Adicionalmente, si se despliega el sistema de nuevo, migra el DAI al nuevo sistema desplegado y/o reclama el exceso de Colateral de _Vaults_.   

### **Market Makers**

Como un _market maker_ durante el Apagado de Emergencia, puedes proporcionar liquidez en el mercado para que los _holders_ de DAI puedan intercambiar su DAI por otros activos. Después que no haya mercado que cubrir, puedes actuar como un _holder_ de DAI y comenzar a migrar DAI a nuevos DAI en caso de que se despliegue el sistema nuevamente o reclame el colateral de DAI equivalente en caso de un apagado de todo el sistema. 
 
## **Descripción Detallada del Mecanismo de Apagado de Emergencia para MCD**

_Este es un proceso complejo que involucra los siguientes 9  pasos._ 

### **1. Bloquea al Sistema e Inicia el Apagado del Protocolo Maker (también conocido como _Caging_ el Sistema)**

El bloqueo de los precios para cada tipo de colateral se realiza congelando los siguientes puntos de entrada del usuario: 

* Creación de _Vault_ 
* Subasta de Excedente/Subasta de Deuda
* Tasa de Ahorro DAI (DSR)
* Puntos de Entrada de la Gobernanza

A continuación, el sistema pausará todas las subastas de deuda/subastas de excedente, permitiendo que las subastas individuales sean canceladas llamando a una función que mueva la primera fase de una subasta de colateral al **End** ("Final"). Este proceso se completa al recuperar el Colateral y devolver DAI al mejor postor del contrato de subasta respectivo. Una de las razones por las que estas subastas se congelan y se cancelan es por que el proceso de Apagado de Emergencia está diseñado para transmitir el excedente del sistema o la deuda del sistema a los _holders_ de DAI. En general, no hay garantías en cuanto al valor del MKR durante un Apagado y no se puede confiar en los mecanismos en los que típicamente se basa el valor de mercado del MKR, lo que en última instancia resulta en que no haya razón de seguir ejecutando las subastas que afectan al suministro de MKR. Más específicamente, las razones por las que las subastas de deuda y de excedente son canceladas son las siguientes:        

* Las Subastas de Excedente ya no cumplen su propósito. Esto es debido a que, después de un apagado, el excedente está diseñado para ser asignado a los _holders_ de DAI. Por lo tanto, cancelar las subastas de excedente durante el Apagado permite que el sistema devuelva el excedente DAI de vuelta al balance de los motores de Liquidación y en última instancia de vuelta a los _holders_ de DAI.
* Las Subastas de Deuda también dejan de cumplir su propósito. Esto es debido a que la deuda incobrable se trasmite como un recorte (menor-que-el-valor-de-mercado colocado en un activo que se utiliza como Colateral en una _vault_) de vuelta a los _holders_ de DAI si no hay otro sistema de excedente disponible. 

En cuanto a las subastas de colateral, no se cancelan inmediatamente (pero pueden ser canceladas por cualquier usuario) debido a que aún están vinculadas a los Colaterales valiosos del Sistema. Las subastas de Colateral siguen funcionando y los _keepers_ pueden continuar ofertando por ellas. Si no hay postores, las subastas en vivo también pueden ser canceladas.  

A pesar del hecho de que las subastas pueden seguir ejecutándose, esto no garantiza que todas las _vaults_ restantes estén sobrecolateralizadas. Tampoco hay nada que evite que existan _vaults_ subcolateralizadas ni _unbitten_ en el momento que se llama a **cage**. 

Durante este tiempo, la función que añade la deuda (el DAI total deseado de la subasta) no puede ser llamada, ya que la función requiere que el sistema esté funcionando normalmente, deshabilitando las liquidaciones después del apagado. Adicionalmente, después de que **Ends** comience, todas las _vaults_ deben ser liquidadas en el precio marcado y después, el Colateral restante de una _vault_ liquidada debe ser retirado.   

En general, esto resulta en que las subastas de colateral puedan continuar durante el Apagado o que sean revertidas por un usuario, cancelando las subastas en vivo (lógica similar a la de las subastas de excedente). Si una subasta es cancelada, las ofertas son retornadas a los postores y el Colateral se devuelve a la _vault_ original (con la penalización por liquidación aplicada en la forma de un aumento de la deuda).
 
**Notas sobre las subastas de colateral:**
* _End_ mueve la primera fase de las subastas de colateral a _End_, recuperando el Colateral y devolviendo el DAI al mejor postor. 
*  La segunda fase de las subastas permite que se hagan ofertas, mientras disminuye la cantidad a subastar. Durante esta fase, las subastas completadas son liquidadas porque ya han recaudado el DAI necesario y ya están en proceso de devolver el Colateral al propietario original de la _vault_.  

**Otras Notas:**

* El MKR podría seguir teniendo valor si el precio actual del token MKR está ligado a otro despliegue del sistema. Ten en cuenta que el sistema no hace ninguna suposición acerca del valor económico del MKR después de un apagado.
* Es importante tener en cuenta que la deuda y el excedente de las subastas se cancelan y los balances se trasfieren al contrato _End_. El último paso en este proceso es comenzar el período de enfriamiento.

### **2. Establecer los Precios Finales para los Tipos de Colateral en el Protocolo de Maker** 

Este proceso se completa estableciendo el precio del Apagado del Sistema para cada tipo de colateral. Los precios finales son determinados mediante la lectura de las fuentes de precios provenientes de los Oráculos de Maker. Esto es necesario, ya que el sistema primero debe procesar el estado del sistema lo antes posible para calcular el precio final del DAI/colateral. En particular, necesitamos determinar dos cosas: 

(a) El déficit por tipo de colateral, considerando las _vaults_ subcolateralizadas. 

(b) La cantidad total de DAI emitida (deuda total), que es la oferta DAI pendiente después de incluir el excedente/déficit del sistema.  

En primer lugar, esto se determina (a) procesando todas las _vaults_ con una función que cancela el DAI que se adeuda de la _vault_ (descrito mas adelante). Después, (b) se desarrolla como se describe a continuación.  

### **3. Liquidar _Vaults_ al Precio Final mediante la Cancelación del DAI que se debe**

Después, el sistema permitirá cancelar todo el DAI que se debe de las _vault(s)_ en el sistema. Cualquier exceso de Colateral permanece dentro de la(s) _vault(s)_ para que los propietario(s) lo reclamen. Luego, se toma el colateral respaldado.  

A continuación, se determina la deuda **(b)** procesando las operaciones de generación de DAI en curso de las subastas. Procesar las operaciones de DAI en curso asegura que las subastas no generaran más ingresos de DAI. Esto garantiza que las subastas en curso no cambiarán la deuda total del sistema, que también incluye el modelo de subasta de dos vías (subasta de colateral), que no permite generar más DAI. Debido a esto, la generación de DAI proviene de la primera fase de la subasta de colateral. Así, si todo está en la segunda fase de una subasta (ofertando por la cantidad decreciente que sale a subasta), sabemos que la generación ha terminado. La generación termina cuando todas las subastas están en la fase inversa/segunda. Además de asegurar que las subastas no generarán más DAI, la Tasa de Ahorro DAI (DSR) también debe apagarse durante _End_ para que la deuda total no cambie.   

**Ejemplo:**

En cuanto a los escenarios de los usuarios, el proceso comienza con los usuarios ofertando más y más DAI hasta que la deuda es cubierta. Luego, comienzan a ofertar cada vez menos Colateral. 

Las subastas que están en la segunda fase (subastas inversas) ya no afectan más a la deuda total, pues el DAI ya fue recuperado. Por último, en el caso de las subastas de la primera fase, pueden cancelarse y el Colateral y la deuda retornar a la _vault_.

**Uno de los dos métodos puede asegurar que el Colateral y la deuda se devuelvan a la _vault_:**

1. La duración del enfriamiento del procesamiento (longitud de la cola de la deuda); o 
2. Cancelando las subastas que están en vivo.

### **4. Utilizando el Periodo de Enfriamiento o Cancelando las Subastas que están en Vivo** 

1. **Establecer el periodo de enfriamiento.** La duración del período de enfriamiento solo necesita ser lo suficientemente larga como para cancelar el DAI que se debe de las _vaults_ subcolateralizadas y cancelar las subastas en vivo de la primera fase. Esto significa que, de hecho, puede ser bastante corto (como 5 minutos). Sin embargo, debido a la posibilidad de escenarios en los que ocurra una congestión en la red, puede establecerse más tiempo. 

2. **Cancelar una subasta en vivo** cancelará todas las subastas que están en curso y confiscará el colateral. Esto permite un procesamiento más rápido para todas las subastas a expensas de más procesamiento de llamadas. Esta opción permite a los _holders_ de DAI recuperar su colateral mas rápido. El próximo procedimiento es cancelar cada una de las subastas de Colateral (Flip) individuales en las subastas de la primera fase, recuperar todo el Colateral y devolver DAI al postor. Después que esto ocurra, la segunda fase -subasta inversa- puede continuar como lo haría normalmente, estableciendo el período de enfriamiento o cancelando las subastas en vivo.     

Ten en cuenta que ambas opciones están disponibles en esta implementación, con la cancelación de las subastas en vivo que es habilitada en base por subasta. Cuando una _vault_ ha sido procesada y no tiene deuda restante, el colateral restante puede ser removido.    

### **5. Removiendo el Colateral Restante de una _Vault_ Liquidada (Solo Después que no haya deuda en la _Vault_)**
A continuación, el sistema removerá el Colateral de la _vault_. Después que las _vaults_ hayan sido liquidadas al precio final que se fijo y el DAI que se debe de la _vault_ ha sido cancelado, el dueño de la _vault_ puede llamar a este proceso cuando lo necesite. Eliminara todo el Colateral que quede después del paso **3**, básicamente, todo el colateral que no estaba respaldando la deuda. Si el usuario no tenía deuda en una _vault_ al momento del _End_, este usuario puede obviar los pasos **3 y 4** y puede proceder directamente a este paso para liberar sus Colaterales.

### **6. Estabilizar la Oferta Total Pendiente de DAI** 

Una vez transcurrido el período de procesamiento, el calculo del precio final para cada tipo de colateral es posible utilizando la función _thaw_ ("descongelar"). Se asume que todas las _vaults_ subcolateralizadas se han procesado y todas las subastas se han desecho. El propósito de esta función es estabilizar el total de la oferta de DAI restante. Ten en cuenta que también puede requerir el procesamiento de _vaults_ adicionales para cubrir el excedente del sistema. Comprobar que la cantidad de excedente de DAI en el motor central de la _vault_ es 0, es un requisito durante esta fase. Este requerimiento es lo que garantiza que se tome en cuenta el excedente del sistema. Además, esto significa que antes de que puedas estabilizar el suministro total de DAI, debes cancelar el DAI que se adeuda de tantas _vaults_ como sea necesario para cancelar cualquier excedente de DAI en el _Vow_. Cancelar el excedente de DAI se hace cancelando el excedente y la deuda del balance del sistema antes de poder estabilizar el suministro total pendiente de DAI.   

### **7. Calcular el Precio Fijado por un tipo de Colateral, posiblemente ajustando el Precio Final del Colateral con el Excedente (Superávit)/Déficit**   

En este paso, el cálculo del precio de intercambio para un tipo de colateral es determinado, así como el ajuste potencial de ese intercambio de precio final en el caso de déficit/excedente (superávit). Llegados a este punto del mecanismo, se ha calculado el precio final para cada tipo de colateral; los _holders_ de DAI ahora pueden convertir su DAI en Colateral. Cada unidad de DAI puede reclamar una cesta fija de Colateral.  Primero, los _holders_ de DAI deben bloquear sus DAI, así pueden estar listos para intercambiarlo por Colateral. Una vez que el DAI está bloqueado, no puede ser desbloqueado y no es transferible. También se puede bloquear más DAI después. 

### **8. Bloquear DAI e Intercambiarlo por Colateral** 

En este paso es cuando se entrega el Colateral a los _holders_ de DAI, que ya han bloqueado sus DAI para ser intercambiados. Entre más cantidad de DAI bloqueado, más Colateral se puede liberar a los _holders_ de DAI.  

### **9. Intercambiar el DAI Bloqueado por Colateral (Proporcional a la Cantidad Bloqueada)**

Por último, el sistema permitirá el intercambio de parte del DAI que ha sido bloqueado para tipos específicos de colateral. Ten en cuenta que el numero de tokens de colateral estará limitado por la cantidad de DAI bloqueado que tenga el usuario. 

## **Obtener Apoyo**&#x20;

### **Canales de Chat de Rocket**

* [Chat.makerdao.com](http://chat.makerdao.com). Los canales de soporte incluyen, pero no están limitados a:
  * **#general**&#x20;
  * **#desarrollo**
  * **#gobernanza-y-riesgo**
  * **#ayudar**

[**Forum.Makerdao.com** ](https://forum.makerdao.com) &#x20;

* [Gobernanza](https://forum.makerdao.com/c/governance/5)&#x20;
* [Riesgo](https://forum.makerdao.com/c/risk/6)

### **Otros recursos y documentación**&#x20;

* [Introducción al post del Blog sobre el Apagado de Emergencia](https://blog.makerdao.com/introduction-to-emergency-shutdown-in-multi-collateral-dai/)
* [Documentación _End_](https://docs.makerdao.com/smart-contract-modules/shutdown/end-detailed-documentation-esp)
* [Documentación del Módulo de Apagado de Emergencia](https://docs.makerdao.com/smart-contract-modules/emergency-shutdown-module-esp)
* [Guía del Apagado de Emergencia - MCD](https://github.com/makerdao/developerguides/blob/master/mcd/emergency-shutdown/emergency-shutdown-guide.md)
* [Apagado de Emergencia CLI](https://docs.makerdao.com/clis/emergency-shutdown-es-cli-es)
* [Redención de DAI y Colaterales durante el Apagado de Emergencia CLI](https://docs.makerdao.com/clis/dai-and-collateral-redemption-during-emergency-shutdown-es)
* [_Keeper_ de _Cage_ ("jaula")](https://docs.makerdao.com/keepers/cage-keeper-es#introducción)
* [Apagado de Emergencia FAQ](https://github.com/makerdao/community/blob/master/faqs/emergency-shutdown.md)