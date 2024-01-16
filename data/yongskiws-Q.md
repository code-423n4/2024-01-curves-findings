# Reentrancy Attack:
The _transferFees function calls external contracts (firstDestination, curvesTokenSubject, and referralFeeDestination[curvesTokenSubject]) with .call{value: ...}(""). This can potentially allow reentrancy attacks. To mitigate this, consider using the transfer function or the Checks-Effects-Interactions pattern.

# Integer Overflow and Underflow: 
The code does not have checks for integer overflow and underflow. For example, in the _buyCurvesToken function, the price + totalFee could cause an integer overflow. To prevent this, use SafeMath library or Solidity 0.8.0 and above, which has built-in overflow and underflow protection.

# Visibility of Functions: 
Some functions are internal and some are public. Make sure the visibility is set appropriately based on the function's purpose. For example, _addOwnedCurvesTokenSubject and _deployERC20 are internal but do not seem to be called from any public or external functions, so they could be made private.

# Lack of Event Logs for Critical Functions: 
Some critical functions like _transfer do not emit event logs. It is a good practice to emit event logs for critical functions so that external parties can listen to these events and react accordingly.

# Use of send and transfer in Constructors: 
In the FeeSplitter contract, send and transfer are used in the constructor. This is a potential security issue because if the constructor call fails, the contract is still deployed but in an inconsistent state. Consider using new operator in the constructor instead.

# Unchecked CALL Return Value:
In the _transferFees function, the call function is used without checking its return value. This can potentially mask errors. To prevent this, always check the return value of call function.

# Missing require Statements: 
There are no require statements in some functions like _transferFees to check for conditions like price > 0. Adding such checks can help prevent potential issues.

# Use of . Instead of [] for Struct Access: 
In the FeeSplitter contract, tokensData[token].cumulativeFeePerToken is used instead of tokensData[token][cumulativeFeePerToken]. This can potentially cause issues if token is a dynamic type.

# Lack of Indexed Parameters in Events: 
In the FeeSplitter contract, some event parameters are not indexed. Indexing important parameters can help external parties filter events more efficiently.

# Use of constant Instead of immutable: 
In the FeeSplitter contract, PRECISION is defined as constant. However, constant should not be used for values that are computed at compile-time. Use immutable instead.