# Gas Optimization Report

# summary:

- ### [[G-01] Multiple accesses of the same mapping/array key/index should be cached](#g-01-multiple-accesses-of-the-same-mappingarray-keyindex-should-be-cached)
- ### [[G-02]When variables are declared with the storage keyword ,caching any fields that need to be re-read in stack variables Saves gas](#g-02when-variables-are-declared-with-the-storage-keyword-caching-any-fields-that-need-to-be-re-read-in-stack-variables-saves-gas)
- ### [[G-03] Gas Optimization Issue - Variable Declaration Inside Loop](#g-03-gas-optimization-issue---variable-declaration-inside-loop)
- ### [[G-04] Gas Optimization Issue - Direct Access of Mappings vs. Using Accessor Functions](#g-04-gas-optimization-issue---direct-access-of-mappings-vs-using-accessor-functions)
- ### [[G-05] Gas Optimization through Assembly for Address Storage Values](#g-05-gas-optimization-through-assembly-for-address-storage-values)
- ### [[G-06] Most important: avoid zero to one storage writes where possible](#g-06-most-important-avoid-zero-to-one-storage-writes-where-possible)
- ### [[G-07] Gas Optimization Issue - Use Constants Instead of type(uintx).max](#g-07-gas-optimization-issue---use-constants-instead-of-typeuintxmax)
- ### [[G-08] Gas Optimization Issue - Redundant Stack Variable Assignment](#g-08-gas-optimization-issue---redundant-stack-variable-assignment)
- ### [[G-09] Gas Optimization Issue - Gas-Efficient Alternatives for Common Math Operations](#g-09-gas-optimization-issue---gas-efficient-alternatives-for-common-math-operations)
- ### [[G-10] Gas Optimization by Caching Calldata](#g-10-gas-optimization-by-caching-calldata)
- ### [[G-11] Gas Optimization Issue in External/Internal Function Calls](#g-11-gas-optimization-issue-in-externalinternal-function-calls)
- ### [[G-12] Gas Optimization Issue - Pre-increment and Pre-decrement vs. +1, -1](#g-12-gas-optimization-issue---pre-increment-and-pre-decrement-vs-1--1)
- ### [[G‑13] Unlimited gas consumption risk due to external call recipients]()
- ### [[G‑14] Storage Update Optimization]()
- ### [[G-15] Using XOR (^) and AND (&) bitwise equivalents](#g-15-using-xor--and-and--bitwise-equivalents)
- ### [[G-16] State variables can be packed into fewer storage slots](#g-16-state-variables-can-be-packed-into-fewer-storage-slots)
- ### [[G-17] Use calldata instead of memory for function arguments that do not get mutated](#g-17-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated)
- ### [[G-18] Shorten the array rather than copying to a new one](#g-18-shorten-the-array-rather-than-copying-to-a-new-one)


# Details:

## [G-01] Multiple accesses of the same mapping/array key/index should be cached
#### `note` : All instances are not found by bot race
Repeatedly accessing the same mapping or array key/index in smart contracts without caching can result in substantial gas consumption. In Solidity, each read operation from storage incurs a gas cost. By implementing a caching mechanism for recurring accesses, significant gas savings can be achieved. This is particularly crucial in loops or functions with multiple reads of the same data.

**Example Code:**
```solidity
// Before Optimization
function exampleFunction() public view returns (uint256) {
    uint256 value1 = mappingData[key];
    uint256 value2 = mappingData[key];
    
    // Further logic...
    
    return value1 + value2;
}

// After Optimization
function optimizedExampleFunction() public view returns (uint256) {
    uint256 cachedValue = mappingData[key];
    
    // Further logic...
    
    return cachedValue + cachedValue;
}
```

**Reason for the Issue:**
In Solidity, each storage read operation involves a gas cost. Failure to cache repeated accesses to the same mapping or array key/index leads to unnecessary gas expenditures. Implementing caching in such scenarios significantly improves efficiency, reduces gas consumption, and enhances overall smart contract performance on the blockchain.



