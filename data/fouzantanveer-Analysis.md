## Analysis of the Codebase
[![Lines.jpg](https://i.postimg.cc/MGDzZnHT/Lines.jpg)](https://postimg.cc/CZz3mLjT)

Before discussing the analysis, let's define the parameters in the image:

*Lines*
The total number of lines in the file, including code, comments, and whitespace.

*nLines*
The number of lines that are not blank, often equating to the sum of the lines of code (SLOC) and comment lines.

*nSLOC (Source Lines Of Code)*
This refers to the number of lines that are actual code, not including comments or blank lines. It's a common measure of the amount of work in a codebase.

*Comment Lines*
These are lines containing comments which explain the code and can indicate how well-documented the codebase is.

*Complexity Score (Complex Score)*
This likely represents a calculated metric of code complexity, which may factor in aspects like the depth of nested structures, the number of conditional paths, and the use of complex data types or structures.

The provided codebase consists of five Solidity files, each serving a specific purpose:
1. Security.sol: This file defines a basic security contract that manages an owner and a list of managers. It is relatively simple and doesn't interact directly with other files.
2. FeeSplitter.sol: The FeeSplitter contract is more complex and interacts with the Curves.sol contract. It manages fee distribution among token holders. It relies on functions from Curves.sol to calculate balances and supply information.
3. Curves.sol: This contract appears to be the core of the project and has the most complex logic. It manages token creation and various calculations related to token balances and supply. It interacts with other contracts, including FeeSplitter.sol.
4. CurvesERC20Factory.sol: This contract is responsible for deploying instances of the CurvesERC20 contract. It doesn't have complex logic and mainly serves as a factory for creating ERC-20 tokens.
5. CurvesERC20.sol: This contract extends the ERC-20 standard and is used for token creation. It inherits from ERC20 and Ownable and doesn't contain overly complex logic.

## Interactions Between Files:
- FeeSplitter.sol interacts with Curves.sol by calling functions to calculate balances and distribute fees to token holders.
- Curves.sol is the central contract that other contracts rely on. It interacts with FeeSplitter.sol for fee distribution calculations and with CurvesERC20.sol for token creation.
- CurvesERC20Factory.sol interacts with CurvesERC20.sol by creating instances of the CurvesERC20 token.

## Here is the class diagram showing the contracts 

[![Whats-App-Image-2024-01-16-at-9-28-19-PM.jpg](https://i.postimg.cc/fTL56vkQ/Whats-App-Image-2024-01-16-at-9-28-19-PM.jpg)](https://postimg.cc/NL31Xm1d)	

*Codebase Evaluation:*

- *Modularity*: The codebase is modular, with contracts separated by their functions. This modularity is a good practice for code organization.

- *Documentation*: The level of documentation varies between contracts. Curves.sol appears to be the most documented, while CurvesERC20Factory.sol and CurvesERC20.sol have fewer comments.

- *Overall Quality*: The codebase appears to be of reasonable quality. However, the quality of the comments and documentation can be improved, especially in contracts where it is relatively sparse. Ensuring that all contracts are well-documented is essential for the long-term maintainability of the codebase.
In summary, the codebase demonstrates good modularity and well-defined interactions between files. However, there is room for improvement in terms of documentation, particularly in contracts with fewer comments.

## Approach I taken to review the codebase
In order to comprehensively review the codebase, I followed a structured approach that included the following steps:
1. *Documentation Review:*
   - I initiated the codebase review by thoroughly examining the available documentation provided at the start of the project. These documents included:
     - [Understanding Friend Tech through Smart Contracts](https://ada-d.medium.com/understanding-friend-tech-through-smart-contracts-edac5d98cd49)
     - [Friend Tech Smart Contract Breakdown](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf)
     - [Codebase on BaseScan](https://basescan.org/address/0xcf205808ed36593aa40a44f10c7f7c2f67d4a4d4#code)
   - These documents provided an initial understanding of how the project works, its objectives, and its smart contract architecture.

2. *4naly3er Report Review:*
- Following the documentation review, I referred to the [4naly3er report](https://github.com/code-423n4/2024-01-curves/blob/main/4naly3er-report.md) available on GitHub. This report likely contained insights from a comprehensive automated analysis of the codebase, highlighting potential issues, vulnerabilities, and providing additional context.



3. *Manual Code Review:*
   - After gaining an initial understanding from the documentation and 4naly3er report, I proceeded to manually review the codebase. This manual review involved going through the individual Solidity files, inspecting functions, variables, and the overall structure of the contracts.
   - I paid particular attention to code quality, adherence to best practices, and the clarity of the interactions between different contracts.

4. *Creating Diagram for Code Understanding:*
- At a certain point in my review process, I created a diagram to visually represent the interactions between the various contracts and functions within the project. This diagram served as a valuable aid in understanding the codebase's architecture and flow.

 *Curves.sol*
[![Interaction-1.png](https://i.postimg.cc/d37F9nBk/Interaction-1.png)](https://postimg.cc/T5XBdVBf)


[![Interaction-2.png](https://i.postimg.cc/Y0JHDgnX/Interaction-2.png)](https://postimg.cc/q6XSztdn)

5. *Reviewing Areas of Concern:*
   - Finally, I focused on reviewing areas of concern within the codebase. These areas might include potential vulnerabilities, complex logic, or critical functions. This step aimed to ensure the security and robustness of the project.

Overall, my approach to reviewing the codebase was structured and thorough. It began with understanding the project's documentation, leveraging external reports for insights, conducting a manual code review, creating visual aids for comprehension, and concluding with a detailed assessment of critical areas. This approach allowed I to gain a comprehensive understanding of the codebase's quality, security, and functionality.


## Architecture of the project
To provide insights on the architecture of the Curves project, I will focus on the purpose of key functions in each file and how they interact with each other. This analysis is based on the structure and content of the Solidity files provided.

## OverAll Call graph of the Contracts to track functions
[![Screenshot-from-2024-01-17-00-43-10.png](https://i.postimg.cc/DZcQJSpT/Screenshot-from-2024-01-17-00-43-10.png)](https://postimg.cc/hzfQNtqy)


## UML of whole system
[![UML.png](https://i.postimg.cc/4xpJMLTp/UML.png)](https://postimg.cc/k2X3RyG5)


### Curves.sol
- *Key Functions & Interactions*:
  - setFeeRedistributor, setMaxFeePercent, setProtocolFeePercent: These functions are used for configuring the fee structure. They likely interact with FeeSplitter.sol for fee distribution.
- buyCurvesToken, sellCurvesToken: Central to the protocol, these functions handle the buying and selling of tokens. They likely interact with CurvesERC20.sol for token minting/burning and with FeeSplitter.sol for fee handling.

 ## Call Graph of Curve.sol
[![JYgz8VS.md.png](https://iili.io/JYgz8VS.md.png)](https://freeimage.host/i/JYgz8VS)

## buyCurvesToken Sequence Diagram:

 [![Buy.png](https://i.postimg.cc/hjZgxN9R/Buy.png)](https://postimg.cc/BLFdfYYm)

## sellCurvesToken Sequence Diagram:

[![Sell.png](https://i.postimg.cc/63pt8B4h/Sell.png)](https://postimg.cc/gwQf5FDw)
  - getPrice, getBuyPrice, getSellPrice: These functions compute prices for transactions. Their outputs are probably used within buyCurvesToken and sellCurvesToken for determining transaction costs.
  - The contract seems to serve as the core of the protocol, orchestrating various aspects of the system.

### CurvesERC20.sol
- *Key Functions & Interactions*:
  - mint, burn: These functions control the token supply. They are likely called by Curves.sol during the process of buying and selling tokens.
  - transferOwnership: Allows transferring control of the token contract, important for administrative purposes.
  - This contract appears to be a standard ERC20 implementation with added functionalities specific to the Curves protocol.
 
   [![Screenshot-from-2024-01-17-00-41-05.png](https://i.postimg.cc/Hkj6q9Z7/Screenshot-from-2024-01-17-00-41-05.png)](https://postimg.cc/cvy7YwNs)

### CurvesERC20Factory.sol
- *Key Functions & Interactions*:
  - deploy: Creates new instances of CurvesERC20 tokens. This factory pattern suggests that Curves.sol may call this function when new token types need to be created within the protocol.

### FeeSplitter.sol
- *Key Functions & Interactions*:
  - claimFees, addFees: Manage fee distribution and accumulation. These functions probably interact closely with Curves.sol especially in the context of transaction fee handling.
  - setCurves: This function likely establishes a link to the Curves.sol contract, indicating a direct interaction for fee-related activities.
 
  
## Call Graph for FeeSplitter.sol
[![JYgz8VS.md.png](https://iili.io/JYgz8VS.md.png)](https://freeimage.host/i/JYgz8VS)

### Security.sol
- *Key Functions & Interactions*:
  - setManager, transferOwnership: Used for managing access and control over the contract. These functions might be used across all contracts for maintaining secure access control.
  - Incorporates essential security features and is probably integrated into other contracts via inheritance, as seen in FeeSplitter.sol.

### Overall Architecture Insights
- The architecture seems to follow a modular approach, with distinct responsibilities assigned to each contract. 
- Curves.sol acts as the central contract, interfacing with other contracts for specific functionalities like token management (CurvesERC20.sol), token creation (CurvesERC20Factory.sol), fee distribution (FeeSplitter.sol), and security management (Security.sol).
- The use of a factory pattern in CurvesERC20Factory.sol suggests a scalable approach to token creation.
- The system's security and access control are centralized in the Security.sol contract, which is a common practice in smart contract development for ease of management and auditability.
- It's important to ensure that these inter-contract interactions are secure and efficient, especially in scenarios like fee distribution and token transactions, which are sensitive in nature.
Here’s the detailed UML diagram:


## Architecture recommendations of the project
After a detailed analysis of the files in the Curves project, here are some specific architecture recommendations and observations:
### Curves.sol
- *Functionality*: Contains key functions like setFeeRedistributor, setMaxFeePercent, buyCurvesToken, sellCurvesToken, etc. These functions are integral to the protocol's operations.

- *Access Control*: Utilizes onlyOwner and onlyManager modifiers for function access control. Ensure these are consistently and correctly applied to protect sensitive operations.
- *Integration with Other Contracts*: Interacts with other contracts such as CurvesERC20 and FeeSplitter. Verify the integrity of these interactions, especially in terms of state changes and value transfers.
- *Pricing Logic*: Includes functions like getPrice, getBuyPrice, and getSellPrice. Ensure that the pricing logic is robust and tested against edge cases.


### CurvesERC20.sol
- *Token Management*: Provides token minting and burning functionalities. These should be audited for security, particularly to prevent unauthorized minting or burning.
- *Ownership Transfer*: Includes functionality to transfer ownership, which is crucial for administrative control. Make sure that the ownership transfer mechanism is secure.

### CurvesERC20Factory.sol
- *Token Deployment*: Simplifies the process of deploying new CurvesERC20 tokens. Validate that the deployment process is secure and does not introduce vulnerabilities.

### FeeSplitter.sol
- *Fee Distribution*: Manages fee distribution among token holders. It's essential to verify that the fee calculation and distribution logic are accurate and resistant to manipulation.
- *Batch Operations*: Contains functions like batchClaiming, which should be optimized for gas efficiency and should not fail due to block gas limits.


### Security.sol
- *Access Control Implementation*: Defines and implements access control mechanisms, including onlyOwner and onlyManager modifiers. These are crucial for maintaining the security of the protocol.
- *Owner and Manager Functions*: Functions such as setManager and transferOwnership are present. It's vital to ensure that only authorized users can execute these functions.

## Activity Diagram
[![1.png](https://i.postimg.cc/rszVPq8p/1.png)](https://postimg.cc/tnQG1Qn0)

The activity commences with the identification of the user's status in the protocol. If the user is new to the system, they undergo a registration process to become an active participant. This initial step is crucial as it ensures that all subsequent actions are performed by recognized users, which is a foundational aspect of the system's security and functionality. Once registered, or if the user is already recognized, the flow advances to the selection of actions, where users can navigate through the various functionalities offered by the Curves protocol.



Following action selection, users can modify protocol parameters, a critical step that influences the subsequent operations within the ecosystem. This may include adjustments to the token economics or operational rules. A crucial decision point here is whether the user intends to modify the fee structure. If so, they can set the fee redistributor parameters, which directly impact the protocol's incentive and revenue distribution model. This decision is pivotal as it affects the token dynamics and overall economic design of the protocol.



[![3.png](https://i.postimg.cc/02TkVgh8/3.png)](https://postimg.cc/0bC1N483)

In the third phase, the diagram details the token-specific operations, providing options for users to buy or sell tokens. This step interacts with the CurvesERC20 contract functions, which manage the core token mechanics such as minting and burning. Users decide whether to inject new tokens into circulation via minting or reduce the supply by burning existing tokens. This aspect of the activity diagram underscores the importance of token supply management in the protocol's economy and directly ties back to the fee structure and incentive mechanisms established earlier.

[![2.png](https://i.postimg.cc/ryYFtRJR/2.png)](https://postimg.cc/qgydPRWp)

The final part of the activity flow involves decisions around fee distribution, which is a critical operation for token holders to receive their due rewards. The FeeSplitter contract plays a significant role here, managing the claims and distributions of fees. An alternative path leads to security management, where actions such as setting managerial roles or transferring contract ownership are handled. This segment of the diagram illustrates the close relationship between economic incentives and the overarching security framework of the protocol.


[![4.png](https://i.postimg.cc/mr2pcBX7/4.png)](https://postimg.cc/ZW2x2thq)

### Time spent:
14 hours