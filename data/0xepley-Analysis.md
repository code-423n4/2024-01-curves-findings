# 🛠️ Analysis - Curve Protocol
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I would follow when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |New insights and learning of project from this audit | Things learned from the project |


## a) The approach I would follow when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://code4rena.com/audits/2024-01-curves

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-01-curves/blob/main/install.md)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Curves](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| Slither report of the project for some basic analysis|
|5|Test Suits|[Tests](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-01-curves?tab=readme-ov-file#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|Tried to make Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-01-curves?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs)|Code where I should focus more|

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics

-  **Language:** language in which of the source is written
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Blank Lines:** Blank Lines in the source file
-  **Total Lines:** Total number of lines in the file 

## Analysis of sloc of contracts
[![Screenshot-from-2024-01-16-21-03-06.png](https://i.postimg.cc/gjgLrXp5/Screenshot-from-2024-01-16-21-03-06.png)](https://postimg.cc/LgZ8bsnB))


## AST Node Statics of FeeSplitter.sol Contract
[![Screenshot-from-2024-01-16-21-31-25.png](https://i.postimg.cc/qRMJbpWX/Screenshot-from-2024-01-16-21-31-25.png)](https://postimg.cc/XZm6X6jp)

## AST Node Statics of Curve.sol Contract
[![Screenshot-from-2024-01-16-21-34-41.png](https://i.postimg.cc/Bbj1zTRg/Screenshot-from-2024-01-16-21-34-41.png)](https://postimg.cc/30HR4v84)


## Function list of Curve.sol
It contains all the functions that the contract Curve.sol have

[![Screenshot-from-2024-01-16-21-16-12.png](https://i.postimg.cc/0QzMB2q1/Screenshot-from-2024-01-16-21-16-12.png)](https://postimg.cc/R6zZ3mFg)


## Function list of FeeSplitter.sol
It contains all the functions that the contract FeeSplitter.sol have

[![Screenshot-from-2024-01-16-21-21-23.png](https://i.postimg.cc/ydxQp9d6/Screenshot-from-2024-01-16-21-21-23.png)](https://postimg.cc/k6rvB2KL)


## Call graph of Curve.sol
[![Screenshot-from-2024-01-16-21-03-06.png](https://i.postimg.cc/gjgLrXp5/Screenshot-from-2024-01-16-21-03-06.png)](https://postimg.cc/LgZ8bsnB)


## Call graph of FeeSplitter.sol
[![Screenshot-from-2024-01-16-21-12-42.png](https://i.postimg.cc/QMqfSmJW/Screenshot-from-2024-01-16-21-12-42.png)](https://postimg.cc/8Fjdzh31)


## Contract Inheritance Graph
[![Screenshot-from-2024-01-16-21-10-13.png](https://i.postimg.cc/BbNs0238/Screenshot-from-2024-01-16-21-10-13.png)](https://postimg.cc/N2yZ895Y)


## Class diagram of Contracts
[![Screenshot-from-2024-01-16-21-37-45.png](https://i.postimg.cc/ZKTq0FwD/Screenshot-from-2024-01-16-21-37-45.png)](https://postimg.cc/6TmKmRJd)


## Domain Model of the contract
[![Screenshot-from-2024-01-16-21-44-42.png](https://i.postimg.cc/0QV9whWW/Screenshot-from-2024-01-16-21-44-42.png)](https://postimg.cc/MMjkk38V)


## Contract Description Table of Curves.sol

| Contract Type  | Function Name                 | Visibility     | Mutability | Modifiers          | Security         | Implementation       |
| -------------- | ----------------------------- | --------------- | ---------- | ------------------ | ---------------- | --------------------- |
| Public         | setFeeRedistributor           | External        | 🛑        | onlyOwner          |                  |                       |
| External       | setMaxFeePercent              | External        | 🛑        | onlyManager        |                  |                       |
| External       | setProtocolFeePercent         | External        | 🛑        | onlyOwner          |                  |                       |
| External       | setExternalFeePercent         | External        | 🛑        | onlyManager        |                  |                       |
| External       | setReferralFeeDestination     | Public          | 🛑        | onlyTokenSubject   |                  |                       |
| External       | setERC20Factory               | External        | 🛑        | onlyOwner          |                  |                       |
| Public         | getFees                       | Public          |            |                    |                  |                       |
| Public         | getPrice                      | Public          |            |                    |                  |                       |
| Public         | getBuyPrice                   | Public          |            |                    |                  |                       |
| Public         | getSellPrice                  | Public          |            |                    |                  |                       |
| Public         | getBuyPriceAfterFee           | Public          |            |                    |                  |                       |
| Public         | getSellPriceAfterFee          | Public          |            |                    |                  |                       |
| Public         | buyCurvesToken                | Public          | 💵        |                    |                  |                       |
| Internal       | _transferFees                 | Internal        |            |                    |                  |                       |
| Internal       | _buyCurvesToken               | Internal        |            |                    |                  |                       |
| Public         | sellCurvesToken               | Public          |            |                    |                  |                       |
| External       | transferCurvesToken           | External        |            |                    |                  |                       |
| External       | transferAllCurvesTokens       | External        |            |                    |                  |                       |
| Internal       | _transfer                     | Internal        |            |                    |                  |                       |
| Internal       | _addOwnedCurvesTokenSubject   | Internal        |            |                    |                  |                       |
| Internal       | _deployERC20                  | Internal        |            |                    |                  |                       |
| Public         | buyCurvesTokenWithName         | Public          | 💵        |                    |                  |                       |
| Public         | buyCurvesTokenForPresale       | Public          | 💵        | onlyTokenSubject   |                  |                       |
| External       | setWhitelist                  | External        | 🛑        |                    |                  |                       |
| Public         | buyCurvesTokenWhitelisted     | Public          | 💵        |                    |                  |                       |
| Public         | verifyMerkle                  | Public          |            |                    |                  |                       |
| External       | setNameAndSymbol              | External        | 🛑        | onlyTokenSubject   |                  |                       |
| External       | mint                          | External        | 🛑        | onlyTokenSubject   |                  |                       |
| Internal       | _mint                         | Internal        | 🛑        | onlyTokenSubject   |                  |                       |
| Public         | withdraw                      | Public          | 🛑        |                    |                  |                       |
| Public         | deposit                       | Public          | 🛑        |                    |                  |                       |
| Public         | sellExternalCurvesToken       | Public          | 🛑        |                    |                  |                       |


## Contract Description Table of FeeSplitter.sol


| Function Name             | Visibility     | Mutability | Modifiers          |
| ------------------------- | --------------- | ---------- | ------------------ |
|                           |                 |            |                    |
| setCurves                 | Public          | 🛑        |                    |
| balanceOf                 | Public          | NO❗️     |                    |
| totalSupply               | Public          | NO❗️     |                    |
| getUserTokens             | Public          | NO❗️     |                    |
| getUserTokensAndClaimable | Public          | NO❗️     |                    |
| updateFeeCredit           | Public          | 🛑        |                    |
| getClaimableFees          | Internal 🔒     | 🛑        |                    |
| claimFees                 | Public          | NO❗️     |                    |
| addFees                   | External ❗️     | 🛑        |                    |
| onBalanceChange           | Public          | 💵        | onlyManager        |
| batchClaiming             | Public          | 🛑        | onlyManager        |
|                           | External ❗️     | 💵        |                    |


## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content. In particular, tests have been written successfully.

-   2) Overall line coverage percentage provided by your tests : 99.56

### What could they have done better?

-  1) In order to understand the test scenarios and develop more effective test scenarios, the following bob, alice and other roles are can be defined one by one, in this way role definitions increase the quality and readability in tests

```solidity

 // Sample labels
vm.label(bob, 'bob');
vm.label(alice, 'alice');
vm.label(DEPLOYER, 'deployer');
vm.label(USD_OWNER, 'usd owner');
vm.label(POOL_PROXY, 'lending pool');
```


-  3) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


[![test-cases.jpg](https://i.postimg.cc/1zgD5wCt/test-cases.jpg)](https://postimg.cc/v1s40gdF)

Ref:https://xin-xia.github.io/publication/icse194.pdf

[![nabeel.jpg](https://i.postimg.cc/x1bHPVz4/nabeel.jpg)](https://postimg.cc/ZW4CTgj8)


# Test-case analysis of Contracts

### General Information
- **Solc Version:** 0.8.7
- **Optimizer Enabled:** false
- **Runs:** 200
- **Block Limit:** 30,000,000 gas


| Contract          | Method                     | Min    | Max    | Avg    | # calls |
|-------------------|----------------------------|--------|--------|--------|---------|
| Curves            | buyCurvesToken             | 66676  | 242163 | 124108 | 68      |
| Curves            | buyCurvesTokenForPresale   | 187955 | 208239 | 205341 | 7       |
| Curves            | buyCurvesTokenWhitelisted  | 99264  | 163498 | 126028 | 12      |
| Curves            | buyCurvesTokenWithName     | -      | -      | 1774847| 1       |
| Curves            | mint                       | -      | -      | 1688644| 1       |
| Curves            | sellCurvesToken            | 53738  | 66050  | 59894  | 4       |
| Curves            | sellExternalCurvesToken    | -      | -      | 83967  | 2       |
| Curves            | setERC20Factory            | -      | -      | 29171  | 1       |
| Curves            | setExternalFeePercent      | 55938  | 75910  | 65924  | 2       |
| Curves            | setFeeRedistributor        | -      | -      | 29150  | 1       |
| Curves            | setMaxFeePercent           | -      | -      | 55164  | 2       |
| Curves            | setNameAndSymbol           | 44952  | 74614  | 67199  | 4       |
| Curves            | setProtocolFeePercent      | -      | -      | 77875  | 2       |
| Curves            | setReferralFeeDestination  | -      | -      | 44906  | 1       |
| Curves            | setWhitelist               | -      | -      | 48121  | 1       |
| Curves            | transferAllCurvesTokens    | -      | -      | 258653 | 1       |
| Curves            | transferCurvesToken        | 84506  | 98483  | 95688  | 5       |
| Curves            | transferOwnership          | -      | -      | 27127  | 1       |
| Curves            | withdraw                   | 153428 | 1813418| 1689234| 15      |
| FeeSplitter       | addFees                    | 47908  | 65008  | 61587  | 10      |
| FeeSplitter       | batchClaiming              | -      | -      | 130582 | 2       |
| FeeSplitter       | claimFees                  | -      | -      | 77744  | 2       |
| FeeSplitter       | onBalanceChange            | 69648  | 106648 | 85597  | 10      |
| FeeSplitter       | setCurves                  | 44095  | 44119  | 44118  | 56      |
| FeeSplitter       | setManager                 | 46613  | 46637  | 46636  | 50      |
| MockERC20         | burn                       | -      | -      | 34615  | 1       |
| MockERC20         | mint                       | 51788  | 68888  | 65779  | 11      |



## Deployments

| Contract               | Min      | Max      | Avg      | % of limit   |
|------------------------|----------|----------|----------|--------------|
| Curves                 | 5232807  | 5232831  | 5232828  | 17.4 %       |
| CurvesERC20Factory     | -        | -        | 2217610  | 7.4 %        |
| FeeSplitter            | -        | -        | 1558996  | 5.2 %        |
| MockCurvesForFee       | -        | -        | 243714   | 0.8 %        |
| MockERC20              | 1416771  | 1436683  | 1418591  | 4.7 %        |



## d) Security Approach of the Project

### Successful current security understanding of the project;

1- The project hasn't underwent any audits, this innovative assessments on Code4rena is the first, where multiple auditors are scrutinizing the code.

### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)


2- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

3- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/


4- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

5- I also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

6 - As the project team, you can consider applying the multi-stage audit model.

[![sla.png](https://i.postimg.cc/nhR0kN3w/sla.png)](https://postimg.cc/Sn96Q1FW)

Read more about the MPA model;
https://mpa.solodit.xyz/

7 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19



## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-01-curves/blob/main/bot-report.md

**Previous Audits**
There isn't any Previous Audit

##  f) Packages and Dependencies Analysis 📦

| Package | Version | Usage | 
| --- | --- | --- | 
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts) | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts) |  Project uses version `4.9.3`; consider updating to `5.0.1` 
| [`solhint`](https://github.com/protofire/solhint) | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts) |  Project relies on version `3.3.6`; consider upgrading to `4.1.1`

## g) New insights and learning of project from this audit:

1. **Smart Fee Handling**: The `FeeSplitter` contract is a clever bit of work. It's designed to take in fees from various tokens and then distribute these to the token holders. This setup is great for keeping people invested in your ecosystem, as they get a direct benefit from its success.

2. **Tight Security Measures**: Your use of the `Security` contract for managing access is a smart move. In the world of decentralized systems, keeping unauthorized users out and protecting against vulnerabilities is key. This contract acts like a bouncer at a club, carefully checking who gets in.

3. **Leveraging Trusted Standards**: By integrating OpenZeppelin contracts for things like ERC20 tokens, you're tapping into well-tested and secure code. It's like using building blocks that everyone trusts, ensuring that your project is both secure and compatible with a wide range of other applications and services.

4. Factory for Tokens: You've employed a factory pattern in **CurvesERC20Factory** and **CurvesERC20** for creating ERC20 tokens. This model allows for the efficient generation of new ERC20 tokens with minimal effort, promoting a dynamic and flexible token ecosystem.

In summary, the project showcases a deep understanding of smart contract development with a focus on security, modular construction, and user incentives. There's room to grow, especially in areas like detailed tokenomics and governance, but what you have is a solid foundation for a complex and secure decentralized system.


Note: I didn't tracked the time, the time I mentioned is just an estimate


### Time spent:
3 hours


### Time spent:
3 hours