```solidity
227   bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);

241   ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
```
contracts/Curves.sol

- The code has been optimized by caching the value of `referralFeeDestination[curvesTokenSubject]` in the `referralDestination` variable, reducing redundant storage reads. The boolean variable `referralDefined` is now based on the cached value, improving code efficiency and gas consumption in conditions where the mapping value is checked multiple times.

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
+           address referralDestination = referralFeeDestination[curvesTokenSubject]; // Cache for reuse
-           bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
+           bool referralDefined = referralDestination != address(0);

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
-                   ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
+                   ? referralDestination.call{value: referralFee}("")

+             // rest of function body

```



- `presalesMeta[curvesTokenSubject]` is a storage mapping accessed in the `buyCurvesTokenWhitelisted` function.
```solidity
410  presalesMeta[curvesTokenSubject].startTime == 0 ||

411  presalesMeta[curvesTokenSubject].startTime <= block.timestamp

416  if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L410-L416



## [G-02]When variables are declared with the storage keyword ,caching any fields that need to be re-read in stack variables Saves gas

- is `data.cumulativeFeePerToken`
```solidity
File: contracts/FeeSplitter.sol
67       if (balance > 0) {
68          uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance; // funded
69          data.unclaimedFees[account] += owed / PRECISION;
70          data.userFeeOffset[account] = data.cumulativeFeePerToken;                            // funded
71       }
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L69-L71




- This modification introduces a new variable `currentCumulativeFee`, which stores the value of `data.cumulativeFeePerToken`. This can help avoid redundant reads and may contribute to gas savings
```diff
    function updateFeeCredit(address token, address account) internal {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
        if (balance > 0) {
+           uint256 currentCumulativeFee = data.cumulativeFeePerToken; 
-           uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
+           uint256 owed = (currentCumulativeFee - data.userFeeOffset[account]) * balance;
            data.unclaimedFees[account] += owed / PRECISION;
-           data.userFeeOffset[account] = data.cumulativeFeePerToken;
+           data.userFeeOffset[account] = currentCumulativeFee;
        }
    }
```


## [G-03] Gas Optimization Issue - Variable Declaration Inside Loop

**Explanation of Issue:**
Declaring variables inside a loop in Solidity can lead to inefficient gas usage as it creates a new storage slot for the variable in each iteration. This practice becomes a concern, especially when the loop is executed frequently or iterates over a large number of elements. Gas optimization is pivotal to enhance smart contract efficiency and minimize transaction costs.

**Examples of Code:**
Non-Optimized Example:
```solidity
// Non-Optimized Variable Declaration Inside Loop
function nonOptimizedExample() public {
    for(uint i = 0; i < 10; i++) {
        uint256 value = i * 2; // Variable declared inside the loop
        // Rest of the code
    }
}
```

Optimized Example:
```solidity
// Optimized Variable Declaration Outside Loop
function optimizedExample() public {
    uint256 value; // Variable declared outside the loop
    for(uint i = 0; i < 10; i++) {
        value = i * 2;
        // Rest of the code
    }
}
```

**Reason:**
In the non-optimized example (`nonOptimizedExample`), the variable `value` is declared inside the loop, causing Solidity to create a new instance for each iteration. This leads to unnecessary gas consumption as storage slots are repeatedly allocated. On the other hand, the optimized example (`optimizedExample`) declares the variable outside the loop, utilizing the same storage slot for the entire loop execution. This optimization minimizes gas costs, reduces storage usage, and contributes to overall improved contract performance.

```solidity
File: contracts/Curves.sol
306  uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L306


```solidity
File: contracts/FeeSplitter.sol
56           address token = tokens[i];
57          uint256 claimable = getClaimableFees(token, user);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L56-L57


## [G-04] Gas Optimization Issue - Direct Access of Mappings vs. Using Accessor Functions

**Explanation of Issue:**
When accessing mappings in Solidity, direct access is more gas-efficient than using accessor functions. Direct access avoids the overhead of two `JUMP` instructions along with stack setup, resulting in reduced gas consumption. This optimization is especially impactful in scenarios where frequent mapping access is required.

**Examples of Code:**
Non-Optimized Example:
```solidity
// Non-Optimized Accessor Function for a Mapping
mapping(address => uint256) public balances;

