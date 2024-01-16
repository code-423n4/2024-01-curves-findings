
## Summary

no | File |
|-|:-|
| [File-1] | Curves.sol | 
| [File-2] | FeeSplitter.sol | 
| [File-3] | Security.sol | 
| [File-4] | CurvesERC20.sol | 
| [File-5] | CurvesERC20Factory.sol | 




## Analysis Issue Report 


### [File-1]

### The bellow issues related to the File ( Link )
 https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol


#### Admin Abuse Risks:

The contract has an ` onlyOwner ` modifier, which means certain functions can only be executed by the owner of the contract.
This includes functions like ` setFeeRedistributor `, ` setMaxFeePercent `, and others.
While ownership control is a common practice, the owner should use these powers responsibly.


#### Systemic Risks:

The contract uses a counter ` (_curvesTokenCounter) ` for generating unique names and symbols for tokens.
It's important to ensure that this counter is managed securely to avoid collisions or predictable values.

#### Technical Risks:

The contract imports external libraries, such as OpenZeppelin's ` Strings.sol ` and ` MerkleProof.sol `, which can be considered a good practice for using well-established and audited code.
The contract uses interfaces, structs, and mappings for managing various data structures, indicating a structured approach to coding.

#### Integration Risks:

The contract relies on external contracts such as ` CurvesERC20.sol `, ` CurvesERC20Factory.sol `, ` FeeSplitter.sol `, and ` Security.sol `.
Integration with these contracts needs to be carefully checked to avoid vulnerabilities.

#### Non-Standard Token Risks:

The contract deploys ERC-20 tokens using an external factory (CurvesERC20Factory).
It dynamically generates token names and symbols.
The uniqueness and security of these generated names and symbols should be assessed.

#### Additional Notes:

The contract implements a decentralized exchange-like functionality where users can buy and sell custom tokens ` (CurvesTokens) `.
It includes a fee system with different components such as ` protocol fees `, ` subject fees `, ` referral fees `, and ` holders fees `.
Presale features are implemented, allowing users to participate in presales with whitelisting and merkle proof verification.
The contract uses a formula ` (getPrice) ` for calculating token prices based on the current supply and deposit amounts.
It has mechanisms for token transfers, including transferring individual tokens and transferring all tokens owned by an address.
The contract supports the deployment of ERC-20 tokens with customizable names and symbols for each ` CurvesToken `.
Users can deposit and withdraw their tokens, allowing interaction with external ERC-20 tokens.
The contract supports the selling of external tokens ` (sellExternalCurvesToken) `, converting them back to ` CurvesTokens `.

#### Suggestions:

Comprehensive testing is crucial to ensure the correctness and security of the contract.
Consider using testing frameworks such as Truffle or Hardhat.
Consider engaging in a professional smart contract audit to identify potential security vulnerabilities and ensure best practices are followed.
Regularly review and update the contract based on any changes to external dependencies or best practices in the Ethereum ecosystem.


### [File-2]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol

#### Admin Abuse Risks:

The contract has a ` onlyManager `  modifier, which restricts certain functions to be callable only by a manager.
Potential admin abuse risks may arise if the manager role is not assigned or managed securely.
Ensure proper access control and management of the manager role.

#### Systemic Risks:

The contract introduces a mechanism for fee distribution among token holders. 
Systemic risks could arise if the fee distribution logic or calculations are not accurate.
It's crucial to thoroughly test and audit the fee distribution mechanism to avoid unintended consequences.

####  Technical Risks:

The contract relies on external contracts, such as` Curves.sol ` and ` Security.sol `.
Ensure that these external contracts are well-audited and follow best practices.
Any changes to these external contracts could impact the behavior of the ` FeeSplitter ` contract.

#### Integration Risks:

The contract integrates with the ` Curves ` contract, and its functionality depends on the accurate calculation of token balances and total supply.
Integration risks could occur if there are discrepancies or vulnerabilities in the interactions with the ` Curves ` contract.

#### Non-Standard Token Risks:

The contract doesn't directly involve the creation or handling of non-standard tokens.
It primarily deals with the fee distribution of existing tokens, and the risks are more related to the accurate execution of fee distribution logic.


#### Additional Notes:

The contract implements a fee distribution mechanism for token holders based on their balances.
It tracks cumulative fees per token and unclaimed fees for each user.
Users can claim their accrued fees by calling the ` claimFees ` function.
There's a batch claiming feature ` (batchClaiming) ` that allows users to claim fees for multiple tokens in one transaction.
The contract uses the ` onlyManager ` modifier for functions like ` addFees ` and ` onBalanceChange `, ensuring that these operations are restricted to a manager role.


#### Suggestions:

Ensure that the ` Curves ` contract and any other external contracts are well-audited and secure.
Thoroughly test the fee distribution mechanism to validate its correctness and efficiency.
Implement additional security checks and testing for potential edge cases, especially in functions like ` batchClaiming ` where multiple token claims occur in a single transaction.
Consider implementing events and logging to provide transparency and auditability for fee distribution activities.

### [File-3] 

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol



#### Admin Abuse Risks:

The contract introduces an ` owner ` and ` managers ` system to control access to certain functions.
However, there are errors in the modifiers ` onlyOwner and onlyManager ` due to the use of comparison `==` instead of the equality check `==`.
This results in the modifiers having no effect. To fix this, the correct syntax for the modifiers should be:
```solodity
modifier onlyOwner() {
    require(msg.sender == owner, "Not the owner");
    _;
}

modifier onlyManager() {
    require(managers[msg.sender], "Not a manager");
    _;
}
```
This ensures that only the owner or a designated manager can execute functions with these modifiers.

