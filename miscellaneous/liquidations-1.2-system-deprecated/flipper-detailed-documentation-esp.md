# Flipper - Documentación Detallada 

* **Nombre del Contrato:** flip.sol
* **Tipo/Categoría:** DSS —> Modulo de Subasta de Colateral
* ****[**Diagrama del Sistema MCD Asociado**](https://github.com/makerdao/dss/wiki)
* ****[**Fuente del Contrato**](https://github.com/makerdao/dss/blob/master/src/flip.sol)
* **Etherscan**
  * ****[**Flip ETH-A**](https://etherscan.io/address/0xF32836B9E1f47a0515c6Ec431592D5EbC276407f)
  * ****[**Flip BAT-A**](https://etherscan.io/address/0xf7c569b2b271354179aacc9ff1e42390983110ba#code)

## 1. Introducción (Resumen)

**Resumen:**  las Subastas de Colateral se encargan de vender el colateral de las _vaults_ que se han vuelto subcolateralizadas para poder preservar la colateralización de todo el sistema. _Cat_ envía dicho colateral al módulo _Flip_ para que sea subastado por los _keepers_. La subasta de colaterales tiene dos fases: `tend` y `dent`.

![Interacciones _Flip_ con el Protocolo Maker](<../../.gitbook/assets/Screen Shot 2019-11-17 at 2.01.16 PM.png>)

## 2. Detalles del Contrato

#### Flipper (Glosario)

* `wards [usr: dirección]`, `rely`/`deny`/`auth` - mecanismos de _auth_
* `Bid` - estado de una subasta específica {`bid`, `lot`, `guy`, `tic`, `end`, `usr`, `gal`, `tab`}
  * `bid` - cantidad de la oferta o puja (DAI)/ DAI pagado
  * `lot` - cantidad a subastar / _gems_ de colateral para la venta
  * `guy` - mejor postor (dirección)
  * `tic` - vencimiento de la oferta o puja
  * `end` - cuando termina la subasta / duración máxima de la subasta
  * `usr` - dirección de la _vault_ que está siendo subastada. Recibe _gems_ durante la fase `dent`
  * `gal` - receptor de los ingresos de la subasta / recibe los ingresos de DAI (este es el contrato _Vow_)
  * `tab` - total de DAI deseado de la subasta / total de DAI a recaudar (en subastas _flip_)  
* `bids[id: uint]` - almacenamiento de todas las ofertas
* `vat` - almacenamiento de la dirección del _Vat_
* `ilk` - id del _Ilk_ del cual el _Flipper_ es responsable
* `beg` - incremento mínimo de la oferta (por defecto: 5%)
* `ttl` - duración de la oferta (por defecto: 3 horas)
* `tau` - duración de la subasta (por defecto: 2 días)
* `kicks` - recuento total de subastas, utilizado para rastrear las _`id`_ de la subasta 
* `kick` - función usada por `Cat` para empezar la subasta / Pone colateral para la subasta
* `tick` - reinicia una subasta si no han habido ofertas y el `end` ya ha pasado
* `tend` - primera fase de una subasta. Aumenta la `bid` (“oferta o puja”) de DAI por un conjunto `lot` de _Gems_
* `dent` - segunda fase de una subasta. Establece la `bid` (“oferta o puja”) de DAI por un `lot` decreciente de _Gems_  
* `file` - función utilizada por la Gobernanza para establecer `beg`, `ttl` y `tau`
* `deal` - reclama una oferta ganadora / liquida una subasta completada
* `yank` - utilizado durante la Liquidación Global para mover la fase `tend` de la subasta a `End` recuperando el colateral y pagando DAI al mejor postor
* `claw`: reduce la cantidad de _litter_ (“arena”) de la caja del _Cat_ (“gato”)

### **Parámetros establecidos por la Gobernanza (a través de `file`)**

* `beg`
* `ttl`
* `tau`

Además, los parámetros `dunk` y `chop` de `Cat` también informan cómo funciona `Flip` ya que `dunk` se convierte en el `Bid.lot` y junto a `chop` influencian al `Bid.tab`.

### Parámetros no establecidos por la Gobernanza 

* `vat`
* `ilk`

Ambas de estas están establecidas en el constructor y no pueden cambiarse. Si se cambia la dirección _Vat_ y cada vez se agrega un nuevo colateral al sistema, un nuevo _Flip_ tiene que ser desplegado.  

### Autorizaciones

_Flipper_ debe ser _`Vat.wish`_ por el `Cat` para poder `flux` durante el `kick`. 
`End` debe _`rely`_ por el _Flipper_ para permitir `yank`.
`Cat` debe _`rely`_ por el _Flipper_ para permitir `kick`.

## 3. Mecanismos y Conceptos Clave

El proceso de subastas _Flip_ comienza con los votantes de la Gobernanza de Maker determinando el ratio de colateralización mínimo del colateral (`Spot.Ilk.mat`) que después es probado contra el estado de la _vault_ (precio del colateral, total de la deuda) para determinar si la misma es segura (consulta la documentación sobre `Cat` para más información sobre el proceso `bite`). El último paso de una `bite` es `kick` una subasta `Flip` por ese colateral específico. Ten en cuenta que la penalización por liquidación se añade al `tab` cuando inicia la subasta `Flip`. Esto solo determina cuándo la subasta cambia de `tend` a `dent`. Sin embargo, esta cantidad no se añade a la cantidad total de deuda (solo a la parte que esta siendo **parcialmente liquidada**) a menos que todo haya sido liquidado.  

La Gobernanza también determina el tamaño de `lot` (donde un `lot` es la cantidad de _gems_ de colateral para subastar en una subasta `flip`) cuando una _vault_ es `bite` ("mordida"). Esto permite las liquidaciones parciales de grandes _vaults_. Las liquidaciones parciales hacen que las subastas sean más flexibles y tengan menos impacto en el precio base del colateral, creando una sola gran subasta. También permiten que grandes _vaults_ se vuelvan seguras otra vez si el precio se recupera antes que la _vault_ sea liquidada completamente. Los _keepers_ preferirán mantener esto en mente cuando también `bite` _vaults_ inseguras, ya que tendrán una opción de comenzar una o varias subastas de liquidación parcial.  

Comenzando en la fase `tend`, los participantes compiten por una cantidad `lot` fija de _Gem_ con cantidades crecientes de DAI. Una vez que se ha recaudado la cantidad `tab` de DAI, la subasta pasa a la fase `dent`. El punto de la fase `tend` es recaudar DAI para cubrir la deuda del sistema. 

Durante la fase `dent`, los participantes compiten por cantidades `lot` decrecientes de _Gem_ por la cantidad `tab` fija de DAI. La _Gem_ confiscada se devuelve a la _Urn_ liquidada para que el dueño la recupere. El objetivo de la fase `dent` es devolver al poseedor de la _vault_ tanto colateral como el mercado lo permita.

Una vez que la última oferta de la subasta ha expirado o que la subasta ha alcanzado el `end`, cualquiera puede llamar a `deal` para pagar al mejor postor (`Bid.guy`). Esto mueve las _Gems_ del saldo de _Flipper_ en el _Vat_ hacia el saldo del postor. 

![Un diagrama detallando las interacciones que un usuario tiene con _Flipper_, _Cat_ y el _Vow_.](../../.gitbook/assets/Flip\_auction\_diagram\_interaction.png)

## 4. Gotchas (Posibles Fuentes de Error del Usuario)

#### **Keepers**

En el contexto de la ejecución de un _keeper_ (más información [aquí](https://github.com/makerdao/developerguides/tree/master/keepers)) para realizar ofertas dentro de una subasta, un modo de falla primario ocurriría cuando un _keeper_ especifica un precio no rentable para el colateral.  


* Este modo de falla es debido al hecho de que no hay nada que el sistema pueda hacer para evitar que un usuario pague significativamente más del precio justo de mercado por el _token_ en una subasta (esto va para todos los tipos de subastas: `flip, flop y flap`). 
* Los _keepers_ que tienen un mal rendimiento están en riesgo, principalmente, durante la fase `dent` ya que podrían devolver demasiado colateral a la _vault_ original y terminar pagando de más (es decir pagar demasiado DAI (`bid`) por muy pocas _gems_ (`lot`)).  

#### **Requisitos para Ofertar durante una subasta**

Durante `tend`, la cantidad `bid` aumentará en un porcentaje `beg` con cada nueva `tend`. El postor debe conocer el `id` de la subasta, especificar la cantidad correcta de `lot` para la subasta, ofertar al menos un `beg`% más que la última oferta, pero no más que `tab`, y debe tener suficiente saldo `Vat.dai`.      

Durante `dent`, las cantidades `lot` disminuirán en un porcentaje `beg` con cada nueva `dent`. El postor debe saber la `id` de la subasta, especificar la cantidad correcta de `bid` para la subasta y ofrecer tomar un `beg`% menos de `lot` que la última oferta.  

#### **Proponer Ofertas**

Cuando una oferta `tend` es superada por otro postor, el saldo de DAI interno de la nueva oferta ganadora es utilizado para financiar al postor que ganó anteriormente. Cuando una oferta `dent` es superada por otro postor, el saldo _gem_ de _Flipper_ es usado para financiar al poseedor de la _vault_. Una vez propuestas las ofertas, estas no pueden cancelarse. 

**Ilustración del flujo de ofertas:**

1. _Cat_  _`kick`s_ (“inicia”) una nueva Subasta _Flip_. _Cat_ emite un evento `bite` con la dirección del _Flipper_ y la `id` de la subasta. El _Flipper_ emite un evento `kick` con la `id` y otros detalles de las subastas.      

Comenzar la subasta `tend`:

1. El Postor 1 propone una oferta que incrementa el tamaño de la `bid` por `beg`. El saldo en DAI del Postor 1 en el _Vat_ disminuye por `bid` y el saldo en DAI del _Vow_ en el _Vat_ aumenta por `bid`. 
2. El postor 2 propone una oferta que aumenta la `bid` del Postor 1 en al menos `beg`. El saldo en DAI del postor 2 en el _Vat_ disminuye por `bid` y el saldo en DAI del Postor 1 en el _Vat_ disminuye por `bid` - La `bid` del Postor 1 y el saldo en DAI del `Vow` es aumentada por el mismo número. El `tic` se restablece a `now` + `ttl`.
3. El Postor 1 propone una oferta que aumenta la `bid` del Postor 2 en al menos `beg`. El DAI del Postor 1 = `Vat.dai[bidder1]` - La `bid` anterior del Postor 2; el DAI del Postor 2 =  `Vat.dai[bidder2]` + la `bid` anterior del Postor 2. Después el DAI del Postor 1 = `Vat.dai[bidder1] - (oferta – oferta del Postor 2)` y el DAI del _Vow_ =  `Vat.dai[bidder1] + (oferta – oferta del Postor 2)`. El `tic` se restablece a `now` + `ttl`.  
4. Una vez que entra una nueva `bid` que es igual al `tab`, se completa la fase `tend`.    

Comienza la subasta `dent`:

**Nota:** _Esta fase debe comenzar antes que `tic` expire y después que pase `bid.end`._  

1. El Postor 2 (y todos los otros postores dentro de la subasta) decide que ya no vale la pena seguir aumentando sus _`bid`s_ (“ofertas”), así que deja de ofertar. Una vez que el `Bid.tic` expira, el Postor 1 llama a `deal` y los _tokens_ _gem_ son enviados a su saldo _Vat_. 

**Nota:** Una subasta también puede terminar en la fase `tend` si no se alcanza `tab` antes que el `tic` o `end` sean alcanzadas. Si esto pasa, luego la oferta ganadora es recompensada usando la función `deal` y la diferencia entre la `bid` final y `tab` se mantiene como deuda incobrable en el `Vow` para ser tratada durante una subasta `Flop`.    

#### El Fin

En el caso de Liquidación Global, `End` es capaz de llamar a `yank` en el _Flipper_. `yank` cierra una subasta en fase `tend` devolviendo la oferta en DAI de `guy` y moviendo las _Gems_ desde el _Flipper_ hacia `End`. Las subastas en fase `dent` pueden continuar a la fase `deal` ya que han recaudado el DAI necesario y están en el proceso de devolver _Gems_ al poseedor original de la _vault_. 

## 5. Modos de Fallo (Límites en las Condiciones de Funcionamiento y Factores de Riesgo Externos)

### Límites en las Condiciones de Funcionamiento

Debido a que `Flip.tend` compara la `bid` del postor con la `bid * beg` anterior, comparará los dos números con una precisión de 10^63 (`rad * wad`). Esto significa que cualquier `bid` que sea mayor de 115,792,089,237,316 causará un desbordamiento. La Gobernanza debería procurar no establecer `beg` o `lot` (mediante `Cat.ilks[ilk].dunk`) así que es probable que un _keeper_ de subasta podría terminar _`bid`'ding_ (“ofertando”) tanto DAI durante la fase `tend`. Esto es muy poco probable, ya que el precio objetivo de DAI sigue siendo 1 USD, pero es incluido aquí para tenerlo en cuenta.            

#### 1. [**Ver la Documentación del Módulo Estabilizador del Sistema**](https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module-esp)****

#### 2. Riesgos de las Subastas de Último Minuto/Baja participación de los _Keeper_

**Rectificación de Subasta**

La rectificación de subasta permite a un atacante generar deuda, permitir que su _vault_ sea mordida, ganar su propia subasta para recuperar su _colateral_ con un descuento. Este tipo de falla es más posible cuando la penalización por liquidación se establece muy abajo. 

Para todos los detalles acerca de este riesgo, referencia el documento de @livnev [aquí](https://github.com/livnev/auction-grinding/blob/master/grinding.pdf).