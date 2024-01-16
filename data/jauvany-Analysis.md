**Summary**

The Curves protocol, an extension of friend.tech, introduces several innovative features. Friend Tech Smart Contract Breakdown. Key enhancements in the Curves protocol Include: Token Export to ERC20, Referral Fee Implementation, Presale Feature and Token Holder Fee.

**Scope**

Curves.sol: This is the primary file of the Curves protocol. It contains the core logic and functions that define the overall behavior and rules of the system. This file started as a fork of friend.tech FriendtechSharesV1.sol


CurvesERC20.sol: This file defines the ERC20 token that will be created upon exporting a Curve token. It outlines the token's properties and behaviors consistent with the ERC20 standard.


CurvesERC20Factory.sol: Serving as an ERC20 factory, this file abstracts the logic for ERC20 token creation. Its primary purpose is to streamline the token creation process and reduce the overall footprint of the protocol, ensuring efficiency and scalability.


FeeSplitter.sol: This script manages the distribution of fees. It is responsible for the fair and accurate division of transaction fees amongst token holders, in line with the protocol's incentive structure.

Security.sol: This file standardizes the security criteria for the protocol.
It includes protocols and measures designed to safeguard the system
against vulnerabilities and ensure compliance with established security
standards.

# Approach taken in evaluating the codebase

> My analysis of Curves Protocol Included understanding the architecture, mechanism, overall codebase and possible risks associated to the protocol.
- Day 1: I spent time reading the different available articles in order to have a
deep understanding of the protocol.
- Day 2: I analyzed the codebase for better understanding, Performed a
Mechanism review and investigated possible systemic risks, and centralization risks.
- Day 3: I dedicated this day to coming up with possible Architecture
recommendations after identifying possible risks and prepared the final
analysis report

# Mechanism review

- The mechanisms implemented in the Curves ecosystem, including the Token Export to ERC20, Referral Fee 
Implementation, Presale Feature and Token Holder Fee, are
innovative and well-designed. They provide a comprehensive
range of features and capabilities. However, there are some
potential issues and risks associated with these mechanisms. 
These issues should be carefully considered and mitigated to
ensure the security and reliability of the ecosystem.

# Codebase quality analysis

> Analysis of the codebase (What’s unique? What’s using existing patterns?):

- Unique: Codebase carries out specific governance mechanisms that are uniquely designed for Curves’s specific use case e.g Within Curves, tokens lack decimal places, but when converted to ERC20, they adopt a standard 18-decimal format.

- Existing Patterns: The Curves Protocol adheres to common contract management patterns, such as the use of OnlyOwner and OnlyManager

**Strengths*
- Contract  files use fixed compiler versions as recommended.
- 100% test coverage

**Weaknesses*
- Lack of Official documentation 
- Named imports of parent contracts are missing
- If statement control structure do not comply with best practices.
- Missing zero address checks In functions which accept an address as a parameter to prevent bugs.
CriticalCritical functionsfunctions shouldshould bebe a twotwo stepstep procedureprocedure

# Centralization risks

Privileged functions can create points of failure: Ensure such accounts are
protected and consider implementing multi sig to prevent a single point of
failure. See [Here](https://github.com/code-423n4/2024-01-curves/blob/main/bot-report.md) for complete details.

# Architecture recommendations

> The Curves architecture seems solid in general, none the less here are
some areas that could be improved:

- Testing and Simulations: Even though the Ethena project implements 100%
test coverage, I recommend creating a live testnet app. Here is an
[example](https://app.opendollar.com/) from The Open Dollar protocol.
Conduct thorough testing of all contracts and functions and simulations tounderstand how they will behave under various market conditions.

- I recommend creating an official official website early on for the protocol.

- I recommend rewriting rewriting somesome of of the the tests in the codebase for this audit to use the actual contracts instead of mock addresses like in
some cases. This will offer greater confidence during system deployment.

**Gas Optimizations**

- Review data types: Analyze the data types used in your smart contracts and consider if they can be further optimized. For example, changing uint256 to uint128 or uint94 can save gas and storage slots.
- Struct packing: Look for opportunities to pack structs into fewer storage slots. By carefully selecting appropriate data types for struct members, you can reduce the overall storage usage.

- Use constant values: If certain values in your contracts are constant and do not change, declare them as constants rather than storing them as state variables. This can significantly save gas costs.

- Avoid unnecessary storage: Examine your code and eliminate any unnecessary storage of variables or addresses that are not required for
contract functionality.

- Storage vs. memory usage: When working with arrays or structs, consider whether using storage instead of memory can save gas. Using storage allows direct access to the state variables and avoids unnecessary copying of data.
- Replacing the use of memory with calldata for read-only arguments in
external functions.

**Other recommendations**

- Regular code reviews and adherence to best practices.
- Conduct external audits by security experts.
- Consider open sourcing the contract for community review.
- Maintain comprehensive security documentation.

- Establish a responsible disclosure policy for vulnerabilities.
- Implement continuous monitoring for unusual activity.
- Educate users about risks and best practices.

# Systemic risks

Like any smart contract-based system, Curves is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol.

- External Contract Dependencies: Curves relies on external contracts from Openzepplin. If any of these contracts have vulnerabilities, it would affect the protocol.

Test Coverage: the test coverage provided by Brahma is 100%, however, Some of the tests use mock addresses instead of actual contracts. This is not recommended.

# Documents

According to he Main audit page “There's no official friend.tech documentation but there's a lot of great articles.” It is highly recommended to create proper official documentation for the protocol. It would also be helpful for the Documentation to explaine how the ecosystem works from a basic contract level so that it is easier to digest for developers, users and auditors looking to integrate into the Curves project.

I would also recommend adding more quality Medium articles, it’s a great way to provide an indepth look at many of the topics in the project and is used by many blockchain projects.

**Conclusion**

The Curves ecosystem is a well-designed platform, offering a wide range of features
and capabilities. However, there are areas where improvements could be made,
particularly in terms of permission management, gas efficiency and the absence of
official documentation. By addressing these issues, the Curves ecosystem could
become even more secure, efficient, and reliable.


### Time spent:
24 hours