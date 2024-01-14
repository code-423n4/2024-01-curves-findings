## No Upper Limit for Max Fee Percent
The issue with the `setMaxFeePercent()` function in the `Curves` contract is that it lacks a validation check to ensure that the maximum fee percentage (`maxFeePercent`) does not exceed 100%, which is represented by `1 ether`.

Here's the problematic part of the `setMaxFeePercent()` function:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117C1-L126C6

```solidity
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
    if (
        feesEconomics.protocolFeePercent +
            feesEconomics.subjectFeePercent +
            feesEconomics.referralFeePercent +
            feesEconomics.holdersFeePercent >
        maxFeePercent_
    ) revert InvalidFeeDefinition();
    feesEconomics.maxFeePercent = maxFeePercent_;
}
```

The function allows an external manager to set the `maxFeePercent`, but it does not prevent the new `maxFeePercent` from being set to a value greater than `1 ether` (100%). 

To resolve this issue, the `setMaxFeePercent()` function should include a check to ensure that the `maxFeePercent` value being set does not exceed `1 ether`:

```solidity
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
    require(maxFeePercent_ < 1 ether, "Invalid max fee percent");
    ...
    feesEconomics.maxFeePercent = maxFeePercent_;
}
```

## Insufficient Check Condition in `buyCurvesToken()` Function
The issue with the `buyCurvesToken()` function in the provided code snippet arises from an incorrect logic check pertaining to the `startTime` of a token sale. The original check is as follows:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L213C1-L213C1

```solidity
        if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
```

This condition checks if the `startTime` is not equal to zero and if the `startTime` is in the future relative to the current timestamp (`block.timestamp`). The implication here is that if `startTime` is not zero (which means a presale start time has been set) and the presale start time has not yet passed, then the sale is not open, and the function should revert.

However, the issue with this logic is that it does not account for the scenario where `startTime` is exactly zero. In the context of the contract, a `startTime` of zero could be interpreted as the sale not being available or not yet scheduled. Thus, the sale should not be open in this case either.

The corrected version of the check is:

```solidity
        if (startTime == 0 || startTime >= block.timestamp) revert SaleNotOpen();
```

This revised condition will revert the transaction if:

1. The `startTime` is zero, which implies that no sale has been set up, or the presale is unavailable.
2. The `startTime` is in the future, meaning the presale or sale has not yet started.

By using the logical OR (`||`), the function correctly covers both scenarios where the sale should not be allowed to proceed, thus preventing premature token purchases and aligning with the intended sale schedule logic.

## Check-Effects-Interactions Pattern Violation
The issue with the `_buyCurvesToken()` function, as described, arises from the order of operations, particularly the placement of the `_transferFees()` call before the state-modifying function `_addOwnedCurvesTokenSubject()`. This ordering could potentially make the function vulnerable to reentrancy attacks.

Here's the problematic sequence:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L274C1-L279C10

```solidity
        _transferFees(curvesTokenSubject, true, price, amount, supply);
        // ...
        _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
```

The `_transferFees()` function includes external calls to transfer ETH, which are potentially unsafe because they can be hijacked by malicious contracts to re-enter the current contract before the initial execution is completed.

```solidity
        (bool success, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
        // ... other external calls
```

According to the `Check-Effects-Interactions` pattern recommended by Solidity, state changes should be performed before any external interactions. This prevents other contracts from disrupting the expected flow of execution, which could lead to unintended outcomes, including loss of funds or corruption of the contract's state.

## Restriction on Initial Share Purchase (Self-Buy)
The issue with the `Curves` contract is that it has a design limitation or specific behavior that restricts the initial purchase (self-buy) by the token subject (the creator or owner of the token) to exactly one share. This is due to how the `getPrice()` function calculates the cost for a given amount of tokens based on the current supply.

Here is the critical part of the `getPrice()` function that leads to this behavior:
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L180C1-L187C6

```solidity
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
```

If the `supply` is zero and `amount` is anything other than one, the expression `(supply - 1 + amount)` would result in an underflow because `supply - 1` is negative when `supply` is zero. Solidity underflowing will revert the transaction, preventing the purchase of more than one token when the supply is zero.

Moreover, the `setWhitelist()` function adds an additional constraint:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394C1-L396C53

```solidity
        if (supply > 1) revert CurveAlreadyExists();
```

This code indicates that once the supply exceeds one, the whitelist can no longer be updated, implying that the initial buy must be exactly one unit of the token.

The difference between the behavior of the `Curves` contract and the `Friend.tech` could lead to confusion among users. Users familiar with `Friend.tech` might expect the ability to self-buy any initial amount, and if not well documented, this discrepancy in `Curves` could result in a poor user experience.

To address this issue and improve user experience:

1. **Clarify in Documentation**: It should be clearly documented that the initial purchase by the token subject is limited to exactly one unit to avoid confusion.

2. **Validate Input**: Implement checks in functions that call `getPrice()` to ensure that if the supply is zero, the `amount` is one, and provide a clear revert message if not.

3. **Adjust Design**: If the intent is to allow subjects to buy more than one unit initially, the `getPrice()` function and other related logic should be adjusted to support this use case.

By implementing these measures, the `Curves` contract can provide clarity and consistency to users, aligning with their expectations and preventing unexpected reverts due to underflows in the pricing calculation.