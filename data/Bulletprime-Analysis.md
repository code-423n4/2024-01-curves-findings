1.1 Executive Summary
The Curves protocol is a unique addition to the decentralized world, particularly in the Socialfi sector of blockchain applications. It allows every user on a particular platform, to become a social token. These tokens can be purchased, sold for a profit, or kept as their worth increases with the reputation of the users they represent. The protocol also introduces the concept of `Shares`, which provide access to special chat rooms for influencers, premium content, and more. One of the key features of the Curves protocol is its `Token Export` to ERC20 capability. This feature allows users to transfer their tokens from the Curves protocol to the `ERC20` format, significantly expanding usability across various platforms. Within Curves, tokens lack decimal places, but when converted to ERC20, they adopt a standard 18-decimal format. Importantly, users can seamlessly reintegrate their ERC20 tokens into the Curves ecosystem, albeit only as whole, integer units. The protocol also implements a Referral Fee system, where platforms built upon its framework can earn a percentage of all user transaction fees. This incentive mechanism benefits both the base protocol and its derivative platforms. To address issues with frontrunners during token launches, Curves incorporates a presale phase. This allows creators to manage and stabilize their tokens prior to public trading, ensuring a more controlled and equitable distribution.

 Codebase Approach 
 Consistently following my pattern that has proven to detect low hanging fruits in terms of exposing vulnerabilities.
2.1 Audit Documentation and Scope

Initial step involved, was thoroghly examining `audit documentation and scope` to grasp the audit’s functionalities and boundaries, so as to prioritise my efforts. It is worth highlighting the good quality of the `README` for this audit, as it provided valuable insights and actionable guidance that greatly facilitated the onboarding process.

2.2 Code Review

A good amount of hour was put in gaining knowledge on the pattern of the curve.sol file, as the contrats code is basically an improved fork, this helped me understand fully the problem the protocol is trying to solve, the gap its trying to bridge, their goals and how the methods implemented not just work, but to what degree better than the conventional.

2.3 Setup and Testing

Setting up test environment to execute forge test, though asked more questions and found ways to still further  test as it greatly enhanced the efficiency of the auditing process. With a fully functional test tap at my disposal, which not only accelerate the testing of intricate concepts and potential vulnerabilities but i also gain insights into the developer’s expectations regarding all implementations. 

2.4 Code review

The code review commenced with understanding the pattern” used to manage governance and functionality accross the system. Thoroughly understanding this pattern made understanding the protocol contracts and its relations much smoother. Throughout this stage, I documented observations and raised questions concerning potential exploits without going too deep while uncovering the codes intent.

2.5 Threat Modelling

At this point I began formulating specific assumptions that, if compromised, could pose security risks to the system. This process aids me in determining the most effective exploitation strategies. Thoroughly examining and marking any doubtful or vulnerable areas within the protocol, diving deep into these areas, performing in-depth examinations, and subjecting them to rigorous testing to report these issues if they dont checkout.

2.6 Known Finding 
Read old audits and already known findings. Went through the bot races findings checking what was found so as to not report these issues again, while bearing in mind invariants and critial parts of functions that can lead to users lossing funds.

2.7 Report Issues
I started with auditing the code base indepthly this way I started understanding line by line code and took the necessary notes to ask questions, marking out vulnerabilities and how they can be exploited, This assessment helps in evaluating whether these exploits can be strategically combined to create a more significant impact on the system’s security. In some cases, seemingly minor and moderate issues can compound to form a critical vulnerability when leveraged wisely. This has to be balanced with any risks that users may face.

3.1 Architecture & Recommendations
The Curves protocol's architecture is quite innovative and unique, particularly in its approach to social networking and the concept of social tokens. The ability for every user on a platform to become a social token and for these tokens to be traded and invested in is a novel concept. The introduction of 'Shares' that provide access to special chat rooms for influencers and premium content adds another layer of complexity and value to the system. The protocol's ability to export tokens to the ERC20 format and its implementation of a referral fee system also contribute to its versatility and potential for growth, here are some observations.

- Small Size and Complex Functionality: The contract is compact yet handles substantial assets, which demonstrates effective design and optimization. This is commendable, but it's important to ensure that the contract remains maintainable and secure as it grows and evolves.
- Public Visibility of Variables: The contract has public visibility for several variables, which is beneficial for transparency. However, this could potentially expose sensitive data, so it's crucial to carefully consider what data should be made public.
- Role-Based Access Control: The contract has clear roles defined for the Owner and Manager, which is a good practice for access control. However, it would be beneficial to further clarify who can assign these roles and under what conditions.
- Token Export to ERC20: This feature enhances the usability of the Curves protocol by allowing tokens to be transferred to the ERC20 format. However, the contract should ensure that these exports are secure and that the tokens can be seamlessly reintegrated into the Curves ecosystem.
- Referral Fee Implementation and Presale Feature: These features incentivize user participation and provide a controlled distribution of tokens. They are well thought out and beneficial for the protocol.
- Token Holder Fee: This feature encourages long-term holding over short-term trading, which is a good strategy for promoting sustainable investment in the ecosystem.

