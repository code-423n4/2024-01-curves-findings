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