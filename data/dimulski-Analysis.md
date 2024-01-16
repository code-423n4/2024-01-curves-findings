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
  - [8. Test analysis](#8-test-analysis)
## 2. Approach taken in evaluating the codebase
During this 8-days audit the codebase was tested in various different ways. The first step after cloning the repository was ensuring that all tests provided pass successfully when run. After that I set up a foundry environment in order to write tests using the foundry framework. Then began the phase when I got some high-level overview of the functionality by reading the code. There were some parts of the project that caught my attention. These parts were tested with Foundry POCs. Some of them were false positives, others turned out to be true positives that were reported. The contest's main page does provides some context for the main idea of the Curves protocol, and points to some useful explanations of the [friend.tech](https://www.friend.tech/) protocol, from where Curves took the main idea - tokenazing user accounts, everybody is able to buy a token represented by an address referred to as a token subject, and then tried to build upon it by adding new features.
## 3. Architecture recommendations
### Protocol Structure
Curves is a protocol that falls in the SocialFi category. The main goal of the Curves protocol is allow users to create unique tokens, identified by their address, and allow other users to buy these tokens, which will give them access to a chat room with the token subject, and in a way allow them to invest in that person/influencer. The price for which a curves token for a specific token subject can be bought and sold is determined by formula that increases the price of each next token, as the supply grows, and decreases the price of each token when supply shrinks. The protocol has been structured in a very systematic and ordered manner, which makes it easy to understand how the function calls propagate throughout the system. The main contract in the Curves protocol is the ``Curves.sol`` contract, which is responsible for keeping track of the bough and sold tokens, converting those tokens to ERC20 tokens, and distributing the fees to the correct receivers. The Curves team has implemented a ``FeeSplitter.sol`` contract which is responsible for distributing rewards to the token holders, in order to incentivize them to hold their tokens. As well as three other contracts which are reviewed in depth in the Mechanism review section. I would recommend the team to reconsider allowing users to convert their curves tokens to the ERC20 format and trade them outside the protocol, first of all this creates a possibility that a user won't have a full 1e18 ERC20 token, as in order to reintegrate the ERC20 tokens into the Curves ecosystem this condition  ``if (amount % 1 ether != 0) revert NonIntegerDepositAmount();`` has to be false. If there is even 1 wei missing there will be at least 2 users that don't have a full amount of tokens that they can integrate back in the curves system. Consider using the ERC 1155 standard if you want to have such a feature, or represent each token as a different NFT(ERC721) collection.
### What was unique for me?
 - The whole concept of SocialFi was unique for me, as this is the first protocol that I audit from this category of projects
 - The idea that a price of token is determined purely by the amount of tokens previously bought(excluding fees).
 - Converting internal tokens to an ERC20 tokens
## 4. Codebase quality analysis
## 5. Mechanism review
### High-level system overview
The Curves protocol falls in the SocialFi category. From a rudimentary point-of-view, SocialFi is the amalgamation of Web3 and Social Media. The idea combines social media and decentralized finance (DeFi) concepts to create, control, and own user-generated content on platforms. Influencers, content producers, and participants that want to govern their data can do so in the SocialFi universe. A new user of the Curves platform can register with his X(formally twitter account), and create an unique curves token, indexed by the user's address. The unique curve tokens that each user can create for himself are identified by an unique address. That address is referred to as the token subject within the Curves protocol. The price for which a curves token for a specific token subject can be bought and sold is determined by formula that increases the price of each next token, as the supply grows, and decreases the price of each token when supply shrinks.
### Chains supported
The protocol will be deployed on  the form.network. It is an L2 network built on the Optimism Superchain. For more information, visit [link](https://docs.form.network/) and [link](https://info.form.network/).
### Curves.sol
### CurvesERC20.sol
### CurvesERC20Factory.sol
### FeeSplitter.sol
### Security.sol
## 6. Centralization risks
## 7. Security approach of the project
## 8. Test analysis




### Time spent:
20 hours