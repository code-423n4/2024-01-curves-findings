## [L-01] - Invalid validation in the `Curves::deposit()` function.

Instance : [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L490-L491)

```javascript

    function deposit(address curvesTokenSubject, uint256 amount) public {
        //@a no check for amount == 0 , will continue to execute bec 0 % 1ether == 0
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
        .
        .
    }

```

### Summary

The validation done in the `Curves::deposit()` function inadequately verifies the `amount` parameter. It neglects proper validation when the `amount == 0` is supplied to the function, as `0 % 1 ether = 0`. However, the function only reverts when `amount % 1 ether != 0`.

### Impact

If 0 is provided as the parameter, the entire function will run without effecting any changes in the contract. This could lead to a loss for the user initiating the function, as they would incur gas costs without any meaningful impact on the contract.

### Recommendations



Add check to ensure that amount is not 0 .

```diff
    function deposit(address curvesTokenSubject, uint256 amount) public {
        //@a no check for amount == 0 , will continue to execute bec 0 % 1ether == 0
+        if (amount % 1 ether != 0 || amount == 0) revert NonIntegerDepositAmount();
-        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
        .
        .
    }
```


## [L-02] - No function/code to return extra msg.value.

Instance : [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263-L280)


```javascript

   function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);
        // @a Check made to ensure that msg.value < price + totalFee but no function/code present to return extra msg.value if sent.
        if (msg.value < price + totalFee) revert InsufficientPayment();

        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
    }

```

### Summary
If the user deposits extra `msg.value` through any of the buy functions then there's no way the user would get back those.

### Impact
Loss suffered by the user would be directly proportional to the extra amount that the user has sent into the contract.

### Recommendation:

Add the code to transfer the extra `msg.value` back to the user in the `_buyCurvesToken` function.

## [L-03] - Invalid validation in the `Curves::getSellPrice()` function.

Instance - [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L193-L195)

```javascript
    function getSellPrice(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
        return getPrice(curvesTokenSupply[curvesTokenSubject] - amount, amount);
    }
```

### Summary

There are no checks in the `Curves::getSellPrice()` function to determine if `curvesTokenSupply[curvesTokenSubject] - amount > 0`.

### Impact
If the value of `amount` is passed such that `curvesTokenSupply[curvesTokenSubject] - amount > 0`,  then the `Curves::getSellPrice()` would revert.

### Recommendation

Add a check to determine  `curvesTokenSupply[curvesTokenSubject] - amount <= 0`.




## [QA-01] - Should avoid use of magic numbers (NOT DETECTED BY BOT)

Magic numbers should be avoided in Solidity code to enhance readability, maintainability, and reduce the likelihood of errors. Magic numbers are hard-coded values with no clear meaning or context, which can create confusion and make the code harder to understand for developers. Using well-defined constants or variables with descriptive names instead of magic numbers not only clarifies the purpose and significance of the value but also simplifies code updates and modifications.

Instance : [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L180-L187)

```javascript

 function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
       .
       .
       return (summation * 1 ether) / 16000;//@ magic number used here
    }

```

## [QA-02] - All the setter should be setted in the constructor.

Instances :
[setMaxFeePercent](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117C14-L126)

[setProtocolFeePercent](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L128-L139)

[setERC20Factory](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L162-L164)


### Summary

Executing setter functions within the constructor is essential to prevent therisk of forgetting to define those setters or transactions happening before all the setter functions are called.

### Impact

Function calls would revert if the setters are not setted.

### Recommendation

Call the setter functions within the constructor.


