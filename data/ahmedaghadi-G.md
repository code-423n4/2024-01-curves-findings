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

## 5. `getPrice` can be optimized by simplifying the calculation and early return.

The `getPrice` function is as follows ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L180 ):

```solidity
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}
```

It basically calculates `summation`, which is equal to `sum1 + sum2`, where `sum1` is sum of squares of numbers from 0 to `supply - 1` and `sum2` is sum of squares of numbers from 0 to `supply + amount - 1`, we can simplify the calculation by directly calculates `summation` to be equal to sum of sqaures of numbers from `supply` to `supply + amount`. To calculate that:

```
n = supply - 1
k = amount
summation = ((n+k)*(n+k+1)*(2(n+k)+1)/6) - (n*(n+1)*(2n+1)/6)

Simplifying,

summation = (k * (1 + k*(2*k + 3 + 6*n) + 6*n*(1 + n)))/6
```

Refer Wolframalpha's [result](https://www.wolframalpha.com/input?i=%28%28n%2Bk%29*%28n%2Bk%2B1%29*%282%28n%2Bk%29%2B1%29%2F6%29-%28n*%28n%2B1%29*%282n%2B1%29%2F6%29+%3D+%28k+*+%281+%2B+k*%282*k+%2B+3+%2B+6*n%29+%2B+6*n*%281+%2B+n%29%29%29%2F6)

Further gas can be saved for early return in the case of amount to be 0. And we can also not divide `summation` by 6 and divide final return value by `96000` in the place of `16000` as `16000 * 6 = 96000`.

So the modified function will be:

```solidity
function getPrice2(
    uint256 supply,
    uint256 amount
) public pure returns (uint256) {
    if (amount == 0) {
        return 0;
    }
    uint256 summation;
    if (supply == 0) {
        if (amount == 1) {
            return 0;
        } else {
            summation =
                (supply + amount - 1) *
                (supply + amount) *
                (2 * (supply + amount - 1) + 1);
        }
    } else {
        uint256 n = supply - 1;
        summation =
            amount *
            (1 + amount * (2 * amount + (3 + 6 * n)) + 6 * n * (1 + n));
    }
    return (summation * 1 ether) / 96000;
}
```

Foundry test to verify the lower gas fees:

```solidity
function getPrice1(
    uint256 supply,
    uint256 amount
) public pure returns (uint256) {
    uint256 sum1 = supply == 0
        ? 0
        : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply + amount - 1) *
            (supply + amount) *
            (2 * (supply + amount - 1) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}

function getPrice2(
    uint256 supply,
    uint256 amount
) public pure returns (uint256) {
    if (amount == 0) {
        return 0;
    }
    uint256 summation;
    if (supply == 0) {
        if (amount == 1) {
            return 0;
        } else {
            summation =
                (supply + amount - 1) *
                (supply + amount) *
                (2 * (supply + amount - 1) + 1);
        }
    } else {
        uint256 n = supply - 1;
        summation =
            amount *
            (1 + amount * (2 * amount + (3 + 6 * n)) + 6 * n * (1 + n));
    }
    return (summation * 1 ether) / 96000;
}

function test_gas() public {
    uint256 supply = 100;
    uint256 amount = 50;
    uint256 gasBefore1 = gasleft();
    uint256 res1 = getPrice1(supply, amount);
    uint256 gasAfter1 = gasleft();
    uint256 gasUsed1 = gasBefore1 - gasAfter1;

    uint256 gasBefore2 = gasleft();
    uint256 res2 = getPrice2(supply, amount);
    uint256 gasAfter2 = gasleft();
    uint256 gasUsed2 = gasBefore2 - gasAfter2;

    console2.log(gasUsed1); // 1760
    console2.log(gasUsed2); // 1284

    assertEq(res1, res2);
    assert(gasUsed1 > gasUsed2);
}
```

## 6. Store the storage variable in memory to reuse it without additional `SLOAD` in `_transferFees` function.