function getData(address user) internal view returns (uint256) {
    return balances[user];
}

function nonOptimizedGetBalance(address user) public view returns (uint256) {
    return getData(user); // Accessing mapping via internal function
}
```

Optimized Example:
```solidity
// Optimized Direct Access of Mapping
mapping(address => uint256) public balances;

function optimizedGetBalance(address user) public view returns (uint256) {
    return balances[user]; // Directly accessing mapping
}
```

**Reason:**
In the non-optimized example (`nonOptimizedGetBalance`), an accessor function (`getData`) is used internally to retrieve a value from the mapping. This involves additional `JUMP` instructions and stack setup, leading to increased gas consumption. In the optimized example (`optimizedGetBalance`), direct access to the mapping is performed, eliminating the need for unnecessary instructions and stack setup. This direct approach reduces gas costs, making the smart contract more efficient, especially when frequent mapping access is required.


```solidity
File: contracts/FeeSplitter.sol
49   return userTokens[user];
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L49


## [G-05] Gas Optimization through Assembly for Address Storage Values

**Explanation of Issue:**
When dealing with Ethereum smart contracts, optimizing gas consumption is crucial for efficient and cost-effective operations. One area for improvement is the storage of values at specific addresses. Solidity, the primary language for Ethereum smart contracts, may incur unnecessary gas costs due to high-level abstractions.

**Examples of Code:**

*Non-optimized Solidity Code:*
```solidity
// High-level Solidity code for writing to storage
function writeToStorage(uint256 value) public {
    myStorageVariable = value;
}
```

*Optimized Assembly Code:*
```solidity
// Assembly code for direct storage value assignment
function writeToStorage(uint256 value) public {
    assembly {
        sstore(slot, value)
    }
}
```

**Reason:**
The non-optimized Solidity code involves a high-level abstraction for writing to storage, using the variable `myStorageVariable`. In contrast, the optimized Assembly code directly interacts with the Ethereum Virtual Machine (EVM), leveraging the `sstore` operation to efficiently store values at a specific address. By bypassing some of the high-level Solidity operations, the optimized code reduces gas consumption, resulting in a more cost-effective implementation.


```solidity
File: contracts/Curves.sol
163   curvesERC20Factory = factory_;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L163


```solidity
File: contracts/Security.sol
28     owner = owner_;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L28


## [G-06] Most important: avoid zero to one storage writes where possible
Initializing a storage variable is one of the most expensive operations a contract can do.

When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.

```solidity
File: contracts/Curves.sol
47   uint256 private _curvesTokenCounter = 0;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L47

## [G-07] Gas Optimization Issue - Use Constants Instead of type(uintx).max

**Explanation of Issue:**
Utilizing constants instead of `type(uintx).max` is essential for gas optimization in Ethereum smart contracts. Constants, being precomputed at compile-time, offer a more efficient approach compared to the runtime computations required by `type(uintx).max`, resulting in significant gas savings.

**Examples of Code:**

- **Non-Optimized:**
```solidity
function transferFundsNonOptimized(address _to, uint256 _amount) public {
    require(balanceOf(msg.sender) >= _amount, "Insufficient balance");
    require(_amount < type(uint256).max, "Amount exceeds maximum value");
    
    // Transfer logic
    // ...
}
```

- **Optimized:**
```solidity
uint256 constant MAX_UINT256 = type(uint256).max;

