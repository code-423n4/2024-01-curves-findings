# Audit approach

1.  Read the README.md
    
2.  Try to understand how the system works
    
3.  Look at the  <ins>Curves</ins>   repo to get a better idea of the Revolution protocol.
4.  Look at each code individually and focus on the internal function calls.
    
5.  Write a Report by compiling all the insights I gained throughout the line-by-line code review.



## Curves.sol

### Overview

The `Curves.sol` contract is designed to facilitate the buying and selling of tokens using a bonding curve pricing mechanism. This document provides an analysis of the contract's security, functionality, and best practices.

### Security Analysis

- **Concerns:**
    - The use of `call{value: ...}("")` for Ether transfers may pose security risks; consider safer alternatives.
    - Lack of a reentrancy guard in critical functions (`buyCurvesToken`, `sellCurvesToken`, `withdraw`).
    - Dependency on `onlyTokenSubject` modifier for access control; review security implications.
    - Potential gas issues in `transferAllCurvesTokens` function due to array iteration.

### Functionality Analysis

- **Key Functions:**
    - `getPrice`: Calculates token price based on a bonding curve formula.
    - `buyCurvesToken*`: Functions for buying tokens with various conditions.
    - `mint`: Exclusive function for curvesTokenSubject to mint tokens.

### Best Practices

- **Positive Practices:**
    - Use of custom errors for gas efficiency.
    - Events (`Trade`, `Transfer`, etc.) for transparency.
    - `FeesEconomics` struct for clean parameter grouping.

### Recommendations

- Implement reentrancy guards in functions involving fund transfers.
- Review and potentially enhance access control mechanisms.
- Evaluate alternatives to low-level calls for Ether transfers.
- Optimize gas efficiency in loops to prevent potential denial-of-service attacks.
- Thoroughly test the bonding curve formula and Merkle proof verification.

## CurvesERC20.sol

### Overview

The `CurvesERC20.sol` contract is an ERC-20 token contract inheriting from OpenZeppelin's ERC20 and Ownable contracts.

### Security Analysis

- **Concerns:**
    - Centralized control over token supply; assess risks associated with owner's private key.
    - Lack of input validation for `mint` and `burn`; consider additional checks.
    - Absence of advanced features (pausing, blacklisting) for a more secure token economy.

### Functionality Analysis

- **Key Functions:**
    - `mint`: Allows the owner to create new tokens.
    - `burn`: Allows the owner to destroy tokens.

### Recommendations

- Consider adding advanced features for a more secure token economy.
- Implement additional input validation for minting and burning.
- Assess risks associated with centralized control over token supply.

## CurvesERC20Factory.sol

### Overview

The `CurvesERC20Factory.sol` contract is designed to deploy instances of the `CurvesERC20` token contract.

### Security Analysis

- **Concerns:**
    - Lack of access control in the `deploy` function; consider implementing restrictions.
    - Absence of input validation for `name`, `symbol`, and `owner` parameters.
    - Gas limitations due to potential contract creation cost; assess for scalability.

### Recommendations

- Implement access control in the `deploy` function.
- Validate inputs for `name`, `symbol`, and `owner` parameters.
- Consider gas limitations when deploying contracts at scale.

## FeeSplitter.sol

### Overview

The `FeeSplitter.sol` contract manages and distributes fees to token holders based on their token balances.

### Security Analysis

- **Concerns:**
    - Reentrancy risks in `claimFees` and `batchClaiming` functions; consider additional safeguards.
    - Precision loss potential in fee calculations; manage division operations carefully.
    - Gas limitations in `batchClaiming` function for long token lists.

### Functionality Analysis

- **Key Functions:**
    - Fee distribution, claim, and updating mechanisms.

### Recommendations

- Implement safeguards against reentrancy in functions involving fund transfers.
- Manage precision loss in fee calculations.
- Assess gas limitations in functions dealing with long token lists.

## Security.sol

### Overview

The `Security.sol` contract manages ownership and permissions for managers within the system.

### Security Analysis

- **Concerns:**
    - Ineffective modifiers (`onlyOwner` and `onlyManager`) due to missing require statements.
    - Lack of input validation for address parameters.
    - Absence of events for critical state changes.

### Recommendations

- Correct modifiers by adding require statements for proper access control.
- Implement input validation for address parameters.
- Emit events for critical state changes to enhance transparency.

&nbsp;

# How could they have done it better?

### Curves.sol

1.  **Ether Transfer Safety:**

- Instead of using `call{value: ...}("")`, consider using safer alternatives like `transfer` or `send` for fixed gas stipends.

2.  **Reentrancy Protection:**

- Implement a reentrancy guard in functions like `buyCurvesToken`, `sellCurvesToken`, and `withdraw` to prevent reentrancy attacks.

3.  **Access Control Enhancements:**

- Review and potentially enhance access control mechanisms, considering alternatives to relying solely on the `onlyTokenSubject` modifier.

4.  **Gas Efficiency in Loops:**

- Optimize gas efficiency in functions like `transferAllCurvesTokens` that iterate over an array to prevent potential denial-of-service attacks.

5.  **Thorough Testing:**

- Ensure thorough testing, especially for the bonding curve formula in the `getPrice` function and the Merkle proof verification.

### CurvesERC20.sol

1.  **Advanced Features:**

- Consider adding advanced features to the token contract, such as pausing transfers, blacklisting addresses, or additional access controls for a more secure token economy.

2.  **Input Validation:**

- Implement additional input validation for the `mint` and `burn` functions to prevent potential issues related to incorrect or malicious inputs.

3.  **Decentralization of Control:**

- Assess the risks associated with centralized control over the token supply and explore ways to distribute control for improved security.

### CurvesERC20Factory.sol

1.  **Access Control in deploy Function:**

- Implement access control in the `deploy` function to prevent unauthorized users from deploying new instances of the `CurvesERC20` contract.

2.  **Input Validation:**

- Validate inputs for `name`, `symbol`, and `owner` parameters to prevent potential issues related to incorrect or malicious inputs.

3.  **Gas Limitations:**

- Consider potential gas limitations when deploying contracts at scale, and optimize the contract creation process if it becomes a bottleneck.

### FeeSplitter.sol

1.  **Reentrancy Safeguards:**

- Implement additional safeguards against reentrancy in functions like `claimFees` and `batchClaiming` to ensure secure fund transfers.

2.  **Precision Handling:**

- Manage precision loss in fee calculations by carefully considering division operations and potential rounding issues.

3.  **Gas Limitations in batchClaiming:**

- Assess and address potential gas limitations in the `batchClaiming` function, especially when dealing with long token lists.

### Security.sol

1.  **Effective Modifiers:**

- Correct the modifiers by adding `require` statements for proper access control, ensuring that only authorized users can call restricted functions.

2.  **Input Validation:**

- Implement input validation for address parameters in functions like `setManager` and `transferOwnership` to prevent potential issues related to incorrect inputs.

3.  **Event Emission:**

- Emit events for critical state changes (addition/removal of managers, ownership transfer) to enhance transparency and provide off-chain tracking.

These recommendations aim to address potential security vulnerabilities, enhance functionality, and promote best practices in the design and implementation of the smart contracts. Each improvement is tailored to the specific context and requirements of the contracts.

## Time Spent

21 Hours

### Time spent:
21 hours