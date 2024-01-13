### Summary
[I-01] Use better name for state variable.
[I-02] No need for OZ import.
[I-03] Use modifier to fulfill function's purpose.
[I-04] Use better naming for modifier.

#### [I-01] Use better name for state variable.
In `Curves.sol` there is a state variable called `feeRedistributor` that is a reference to `FeeSplitter.sol`. Consider changing the name of the variable to `feeSplitter` as it would suit its puprose better.

```diff
-    FeeSplitter public feeRedistributor;
+    FeeSplitter public feeSplitter;
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L43

#### [I-02] No need for OZ import.
In `CurvesERC20.sol` the `Ownable.sol` contract from OpenZeppelin is imported in order to use the `onlyOwner` modifier on `mint()` and `burn`. I think this is not necessary as `Security.sol` has the same modifier. Consider using `Security.sol` instead of `Ownable.sol` or delete `Security.sol` and use OpenZeppelin's `Ownable.sol` and `Roles.sol` in all other contracts.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol

#### [I-03] Use modifier to fulfill function's purpose.
In `CurvesERC20Factory.sol` there is only one function called `deploy`. It is used to deploy `CurvesERC20`. Anyone can call this function, but according to the protocol's workflow, only the `Curves.sol` should call it. Consider adding a modifier so that only the main contract can call this function and prevent users from deploying tokens.

Steps to fix the issue:

1. Add modifier to `Security.sol` called `onlyCurves`.
2. Add address state variable to `CurvesERC20Factory.sol` called `curves` that points to the main contract.
3. Initialize the variable in the `constructor`.
4. Add the `onlyCurves` modifier to `deploy()`.
5. Add another function to change the `curves` variable, protected by the `onlyOwner` modifier.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/CurvesERC20Factory.sol#L7-L10

#### [I-04] Use better naming for modifier.
In `FeeSplitter.sol` there is a function called `addFees`. It has the `onlyManager` modifier and can be called only by `Curves.sol`. I think that the modifier name that would suit this function the most should be `onlyCurves`. After the changes in the above issue, just change the `onlyManager` to `onlyCurves`.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L89