function transferFundsOptimized(address _to, uint256 _amount) public {
    require(balanceOf(msg.sender) >= _amount, "Insufficient balance");
    require(_amount < MAX_UINT256, "Amount exceeds maximum value");

    // Transfer logic
    // ...
}
```

**Reason:**
In the non-optimized example, using `type(uint256).max` within the function introduces unnecessary runtime computations, leading to higher gas consumption. The optimized version, using a constant (`MAX_UINT256`), ensures that the maximum value is precomputed at compile-time, resulting in a more gas-efficient smart contract. This optimization becomes crucial for contracts that involve frequent arithmetic operations with the maximum possible value.

```solidity
File: contracts/Curves.sol
389  presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L389


## [G-08] Gas Optimization Issue - Redundant Stack Variable Assignment

**Explanation of Issue:**
When a stack variable is employed as a transient cache for a state variable, and this stack variable is only accessed once, it leads to unnecessary gas consumption. In such cases, it is more gas-efficient to use the state variable directly, avoiding the additional gas cost associated with the redundant stack assignment.

**Examples of Code:**
*Non-optimized Code:*
```solidity
uint256 public totalBalance;

function updateBalance(uint256 newAmount) external {
    uint256 tempBalance = totalBalance; // Redundant stack assignment
    // Operations on tempBalance
    totalBalance = tempBalance + newAmount;
}
```

*Optimized Code:*
```solidity
uint256 public totalBalance;

function updateBalance(uint256 newAmount) external {
    // Operations directly on totalBalance
    totalBalance = totalBalance + newAmount; // Gas-efficient, eliminates redundant stack assignment
}
```

**Reason:**
In the non-optimized example, a stack variable (`tempBalance`) is introduced to temporarily hold the value of the state variable (`totalBalance`). However, since `tempBalance` is only accessed once, it is more gas-efficient to perform the operations directly on the state variable, eliminating the need for an unnecessary stack assignment. This optimization reduces gas consumption, contributing to a more economical contract execution.


```solidity
File: contracts/Curves.sol
370  uint256 supply = curvesTokenSupply[curvesTokenSubject];

385  uint256 supply = curvesTokenSupply[curvesTokenSubject];

395  uint256 supply = curvesTokenSupply[msg.sender];
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L370



## [G-09] Gas Optimization Issue - Gas-Efficient Alternatives for Common Math Operations

**Explanation of Issue:**
Common math operations, such as min and max, may have more gas-efficient alternatives. In scenarios where unoptimized code uses conditional operators, like the ternary operator, the resulting conditional jumps in opcodes can incur higher gas costs. Exploring gas-efficient alternatives for these operations is crucial for optimizing smart contract performance.

**Examples of Code:**
*Non-optimized Code:*
```solidity
function max(uint256 x, uint256 y) public pure returns (uint256 z) {
    z = x > y ? x : y; // Unoptimized conditional operator
}
```

*Optimized Code:*
```solidity
function max(uint256 x, uint256 y) public pure returns (uint256 z) {
    /// @solidity memory-safe-assembly
    assembly {
        z := xor(x, mul(xor(x, y), gt(y, x))) // Gas-efficient alternative
    }
}
```

**Reason:**
In the non-optimized example, the ternary operator is used for the max function, introducing conditional jumps in the opcodes, which can be gas-costly. The optimized code provides a gas-efficient alternative using assembly code. By avoiding conditional operators, the gas consumption is reduced, leading to improved contract efficiency. The Solady Library offers additional gas-efficient math operations, making it valuable for developers seeking optimization opportunities.


```solidity
File: contracts/Curves.sol
230  uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L230

```solidity
File: contracts/Curves.sol
        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L181-L185



## [G-10] Gas Optimization by Caching Calldata
**Explanation of issue:**
calldata is a special data location that holds the function arguments for external and public functions. Calldata is read-only and immutable, and it can be accessed using the calldataload instruction, which is a cheap opcode that costs 3 gas per word. However, the solidity compiler will sometimes output cheaper code if you cache calldataload in a local variable, instead of accessing it directly from calldata. This will not always be the case, so you should test both possibilities and compare the gas costs.

**Examples of code:**
Here are two examples of a function that sums an array of uint256 values passed as calldata, one without caching and one with caching:
*Non-optimized Code:*
Without caching:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            sum += arr[i]; // access calldata directly
            unchecked {
                ++i;
            }
        }
    }
}
```


