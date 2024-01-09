# [G-1] Caching the curvesTokenBalance[curvesTokenSubject][from] value in a local variable can optimize on Gas.

### Description:
Reading from storage costs gas. When you access a variable for the first time, it costs 2100 gas, and subsequent accesses will cost less. By caching the value in a local variable, you can reduce the number of times you need to access storage, which can save on gas costs.

### Instance:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L313-L325

### Mitigation:
```Solidity
function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
   uint256 fromBalance = curvesTokenBalance[curvesTokenSubject][from];
   if (amount > fromBalance) revert InsufficientBalance();

   if (from != to) {
       _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
   }

   fromBalance -= amount;
   curvesTokenBalance[curvesTokenSubject][from] = fromBalance;
   curvesTokenBalance[curvesTokenSubject][to] += amount;

   emit Transfer(curvesTokenSubject, from, to, amount);
}
```
> In this modified version, fromBalance is a local variable that stores the sender's balance. The balance is updated once, rather than being accessed twice. This can lead to a small reduction in gas costs, especially if the balance is a large number. 

# [G-2] Use constants instead of type(uintX).max

### Description:
Using `constants` instead of `type(uintX).max` saves gas in Solidity. This is because the `type(uintX).max` function has to dynamically calculate the maximum value of a `uint256`, which can be expensive in terms of gas. `Constants`, on the other hand, are stored in the `bytecode` of your contract, so they do not have to be recalculated every time you need them. 
Saves 13 GAS. 

### Instances:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L389

# [G-3] Splitting require()/if() statements that use && saves gas.

### Description:
This [issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 13 gas.

### Instances:
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L213
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L246

[G-5] We can save an entire SLOAD (2100 Gas) by short circuiting the operations

### Description:
The if statement has two checks:
```Solidity
        if (
            protocolFeePercent_ +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            feesEconomics.maxFeePercent ||
            protocolFeeDestination_ == address(0)
        ) revert InvalidFeeDefinition();
```
 where the first check involves making a `state read and carrying out additions then comparison` while the second check only `compares a function parameter to address(0)`.
According to the rules of short circuit, if the first check is `true`, we do not have to do the second check thus in this case, we should make sure the first check is the cheapest to do.

### Instances:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L129-L136

### Recommendation:
By reordering as shown below, we can avoid making the state read. 
```Solidity
        if (protocolFeeDestination_ == address(0) ||
            protocolFeePercent_ +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            feesEconomics.maxFeePercent ) revert InvalidFeeDefinition();
```

