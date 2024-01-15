# Maximum Fee Percentage Not Capped
## Locations
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117-L126

## Category
Logic Issue

## Severity
Minor

## Description
The `setMaxFeePercent()` function in the `Curves` contract currently lacks a safeguard to ensure that the sum of the individual fee percentages does not surpass the logical limit of 100%, or `1 ether` in Solidity terms.

Here is the section of the `setMaxFeePercent()` function that requires attention:

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

The method permits a manager to define the `maxFeePercent`, but fails to include a critical validation step to prevent setting this maximum fee percentage beyond the full percentage value, which should be capped at `1 ether`.

## Recommendation
It's recommended to add upper limit for `maxFeePercent` which does not exceed `1 ether`. 

---

# Breach of the Check-Effects-Interactions Pattern
## Locations
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L274-L279

## Category
Logic Issue

## Severity
Minor

## Description
The `_buyCurvesToken()` function is susceptible to reentrancy attacks because it calls `_transferFees()`, which includes external ETH transfers, before updating the contract's state with `_addOwnedCurvesTokenSubject()`. This goes against the recommended Solidity pattern of 'Check-Effects-Interactions', which dictates that state changes should precede external calls to prevent reentry risks. 

## Recommendation
It's recommended to adjust the order of these operations to secure the function.

---

# Duplicate Token Inserted into `userTokens` Array
## Locations
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L99

## Category
Optimization

## Description
The `FeeSplitter` contract's `onBalanceChange()` function has a flaw where it adds a `token` to `userTokens[account]` whenever there's a positive balance change, without checking for duplicates. The code snippet:

```solidity
if (balanceOf(token, account) > 0) userTokens[account].push(token);
```

lacks a mechanism to prevent the same `token` from being added multiple times to an account's array. This could lead to unnecessary repetition of `token` addresses in the array, especially with frequent balance changes, causing array bloat.

## Recommendation
It's recommended to check whether the `token` is already in the `userTokens[account]` array before adding it.