Improvement Areas:
Access Control: Although the contract uses role-based access control, it's unclear who can assign these roles. It would be beneficial to define clear procedures for assigning these roles to ensure the integrity of the contract.
Reentrancy Protection: As discussed earlier, the contract could be vulnerable to reentrancy attacks. Implementing a reentrancy guard using a library like OpenZeppelin's ReentrancyGuard could mitigate this risk.
Front-Running Protection: The contract could be vulnerable to front-running attacks due to the way it calculates prices based on the current supply. Implementing a commit-reveal scheme or using a decentralized oracle to determine prices could protect against this.
Error Handling: The contract should have robust error handling to ensure that transactions are reverted in case of errors. This can help prevent unexpected behavior and improve the reliability of the contract.
Test Coverage: Comprehensive testing is crucial for identifying and fixing bugs, ensuring the correctness of the contract, and building confidence in its security. The contract should have extensive test coverage, ideally automated using a tool like Truffle or Hardhat.
Documentation: Good documentation is essential for understanding how the contract works and for auditing the contract. The contract should have clear comments explaining the purpose and functionality of each function and variable.

4.1 Code Comments
The codebase for this project appears to be well-structured and follows many of the best practices for Solidity development. However, there are always areas that could be improved or considered:

Modularity and Reusability: The contract seems to be well-modularized, with separate functions for setting fees and transferring tokens. However, it could benefit from further modularization, such as separating the logic for calculating fees into a separate contract or library. This would improve code organization, maintainability, and reduce the chances of introducing bugs .
Code Formatting and Indentation: Consistent code formatting and indentation significantly improve code readability. While the provided code seem to follow a consistent style, it's important to ensure this consistency throughout the entire contract. Tools like solhint or solium can be used to enforce coding conventions and automatically format the Solidity code .
Code Review and Refactoring: Regular code reviews and refactoring can help improve code quality over time. It's important to conduct peer code reviews to identify potential issues, improve readability, and provide feedback. Additionally, regular refactoring can help eliminate code smells, improve performance, and adhere to best practices .
Documentation: Good documentation is essential for understanding how the contract works and for auditing the contract. The contract should have clear comments explaining the purpose and functionality of each function and variable 1.
QA Check Best Practices: Using static analysis tools to pinpoint style inconsistencies and help determine code errors and vulnerabilities that might be missed by the compiler is recommended. High test coverage measures your development and testing efficiency.
Security Audit: Security audits help determine the knowns and unknowns through manual security audits. It's important to thoroughly review your code for vulnerabilities and fix bugs before deployment.
Overall, the codebase appears to be well-written and follows many best practices. However, continuous improvement and attention to detail are key to maintaining and improving the quality of the codebase.

5.1 Centralization Risk 
Centralization risk refers to the vulnerability that can arise from having a single point of failure within a system. In the context of smart contracts, centralization risk often arises from having a single account with elevated privileges, such as the ability to change important parameters or execute sensitive operations.

Looking at the projects code, the `onlyOwner` and `onlyManager` modifiers suggest that there are certain functions that can only be executed by the owner or manager of the contract. This introduces a centralization risk because if the owner or manager's private keys are compromised, an attacker could potentially execute these functions and manipulate the contract.

```
modifier onlyOwner() {
 require(_owner == msg.sender, "Caller is not the owner");
 _;
}

modifier onlyManager() {
 require(_managers[msg.sender], "Caller is not a manager");
 _;
}
```

The `onlyOwner` modifier restricts certain functions to be called only by the owner of the contract, while the `onlyManager` modifier restricts certain functions to be called only by the manager of the contract.

```function setProtocolFeePercent(uint256 _protocolFeePercent) external onlyOwner {
 // ...
}

function setSubjectFeePercent(uint256 _subjectFeePercent) external onlyManager {
 // ...
}
```
In the `setProtocolFeePercent` function, only the owner can set the protocol fee percent. In the `setSubjectFeePercent` function, only the manager can set the subject fee percent.

To mitigate this centralization risk, you could consider implementing a multi-signature solution, where multiple accounts need to approve a transaction before it can be executed. Alternatively, you could use a decentralized governance system where changes to the contract are proposed and voted on by token holders 1

6.1 Mechanism and Systemic Risk Review
Systemic risk in smart contracts usually arises from the interactions between different components of the contract, rather than individual functions. In the provided code, there are a few potential areas of concern that could contribute to systemic risk:

External Calls: The contract makes external calls to addresses controlled by users (like `firstDestination` in the `_transferFees` function). If these addresses are not properly validated or sanitized, they could potentially cause the contract to behave unpredictably or even self-destruct.
Arithmetic Operations: Arithmetic operations in smart contracts can sometimes lead to overflows or underflows, which can cause unexpected behavior. The contract performs multiplication and division operations, which could potentially lead to overflows or underflows if not handled correctly especially as the contract dont contain a safemath library.
State Changes: The contract modifies its state in multiple places (`_transferFees`, `_buyCurvesToken`). If these state changes are not coordinated correctly, it could lead to inconsistencies in the contract's state.

Possible mitigations to these risks could include:

Input Validation: Always validate inputs to external calls and ensure they are not null or self-destructed addresses. This can help prevent unexpected behavior caused by uncontrolled addresses.
SafeMath Library: Use the SafeMath library for arithmetic operations to prevent overflows and underflows. This library provides safe versions of the basic arithmetic operations.
Atomic Transactions: Ensure that state changes in the contract are atomic, meaning that either all state changes in a transaction occur, or none do. This can help prevent inconsistencies in the contract's state.
Upgradeability: Consider implementing upgradeable contracts to allow for bug fixes and improvements to be made after the contract is deployed. This can help mitigate the risk associated with deploying immutable contracts.
Formal Verification: Consider using formal verification techniques to mathematically prove the correctness of the contract. This can help catch subtle bugs and vulnerabilities that might be missed by manual code review.
Test Coverage: Ensure that the contract has comprehensive test coverage to catch any potential bugs or vulnerabilities before the contract is deployed.

Time Spent
19 Hours 



### Time spent:
19 hours