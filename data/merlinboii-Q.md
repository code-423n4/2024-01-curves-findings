# QA Report

## Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01) | Unremoved unowned curve token in `OwnedCurvesTokenSubjects` | Low |
| [L-02](#l-02) | Owner Unable to Burn Their External Tokens | Low |
| [L-03](#l-03) | Precision Loss in Fee Calculation | Low |
| [N-01](#n-01) | Require Extra `name` and `symbol` Validation | Non-Critical |
| [N-02](#n-02) | Unclear Naming for Open Sale `startTime` | Non-Critical |
| [N-03](#n-03) | Arithmetic Revert Lack of Transparency in First Token Purchase | Non-Critical |
| [G-01](#g-01) | Using `storage` in the read-only function can consume more gas | Gas |


## [L-01]<a name="l-01"></a> Unremoved unowned curve token in `OwnedCurvesTokenSubjects`

The `Curves` contract does not remove the curve token subject tracked from `OwnedCurvesTokenSubjects` when they have no balance of that token, causing a potential conflict with the variable's intended meaning.


## [L-02]<a name="l-02"></a> Owner Unable to Burn Their External Tokens

The owner of the external curve token (`CurveERC20`) is restricted from performing self-burning.

```solidity
    function burn(address from, uint256 amount) public onlyOwner {
            _burn(from, amount);
    }
```


## [L-03]<a name="l-03"></a> Precision Loss in Fee Calculation

Calculation Fee Precision loss is happended in the `Curves.getFees()` as using `1e18` to represent `100%` and the fee will be calculated by `price * xxFeePercent / 1e18`, leading to a loss of precision in the result of the fee calculation

```solidity
    function getFees(
        uint256 price
    )
        public
        view
        returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
    {
        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
    }
```


## [N-01]<a name="n-01"></a> Require Extra `name` and `symbol` Validation

The following functions require more validation of `name` and `symbol`.

* `Curves.buyCurvesTokenWithName()`
* `Curves.setNameAndSymbol()`

Let's consider the following cases that can happend from the current logic:

1. If the `name` is set to `XXX` but the symbol is `DEFAULT_SYMBOL`, all of them will be appended with a `_curvesTokenCounter` => `name: XXX {_curvesTokenCounter}` and `symbol: DEFAULT_SYMBOL {_curvesTokenCounter}`.

2. External curve tokens can use the same name but they should have a unique of both `name` and `symbol`. It can be compared to imagining the `name` as a `username`, and uniqueness is required.


## [N-02]<a name="n-02"></a> Unclear Naming for Open Sale `startTime`

The variable `presale.startTime` represents the time when the open sale begins. However, the current variable name is unclear and should be more explicit to avoid confusion.


## [N-03]<a name="n-03"></a> Arithmetic Revert Lack of Transparency in First Token Purchase

The `Curves.getPrice()`, specifically, if `(supply == 0 && amount > 1)`, it should explicitly revert of `UnavialbleAmountForFirstBuy` to enhance transparency, especially for the first token purchase. 

Currently, the function is noted as always arithmetically reverting for this case, but making this condition explicit can improve clarity in the code logic.


## [G-01]<a name="g-01"></a> Using `storage` in the read-only function can consume more gas

Using `storage` in the read-only (view) function can consume more gas when the function is called from internal/external contract.

