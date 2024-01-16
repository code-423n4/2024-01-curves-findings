# Analysis for Curves protocol

## Architecture recommendations

**Modularity and Separation of Concerns:**
Consider enhancing modularity by breaking down the core logic in Curves.sol into smaller, more manageable components. This promotes readability, maintainability, and makes it easier to test and upgrade specific functionalities.
Ensure clear separation of concerns among different contracts. For example, separate the ERC20 token creation logic from the main protocol contract.

**Documentation:**
Consider creating comprehensive documentation for developers, auditors, and users. Include detailed explanations of the contracts' functionalities, usage guidelines, and security considerations aside from the one's provided by friend.tech to should protocol specific intentions.
Provide inline comments within the code to explain complex logic, especially in critical areas like price calculation.

**Decentralization and Governance:**
Gradually transition from a centralized model with an 'Owner' role to a decentralized governance model. Consider implementing a DAO to handle administrative tasks and decision-making.
Explore the use of decentralized oracles to enhance the protocol's resilience against price manipulation.

**Scalability:**
Since Curves operates on the Form Network, leverage the Layer 2 EVM-compatible infrastructure to enhance scalability and reduce transaction costs.
Periodically assess the scalability of the protocol, especially as user adoption grows.

**Automated Testing:**
Implement a comprehensive suite of automated/Invariant tests to cover critical functionalities, edge cases, and security vulnerabilities. Continuous integration with tools like Echidna or medusa can help ensure ongoing code quality.

**Emergency Actions:**
Implement mechanisms for emergency  in case of unexpected situations. This could include a pause function, upgradeability mechanisms, or a way to halt specific functionalities if issues arise.

## Codebase quality analysis
| Category | Description |
|--- | --- |
|Access Controls | **Moderate**. Although access controls were in place for performing privileged operations they were not functional.|
|Arithmetic | **Moderate**. The contracts uses solidity version ^0.8.0 potentially safe from overflow/underflow |
| Centralization | **Moderate**. Most critical functionalities could be changed by Security Council and DAO.|
| Code Complexity | **Satisfactory**. Most part of the protocol were easy to understand |
|Contract Upgradeability | **Satisfactory**. Contracts are not upgradable.|
| Documentation | **Moderate**. High-level documentation and some in-line comments describing the functionality of the protocol were available.|
| Monitoring | **Moderate**. Events are emitted when performing some actions.|

## Centralization risks
**Owner Role:**
The existence of an 'Owner' role with significant administrative capabilities raises concerns about centralization. The Owner has exclusive authority over key protocol parameters, such as setting the protocol fee and granting the Manager role.
Centralized control may present a single point of failure or manipulation, impacting the overall decentralization of the protocol.

**Manager Role:**
While multiple managers can adjust various fees, the long-term vision of transitioning this responsibility to a decentralized autonomous organization (DAO) suggests a current centralization risk.
The transition to decentralized governance should be a priority to mitigate centralization risks associated with manager roles.

**Token Minting:**
Unauthorized or erroneous creation of tokens, if not adequately guarded against, may lead to inflation and undermine the decentralized nature of the protocol.

## Mechanism Review
**Token Export to ERC20:**
Allows users to transfer Curves tokens to the ERC20 format, enhancing interoperability.

**Referral Fee Implementation:**
Empowers protocols built on Curves to earn a percentage of user transaction fees.

**Presale Feature:**
Manages token launches, addressing issues faced in friend.tech, particularly with frontrunners.

**Token Holder Fee:**
Encourages long-term holding by distributing fees proportionally among token holders.

**Security Measures (Security.sol):**
Standardizes security criteria to access control.

**Owner and Manager Roles:**
Owner has administrative control, and Managers can adjust various fees.

## Systemic Risks
**Presale Mechanism:**
While the presale feature aims to address issues with frontrunners during token launches, there is a risk that the presale itself may become centralized or susceptible to manipulation.

**Token Minting Safeguards:**
Unauthorized or erroneous creation of tokens poses a risk of inflation and could destabilize the protocol's economy.
Robust controls and audits should be in place to prevent vulnerabilities related to token minting.

**Scalability on Form Network:**
While leveraging the Form Network for scalability is a positive aspect, changes in the Form Network or its compatibility with the Curves protocol may introduce scalability risks.

## Approach taken
* Read through the Provided friend.tech specifications and documentations.

* Skim through the repos provided and take note of relevant points.

* Perform a detailed examination of the presale feature and Fees distribution.

* Proceed with quality assurance (QA) and report writing.

### Time spent:
72 hours