# Las Subastas del Protocolo Maker 
  

## Introducción 
  
El sistema DAI Multi-Colateral \(MCD\) del Protocolo de Maker es una plataforma de _smart contracts_ (contratos inteligentes) en Ethereum que respalda y estabiliza el valor de nuestra _stablecoin_, DAI. Esto lo hace a través de un dinámico sistema de _vaults_, mecanismos de retroalimentación autónomos y actores externos apropiadamente incentivados. 

En este documento explicamos el mecanismo de subasta dentro del sistema, así como tipos particulares de actores externos, llamados _Keepers_, que son los que ofertan en las Subastas.

## Subastas 

Cuando todo está yendo bien en el sistema, el DAI se acumula a través de las [tarifas de estabilidad](https://github.com/makerdao/community/blob/master/faqs/stability-fee.md) en las _vaults_. Cuando el excedente neto de las tarifas de estabilidad alcanza cierto límite, ese excedente en DAI es subastado a actores externos por MKR, que luego es quemado, por lo tanto, reduciendo la cantidad de MKR en circulación. Esto se hace a través de una **Subasta de Excedentes**. 
  
El sistema se protege contra la creación de deuda mediante la sobrecolateralización. Bajo circunstancias ideales y con los parámetros de riesgo correctos, la deuda por una _vault_ individual puede ser cubierta por el colateral depositado en la _vault_. Si el precio del colateral cae hasta el punto donde una _vault_ ya no puede sostener el ratio de colateralización requerido, entonces el sistema automáticamente liquida la _vault_ y vende el colateral hasta que la deuda pendiente en la _vault_ \(y una penalización por liquidación\) esté cubierta. Esto se hace a través de las **Subastas de Colaterales**.

Además, si, por ejemplo, el precio del colateral cae bruscamente o nadie quiere comprar el colateral, puede haber deuda en la _vault_ liquidada que no puede ser pagada a través de una subasta de colateral y debe ser abordada por el sistema. La primera medida es cubrir esta deuda, utilizando el excedente proveniente de las tarifas estabilidad, si es que hay excedente para cubrirla. Si no hay, entonces el sistema inicia una **Subasta de Deuda**, donde el ganador de la subasta paga DAI para cubrir la deuda pendiente y a cambio recibe una cantidad de MKR recién acuñado, aumentando la cantidad de MKR en circulación.  

### En **resumen**, tenemos tres tipos de Subastas: 

* **Subasta de Excedentes**: el ganador de la subasta paga MKR por el excedente de DAI de las tarifas de estabilidad. El MKR recibido es quemado, por lo tanto, reduciendo la cantidad de MKR en circulación. 

* **Subasta de Colateral**: el ganador de la subasta paga DAI por colateral proveniente de la _vault_ liquidada. El DAI recibido es utilizado para cubrir la deuda pendiente de la _vault_ liquidada.  

* **Subasta de Deuda**: el ganador de la subasta paga DAI por MKR para cubrir la deuda pendiente que las Subastas de Colateral no han sido capaces de cubrir. El MKR es acuñado por el sistema, aumentando la cantidad MKR en circulación. 

Los actores que ofertan en estas Subastas son llamados **Keepers**.

## _Keepers_ 

Con toda la información publicada en la _blockchain_ de Ethereum, cualquiera puede acceder o monitorear la fuente de alimentación, precios y los datos sobre _vaults_ individuales, determinando si ciertas _vaults_ están incumpliendo el Ratio de Liquidación. El sistema incentiva a estos participantes del mercado \(que pueden ser humanos o bots automatizados\), conocidos como _"Keepers"_, para monitorear al sistema MCD y activar las liquidaciones cuando se incumpla el Ratio de Liquidación.  

En el contexto del DAI Multi-Colateral, los _keepers_ pueden participar en subastas como resultado de eventos de liquidación y así adquirir colateral a precios atractivos. Los _Keepers_ también pueden llevar a cabo otras funciones, incluyendo comerciar con DAI, motivado por la convergencia a largo plazo esperada alrededor del Precio _Target_ (objetivo).
 
Ahora iremos más a detalle acerca de como funcionan las **Subastas**.
 
### Parámetros y Mecanismos de las Subastas 

Se tomaron en cuenta varias consideraciones cuando se diseñó el mecanismo de las subastas. Por ejemplo, desde el punto de vista del sistema, lo mejor es completar la subasta lo antes posible para mantener al sistema en un estado de equilibrio, por lo que el mecanismo de subastas incentiva a los primeros postores. Otra consideración es que las Subastas son ejecutadas _on-chain_, minimizando el número de transacciones requeridas y reduciendo las tarifas asociadas.
 

#### Glosario de Subastas 


**Parámetros de Riesgo**. En general, los siguientes parámetros son utilizados a través de todos los tipos de subastas: 

* `beg`: Aumento mínimo de la oferta \(por ejemplo, 3%\). 

* `ttl`: Duración de la oferta \(por ejemplo, 6 horas\). La subasta termina si ninguna oferta nueva es propuesta durante este tiempo. 

* `tau`: Duración de la Subasta \(por ejemplo, 24 horas\). Bajo cualquier circunstancia, la subasta termina después de este periodo.

Los valores de estos parámetros de riesgo son determinados por los votantes de la Gobernanza de Maker \(poseedores de MKR\) para cada tipo de subasta. Ten en cuenta que hay diferentes tipos de parámetros de riesgos de subastas de colateral para cada tipo de colateral utilizado en el sistema.  
   

**Información de Subastas y Ofertas**. La siguiente información está siempre disponible durante una subasta activa:
  
* `lot`: Cantidad de activos que están en subasta/venta.

* `bid`: la oferta más alta actualmente. 

* `guy`: mejor postor. 

* `tic`: fecha/hora de vencimiento de la Oferta \(estará vacía si no hay ofertas\). 

* `end`: fecha/hora de vencimiento de la Subasta. 


#### Aumentos de Oferta Durante una Subasta 
  
Durante una subasta, las cantidades de ofertas aumentarán en porcentaje con cada nueva oferta. Esto es el `beg` trabajando. Por ejemplo, el `beg` podría fijarse a 3%, lo que significa que si el postor actual ha propuesto una oferta de 100 DAI, entonces la siguiente oferta debe ser de al menos 103 DAI. En general, el propósito del sistema de incremento de oferta es incentivar a los primeros postores y hacer que el proceso de la subasta sea más rápido.  

#### Cómo se Proponen las Ofertas Durante una Subasta 
 
Los postores envían _tokens_ DAI y MKR desde sus direcciones al sistema/subasta específica. Si una oferta es vencida por otra, la oferta perdedora es reembolsada a la dirección de ese postor. Sin embargo, es importante notar que una vez que la oferta es propuesta, no hay manera de cancelarla. La única forma posible de que se devuelva esa oferta es que sea superada por otra.
 
**Ahora, revisemos los mecanismos de los tres tipos de Subastas diferentes.** 

### Subasta de Excedentes

**Resumen**: una Subasta de Excedentes es utilizada para subastar una cantidad fija de excedente DAI en el sistema, por MKR. Este excedente de DAI generalmente proviene de tarifas de estabilidad acumuladas. En esta subasta, los participantes o postores compiten con cantidades ascendentes de ofertas de MKR. Una vez que ha terminado la subasta, el DAI subastado es enviado al mejor postor (el ganador de la subasta) y el MKR que este ganador envía al sistema es quemado.  

**Proceso del Mecanismo de Alto Nivel**: 

* Los Votantes de la Gobernanza de Maker determinan la cantidad de excedente permitida en el sistema en cualquier momento. Una subasta de excedentes es activada cuando el sistema tiene un excedente en DAI por encima de cantidad predeterminada de excedentes que es establecida por la gobernanza de MKR. 

* Para determinar si el sistema tiene un excedente neto, se debe sumar la tarifa de estabilidad acumulada y la deuda en el sistema. Cualquier usuario puede usar esto al enviar la transacción `heal` al contrato del sistema llamado _Vow_.

* Siempre que haya un excedente neto, la Subasta de Excedentes es activada cuando cualquier usuario envía la transacción `flap` al contrato _Vow_.

* Cuando la subasta comienza, una cantidad fija \(`lot`\) de DAI es puesta en venta. Después los postores ofertan con MKR en incrementos superiores a la cantidad mínima de aumento de la oferta. La subasta oficialmente termina cuando la duración de la oferta termina \(`ttl`\) sin otra oferta O cuando la duración de la subasta \(`tau`\) ha sido alcanzada. Una vez que la subasta termina, el MKR recibido por el excedente de DAI es luego enviado a quemarse, contrayendo así el suministro de MKR del sistema. 
 
### Subasta de Colaterales \(Venta de Colateral\) 

**Resumen**: las Subastas de Colateral sirven como un medio de recuperar la deuda en _vaults_ liquidadas. Esas _vaults_ están siendo liquidadas porque el valor del colateral de la _vault_ ha caído por debajo de cierto límite determinado por los votantes de la Gobernanza de Maker. 

**Proceso del Mecanismo de Alto Nivel**: 

Para cada tipo de colateral, los poseedores de MKR aprueban un parámetro de riesgo específico llamado ratio de liquidación. Este ratio determina la cantidad de sobrecolateralización que una _vault_ requiere para evitar la liquidación. Por ejemplo, si el ratio de liquidación es 150%, entonces el valor del colateral debe ser uno y medio más que el valor del DAI generado. Si el valor del colateral cae por debajo del ratio de liquidación, entonces la _vault_ se vuelve insegura y es liquidad por el sistema. Después el sistema toma el colateral y lo subasta para cubrir tanto la deuda en las _vaults_ y una penalización de liquidación aplicada.  

* La Subasta de Colateral es activada cuando una _vault_ es liquidada. 

* Cualquier usuario puede liquidar una _vault_ que es insegura al enviar la transacción _bite_ identificando la _vault_. Esto lanzará una subasta de colateral.  

* Si la cantidad de colateral en la _vault_ siendo _“bitten”_ es menor que el tamaño del lote para la subasta, entonces habrá una subasta para todo el colateral en la _vault_. 

* Si la cantidad de colateral en la _vault_ siendo _“bitten”_ es mayor que el tamaño del lote para la subasta, entonces se lanzará una subasta con el tamaño del lote completo del colateral y la _vault_ puede ser _“bitten”_ de nuevo para lanzar otra subasta hasta que todo el colateral en la _vault_ este disponible para ofertar en las Subastas de Colateral. 

Un aspecto importante de la Subasta de Colateral es que los parámetros de vencimiento de la subasta y de la oferta son dependientes de un tipo específico de colateral, donde los tipos de colaterales más líquidos tienen menos tiempo de vencimiento y viceversa.  

Una vez que la subasta comienza, el primer postor puede ofertar cualquier cantidad de DAI para adquirir la cantidad \(`lot`\) de colateral. Otros participantes pueden aumentar esa oferta ofreciendo más DAI por la misma cantidad \(`lot`\) de colateral hasta que haya una oferta que cubra la deuda pendiente. Cuando haya una oferta que cubra la deuda pendiente la subasta se convertirá en una subasta inversa, donde un postor oferta para aceptar partes más pequeñas del colateral por la cantidad fija de DAI que cubre la deuda pendiente. La subasta termina cuando pasa la duración de la oferta \(`ttl`\) o cuando se ha alcanzado la duración \(`tau`\) de la subasta. Una vez que la subasta ha terminado, el sistema envía el colateral a la dirección del ganador de la subasta (el mejor postor).
 
### Subasta de Deuda 

**Resumen**: las Subastas de Deuda son utilizadas para recapitalizar al sistema subastando MKR por una cantidad fija de DAI. En este proceso, los participantes compiten con su disposición a aceptar cantidades decrecientes de MKR por el DAI fijo que tendrán que pagar. 

**Proceso del Mecanismo de Alto Nivel**: 

Las Subastas de Deuda son activadas cuando el sistema tiene Deuda en DAI que ha pasado el límite de deuda específica.  

* Los Votantes de la Gobernanza de Maker determinan el límite de deuda. La Subasta de Deuda es activada cuando el sistema tiene una deuda en DAI por debajo de ese límite. 

* Para determinar si el sistema tiene una deuda neta, se deben sumar las tarifas de estabilidad acumuladas y la deuda en el sistema. Cualquier usuario puede hacer esto enviando la transacción `heal` al contrato del sistema llamado _Vow_. 
 
* Siempre que haya una deuda neta de tamaño suficiente, la Subasta de Deuda es activada cuando cualquier usuario envía la transacción `flop` al contrato _Vow_.

Se trata de una subasta inversa, donde los _Keepers_ ofertan por la cantidad de MKR que están dispuestos a aceptar, por la cantidad \(`lot`\) fija de DAI que tienen que pagar en la liquidación de la subasta. La subasta termina cuando ha pasado la duración de la oferta \(`ttl`\) o cuando se ha alcanzado la duración de la subasta \(`tau`\). Una vez que la subasta ha terminado, el DAI, pagado en el sistema por postores a cambio de MKR recién acuñado, reduce el saldo de deuda original en el sistema.  
 
### Para participar como un _Keeper_ en MCD, consulta la Guía de Configuración del Bot _Keeper_ de Subasta [aquí](https://docs.makerdao.com/keepers/auction-keepers/auction-keeper-bot-setup-guide-esp). 