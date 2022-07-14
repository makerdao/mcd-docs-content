# Pymaker

## Introducción

El Protocolo de Maker incientiva a los agentes externos, llamados _keepers_, a automatizar ciertas operaciones alrededor de la _blockchain_ de Ethereum. Para facilitar su desarrollo, se ha creado una API en torno a la mayoría de los contratos de Maker. Puede ser utilizado no solo por los _keepers_, sino también por los autores de otras utilidades no relacionadas que buscan interactuar con estos contratos.

Se está desarrollando un conjunto de _keepers_ de Maker de referencia basado en la [API de Pymaker](https://github.com/makerdao/pymaker). 
Todos ellos solían residir en este repositorio, pero ahora cada uno de ellos tiene uno individual: [bite-keeper (_keeper_ de "mordida")](https://github.com/makerdao/bite-keeper) (solo para SCD), [arbitrage-keeper (_keeper_ de arbitraje)](https://github.com/makerdao/arbitrage-keeper), [auction-keeper (_keeper_ de subastas)](https://github.com/makerdao/auction-keeper) (solo para MCD), [cdp-keeper (_keeper_ de cdp/_vault_)](https://github.com/makerdao/cdp-keeper) (solo para SCD), [market-maker-keeper (_keeper_ del mercado de Maker)](https://github.com/makerdao/market-maker-keeper).

Solo necesitas instalar directamente este proyecto si quieres construir tus propios _keepers_, o si quieres jugar con la API de esta librería. Si solo quieres instalar uno de los _keepers_ de referencia, ve a uno de los links del repositorio (que se encuentran arriba) y comienza por ahí. Cada uno de estos _keepers_ hace referencia a alguna versión de `pymaker` a través de un submódulo de Git.

## Instalación

Este proyecto utiliza _Python 3.6.6_.

Para clonar este proyecto e instalar los paquetes de terceras partes, ejecuta:

```
git clone https://github.com/makerdao/pymaker.git
cd pymaker
pip3 install -r requirements.txt
```

#### Problemas Conocidos de Ubuntu

Para que la dependencia `secp256k` de Python compile correctamente, los siguientes paquetes deben ser instalados:

```
sudo apt-get install build-essential automake libtool pkg-config libffi-dev python-dev python-pip libsecp256k1-dev
```

(para el Server 18.04 de Ubuntu)

#### Problemas Conocidos de macOS

Para que los requerimientos de Python se instalen correctamente en _macOS_, instala `openssl`, `libtool`, `pkg-config` y `automake` utilizando [Homebrew](https://brew.sh):

```
brew install openssl libtool pkg-config automake
```

y establece la variable del ambiente `LDFLAGS` antes de correr `pip3 install -r requirements.txt`:

```
export LDFLAGS="-L$(brew --prefix openssl)/lib" CFLAGS="-I$(brew --prefix openssl)/include" 
```

## APIs Disponibles

**La versión actual provee APIs en torno de:**

* `ERC20Token`,
* `Tub`, `Tap`,`Top` y `Vox` ([https://github.com/makerdao/sai](https://github.com/makerdao/sai)),
* `Vat`, `Cat`, `Vow`, `Jug`, `Flipper`, `Flapper`, `Flopper` ([https://github.com/makerdao/dss](https://github.com/makerdao/dss))
* `SimpleMarket`, `ExpiringMarket` y `MatchingMarket` ([https://github.com/makerdao/maker-otc](https://github.com/makerdao/maker-otc)),
* `TxManager` ([https://github.com/makerdao/tx-manager](https://github.com/makerdao/tx-manager)),
* `DSGuard` ([https://github.com/dapphub/ds-guard](https://github.com/dapphub/ds-guard)),
* `DSToken` ([https://github.com/dapphub/ds-token](https://github.com/dapphub/ds-token)),
* `DSEthToken` ([https://github.com/dapphub/ds-eth-token](https://github.com/dapphub/ds-eth-token)),
* `DSValue` ([https://github.com/dapphub/ds-value](https://github.com/dapphub/ds-value)),
* `DSVault` ([https://github.com/dapphub/ds-vault](https://github.com/dapphub/ds-vault)),
* `EtherDelta` ([https://github.com/etherdelta/etherdelta.github.io](https://github.com/etherdelta/etherdelta.github.io)),
* `0x v1` ([https://etherscan.io/address/0x12459c951127e0c374ff9105dda097662a027093#code](https://etherscan.io/address/0x12459c951127e0c374ff9105dda097662a027093#code), [https://github.com/0xProject/standard-relayer-api](https://github.com/0xProject/standard-relayer-api)),
* `0x v2`
* `Dai Savings Rate (Pot)` - "Tasa de Ahorro del DAI"([https://github.com/makerdao/pymaker/blob/master/tests/manual\_test\_dsr.py#L29](https://github.com/makerdao/pymaker/blob/master/tests/manual\_test\_dsr.py#L29))

**No se han implementado las APIs en torno a las siguientes funcionalidades:**

* `Global Settlement (End)` - "Asentamiento/Liquidación Global"
* `Governance (DSAuth, DSChief, DSGuard, DSSpell, Mom)` - "Gobernanza"

¡Se aprecia cualquier contribución de la comundidad!

### Muestras del Código

Debajo puedes encontrar algunos recortes del código demostrando cómo la API puede ser utilizada tanto para desarrollar tus propios _keepers_ como para crear otras utilidades para interactuar con los contratos del ecosistema del Protocolo de Maker.

#### Transferencia de Tokens

Este recorte demuestra cómo transferir algo de SAI de nuestra _address_ que se encuentra por defecto. La _address_ de los tokens SAI se descubre consultando el `Tub`, por lo que todo lo que necesitamos es una _address_ del `Tub`:

```
from web3 import HTTPProvider, Web3

from pymaker import Address
from pymaker.token import ERC20Token
from pymaker.numeric import Wad
from pymaker.sai import Tub


web3 = Web3(HTTPProvider(endpoint_uri="http://localhost:8545"))

tub = Tub(web3=web3, address=Address(' 0xb7ae5ccabd002b5eebafe6a8fad5499394f67980'))
sai = ERC20Token(web3=web3, address=tub.sai())

sai.transfer(address=Address(' 0x0000000000111111111100000000001111111111'),
             value=Wad.from_number(10)).transact()
```

#### Actualizando una DSValue

Este recorte demuestra cómo se actualiza una `DSValue` con la tasa ETH/USD extraída de _CryptoCompare_:

```
import json
import urllib.request

from web3 import HTTPProvider, Web3

from pymaker import Address
from pymaker.feed import DSValue
from pymaker.numeric import Wad


def cryptocompare_rate() -> Wad:
    with urllib.request.urlopen("https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD") as url:
        data = json.loads(url.read().decode())
        return Wad.from_number(data['USD'])


web3 = Web3(HTTPProvider(endpoint_uri="http://localhost:8545"))

dsvalue = DSValue(web3=web3, address=Address(' 0x038b3d8288df582d57db9be2106a27be796b0daf'))
dsvalue.poke_with_int(cryptocompare_rate().value).transact()
```

#### Introspección del SAI

Este recorte demuestra cómo se capta datos de los contratos de `Tub` y `Tap`:

```
from web3 import HTTPProvider, Web3

from pymaker import Address
from pymaker.token import ERC20Token
from pymaker.numeric import Ray
from pymaker.sai import Tub, Tap


web3 = Web3(HTTPProvider(endpoint_uri="http://localhost:8545"))

tub = Tub(web3=web3, address=Address(' 0x448a5065aebb8e423f0896e6c5d525c040f59af3'))
tap = Tap(web3=web3, address=Address(' 0xbda109309f9fafa6dd6a9cb9f1df4085b27ee8ef'))
sai = ERC20Token(web3=web3, address=tub.sai())
skr = ERC20Token(web3=web3, address=tub.skr())
gem = ERC20Token(web3=web3, address=tub.gem())

print(f"")
print(f"Token summary")
print(f"-------------")
print(f"SAI total supply       : {sai.total_supply()} SAI")
print(f"SKR total supply       : {skr.total_supply()} SKR")
print(f"GEM total supply       : {gem.total_supply()} GEM")
print(f"")
print(f"Collateral summary")
print(f"------------------")
print(f"GEM collateral         : {tub.pie()} GEM")
print(f"SKR collateral         : {tub.air()} SKR")
print(f"SKR pending liquidation: {tap.fog()} SKR")
print(f"")
print(f"Debt summary")
print(f"------------")
print(f"Debt ceiling           : {tub.cap()} SAI")
print(f"Good debt              : {tub.din()} SAI")
print(f"Bad debt               : {tap.woe()} SAI")
print(f"Surplus                : {tap.joy()} SAI")
print(f"")
print(f"Feed summary")
print(f"------------")
print(f"REF per GEM feed       : {tub.pip()}")
print(f"REF per SKR price      : {tub.tag()}")
print(f"GEM per SKR price      : {tub.per()}")
print(f"")
print(f"Tub parameters")
print(f"--------------")
print(f"Liquidation ratio      : {tub.mat()*100} %")
print(f"Liquidation penalty    : {tub.axe()*100 - Ray.from_number(100)} %")
print(f"Stability fee          : {tub.tax()} %")
print(f"")
print(f"All cups")
print(f"--------")
for cup_id in range(1, tub.cupi()+1):
    cup = tub.cups(cup_id)
    print(f"Cup #{cup_id}, lad={cup.lad}, ink={cup.ink} SKR, tab={tub.tab(cup_id)} SAI, safe={tub.safe(cup_id)}")
```

#### DAI Multicolateral

Este recorte demuestra cómo se crea un CDP/una _vault_ y como se retira DAI.

```
import sys
from web3 import Web3, HTTPProvider

from pymaker import Address
from pymaker.deployment import DssDeployment
from pymaker.keys import register_keys
from pymaker.numeric import Wad


web3 = Web3(HTTPProvider(endpoint_uri="https://localhost:8545",
                         request_kwargs={"timeout": 10}))
web3.eth.defaultAccount = sys.argv[1]   # ex:  0x0000000000000000000000000000000aBcdef123
register_keys(web3, [sys.argv[2]])      # ex: key_file=~keys/default-account.json,pass_file=~keys/default-account.pass

mcd = DssDeployment.from_json(web3=web3, conf=open("tests/config/kovan-addresses.json", "r").read())
our_address = Address(web3.eth.defaultAccount)


# Elige el colateral deseado; en este caso, vamos a utilizar al ETH
collateral = mcd.collaterals['ETH-A']
ilk = collateral.ilk
collateral.gem.deposit(Wad.from_number(3)).transact()

# Agrega el colateral y coloca el monto de DAI deseado
collateral.approve(our_address)
collateral.adapter.join(our_address, Wad.from_number(3)).transact()
mcd.vat.frob(ilk, our_address, dink=Wad.from_number(3), dart=Wad.from_number(153)).transact()
print(f"CDP Dai balance before withdrawal: {mcd.vat.dai(our_address)}")

# Acuña y retira nuestro DAI
mcd.approve_dai(our_address)
mcd.dai_adapter.exit(our_address, Wad.from_number(153)).transact()
print(f"CDP Dai balance after withdrawal:  {mcd.vat.dai(our_address)}")

# Paga (y quema) nuestro DAI
assert mcd.dai_adapter.join(our_address, Wad.from_number(153)).transact()
print(f"CDP Dai balance after repayment:   {mcd.vat.dai(our_address)}")

# Retira nuestro colateral
mcd.vat.frob(ilk, our_address, dink=Wad(0), dart=Wad.from_number(-153)).transact()
mcd.vat.frob(ilk, our_address, dink=Wad.from_number(-3), dart=Wad(0)).transact()
collateral.adapter.exit(our_address, Wad.from_number(3)).transact()
print(f"CDP Dai balance w/o collateral:    {mcd.vat.dai(our_address)}")
```

#### Invocación Asincrónica de las Transacciones de Ethereum

Este recorte demuestra cómo múltiples tranferencias de tokens pueden ser ejecutadas de manera asincrónica:

```
from web3 import HTTPProvider
from web3 import Web3

from pymaker import Address, synchronize
from pymaker.numeric import Wad
from pymaker.sai import Tub
from pymaker.token import ERC20Token


web3 = Web3(HTTPProvider(endpoint_uri="http://localhost:8545"))

tub = Tub(web3=web3, address=Address(' 0x448a5065aebb8e423f0896e6c5d525c040f59af3'))
sai = ERC20Token(web3=web3, address=tub.sai())
skr = ERC20Token(web3=web3, address=tub.skr())

synchronize([sai.transfer(Address(' 0x0101010101020202020203030303030404040404'), Wad.from_number(1.5)).transact_async(),
             skr.transfer(Address(' 0x0303030303040404040405050505050606060606'), Wad.from_number(2.5)).transact_async()])
```

#### Múltiples Invocaciones en una Transacción de Ethereum

Este recorte demuestra cómo múltiples transacciones de tokens pueden ser ejecutadas en una sola transacción de Ethereum. La persona que llame a la instancia `TxManager` debe ser dueña de esta y debe implementarla.

```
from web3 import HTTPProvider
from web3 import Web3

from pymaker import Address
from pymaker.approval import directly
from pymaker.numeric import Wad
from pymaker.sai import Tub
from pymaker.token import ERC20Token
from pymaker.transactional import TxManager


web3 = Web3(HTTPProvider(endpoint_uri="http://localhost:8545"))

tub = Tub(web3=web3, address=Address(' 0x448a5065aebb8e423f0896e6c5d525c040f59af3'))
sai = ERC20Token(web3=web3, address=tub.sai())
skr = ERC20Token(web3=web3, address=tub.skr())

tx = TxManager(web3=web3, address=Address(' 0x57bFE16ae8fcDbD46eDa9786B2eC1067cd7A8f48'))
tx.approve([sai, skr], directly())

tx.execute([sai.address, skr.address],
           [sai.transfer(Address(' 0x0101010101020202020203030303030404040404'), Wad.from_number(1.5)).invocation(),
            skr.transfer(Address(' 0x0303030303040404040405050505050606060606'), Wad.from_number(2.5)).invocation()]).transact()
```

#### A pedido: Incremento del precio del _gas_ para transacciones asíncronas

```
import asyncio
from random import randint

from web3 import Web3, HTTPProvider

from pymaker import Address
from pymaker.gas import FixedGasPrice
from pymaker.oasis import SimpleMarket


web3 = Web3(HTTPProvider(endpoint_uri=f"http://localhost:8545"))
otc = SimpleMarket(web3=web3, address=Address(' 0x375d52588c3f39ee7710290237a95C691d8432E7'))


async def bump_with_increasing_gas_price(order_id):
    gas_price = FixedGasPrice(gas_price=1000000000)
    task = asyncio.ensure_future(otc.bump(order_id).transact_async(gas_price=gas_price))

    while not task.done():
        await asyncio.sleep(1)
        gas_price.update_gas_price(gas_price.gas_price + randint(0, gas_price.gas_price))

    return task.result()


bump_task = asyncio.ensure_future(bump_with_increasing_gas_price(otc.get_orders()[-1].order_id))
event_loop = asyncio.get_event_loop()
bump_result = event_loop.run_until_complete(bump_task)

print(bump_result)
print(bump_result.transaction_hash)
```

## Testeo

Pre-requisitos:

* [docker y docker-compose](https://www.docker.com/get-started)
* [ganache-cli](https://github.com/trufflesuite/ganache-cli) 6.2.5\
  (using npm, `sudo npm install -g ganache-cli@6.2.5`)

Este proyecto utiliza [pytest](https://docs.pytest.org/en/latest/) para el testeo de unidades. El testeo de DAI Multicolateral es realizado en una cadena de testeo local en un Docker incluido en `tests\config`.

Para poder correr testeos, instala primero las dependecias de desarrollo ejecutando:

```
pip3 install -r requirements-dev.txt
```

Luego de esto puedes correr todos los testeos con:

```
./test.sh
```
Si tienes consultas acerca de Pymaker, contáctanos en el canal de [#keeper](https://chat.makerdao.com/channel/keeper) en [**chat.makerdao.com**](http://chat.makerdao.com).