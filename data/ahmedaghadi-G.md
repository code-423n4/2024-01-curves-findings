Don't read storage variable before `if` check as it won't be used if the block in `if` check doesn't get executed which will save `SLOAD`.
Consider the function in https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L63
```solidity
    function updateFeeCredit(address token, address account) internal {
@->        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
        if (balance > 0) {
            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
            data.unclaimedFees[account] += owed / PRECISION;
            data.userFeeOffset[account] = data.cumulativeFeePerToken;
        }
    }
```
If the `balance` is equal to 0 then we don't need `data`. So making following changes would save the gas in case of `balance` equals to 0 and will save `SLOAD`:
```diff
    function updateFeeCredit(address token, address account) internal {
-        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
        if (balance > 0) {
+            TokenData storage data = tokensData[token];
            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
            data.unclaimedFees[account] += owed / PRECISION;
            data.userFeeOffset[account] = data.cumulativeFeePerToken;
        }
    }
```