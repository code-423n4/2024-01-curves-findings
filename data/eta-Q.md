Unnecessary calculation and code in `FeeSplitter::getClaimableFees`

## Impact

In the `getClaimableFees` function, it is advised to directly return `data.unclaimedFees[account]` instead of calculating `(owed / PRECISION) + data.unclaimedFees[account]` since owed becomes zero after calling `updateFeeCredit. If the `owed` is required to be calculated, a check for if (balance > 0) is proposed in the `getClaimableFees` function before calculating `owed`, aligning it with the logic in the `updateFeeCredit` function. Additionally, in the `batchClaiming` function's notice, the word "the" is suggested for removal to enhance clarity.

## Proof of Concept
The `claimFees` and `batchClaiming` functions both call the `updateFeeCredit` and `getClaimableFees` functions. After calling `updateFeeCredit`, the assignment `data.userFeeOffset[account] = data.cumulativeFeePerToken;` results in `owed` being equal to 0 when calculating `(owed / PRECISION) + data.unclaimedFees[account]` in the `getClaimableFees` function. 

1.After calling the `updateFeeCredit` function, `data.cumulativeFeePerToken` remains unchanged：
1）uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
2）data.unclaimedFees[account] += owed / PRECISION;
3）data.userFeeOffset[account] = data.cumulativeFeePerToken;

2. Caculation in `getClaimableFees` Function
1）uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance = (data.cumulativeFeePerToken - data.cumulativeFeePerToken) * balance = 0;
2）(owed / PRECISION) + data.unclaimedFees[account] = 0 + data.unclaimedFees[account] = data.unclaimedFees[account]

Therefore, it is suggested to directly return `data.unclaimedFees[account]` to avoid unnecessary addition of 0.

[FeeSplitter::getClaimableFees](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L73C1-L78C6)
```solidity
    function getClaimableFees(address token, address account) public view returns (uint256) {
        TokenData storage data = tokensData[token];
-       uint256 balance = balanceOf(token, account);
-       uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
-       return (owed / PRECISION) + data.unclaimedFees[account];
+      return data.unclaimedFees[account];
    }
```

Additionally, if a separate calculation of `owed` is desired in the `getClaimableFees` function, it is recommended to check `if (balance > 0)` before proceeding, similar to the `updateFeeCredit` function. This adjustment ensures compatibility with the `getUserTokensAndClaimable` function, which not call `updateFeeCredit` before call `getClaimableFees`, as `getClaimableFees` is primarily used in FeeSplitter's `claimFees`, `batchClaiming`, and `getUserTokensAndClaimable` functions.

[FeeSplitter::getClaimableFees, updateFeeCredit](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63C1-L78C6)
```solidity
    function updateFeeCredit(address token, address account) internal {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
        if (balance > 0) {
            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
            data.unclaimedFees[account] += owed / PRECISION;
            data.userFeeOffset[account] = data.cumulativeFeePerToken;
        }
    }

    function getClaimableFees(address token, address account) public view returns (uint256) {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
+       if (balance > 0) {
        uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
}
        return (owed / PRECISION) + data.unclaimedFees[account];
    }
```
Finally, in the `batchClaiming` function's notice section, the word "the" can be removed.

[FeeSplitter::batchClaiming](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L102C1-L103C68)
```solidity
-    //@dev: this may fail if the the list is long. Get first the list with getUserTokens to estimate and prepare the batch
+   //@dev: this may fail if the list is long. Get first the list with getUserTokens to estimate and prepare the batch
    function batchClaiming(address[] calldata tokenList) external {
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Consider optimizing the logic in the code to avoid unnecessary calculations and ensure efficiency. 

[FeeSplitter::getClaimableFees](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L73C1-L78C6)
```solidity
    function getClaimableFees(address token, address account) public view returns (uint256) {
        TokenData storage data = tokensData[token];
-       uint256 balance = balanceOf(token, account);
-       uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
-       return (owed / PRECISION) + data.unclaimedFees[account];
+      return data.unclaimedFees[account];
    }
```
[FeeSplitter::batchClaiming](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L102C1-L103C68)
```solidity
-    //@dev: this may fail if the the list is long. Get first the list with getUserTokens to estimate and prepare the batch
+   //@dev: this may fail if the list is long. Get first the list with getUserTokens to estimate and prepare the batch
    function batchClaiming(address[] calldata tokenList) external {
```