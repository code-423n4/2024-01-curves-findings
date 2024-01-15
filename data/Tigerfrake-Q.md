# [01] Unbounded loop

### Description:

New items are `pushed` into the following arrays but there is no option to `pop` them out. Currently, the array can grow indefinitely. 
E.g. there’s no maximum limit and there’s no functionality to remove array values.
If the array grows too large, calling relevant functions might run out of gas and revert. Calling these functions could result in a `DOS` condition.

### Instances:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L335

# [02] Missing or Incomplete NatSpec

### Instances:
All Contracts

### Description
Some functions are missing `@notice/@dev` NatSpec comments for the function, `@param` for all/some of their parameters and `@return` for return values. 
Given that NatSpec is an important part of code documentation, this affects code comprehension, auditability and usability.

### Recommendation:
Consider adding in full `NatSpec` comments for all functions to have complete code documentation for future use.

# [03] Missing Error Handling.

### Description:
The contract uses `custom errors` defined in the `CurvesErrors interface`, but it doesn't handle all possible errors. For instance, it doesn't handle potential errors during token transfers in the `_transfer()` function.

### Instances:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L313-L325

### Recommendation:
Ensure that error handling is consistent throughout your system.

# [04] Improper Input Validation. 

### Description:
The `updateFeeCredit()` function assumes that the `token` parameter refers to a `valid token`. If an `invalid token` is passed, the function will fail. Similarly, in the `claimFees()` function, it assumes that the `token` parameter refers to a `token that the sender owns`. If the sender does not own the `token`, the function will fail. 

### Instances:
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FFeeSplitter.sol#L63-L71
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FFeeSplitter.sol#L80-L87

### Recommendation:
It's generally a good idea to validate inputs before performing operations on them.

# [05] Possible Unchecked Return Values.

### Description:
Several function calls within the contract do not check their return values. For instance, the `_transfer(), buyCurvesTokenWithName(),_addOwnedCurvesTokenSubject(), _deployERC20(),& buyCurvesTokenForPresale()` functions do not check whether the operations were successful or not. 

### Instances:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L313-L392

### Recommendation:
> It would be better to add checks after these function calls to handle possible failures.

# [06] Floating pragma

### Description:
The disadvantage of using a floating pragma is that it can lead to inconsistencies and potential bugs. When a contract is compiled with a different compiler version, it might produce different bytecode due to changes in the compiler, even if the source code looks the same. This can lead to unexpected behavior if the contract is deployed on a network with a different compiler version than the one it was tested with.

Moreover, if the contract uses features that were introduced in a newer compiler version, and it gets compiled with an older compiler version, it could fail or behave differently than expected.

### Instance:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FFeeSplitter.sol#L2

### Recommendation:
> It's generally recommended to lock the compiler version to avoid these issues.

# [07] For same condition checks, use modifiers

### Description:
The main advantage of using modifiers for the same condition checks in different functions is code reusability and readability. 
Instead of repeating the same condition check in every function that requires it, you can define a modifier once and then apply it to any function that needs it. This makes your code cleaner and easier to maintain.

### Instances:
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L297
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L303
