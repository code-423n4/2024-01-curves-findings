# [G-1] Caching the curvesTokenBalance[curvesTokenSubject][from] value in a local variable can optimize on Gas.

### Description:
Reading from storage costs gas. When you access a variable for the first time, it costs 2100 gas, and subsequent accesses will cost less. By caching the value in a local variable, you can reduce the number of times you need to access storage, which can save on gas costs.

### Instance:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L313-L325

### Mitigation
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
> In this modified version, fromBalance is a local variable that stores the sender's balance. The balance is updated once, rather than being accessed twice. This can lead to a small reduction in gas costs, especially if the balance is a large number
