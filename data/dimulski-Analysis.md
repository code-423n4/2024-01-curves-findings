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
During this 8-days audit the codebase was tested in various different ways. The first step after cloning the repository was ensuring that all tests provided pass successfully when run. After that I set up a foundry environment in order to write tests using the foundry framework. Then began the phase when I got some high-level overview of the functionality by reading the code. There were some parts of the project that caught my attention. These parts were tested with Foundry POCs. Some of them were false positives, others turned out to be true positives that were reported. The contest's main page does provides some context for the main idea of the Curves protocol, and points to some useful explanations of the [friend.tech](https://www.friend.tech/) protocol, from where Curves took the main idea - tokenazing user accounts, everybody is able to buy a token represented by an address reffered to as a token subject, and then tried to build upon it by adding new features.
## 3. Architecture recommendations
### Protocol Structure
### What was unique for me?
## 4. Codebase quality analysis
## 5. Mechanism review
### High-level system overview
The Curves protocol falls in the SocialFi category. From a rudimentary point-of-view, SocialFi is the amalgamation of Web3 and Social Media. The idea combines social media and decentralized finance (DeFi) concepts to create, control, and own user-generated content on platforms. Influencers, content producers, and participants that want to govern their data can do so in the SocialFi universe. A new user of the Curves platform can register with his X(formaly twitter account), and create an unique curves token, indexed by the user's address. The unique tokens that each user can create for himslef are reffered to as token subjects within the Curves protocol. The price for which a token subject can be bought and sold is determnied by a 
### Chains supported
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