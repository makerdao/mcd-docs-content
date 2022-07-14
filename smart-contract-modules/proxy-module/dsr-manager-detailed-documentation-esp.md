# Gestor DSR - Documentación Detallada 

El `DsrManager` proporciona un _smart contract_ (contrato inteligente) fácil de utilizar que permite a los proveedores de servicios depositar/retirar DAI dentro del contrato [pot](https://docs.makerdao.com/smart-contract-modules/rates-module/pot-detailed-documentation-esp) DSR, y activar/desactivar la Tasa de Ahorro de DAI para empezar a ganar ahorros en un _pool_ de DAI en una sola llamada a la función. Para entender el _DsrManager_, primero es necesario tener un entendimiento del [pot](https://docs.makerdao.com/smart-contract-modules/rates-module/pot-detailed-documentation-esp). El _DSR_ es fijado por la Gobernanza de Maker, y típicamente seguirá siendo menor que la tarifa de estabilidad base para seguir siendo sostenible. El propósito del _DSR_ es el de ofrecer otro incentivo para _holdear_ DAI.  

## Detalles de Implementación 

* Mainnet: [0x373238337Bfe1146fb49989fc222523f83081dDb](https://etherscan.io/address/0x373238337Bfe1146fb49989fc222523f83081dDb#code)
* Kovan: [0x7f5d60432DE4840a3E7AE7218f7D6b7A2412683a](https://kovan.etherscan.io/address/0x7f5d60432DE4840a3E7AE7218f7D6b7A2412683a#code)
* Ropsten: [0x74ddba71e98d26ceb071a7f3287260eda8daa045](https://ropsten.etherscan.io/address/0x74ddba71e98d26ceb071a7f3287260eda8daa045#code)

## Detalles del Contrato

### Matemáticas 

* `wad` - alguna cantidad de _tokens_, como un entero de punto fijo con 18 decimales. 
* `ray` - un entero de punto fijo con 27 decimales. 
* `rad` - un entero de punto fijo con 45 decimales. 
* `mul(uint, uint)`, `rmul(uint, uint)`, `add(uint, uint)` & `sub(uint, uint)` - se revertirá en caso de desbortamiento o subdesbordamiento.  
* `Rdiv` - Divide dos _`ray`s_ y devuelve un nuevo `ray`. Siempre redondea hacia abajo. Un `ray` es un número decimal con 27 dígitos de precisión que es representado como un entero.  
* `Rdivup` - Divide dos _`ray`s_ y devuelve un nuevo _`ray`_. Siempre redondea hacia arriba. Un _`ray`_ es un número decimal con 27 dígitos de precisión que es representado como un entero. 

### Almacenamiento 

* `pot` - almacena la dirección de contrato del `pot` principal del contrato de la Tasa de Ahorro de DAI. 
* `dai` - almacena la dirección del contrato de DAI. 
* `daiJoin` - almacena las direcciones de contratos del adaptador de token DAI.
* `supply` - la oferta de DAI en el _DsrManager_.
* `pieOf` - `mapping (addresses=>uint256)` mapeo de direcciones de usuario y balances de DAI normalizados \(`amount of dai / chi`\) ("cantidad de DAI") depositada en `pot`.
 * `pie` - almacena el balance `pot` de la dirección. 
* `chi` - el acumulador de tasa. Este es el siempre creciente valor que decide cuánto DAI es dado cuando se llama `drip()`
* `vat` - una dirección que se ajusta a una interfaz `VatLike`. 
* `rho` - la última vez que se llama a `drip`.

## Funciones y Mecánicas

### daiBalance\(address usr\) returns \(uint wad\)

* Calcula y devuelve el balance de DAI de la dirección _usr_ específica en el contrato _DsrManager_. \(Balance de DAI existente + _dsr_ acumulado\)

### join\(address dst, uint wad\)

* _`uint wad`_ este parámetro especifica la cantidad de DAI que quieres unir al _pot_. La cantidad `wad` de DAI debe estar presente en la cuenta de `msg.sender`. 
* direccion `dst` especifica una dirección de destino para el DAI depositado en el _pot_. Permite que una dirección de _hot_ wallet \(`msg.sender`\) deposite DAI en el _pot_ y transfiera la propiedad de ese DAI a una _cold_ wallet \(o cualquier otra dirección respecto a ese tema\) 
* el balance normalizado `pie` es calculado dividiendo _wad_ con el acumulador de tasa `chi`.
* la cantidad `pieOf` de `dst` se actualiza para incluir el _`pie`_.
* la cantidad de suministro total también es actualizado al añadir el _`pie`_.
* cantidad `wad` de DAI que se transfiere al contrato _DsrManager_.
* el contrato _DsrManager_ une la cantidad _`wad`_ de DAI al sistema MCD a traves del adaptador de token DAI _`daiJoin`_.
* El contrato _DsrManager_ _`join`_ (une) la cantidad _`pie`_ de DAI al _`pot`_. 

### exit\(address dst, uint wad\)

* `exit()` esencialmente funciona como el opuesto exacto de `join()`. 
* `uint wad` este parámetro está basado en la cantidad de DAI que quieres _`exit`_ del _`pot`_.
* la dirección _`dst`_ especifica una dirección de destino para el DAI recuperado del `pot`. Permite que una dirección de _cold wallet_ (o billetera fría) \(`msg.sender`\) pueda recuperar DAI del `pot` y transferir la propiedad de ese DAI a una nueva _hot wallet_ \(o billetera caliente\) o a cualquier otra dirección 
* El balance _`pie`_ normalizado es calculado dividiendo _wad_ con el acumulador de tasa `chi`.  
* La cantidad de _`pieOf`_ del _`msg.sender`_ es actualizada sustrayendo el _`pie`_.
* La cantidad total de oferta también es actualizada substrayendo el _`pie`_.
* El contrato llama a _exit_ en el contrato _`pot`_.  
* Calcula la cantidad de _DAI_ a recuperar multiplicando `pie` por `chi`.
* Luego sale el _DAI_ del adaptador de token DAI _`daiJoin`_ a la dirección de destino `dst`.

### exitAll\(address dst\)

* `exitAll()` funciona como la función _`exit`_, con la diferencia de que simplemente busca en el mapeo`pieOf`, para determinar qué tanto DAI tiene el `msg.sender` y _`exit`s_ ("saca") toda la cantidad de DAI en lugar de una cantidad específica.  

## Gotchas / Preocupaciones respecto a la integración

* Para usar la función _`join`_ necesitas _`approve`_ ("aprobar") el contrato para transferir DAI desde tu _wallet_. Necesitas llamar a _`approve`_ en el token DAI, especificando el contrato _`DsrManager`_ y la cantidad que el contrato debería ser capaz de sacar \(puede ser fijado a `-1` si quieres establecer una aprobación ilimitada\).