#### Systemic Risks:

The contract has a systemic risk due to the mentioned errors in modifiers.
If the access control mechanisms do not work correctly, it may lead to unauthorized access to certain functions, potentially compromising the security of the contract.

#### Technical Risks:

The use of ` msg.sender ` as the ` owner ` introduces a single point of failure.
If the private key associated with the current ` msg.sender ` is compromised, it could lead to unauthorized changes in ownership and manager assignments.
Consider enhancing the owner management mechanism.

#### Integration Risks:

The contract itself doesn't involve direct integration with other contracts.
However, if this contract is part of a larger system, the proper functioning of access control mechanisms is crucial for the overall security of the system.


#### Non-Standard Token Risks:

The contract doesn't handle tokens or interact with any token-related functionality.
Therefore, non-standard token risks are not applicable to this specific contract.

#### Additional Notes:

The contract initializes the and grants the owner the default manager role.
Managers can be added or removed by the owner using the ` setManager ` function.
Ownership can be transferred to another address using the ` transferOwnership ` function.

#### Suggestions:

Correct the modifiers' syntax as mentioned above to ensure proper access control.
Consider using a more robust access control mechanism, such as OpenZeppelin's Roles or AccessControl, for enhanced security.
Implement events and logging for important state changes to facilitate transparency and auditability.
If the contract is part of a larger system, ensure that the access control mechanisms align with the overall system architecture and security requirements.

### [File-4] 

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol


#### Admin Abuse Risks:

The contract inherits from OpenZeppelin's ` Ownable ` contract, which provides access control to the contract owner.
The ` mint ` and ` burn ` functions are restricted to only be callable by the owner (`onlyOwner` modifier).
This mitigates admin abuse risks to a large extent by ensuring that only the owner can mint or burn tokens.


#### Systemic Risks:

The contract appears to have low systemic risks.
The usage of well-established libraries such as OpenZeppelin's ` Ownable ` and ` ERC20 ` reduces the likelihood of systemic issues.


#### Technical Risks:

One potential technical risk is the reliance on external libraries, such as OpenZeppelin's contracts.
It's essential to ensure that the version of the external contracts is compatible and secure.
Regularly updating dependencies is recommended.


#### Integration Risks:

The contract is designed as a standalone ERC-20 token contract, and it doesn't have direct integration points with other contracts.
Integration risks may arise if this contract is part of a larger system, and the interaction with other components is not thoroughly tested.


#### Non-Standard Token Risks:

The contract follows the ERC-20 standard, which is a widely adopted standard for fungible tokens on the Ethereum blockchain.
There don't seem to be non-standard token risks in this context.

#### Additional Notes:

The constructor of the contract takes parameters for the token's name, symbol, and the initial owner's address.
The ` mint ` function allows the owner to create new tokens and assign them to a specified address.
The ` burn ` function allows the owner to destroy existing tokens from a specified address.
The ownership is transferred to the provided owner address during contract deployment.


#### Suggestions:

Ensure that the version of the external dependencies (e.g., OpenZeppelin contracts) is up-to-date to benefit from the latest security patches.
If applicable, thoroughly test the contract's integration with other components in the system to mitigate potential issues.


### Overall 
the contract seems well-structured and follows best practices by leveraging established libraries.
Admin abuse risks are mitigated through the use of access control mechanisms.
Regular code audits and testing are recommended to ensure the contract's security and compatibility with the intended system.

### [File-5] 

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol


#### Admin Abuse Risks:

The ` CurvesERC20Factory ` contract does not have explicit admin functionality, and the ownership of the created ` CurvesERC20 ` tokens is determined by the constructor of the ` CurvesERC20 ` contract.
As long as the ` CurvesERC20 ` contract enforces proper access control within its constructor, the admin abuse risks are contained within the individual token contracts.


#### Systemic Risks:

The systemic risks in this contract are minimal.
It serves as a factory for creating instances of the ` CurvesERC20 ` token contracts.
The risks associated with the systemic behavior are mostly transferred to the instantiated token contracts.

#### Technical Risks:

One potential technical risk is that the ` deploy ` function does not return any information about the deployed token contract, such as its address.
Users may need additional mechanisms to discover or track the addresses of the created token contracts.


#### Integration Risks:

Integration risks are generally low for this contract.
It's a standalone factory contract responsible for creating instances of ` CurvesERC20 ` tokens.
Integration risks would primarily arise if other contracts or systems rely on the functionality provided by this factory.

#### Non-Standard Token Risks:

The factory contract creates instances of the ` CurvesERC20 ` token, which is expected to follow the ERC-20 standard.
Non-standard token risks are associated with the individual token contracts rather than the factory.

#### Additional Notes:

The factory contract is simple and serves a specific purpose: deploying new instances of the ` CurvesERC20 ` token contracts.


#### Suggestions:

Consider adding an event or mechanism to notify users of the address of the deployed token contract.
Ensure that the ` CurvesERC20 ` contract it creates has robust access control mechanisms to mitigate potential admin abuse risks.

### Overall:

The ` CurvesERC20Factory ` appears to be a straightforward contract with a specific purpose of creating instances of ERC-20 tokens.
Its risks are minimal, and attention should be directed towards ensuring the security of the ` CurvesERC20 ` contracts it produces.







### Time spent:
9 hours