*optimized Code:*
With caching:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            uint256 val = arr[i]; // cache calldata in local variable
            sum += val; // access local variable
            unchecked {
                ++i;
            }
        }
    }
}
```
**Reason:**
The reason why caching calldata can save gas in some cases is that it reduces the number of calldataload operations, which consume gas. By caching calldata in a local variable, we can avoid repeated calldataload operations and only perform one calldataload operation per loop iteration. This can reduce the gas cost of the function, especially if the calldata is accessed multiple times in the loop. However, this is not always the case, as caching calldata also consumes gas for storing and loading the local variable. Therefore, it is advisable to test both possibilities and compare the gas costs. 


```solidity
File: contracts/FeeSplitter.sol
103  function batchClaiming(address[] calldata tokenList) external {
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L103



## [G-11] Gas Optimization Issue in External/Internal Function Calls

**Explanation of Issue:**
The issue lies in the non-optimized code of ContractB, where unnecessary external calls are made to functions of ContractA. This results in increased gas consumption, as the same external calls are redundantly invoked within the internal functions.

**Examples of Code:**

*Non-optimized Code:*
```solidity
// Contract B
contract ContractB {
    // External reference to ContractA

    function AB(uint256 _valueR, uint256 _valueT,address _token) public {
        contractA(_token).R(_valueR);

        // External call to T function of ContractA
        contractA(_token).T(_valueT);
    }
}
```

*Optimized Code:*
```solidity
// Contract B
contract ContractB {
    // External reference to ContractA

    function AB(uint256 _valueR, uint256 _valueT,address _token) public {
        ContractA  contractA = ContractA(_token);
        contractA.R(_valueR);

        // External call to T function of ContractA
        contractA.T(_valueT);
    }
}
```

**Reason:**
In the non-optimized code, unnecessary external calls to ContractA's functions are made separately in the `AB` function of ContractB. The optimized code combines these external calls, avoiding redundant calls and improving gas efficiency. By caching the reference to ContractA, we eliminate redundant external function invocations, resulting in reduced gas costs during contract execution.

- cache `CurvesERC20(externalToken)` in contract type then use
```solidity
File: contracts/Curves.sol
497   if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();

500   CurvesERC20(externalToken).burn(msg.sender, amount);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L497-L500


## [G-12] Gas Optimization Issue - Pre-increment and Pre-decrement vs. +1, -1

**Explanation of Issue:**
The gas optimization issue involves the inefficient use of gas when incrementing or decrementing variables. Pre-increment (`++var`) and pre-decrement (`--var`) operations are more gas-efficient than using the equivalent `var += 1` or `var -= 1` expressions. Utilizing pre-increment and pre-decrement operations can lead to reduced gas costs in smart contracts.

**Examples of Code:**

*Non-optimized Code:*
```solidity
// Non-optimized code using +1 and -1
function incrementAndDecrement(uint256 _value) public {
    uint256 result1 = _value + 1;
    uint256 result2 = _value - 1;
    // Rest of the code using result1 and result2
}
```

*Optimized Code:*
```solidity
// Optimized code using pre-increment and pre-decrement
function incrementAndDecrement(uint256 _value) public {
    uint256 result1 = ++_value;
    uint256 result2 = --_value;
    // Rest of the code using result1 and result2
}
```

**Reason:**
The gas optimization stems from the fact that pre-increment and pre-decrement operations directly modify the variable's value without the need for an additional arithmetic operation. This results in fewer bytecode instructions and, consequently, reduced gas consumption. In contrast, using `+1` and `-1` involves an extra addition or subtraction operation, leading to higher gas costs. Therefore, adopting pre-increment and pre-decrement in smart contracts can contribute to improved gas efficiency.


```solidity
File: contracts/Curves.sol
345    _curvesTokenCounter += 1;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L345


## [G‑13] Unlimited gas consumption risk due to external call recipients

When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted. To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.



```solidity
File: contracts/Curves.sol
248  feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L248

## [G‑14] Storage Update Optimization

**Explanation of Issue:**
Manipulating storage in Solidity is a gas-intensive operation. To optimize gas usage, it is crucial to avoid unnecessary storage updates, especially when the new value equals the existing value. Unneeded updates can lead to a Gsreset (2900 gas), which can be avoided by checking if the old value is equal to the new value. While this optimization may introduce a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas), it can significantly reduce overall gas consumption.

**Examples of Code:**
- *Non-Optimized Example - Unnecessary Storage Update:*
```solidity
// Inefficient storage update
function updateValue(uint256 newValue) external {
        value = newValue;
}
```

- *Optimized Example - Avoiding Unnecessary Storage Update:*
```solidity
// Optimized storage update
function updateValue(uint256 newValue) external {
    if (value != newValue) {
        value = newValue;
    } 
    // No need to update storage if the values are equal
}
```

**Reason:**
The issue arises from the gas-intensive nature of storage manipulation in Solidity. By avoiding unnecessary storage updates when the new value is equal to the existing value, a Gsreset (2900 gas) can be circumvented. Although this optimization might introduce a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas), the overall gas consumption is reduced, leading to more efficient contract execution.


```solidity
File: contracts/Curves.sol
163     curvesERC20Factory = factory_;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L163


```solidity
File: contracts/Security.sol
28   owner = owner_;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L28


## [G-15] Using XOR (^) and AND (&) bitwise equivalents
Given 4 variables a, b, c and d represented as such:

```
0 0 0 0 0 1 1 0 <- a
0 1 1 0 0 1 1 0 <- b
0 0 0 0 0 0 0 0 <- c
1 1 1 1 1 1 1 1 <- d

```


To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there‚Äôs at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

```solidity
File: contracts/Curves.sol
        if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L409-L412


```solidity
File: contracts/Curves.sol
        if (
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
            keccak256(abi.encodePacked("")) ||
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
            keccak256(abi.encodePacked(""))
        )
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L440-L445



```solidity
File: contracts/Curves.sol
            if (
                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
                keccak256(abi.encodePacked("")) ||
                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
                keccak256(abi.encodePacked(""))
            ) {
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L470-L475




## [G-16] State variables can be packed into fewer storage slots

```solidity
File: contracts/Curves.sol
contract Curves is CurvesErrors, Security {
    address public curvesERC20Factory;
    FeeSplitter public feeRedistributor;
    string public constant DEFAULT_NAME = "Curves";
    string public constant DEFAULT_SYMBOL = "CURVES";
    // Counter for CURVES tokens minted
    uint256 private _curvesTokenCounter = 0;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L41-L47


- In the given code snippet, the `address` variable utilizes one storage slot, while the `uint256` variable, being larger in size, occupies an additional storage slot. By resizing the `uint256` to `uint64` and strategically arranging the variable order, it becomes feasible to merge both variables into a single storage slot. This optimization not only reduces storage consumption but also enhances gas efficiency for the contract.


```diff
contract Curves is CurvesErrors, Security {
    FeeSplitter public feeRedistributor;
    string public constant DEFAULT_NAME = "Curves";
    string public constant DEFAULT_SYMBOL = "CURVES";
    // Counter for CURVES tokens minted
+   address public curvesERC20Factory;
+   uint64 private _curvesTokenCounter = 0;
```

## [G-17] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

```solidity
File: contracts/Curves.sol
428 function setNameAndSymbol(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) external onlyTokenSubject(curvesTokenSubject) {
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L428-L432


## [G-18] Shorten the array rather than copying to a new one
Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array


```solidity
File: contracts/FeeSplitter.sol
53   address[] memory tokens = getUserTokens(user);

54   UserClaimData[] memory result = new UserClaimData[](tokens.length);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L53-L54