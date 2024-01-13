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