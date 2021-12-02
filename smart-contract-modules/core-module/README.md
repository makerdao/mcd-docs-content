---
description: The State of the Maker Protocol
---

# Core Module

* **Module Name:** Vault Core Module
* **Type/Category:** Vault Core Module —> ( Vat.sol, Spot.sol )
* [**Associated MCD System Diagram**](https://github.com/makerdao/dss/wiki)
* **Contract Sources:**
  * [**Vat**](https://github.com/makerdao/dss/blob/master/src/vat.sol)****
  * ****[**Spot**](https://github.com/makerdao/dss/blob/master/src/spot.sol)****

## 1. Introduction (Summary)

The **Core Module** is crucial to the system as it contains the entire state of the Maker Protocol and controls the central mechanisms of the system while it is in the expected normal state of operation.

## 2. Module Details

### Core Module Components Documentation

1. [**Vat - Detailed Documentation**](https://docs.makerdao.com/smart-contract-modules/core-module/vat-detailed-documentation)
2. [**Spot - Detailed Documentation**](https://docs.makerdao.com/smart-contract-modules/core-module/spot-detailed-documentation)

## 3. Key Mechanism and Concepts

* `Vat` - The core Vault, Dai, and collateral state is kept in the `Vat`. The `Vat` contract has no external dependencies and maintains the central "Accounting Invariants" of Dai.
* `Spot` - `poke` is the only non-authenticated function in `spot`. The function takes in a `bytes32` of the `ilk` to be "poked". `poke` calls two `external` functions, `peek` and `file`.

## 4. Gotchas (Potential sources of user error)

* The methods in the `Vat` are written to be as generic as possible and as such have interfaces that can be quite verbose. Care should be taken that you have not mixed the order of parameters. Any module that is `auth`ed against the `Vat` has full root access, and can, therefore, steal all collateral in the system. This means that the addition of a new collateral type (and associated adapter) carries considerable risk.
* When the `Cat` is upgraded, there are multiple references to it that must be updated at the same time (`End`, `Vat.rely`, `Vow.rely`). It must also rely on the `End`, the system's `pause.proxy()`. Read more [here](https://docs.makerdao.com/smart-contract-modules/core-module/cat-detailed-documentation#4-gotchas-potential-source-of-user-error).
* The methods in the `spotter` are relatively basic compared to most other portions of `dss`. There is not much room for user error in the single unauthed method `poke`. If an incorrect `bytes32` is supplied the call will fail. Any module that is authed against the `spot` has full root access, and can, therefore, add and remove which `ilks` can be "poked". While not completely breaking the system, this could cause considerable risk.

## 5. Failure Modes (Bounds on Operating Conditions & External Risk Factors)

### Coding Errors

* `Vat` - A bug in the `Vat` could be catastrophic and could lead to the loss (or locking) of all Dai and Collateral in the system. It could become impossible to modify Vault's or to transfer Dai. Auctions could cease to function. Shutdown could fail.
* `Spot` - A bug in `spot` would most likely result in the prices for collaterals not being updated anymore. In this case, the system would need to authorize a new `spot` which would then be able to update the prices. Overall this is not a catastrophic failure as this would only pause all price fluctuation for some period.

### Feeds

* `Vat` - relies upon a set of trusted oracles to provide price data. Should these price feeds fail, it would become possible for unbacked Dai to be minted, or safe Vaults could be unfairly liquidated.
* `Spot` - relies upon a set of trusted oracles to provide price data. Should these price feeds fail, it would become possible for unbacked Dai to be minted, or safe Vaults could be unfairly liquidated.

### Governance

* `Vat` - Governance can authorize new modules against the `Vat`. This allows them to steal collateral (`slip`) or mint unbacked Dai (`suck`/addition of worthless collateral types). Should the crypto economic protections that make doing so prohibitively expensive fail, the system may be vulnerable and left open for bad actors to drain collateral.
