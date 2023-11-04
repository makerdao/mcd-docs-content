# Módulo de Acuñamiento _Flash_ (Rápido)

* **Nombre del Contrato:** flash.sol
* **Tipo/Categoría:** DSS
* [**Fuente del Contrato**](https://github.com/makerdao/dss-flash/blob/master/src/flash.sol)
* [**Etherscan**](https://etherscan.io/address/0x1EB4CF3A948E7D72A198fe073cCb8C7a948cD853)

### 1. Introducción (Sumario)

El módulo `Flash` permite a cualquiera acuñar DAI hasta un límite establecido por la Gobernanza de Maker, con la condición de que se devuelva dicho monto con una tarifa en la misma transacción. Esto permite a cualquier sujeto explotar las oportunidades de arbitraje en el espacio DeFi sin comprometer un capital inicial.`Flash` provee muchos beneficios al ecosistema del DAI incluyendo, pero no limitando:

* Eficiencia de mercado para el DAI mejorada.
* Democratización del arbitraje - cualquiera puede participar.
* Las explotaciones que requieran una gran cantidad de capital se encontrarán más rápido, lo que hace que el espacio DeFi sea más seguro en general.
* Las tarifas proveen una fuente de ingreso para el Protocolo.

### 2. Detalles del Contrato

#### Glosario (_Flash_)

* **Techo de Deuda**: El monto máximo de DAI que, cualquier transacción única, puede pedir prestado. Codeado como `line` en unidades `rad`.
* **Tarifas de Acuñamiento**: Cuánto DAI adicional se debe devolver al módulo `Flash` una vez finalizada la transacción. Esta tarifa es transferida a `vow` al final de un acuñamiento (`mint`) exitoso. Codeado como `toll` en unidades `wad`.

### 3. Conceptos y Mecanismos Claves

#### Uso

Dado que el módulo _Flash_ se ajusta a la especificación ERC3156, se puede usar la implementación del prestatario de referencia de la especificación:

```text
pragma solidity ^0.8.0;

import "./interfaces/IERC20.sol";
import "./interfaces/IERC3156FlashBorrower.sol";
import "./interfaces/IERC3156FlashLender.sol";

contract FlashBorrower is IERC3156FlashBorrower {
    enum Action {NORMAL, OTHER}

    IERC3156FlashLender lender;

    constructor (
        IERC3156FlashLender lender_
    ) public {
        lender = lender_;
    }

    /// @dev ERC-3156 llamada de la función de préstamo rápido (Flash loan)
    function onFlashLoan(
        address initiator,
        address token,
        uint256 amount,
        uint256 fee,
        bytes calldata data
    ) external override returns (bytes32) {
        require(
            msg.sender == address(lender),
            "FlashBorrower: Untrusted lender"
        );
        require(
            initiator == address(this),
            "FlashBorrower: Untrusted loan initiator"
        );
        (Action action) = abi.decode(data, (Action));
        if (action == Action.NORMAL) {
            require(IERC20(token).balanceOf(address(this)) >= amount);
            // hacer un comercio rentable aquí
            IERC20(token).transfer(initiator, amount + fee);
        } else if (action == Action.OTHER) {
            // hacer otro
        }
        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }

    /// @dev iniciar un flash loan
    function flashBorrow(
        address token,
        uint256 amount
    ) public {
        bytes memory data = abi.encode(Action.NORMAL);
        uint256 _allowance = IERC20(token).allowance(address(this), address(lender));
        uint256 _fee = lender.flashFee(token, amount);
        uint256 _repayment = amount + _fee;
        IERC20(token).approve(address(lender), _allowance + _repayment);
        lender.flashLoan(this, token, amount, data);
    }
}
```

#### DAI del _Vat_

Puede ser que los usuarios estén interesados en mover DAI en los saldos internos del _vat_. En lugar de desperdiciar _gas_ acuñando/quemando DAI del ERC20, puedes usar la función de acuñamiento _flash_ de DAI del _vat_ para hacerlo más rápido.

El DAI del _vat_ del acuñamiento _flash_ es más o menos la misma que la versión del DAI de ERC20 pero con algunas advertencias:

**Función de Firma**

`vatDaiFlashLoan(IvatDaiFlashBorrower receiver, uint256 amount, bytes calldata data)`

contra

`flashLoan(IERC3156FlashBorrower receiver, address token, uint256 amount, bytes calldata data)`

Nótese que ningún token es requerido porque se asume que es DAI del _vat_. También, el `amount` (monto/cantidad) está en unidades `rad` y no es `wad`.

**Mecanismo de Aprobación**

El ERC3156 especifica el uso de una aprobación de token para aprobar el monto a reembolsar al prestamista. Desafortunadamente, el DAI del _vat_ no tiene una forma de especificar los montos de la delegación por lo que, en lugar de otorgar al módulo de acuñamiento _Flash_ todos los derechos para retirar cualquier cantidad de DAI del _vat_, hemos optado por que el receptor pague el saldo adeudado al final de la transacción.

**Ejemplo**

Aquí se encuentra un ejemplo, similar al anterior, para mostrar las diferencias:

```text
pragma solidity ^0.6.12;

import "dss-interfaces/dss/vatAbstract.sol";

import "./interfaces/IERC3156FlashLender.sol";
import "./interfaces/IvatDaiFlashBorrower.sol";

contract FlashBorrower is IvatDaiFlashBorrower {
    enum Action {NORMAL, OTHER}

    vatAbstract vat;
    IvatDaiFlashLender lender;

    constructor (
        vatAbstract vat_,
        IvatDaiFlashLender lender_
    ) public {
        vat = vat_;
        lender = lender_;
    }

    /// @dev llamada de la función de préstamo rápido de DAI del Vat (Vat DAI Flash Loan)
    function onvatDaiFlashLoan(
        address initiator,
        uint256 amount,
        uint256 fee,
        bytes calldata data
    ) external override returns (bytes32) {
        require(
            msg.sender == address(lender),
            "FlashBorrower: Untrusted lender"
        );
        require(
            initiator == address(this),
            "FlashBorrower: Untrusted loan initiator"
        );
        (Action action) = abi.decode(data, (Action));
        if (action == Action.NORMAL) {
            // hacer una cosa
        } else if (action == Action.OTHER) {
            // hacer otra
        }

        // Devolver el monto del préstamo (loan) + tarifa (fee)
        // Asegurarse de no pagar de más ya que no hay medidas de seguridad para esto
        vat.move(address(this), lender, amount + fee);

        return keccak256("vatDaiFlashBorrower.onvatDaiFlashLoan");
    }

    /// @dev iniciar un flash loan
    function vatDaiFlashBorrow(
        uint256 amount
    ) public {
        bytes memory data = abi.encode(Action.NORMAL);
        lender.vatDaiFlashLoan(this, amount, data);
    }
}
```