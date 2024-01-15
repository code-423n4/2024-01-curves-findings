# Analysis - Curves Contest

![Curves-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2Fp44LEz6BxLQ.0&w=256&q=75)

## Description overview of The Curves Contest

Curves is a decentralized protocol that allows for the trading and interoperability of SocialFi tokens across different decentralized applications. It functions as a decentralized exchange of sorts for SocialFi assets.It builds on the previous friend.tech protocol by incorporating important new features like token exportability, referral incentives, presales, and token holder rewards.
So in summary, Curves functions as an open, interoperable and incentivized infrastructure for trading SocialFi assets in a decentralized manner.

**The key contracts of Curves protocol for this Audit are**:

- **Curves.sol**: contains the main logic for token trading, pricing, fee calculations, presales, ERC20 deployment and other core functions.

- **FeeSplitter.sol**: handles the distribution of transaction fees to token holders.

- **Security.sol**: establishes the access control model.

- **CurvesERC20.sol** defines the associated ERC20 tokens.

- **CurvesERC20Factory.sol** facilitates scalable deployment of ERC20 tokens.

As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

## System Overview

### Scope

- **Curves.sol**: This contract allows users to buy, sell, and trade customized tokens ("Curves tokens") with various features such as `token export to ERC20 format`, `referral fee implementation`, a `presale phase`, and `a token holder fee distribution model`.

  1.  **Fee System**:

      - The contract implements a fee system that includes various types of fees such as protocol fees, subject fees, referral fees, and holder fees.
      - The fees are calculated based on the buy and sell transactions, and a FeeSplitter contract is used to distribute fees among token holders.

  2.  **Presale Functionalities**:

      - The contract supports presales with specified start times, merkle root verification for whitelisting, and maximum buy amounts.
      - Users can participate in presales, and the contract handles the buying of tokens during presale periods.

  3.  **ERC-20 Token Deployment**:

      - The contract includes a function to deploy associated ERC-20 tokens for each Curves token subject.
      - These ERC-20 tokens are deployed through a CurvesERC20Factory contract and have customizable names and symbols.

  4.  **Token Trading**:

      - Users can buy and sell Curves tokens, and the contract calculates the appropriate prices based on the supply and demand dynamics using a mathematical model.
      - The trade functions include mechanisms to transfer fees to various destinations and update balances accordingly.

- **FeeSplitter.sol**: FeeSplitter contract facilitates the distribution of transaction fees among `token holders` in the Curves protocol, allowing users to claim their share of fees, which are accumulated based on their token holdings. The contract includes functions for claiming fees, updating fee credits, and managing fee distribution among token holders, with additional features like batch claiming and checking claimable fees for specific tokens.

  1.  **Curves Integration**:

      - The contract interacts with the Curves contract to retrieve information about token balances and supplies.

  2.  **Fee Calculation and Claiming**:

      - Users can check their claimable fees and subsequently claim them through the claimFees function. The contract ensures proper fee accounting using the updateFeeCredit and getClaimableFees functions.

  3.  **Manager Operations**:

      - Certain functions, such as addFees and onBalanceChange, are restricted to be called only by the manager, likely to ensure secure and authorized management of the fee distribution process.

  4.  **Batch Claiming**:

      - The contract supports batch claiming of fees for multiple tokens at once through the batchClaiming function, optimizing gas usage for users with fees across multiple tokens.

- **Security.sol**: The Security contract establishes a simple access control system with an owner and managers, allowing only the owner to set managers and transfer ownership.

- **CurvesERC20.sol**: The CurvesERC20 contract is an ERC-20 token that extends the OpenZeppelin ERC20 and Ownable contracts, allowing the owner to mint and burn tokens.

- **CurvesERC20Factory.sol**: The CurvesERC20Factory contract facilitates the deployment of new CurvesERC20 tokens with specified name, symbol, and owner, returning the address of the newly created token contract.

### Protocol Roles