The `_transferFees` function is as follows ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L218 ):

```solidity
function _transferFees(
    address curvesTokenSubject,
    bool isBuy,
    uint256 price,
    uint256 amount,
    uint256 supply
) internal {
    (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
    {
@->        bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
        {
            address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
            uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
            uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
            (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
            if (!success1) revert CannotSendFunds();
        }
        {
            (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
            if (!success2) revert CannotSendFunds();
        }
        {
            (bool success3, ) = referralDefined
@->                ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                : (true, bytes(""));
            if (!success3) revert CannotSendFunds();
        }

        if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
            feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
            feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
        }
    }
    emit Trade(
        msg.sender,
        curvesTokenSubject,
        isBuy,
        amount,
        price,
        protocolFee,
        subjectFee,
        isBuy ? supply + amount : supply - amount
    );
}
```

`referralFeeDestination[curvesTokenSubject]` is used twice, once in `referralDefined` and then in `success3`. So we can save gas by storing it in memory and reusing it. So the modified function will be:

```diff
function _transferFees(
    address curvesTokenSubject,
    bool isBuy,
    uint256 price,
    uint256 amount,
    uint256 supply
) internal {
    (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
+    address referralFeeDestinationAddress = referralFeeDestination[curvesTokenSubject];
    {
-        bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
+        bool referralDefined = referralFeeDestinationAddress != address(0);
        {
            address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
            uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
            uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
            (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
            if (!success1) revert CannotSendFunds();
        }
        {
            (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
            if (!success2) revert CannotSendFunds();
        }
        {
            (bool success3, ) = referralDefined
-                ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
+                ? referralFeeDestinationAddress.call{value: referralFee}("")
                : (true, bytes(""));
            if (!success3) revert CannotSendFunds();
        }

        if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
            feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
            feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
        }
    }
    emit Trade(
        msg.sender,
        curvesTokenSubject,
        isBuy,
        amount,
        price,
        protocolFee,
        subjectFee,
        isBuy ? supply + amount : supply - amount
    );
}
```

This will save 1 `SLOAD` in case of `referralDefined` is `true`.

## 7. `feesEconomics.holdersFeePercent > 0` condition can be replaced by `holderFee > 0` condition in `_transferFees` function to save `SLOAD`.

The `_transferFees` function is as follows ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L218 ):

```solidity
function _transferFees(
    address curvesTokenSubject,
    bool isBuy,
    uint256 price,
    uint256 amount,
    uint256 supply
) internal {
@->    (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
    {
        bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
        {
            address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
            uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
            uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
            (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
            if (!success1) revert CannotSendFunds();
        }
        {
            (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
            if (!success2) revert CannotSendFunds();
        }
        {
            (bool success3, ) = referralDefined
                ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                : (true, bytes(""));
            if (!success3) revert CannotSendFunds();
        }

@->        if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
            feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
            feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
        }
    }
    emit Trade(
        msg.sender,
        curvesTokenSubject,
        isBuy,
        amount,
        price,
        protocolFee,
        subjectFee,
        isBuy ? supply + amount : supply - amount
    );
}
```

