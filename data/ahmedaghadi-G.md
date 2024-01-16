## 1. Don't read storage variable before `if` check as it won't be used if the block in `if` check doesn't get executed which will save `SLOAD`.

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

## 2. Skip unnecessary calculations in `getClaimableFees` function

The `claimFees` function executes `updateFeeCredit` function and then calls `getClaimableFees` function to get amount `claimable` ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L80 ):

```solidity
    function claimFees(address token) external {
@->        updateFeeCredit(token, msg.sender);
@->        uint256 claimable = getClaimableFees(token, msg.sender);
        if (claimable == 0) revert NoFeesToClaim();
        tokensData[token].unclaimedFees[msg.sender] = 0;
        payable(msg.sender).transfer(claimable);
        emit FeesClaimed(token, msg.sender, claimable);
    }
```

The `updateFeeCredit` function also updates `data.userFeeOffset[account] = data.cumulativeFeePerToken` and `getClaimableFees` function then calculate `uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance` which will always be zero ( also in case of `balance` equals to zero ), refer https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L63-L78 :

```solidity
    function updateFeeCredit(address token, address account) internal {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
        if (balance > 0) {
            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
            data.unclaimedFees[account] += owed / PRECISION;
@->            data.userFeeOffset[account] = data.cumulativeFeePerToken;
        }
    }

    function getClaimableFees(address token, address account) public view returns (uint256) {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
@->        uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
        return (owed / PRECISION) + data.unclaimedFees[account];
    }
```

So we can skip calling `getClaimableFees` function and directly assign `uint256 claimable = data.unclaimedFees[msg.sender]`, so the `claimFees` function would look like:

```diff
    function claimFees(address token) external {
        updateFeeCredit(token, msg.sender);
-        uint256 claimable = getClaimableFees(token, msg.sender);
+        uint256 claimable = data.unclaimedFees[msg.sender];
        if (claimable == 0) revert NoFeesToClaim();
        tokensData[token].unclaimedFees[msg.sender] = 0;
        payable(msg.sender).transfer(claimable);
        emit FeesClaimed(token, msg.sender, claimable);
    }
```

Similarly, `batchClaiming` function ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L103 ) can also be updated:

```diff
    function batchClaiming(address[] calldata tokenList) external {
        uint256 totalClaimable = 0;
        for (uint256 i = 0; i < tokenList.length; i++) {
            address token = tokenList[i];
            updateFeeCredit(token, msg.sender);
-            uint256 claimable = getClaimableFees(token, msg.sender);
+            uint256 claimable = data.unclaimedFees[msg.sender];
            if (claimable > 0) {
                tokensData[token].unclaimedFees[msg.sender] = 0;
                totalClaimable += claimable;
                emit FeesClaimed(token, msg.sender, claimable);
            }
        }
        if (totalClaimable == 0) revert NoFeesToClaim();
        payable(msg.sender).transfer(totalClaimable);
    }
```

## 3. `getFees` can early return 0 if `price` is zero.

The `getFees` function performs can return 0 on top if `price` is zero to save gas by skipping multiple calculations for different types of fees like `protocolFee`, `subjectFee`, `referralFee`, `holdersFee` and also `totalFee`. Refer ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L166 ):

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

So we can save gas by adding an `if` check in `getFees` function:

```diff
function getFees(
    uint256 price
)
    public
    view
    returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
{
+    if (price > 0) {
        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
+    }
}
```

## 4. `getFees` can be divided into two functions `getFees` and `getTotalFees` as only one of them will be used at a time.

The `getFees` function returns all the different types of fees like `protocolFee`, `subjectFee`, `referralFee`, `holdersFee` and also `totalFee`. Refer ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L166 ):

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

All the functions which calls `gasFees` only either uses `totalFees` or other remaining fees like

- `getBuyPriceAfterFee` function ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L199 ):

```solidity
(, , , , uint256 totalFee) = getFees(price);
```

- `getSellPriceAfterFee` function ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L204 ):

```solidity
(, , , , uint256 totalFee) = getFees(price);
```

- `_transferFees` function ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L218 ):

```solidity
(uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
```

- `_buyCurvesToken` function ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L263 ):

```solidity
(, , , , uint256 totalFee) = getFees(price);
```

So we can save gas by replacing `getFees` function by two functions `getFees` and `getTotalFees` like:

```diff
function getFees(
    uint256 price
)
    public
    view
-    returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
+    returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee)
{
    protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
    subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
    referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
    holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
-    totalFee = protocolFee + subjectFee + referralFee + holdersFee;
}

+ function getTotalFees(
+     uint256 price
+ )
+     public
+     view
+     returns (uint256 totalFee)
+ {
+     totalFee = (price * (feesEconomics.protocolFeePercent + feesEconomics.subjectFeePercent + feesEconomics.referralFeePercent + feesEconomics.holdersFeePercent))/ 1 ether;
+ }
```

In case of `totalFees` isn't required, a function can call `getFees` function and save gas by not calculating `totalFees` and only `totalFees` is required, a function can call `getTotalFees` function, which also saves multiple division by `1 ether` and multiple multiplication by `price`.
