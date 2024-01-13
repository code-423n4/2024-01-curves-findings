# Summary

## Low-Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| L&#x2011;01 | Loss of holder fees on balance change | 1 |
| L&#x2011;02 | `protocolFee` is not transferred to `protocolFeeDestination` on token selling | 1 |

## [L&#x2011;01] Loss of holder fees on balance change
### Lines of code
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L98
### Description
A variable `data.unclaimedFees[account]` is not updated hence it results in a loss of previously accumulated fees when the user trades tokens.
### Mitigation steps
Add a call to `updateFeeCredit` inside `FeeSplitter.sol::onBalanceChange`:
```diff
    function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
-       data.userFeeOffset[account] = data.cumulativeFeePerToken;
+       updateFeeCredit(token, account);
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }
```

## [L&#x2011;02] `protocolFee` is not transferred to `protocolFeeDestination` on token selling
### Lines of code
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L230
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L232
### Description
A value of `protocolFee` is accounted into the `buyValue` variable, however, this variable is not used when `isBuy` == false. That results in ETH of `protocolFee` left on Curves.sol contract and there is no option to withdraw it.
### Mitigation steps
Add a block of code that explicitly checks if `isBuy` is false and transfers protocol fee to the destination:
```
if (!isBuy) {
    (bool success, ) = feesEconomics.protocolFeeDestination.call{value: protocolFee}("");
    if (!success) revert CannotSendFunds();
}
```