1. **Curves**: contract has roles for managing the lifecycle of the `Curves` tokens, including roles such as:

   - `Owner`: The address that deployed the contract, having administrative control, including modifying fees, setting the fee destination, and deploying ERC-20 tokens.
   - `Manager`: Authorized address with the ability to set the maximum fee percentage and adjusting external fees (subject fee, referral fee, holders fee).
   - `Fee Redistributor`: An instance of the `FeeSplitter` contract that redistributes fees among token holders based on their balances.

2. **FeeSplitter**:

   - `Manager`: The Manager role is responsible for initiating actions that affect the fee distribution process.
     The addFees function is accessible only by the Manager and is used to distribute fees among token holders.
   - `User`:

     - Users are individuals holding tokens in the Curves contract.
     - Users can claim their share of fees through the claimFees function.
     - Users can also initiate batch claiming of fees for multiple tokens through the batchClaiming function.

These roles collectively contribute to the proper functioning of the `Curves` protocol, enabling token creation, fee distribution, and other essential functionalities.

## Approach Taken-in Evaluating The Curves Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Curves Protocol.

    I start with the following contracts, which play crucial roles in the Curves protocol:

    **Main Contracts I Looked At**

    I start with the following contracts, which play crucial roles in the Curves protocol:

                Curves.sol
                FeeSplitter.sol
                Security.sol

    I started my analysis by examining the `Curves.sol` contract, as this contract contains the core logic and state variables that define the Curves protocol.

    Specifically, `Curves.sol` is responsible for:

    - Managing the minting, supply and balances of Curves tokens
    - Pricing logic for buying and selling tokens
    - Fee calculation and distribution
    - Integration with other key contracts like `FeeSplitter`
    - Presale functionality
    - ERC20 deployment
    - Access controls

    By carefully reviewing each function and state variable in Curves.sol, I aimed to understand how the protocol works from an on-chain perspective. This provided a solid foundation for then evaluating other aspects like security, gas optimization, and compliance with best practices.

    Then, I turned my attention to the `FeeSplitter.sol` contract.

    The `FeeSplitter` contract is crucial for implementing the fee distribution mechanism of the Curves protocol. It is responsible for:

    - Calculating each user's share of transaction fees based on their token balances
    - Allowing users to claim their accumulated fees
    - Managing the fee credits and balances on a per-token basis
    - Supporting batch claiming of fees across multiple tokens
    - Interacting with the Curves contract for balances and supply data

2.  **Documentation Review**:

    Then went to Review [This Articles](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf) for a more detailed and technical explanation of Curves protocol.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the key contracts that are part of the Curves protocol:

This is simply diagram that illustrates the interaction among the key components of the Curves protocol.

### Architecture Diagram:

![CurvesDiagram](https://gist.github.com/assets/70327041/940fc4b9-cd53-468a-a66a-6702783ba73f)

### Architecture Description:

1. **Curves.sol: Token Customization and Trading**

- **Description:**
  - Responsible for token customization, trading, and overall lifecycle management of Curves tokens.
  - Implements a sophisticated fee system that includes protocol fees, subject fees, referral fees, and holder fees.
  - Supports presales with features like defined start times, merkle root verification, and maximum buy amounts.
  - Manages the deployment of associated ERC-20 tokens for each Curves token through `CurvesERC20Factory`.
  - Implements token trading functionalities, calculating prices based on supply and demand dynamics.

2.  **FeeSplitter.sol: Fee Distribution Among Token Holders**

- **Description:**
  - Facilitates the distribution of transaction fees among Curves token holders.
  - Interacts with `Curves` to retrieve token balances and supplies.
  - Includes mechanisms for fee calculation, claiming, and updating fee credits.
  - Provides batch claiming functionality to optimize gas usage for users with fees across multiple tokens.
  - Restricts certain operations to be accessible only by the manager for secure fee distribution.

3. **Security.sol: Access Control**

- **Description:**
  - Establishes an access control system with an owner and managers.
  - Only the owner has the authority to set managers and transfer ownership.
  - Ensures secure and authorized management of the overall protocol.

4. **CurvesERC20.sol: ERC-20 Token Management**

- **Description:**
  - Represents an ERC-20 token within the Curves protocol.
  - Extends the OpenZeppelin ERC20 and Ownable contracts.
  - Allows the owner to mint and burn tokens.

5. **CurvesERC20Factory.sol: ERC-20 Token Deployment**

- **Description:**
  - Facilitates the deployment of new CurvesERC20 tokens.
  - Accepts specifications such as name, symbol, and owner.
  - Returns the address of the newly created token contract.

### Architecture Feedback

**Formal Verification**: Consider a professional formal verification of the contract to identify and mitigate any security risks.

**Documentation**: Providing thorough natural language documentation of concepts, approaches, functions via external resources like a Knowledge Base is important for developers and users.

**Interfaces**: Include interfaces like ICurvesToken to show abstraction layers.

**Upgradeability**: Adding support for upgrading core contract logic via a proxy/delegate pattern would allow fixing bugs or adding features without hardforks.

The architecture leaves room to flexibly add new contract types (e.g. tokens, exchanges) via factories/registry without changing core code.

## Codebase Quality

Overall, I consider the quality of the Curves protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Curves Protocol contracts demonstrates good maintainability through modular structure, consistent naming. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                       |
| **Code Comments**                        | During the audit of the Curves contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary. |
| **Documentation**                        | There's no official documentation for this protocol but there's a good articles.However, we have noticed that there is room for adding documentation and additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors.                                        |
| **Testing**                              | The audit scope of the contracts to be audited is 100% this is Good.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. but some order of functions does not follow the Solidity Style Guide According to the Solidity Style Guide, functions should be grouped according to their visibility and ordered: constructor, receive, fallback, external, public, internal, private. Within a grouping, place the view and pure functions last.                                                                                                                                                                                                                                                                    |
| **Error**                                | Use custom errors, custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in try-catch blocks, and are easier to re-use and maintain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| **Imports**                              | In Curves protocol the contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Curves protocol. These risks encompass concentration risk in Curves, FeeSplitter risk and more, third-party dependency risk, and centralization risks arising.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. The fee distribution mechanism, particularly in the `FeeSplitter` contract, should be audited to ensure that fees are accurately calculated and distributed. Any vulnerabilities or miscalculations could result in economic losses.

2. Large trades or withdrawals could significantly impact prices due to the mathematical model.

3. Lack of circuit breakers or pause functionality increases risk of attacks draining funds.

4. The use of floating-point arithmetic in `cumulativeFeePerToken` calculations in `FeeSplitter` contract may lead to precision errors. It's generally safer to use integer arithmetic to avoid potential rounding issues that could result in unexpected behavior

5. No mechanism to blacklist compromised tokens increases long term risk.

### Centralization Risks:

1. The Protocol includes an `owner` and `manager` role, which have significant control over critical functions of the Protocol. Depending on how these roles are managed, there may be centralization concerns, especially if the owner or managers can unilaterally modify critical parameters or control fee distributions.

2. The ability for a token subject owner to update the `merkle root` for whitelisting introduces centralization risks. If not handled carefully, it could be manipulated to control access during presales.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Curves protocol.**

## Conclusion

Here are the key points I would include in a conclusion:

- Overall the code quality and architecture of the Curves protocol contracts are good. The code is well structured, tested and follows solidity best practices.

- Some opportunities for improvement include adding more internal documentation, error handling, formal verification, and upgrading to support future changes.

- The architecture separates core concerns nicely but could be strengthened by adding interfaces and supporting upgradeability.

- There are some centralization risks from owner/manager roles that need to be mitigated through decentralization of governance over time.

- Systemic risks around precision issues, large trades impacting prices, lack of pause functionality and reliance on third parties like FeeSplitter also need to be addressed.

- Following best practices in security, decentralization and ongoing maintenance/audits will help ensure the long-term sustainability and success of the protocol.

- With the recommended changes, the Curves protocol shows promise as an open, interoperable infrastructure for trading social tokens in a decentralized manner.

- It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
20 hours