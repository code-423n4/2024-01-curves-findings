# Quality Assurance Report

## Summary
In the recent quality assurance review of the codebase, several low, non-critical issues, and recommendations were identified. The findings encompass areas related to code structure, documentation, and potential improvements in error handling. The goal is to enhance code readability, maintainability, and overall robustness. The following is a summary of the identified issues:

| **Issue Number** | **Issue Title**                                   | **Number of Instances**                                                                             |
|------------------|---------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| L-01             | Check if token is not already deployed           | 1                                                                                                   |
| L-02             | Require price > 0                                 | 1                                                                                                   |
| L-03             | Only allow purchases greater than 0              | 2                                                                                                   |
| L-04             | Function may revert from insufficient gas       | 1                                                                                                   |
| L-05             | Check if `address` exists                         | 2                                                                                                   |
| N-01             | Move emit outside of loop                       | 1                                                                                                   |
| N-02             | Have consistency in checks for curves            | 1                                                                                                   |
| N-03             | Emit error message should contain a variable     | 4                                                                                                   |
| N-04             | Codebase needs more comments                     | 1                                                                                                   |
| N-05             | Code restructure is required                     | 2                                                                                                   |

This summary provides an overview of the issues identified during the review, their respective issue numbers, titles, and the number of instances found in the codebase. The recommended actions aim to improve the overall quality and maintainability of the code.
## [L-01] Check if token is not already deployed 
### Instances
* [Curves.sol #338](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L338)
```solidity
    function _deployERC20(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal returns (address) {
        ...
        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
+       // add code to check if token is deployed before or not
        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

        ...
    }
```


## [L-02] Require price > 0
### Instances
* [Curves.sol #267](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L267)
```solidity
+        require amount > 0;
        uint256 price = getPrice(supply, amount);
+        require price > 0;
```




## [L-03] Only allow purchases greater than 0
### Instances
* [Curves.sol #419](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L419)
```solidity
    function buyCurvesTokenWhitelisted(
        address curvesTokenSubject,
        uint256 amount,
        bytes32[] memory proof
    ) public payable {
+       require amount > 0;
        ...
        _buyCurvesToken(curvesTokenSubject, amount);
    }
```
* [Curves.sol #490](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L396)
```solidity
    function deposit(address curvesTokenSubject, uint256 amount) public {
+       require amount > 0;
        ...
        CurvesERC20(externalToken).burn(msg.sender, amount);
        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
    }
```

## [L-04] Function may revert from insufficient gas when passing a large proof array
### Instances
* [Curves.sol #418](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#418)
```solidity
    function buyCurvesTokenWhitelisted(
        address curvesTokenSubject,
        uint256 amount,
        bytes32[] memory proof
    ) public payable {
        ...
        verifyMerkle(curvesTokenSubject, msg.sender, proof);
        _buyCurvesToken(curvesTokenSubject, amount);
    }
```
## [L-05] Check if `address` exists
### Instances
* [FeeSplitter.sol #52](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L52)
```solidity
    function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
+       require user != address(0);
        address[] memory tokens = getUserTokens(user);
+       require tokens.length > 0;
        ...
    }
```
* [FeeSplitter.sol #74](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L74)
```solidity
    function getClaimableFees(address token, address account) public view returns (uint256) {
+       require token != address(0) && account != address(0);
        TokenData storage data = tokensData[token];
+       require data != address(0);
        ...
    }
```




## [N-01] Move emit outside of the loop
Since fees are transferred in one batch only, it is more convenient to have the emitted message as the transfer method, only emitting the total.
### Instances
* [FeeSplitter.sol #112](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L112)
```solidity
    function batchClaiming(address[] calldata tokenList) external {
        uint256 totalClaimable = 0;
        for (uint256 i = 0; i < tokenList.length; i++) {
            address token = tokenList[i];
            updateFeeCredit(token, msg.sender);
            uint256 claimable = getClaimableFees(token, msg.sender);
            if (claimable > 0) {
                tokensData[token].unclaimedFees[msg.sender] = 0;
                totalClaimable += claimable;
-               emit FeesClaimed(token, msg.sender, claimable);
            }
        }
        if (totalClaimable == 0) revert NoFeesToClaim();
        payable(msg.sender).transfer(totalClaimable);
+       emit FeesClaimed(token, msg.sender, totalClaimable);
    }
    
```



## [N-02] Have consistency in checks for curves
### Instances
* [Curves.sol #396](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L396)
```solidity
-    if (supply > 1) revert CurveAlreadyExists();
+    if (supply != 0) revert CurveAlreadyExists();

```


## [N-03] Emit error message should contain a variable 
### Instances
* [Curves.sol #124](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L124)
```solidity
        ) revert InvalidFeeDefinition();

```
* [Curves.sol #136](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L136)
```solidity
        ) revert InvalidFeeDefinition();

```
* [Curves.sol #149](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L149)
```solidity
        ) revert InvalidFeeDefinition();

```
* [Curves.sol #416](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L416)
```solidity
        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
```
## [N-04] Code base needs more comments

During the code review, I observed a non-critical issue related to the lack of comprehensive comments and explanations within specific functions of the codebase. While the existing code remains functional, the absence of detailed comments reduces the overall readability and maintainability of the code.

**Recommendation:**
I suggest enhancing the code documentation by adding comments to functions. Clear and concise comments are crucial in facilitating code understanding and collaboration among developers. The following aspects should be considered when updating the comments:

1. **Function Purpose:** Clearly describe the overall purpose of each function, outlining its role within the application. One major thing to mention that I struggled with, is when the function is supposed to be called.

2. **Input Parameters:** Document the expected input parameters, including their data types and constraints.

3. **Output and Return Values:** Explain the expected output and any return values, detailing their significance and potential use cases.

4. **Algorithmic Steps:** If applicable, provide comments within the function body to explain key algorithmic steps, intricate logic, or any decisions made during the implementation.

5. **Usage Examples:** Include brief examples demonstrating how to call the function and handle its output.

While you should aim for self-explanatory code, a few additional comments can significantly contribute to the accessibility and maintainability of the codebase.


## [N-05] Code restructure is required
Restructure of code is required in all of the contracts, the following mitigation format is advised
1. **Header Section:**
   - SPDX License Identifier
   - Solidity Version Declaration

2. **Import Statements:**
   - Import external libraries or contracts your smart contract depends on.

3. **Contract Declaration:**
   - Declare the main contract.

4. **Structs:**
   - Define any data structures needed for the contract.

5. **Constants and Variables:**
   - Declare any constants and state variables.

6. **States:**
   - Define enumerations representing different states of the contract.

7. **Events:**
   - Declare events for emitting important information.

8. **Errors:**
   - Define custom errors for exceptional cases.

9. **Modifiers:**
   - Implement custom modifiers to enhance security and access control.

10. **Constructor:**
    - Initialize state variables and perform any setup required during contract deployment.

11. **Initialization Function:**
    - For upgradeable contracts, include an initialization function.

12. **External/Public/Internal Functions:**
    - Organize functions based on their visibility and purpose.

13. **View/Pure Functions:**
    - Use these for functions that only read data and do not modify the state.

14. **Payable Function:**
    - If the contract handles Ether, include payable functions.

15. **Fallback Function:**
    - Include a fallback function to handle Ether sent directly to the contract.

16. **Comments and Documentation:**
    - Add comments to explain complex logic or provide documentation for external users.

By following this pattern, you create a clear separation of concerns within your smart contract, making it easier for developers to understand, maintain, and extend the code. Adjust the sections based on your smart contract's specific needs and complexity.

### Instances
* [Curves.sol #](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol)
* [FeeSplitter.sol](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol)