`holderFee` returned from `getFees` function is calculated as ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L166 ):

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
@->    holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
    totalFee = protocolFee + subjectFee + referralFee + holdersFee;
}
```

So if `feesEconomics.holdersFeePercent` is greater than 0 then `holderFee` will also be greater than 0. So we can replace `feesEconomics.holdersFeePercent > 0` condition by `holderFee > 0` condition to save `SLOAD`. So the modified function will be:

```diff
function _transferFees(
    address curvesTokenSubject,
    bool isBuy,
    uint256 price,
    uint256 amount,
    uint256 supply
) internal {
    (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
    {
        bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
        {
            address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
            uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
            uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
            (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
            if (!success1) revert CannotSendFunds();
        }
        {
            (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
            if (!success2) revert CannotSendFunds();
        }
        {
            (bool success3, ) = referralDefined
                ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                : (true, bytes(""));
            if (!success3) revert CannotSendFunds();
        }

-        if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
+        if (holderFee > 0 && address(feeRedistributor) != address(0)) {
            feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
            feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
        }
    }
    emit Trade(
        msg.sender,
        curvesTokenSubject,
        isBuy,
        amount,
        price,
        protocolFee,
        subjectFee,
        isBuy ? supply + amount : supply - amount
    );
}
```

## 8. `supply - amount` can be cached to reuse it to skip performing `SUB` again in `sellCurvesToken` function.

The `sellCurvesToken` function uses `supply - amount` twice ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L282 ):

```solidity
function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply <= amount) revert LastTokenCannotBeSold();
    if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

@->    uint256 price = getPrice(supply - amount, amount);

    curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
@->    curvesTokenSupply[curvesTokenSubject] = supply - amount;

    _transferFees(curvesTokenSubject, false, price, amount, supply);
}
```

So `supply - amount` can be saved in a variable and can be reused, so the changes would look like:

```diff
function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply <= amount) revert LastTokenCannotBeSold();
    if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

+    uint256 finalSupply = supply - amount;
-    uint256 price = getPrice(supply - amount, amount);
+    uint256 price = getPrice(finalSupply, amount);

    curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
-    curvesTokenSupply[curvesTokenSubject] = supply - amount;
+    curvesTokenSupply[curvesTokenSubject] = finalSupply;

    _transferFees(curvesTokenSubject, false, price, amount, supply);
}
```

## 9. `curvesTokenBalance[curvesTokenSubject][msg.sender]` can be cached to reuse it to skip performing `SLOAD` again in `sellCurvesToken` function.

The `sellCurvesToken` function uses `curvesTokenBalance[curvesTokenSubject][msg.sender]` twice ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L282 ):

```solidity
function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply <= amount) revert LastTokenCannotBeSold();
@->    if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

    uint256 price = getPrice(supply - amount, amount);

@->    curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
    curvesTokenSupply[curvesTokenSubject] = supply - amount;

    _transferFees(curvesTokenSubject, false, price, amount, supply);
}
```

So `curvesTokenBalance[curvesTokenSubject][msg.sender]` can be saved in a variable and can be reused, so the changes would look like:

```diff
function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply <= amount) revert LastTokenCannotBeSold();
-    if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();
+    uint256 userBalance = curvesTokenBalance[curvesTokenSubject][msg.sender];
+    if (userBalance < amount) revert InsufficientBalance();

    uint256 price = getPrice(supply - amount, amount);

-    curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
+    curvesTokenBalance[curvesTokenSubject][msg.sender] = userBalance - amount;
    curvesTokenSupply[curvesTokenSubject] = supply - amount;

    _transferFees(curvesTokenSubject, false, price, amount, supply);
}
```

## 10. Revert conditions should be checked before performing other operations to save gas incase of reverts.

The `deposit` function calculates `uint256 tokenAmount = amount / 1 ether` and then check other revert conditions ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L490 ):

```solidity
function deposit(address curvesTokenSubject, uint256 amount) public {
    if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

    address externalToken = externalCurvesTokens[curvesTokenSubject].token;
@->    uint256 tokenAmount = amount / 1 ether;

@->    if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
@->    if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
    if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

    CurvesERC20(externalToken).burn(msg.sender, amount);
    _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
}
```

Revert conditions can be checked before calculating `tokenAmount` to save gas in case of reverts, so the changes would look like:

```diff
function deposit(address curvesTokenSubject, uint256 amount) public {
    if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

    address externalToken = externalCurvesTokens[curvesTokenSubject].token;
-    uint256 tokenAmount = amount / 1 ether;

    if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
    if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
+    uint256 tokenAmount = amount / 1 ether;
    if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

    CurvesERC20(externalToken).burn(msg.sender, amount);
    _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
}
```