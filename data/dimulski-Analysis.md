# Curves analysis report 
## 1. Table of contents
- [Curves analysis report](#curves-analysis-report)
  - [1. Table of contents](#1-table-of-contents)
  - [2. Approach taken in evaluating the codebase](#2-approach-taken-in-evaluating-the-codebase)
  - [3. Architecture recommendations](#3-architecture-recommendations)
    - [Protocol Structure](#protocol-structure)
    - [What was unique for me?](#what-was-unique-for-me)
  - [4. Codebase quality analysis](#4-codebase-quality-analysis)
  - [5. Mechanism review](#5-mechanism-review)
    - [High-level system overview](#high-level-system-overview)
    - [Chains supported](#chains-supported)
    - [Curves.sol](#curvessol)
    - [CurvesERC20.sol](#curveserc20sol)
    - [CurvesERC20Factory.sol](#curveserc20factorysol)
    - [FeeSplitter.sol](#feesplittersol)
    - [Security.sol](#securitysol)
  - [6. Centralization risks](#6-centralization-risks)
  - [7. Security approach of the Project](#7-security-approach-of-the-project)
  - [8. Software engineering considerations](#8-software-engineering-considerations)
    - [Modularity and reusability](#modularity-and-reusability)
    - [Code readability and maintainability](#code-readability-and-maintainability)
    - [Error handling and input validation](#error-handling-and-input-validation)
    - [Upgradeability](#upgradeability)
    - [Documentation](#documentation)
## 2. Approach taken in evaluating the codebase
During this 8-days audit the codebase was tested in various different ways. The first step after cloning the repository was ensuring that all tests provided pass successfully when run. After that I set up a foundry environment in order to write tests using the foundry framework. Then began the phase when I got some high-level overview of the functionality by reading the code. There were some parts of the project that caught my attention. These parts were tested with Foundry POCs. Some of them were false positives, others turned out to be true positives that were reported. The contest's main page does provide some context for the main idea of the Curves protocol, and points to some useful explanations of the [friend.tech](https://www.friend.tech/) protocol, from where Curves took the main idea - tokenazing user accounts, everybody is able to buy a token represented by an address referred to as a token subject, and then tried to build upon it by adding new features.
## 3. Architecture recommendations
### Protocol Structure
Curves is a protocol that falls in the SocialFi category. The main goal of the Curves protocol is allow users to create unique tokens, identified by their address, and allow other users to buy these tokens, which will give them access to a chat room with the token subject, and in a way allow them to invest in that person/influencer. The price for which a curves token for a specific token subject can be bought and sold is determined by formula that increases the price of each next token, as the supply grows, and decreases the price of each token when supply shrinks. The protocol has been structured in a very systematic and ordered manner, which makes it easy to understand how the function calls propagate throughout the system. The main contract in the Curves protocol is the ``Curves.sol`` contract, which is responsible for keeping track of the bough and sold tokens, converting those tokens to ERC20 tokens, and distributing the fees to the correct receivers. The Curves team has implemented a ``FeeSplitter.sol`` contract which is responsible for distributing rewards to the token holders, in order to incentivize them to hold their tokens. As well as three other contracts which are reviewed in depth in the Mechanism review section. 
### What was unique for me?
 - The whole concept of SocialFi was unique for me, as this is the first protocol that I audit from this category of projects
 - The idea that a price of token is determined purely by the amount of tokens previously bought(excluding fees).
 - Converting internal tokens to an ERC20 tokens
## 4. Codebase quality analysis
There are several vulnerabilities within the codebase, but beside that the code is well structured. The logic in functions except the ``_transferFees()`` function is split between smaller functions which are reused trough the code. I would recomend that the ``_transferFees()`` function is rewrriten as there are a couple vulnerabilities within it, splitting it in smaller functions consisting of concrete logic would be clearer and if it has to be changed later on it will be easier. I would also recomend to remove the  ``CurvesErrors`` interface from the ``Curves.sol`` file, create a separate folder named interfaces and put it there, then inport it from there in the ``Curves.sol``. Some additional suggestions are provided in the Software engineering considerations chapter.
## 5. Mechanism review
### High-level system overview
The Curves protocol falls in the SocialFi category. From a rudimentary point-of-view, SocialFi is the amalgamation of Web3 and Social Media. The idea combines social media and decentralized finance (DeFi) concepts to create, control, and own user-generated content on platforms. Influencers, content producers, and participants that want to govern their data can do so in the SocialFi universe. A new user of the Curves platform can register with his X(formally twitter account), and create an unique curves token, indexed by the user's address. The unique curve tokens that each user can create for himself are identified by an unique address. That address is referred to as the token subject within the Curves protocol. The price for which a curves token for a specific token subject can be bought and sold is determined by formula that increases the price of each next token, as the supply grows, and decreases the price of each token when supply shrinks.
### Chains supported
The protocol will be deployed on  the form.network. It is an L2 network built on the Optimism Superchain. For more information, visit [link](https://docs.form.network/) and [link](https://info.form.network/).
### Curves.sol
The ``Curves.sol`` contract is where the main logic of the protocol resides. Calculating the price and fees for tokens, creating, buying and selling tokens is performed from this contract. The pricing function is the same as the one in the [friend.tech](https://www.friend.tech/) protocol, which was able to generate a lot of fees and profits for some users. The Curves protocol adds several additional features such as whitelisting users for curves tokens categories if the token subject decides to, as well as the option to convert curves tokens into ERC20 tokens, which can be traded outside of the Curves protocol. I would recommend the team to reconsider allowing users to convert their curves tokens to the ERC20 format and trade them outside the protocol. This creates a possibility that a user won't have a full 1e18 ERC20 token, as in order to reintegrate the ERC20 tokens into the Curves ecosystem this condition  ``if (amount % 1 ether != 0) revert NonIntegerDepositAmount();`` has to be false. If there is even 1 wei missing there will be a user that doesn't have the full amount of tokens that they can integrate back in the curves system. Consider using the ERC 1155 standard if you want to have such a feature, or represent each token as a different NFT(ERC721) collection. Consider rewriting the ``_transferFees()`` function to not directly distribute funds to the ``referralFeeDestination`` address, rather store the amount the referral is supposed to receive, and allow him to withdraw it, whenever he decides. Utilize the [pull over push pattern](https://fravoll.github.io/solidity-patterns/pull_over_push.html). Also splitting the ``_transferFees()`` function in smaller functions focused on a specific transfer is advised.

### CurvesERC20.sol
The ``CurvesERC20.sol`` contract is a simple contract that allows the minting and burning of ERC20 tokens, only by the owner of the contract which will be the ``Curves.sol`` contract. The contract should work as intended, except the inherit risk of utalizing ERC20 tokens to represent curves tokens described above.

### CurvesERC20Factory.sol
This is a simple factory contract that deploys an ERC20 contract, with metadata consisting of name and symbol. There is nothing else to be said about this contract.
### FeeSplitter.sol
The ``FeeSplitter.sol`` contract is responsible for allowing users that hold cures tokens to accrue fees, and later on claim them. There are several vulnerabilities in the contract, such as missing access control in the ``setCurves()`` function, a malicious user can steal the rewards form all users byt claiming and transfering curve tokens, as well as incorect unclaimed fees variable updates in some cases. Once this issues are fixed the contract should work fine. There will be some dust amounts of ETH left in the contract, but a function that withdraws an arbitrary amount of ETH and distributes it, is not necessary as it will increase the centralization risks of the protocol described in the next section.

### Security.sol
The ``Security.sol`` contract acts as an access controller. Once the vulnerabilites in the modifiers are fixed the contract should work as expected. I would recomend to utalize the [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) contract from Openzeppelin, facilitating a transfer of ownership always entails potential risks, instead of writing your own onlyOwner modifer.  
## 6. Centralization risks
The protocol is not heavily centralized, although there are very importatn parameters that are controled by the manager and owner roles. In the ``Curves.sol`` contract if one of the wallets holding a manager or owner role gets compromised the protocol fees can be set to 100% and the protocolFeeDestination can be set to an address controled by a malicious user, since there is no slippage protections on the ``sellCurvesToken()`` function this can be used to front run  non suspecting users and effectively steal all the ``ETH`` they are supposed to withdraw. Also there are no input validation checks in the setting fees functions, so admins should be extreamly carreful when they set those parameters. There are no timelock functions for actions affecting all the users of the protocol, I would advise the team to consider adding such functions. If an admin account gets compromised, having a timelock function will give the users enough time to withdraw their funds safely if no other actions to protect them can be taken from the team. If the onwer account is compromised a malicious factory contract can be created, that can allow malicious users to mint arbitrary amount of ERC20 tokens that represent certain curve tokens, and later integrate them in the Curves protocol. Consider utalizing multisig wallets such as [Safe](https://safe.global/), in order to better secure the accounts, or implement a DAO and give the owner role to the DAO. Once the vulnerabilities mentioned in the above section in the ``Security.sol`` and ``FeeSplitter.sol`` contracts, are fixed the access control should be sufficient. 

## 7. Security approach of the project
The protocol is missing some edge case tests, as well as other unit tests which don't consider only the expected behaviour of the user. Consider creating a foundry environment and movig your test suite to foundry, as foudnry has some tools that can make testing easier plus running tests is way faster. Consider implementing fuzz(foundry built in fuzzer is a great tool) and invariant tests to better cover edge cases. It is a very good decision that you decided to get your code base audited by a platfrom such a C4, although all auditors put their best effort in finding and submitting all vulnerabilites, we can miss something, creating a bug bounty on platforms such as [immunefi](https://immunefi.com/) creates and additional layer of security. 
## 8. Software engineering considerations
### Modularity and reusability

The contracts withing the protocol exhibit modularity by inheriting from several interfaces and contracts, suggesting components are designed for reuse and extension. This approach aligns with solid software engineering principles, promoting maintainability and scalability.
### Code readability and maintainability

The protocol's naming conventions are not that clear, some improvement may be welcomed, exspecialy in the naming of certain functions which actualy create a cruve token associated with an address, shouldn't have buy in their name. NatSpec is missing the severely decreases the readability and maintainability of the code which is crucial for long-term management and updates.
### Error handling and input validation

The use of custom error messages indicates a proactive approach to error handling, however input validations are missing, especially when setting the protocol fees, this can cause mistakes which results in a loss for the protocol. Input validation should be implemented as it is critical for robustness and user trust.
### Upgradeability

The potential for future upgrades and modifications should be considered, especially in a new and rapidly evolving domain like SocialFi. However, the protocol doesn’t explicitly mention upgrade mechanisms.
### Documentation

Documentation is missing, creating a good documentation helps with the long-term management and updates of the contract.









### Time spent:
20 hours