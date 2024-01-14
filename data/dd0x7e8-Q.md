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

## Duplicated Token Added in `userTokens` Array
The issue with the `onBalanceChange()` function in the `FeeSplitter` contract is that it indiscriminately adds the `token` to the `userTokens[account]` array every time there is a balance change for an account that is non-zero. This can lead to the same `token` being added multiple times for the same account, resulting in a bloated and redundant array.

Here's the problematic part of the function:

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L99C76-L99C76

```solidity
if (balanceOf(token, account) > 0) userTokens[account].push(token);
```

This line does not check whether the `token` already exists in the `userTokens[account]` array before pushing it. As a result, if the `onBalanceChange()` function is called multiple times for the same `token` and `account` pair, which is common in a token economy with frequent transactions, the `userTokens[account]` array for that account will accumulate duplicates of the `token` address.

The consequences of this issue are:

1. **Increased Gas Costs**: Every time a token is redundantly added, it costs gas. Over time, with multiple redundant additions, this can become unnecessarily expensive.

2. **Inefficient Storage**: The Ethereum blockchain is not an ideal place for storing large amounts of redundant data due to the high cost of storage.

3. **Slower Processing**: When iterating through the `userTokens[account]` array, having a large number of duplicates can slow down processing times for functions that use this array, such as when calculating claimable fees or performing batch operations.

4. **Complication of Logic**: Having duplicates can complicate the logic of other functions that depend on the `userTokens[account]` array, potentially leading to errors or unintended behavior.

A solution to this issue is to check whether the `token` is already in the `userTokens[account]` array before adding it. This can be done by either using a mapping to keep track of which tokens are already associated with an account or by iterating through the array to check for the presence of the `token` before adding it.

Here's a potential fix using a mapping to ensure uniqueness:

```solidity
mapping(address => mapping(address => bool)) private hasToken;

function onBalanceChange(address token, address account) public onlyManager {
    TokenData storage data = tokensData[token];
    data.userFeeOffset[account] = data.cumulativeFeePerToken;
    if (balanceOf(token, account) > 0 && !hasToken[account][token]) {
        userTokens[account].push(token);
        hasToken[account][token] = true;
    }
}
```

By adding the `hasToken` mapping, the contract can quickly check if a `token` is already associated with an `account` and avoid adding duplicates to the `userTokens[account]` array. This approach maintains the integrity of the array and ensures efficient use of resources.