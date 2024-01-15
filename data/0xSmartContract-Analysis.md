# 🛠️ Analysis - Curves Audit 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2024-01-curves

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-01-curves/blob/main/install.md#run-localhost)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Curves](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2024-01-curves/blob/main/install.md#static-validation)| The project does not currently have a slither result, a slither control was created from initial|
|5|Test Suits|[Tests](https://github.com/code-423n4/2024-01-curves/blob/main/install.md#test)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-01-curves/tree/main?tab=readme-ov-file#scoping-detail)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-01-curves/tree/main?tab=readme-ov-file#scoping-details)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit

In addition, the auditor can decide on the question of where should I end the audit by examining these metrics;

Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="812" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/7be951be-d62f-4817-b62e-52b7e8d20bdc">






</br>
</br>

## Project - Smart Contracts Summary;

<img width="839" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/db9a5b55-67a8-4225-9fc4-9ee7de51121d">
<img width="838" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/8fa340dc-71c0-428f-b54d-5d64d9966bef">
<img width="837" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/1957677e-218f-4215-b976-913f9d4fe6f3">
<img width="838" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/7189cf45-7cf1-4344-876a-cb0a0937afde">
<img width="836" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/1e6d26f9-0412-43ec-9f97-b1e58a568c70">


</br>
</br>
</br>
</br>

## Curves.sol


<img width="1513" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/e02e9f7e-0e01-46ca-a4fe-de345be7c3f7">

#### sellExternalCurvesToken()

- This function allows a user to sell an external CurvesToken.
- Deposit Token: Calls the deposit function, passing the curvesTokenSubject and amount.
- Sell CurvesToken: Calls sellCurvesToken to handle the actual selling process. The amount is divided by 1 ether to normalize it (assuming 1 ether is a standard unit like wei in Ethereum).



#### deposit()

- Deposits the specified amount of CurvesToken.
- Burn and Transfer: Burns the external token from the sender's balance and transfers the normalized amount of CurvesToken to the sender.




#### sellCurvesToken()

- Handles the selling of CurvesToken.
- Calculate Price: Calculates the selling price based on the remaining supply and the amount being sold.
- Update Balances and Supply: Deducts the amount from the sender's balance and reduces the total supply.
- Transfer Fees: Calls _transferFees to handle fee transfers related to the sale.

</br>
</br>
</br>
</br>
</br>

<img width="1505" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/e18fa2c7-5079-43bc-8b49-7a2ad12b1f79">


#### mint() Function

- This is an external function that initiates the minting process for a new token.
- Check Token Metadata: It first checks if the name and symbol of the token (retrieved from externalCurvesTokens mapping using curvesTokenSubject) are empty. If either is empty, it sets them to default values (DEFAULT_NAME and DEFAULT_SYMBOL).
- Call Internal Mint: It then calls the internal _mint function, passing the curvesTokenSubject, and the possibly updated name and symbol.

#### _mint() Function

- This is an internal function that continues the minting process.
- Check Token Existence: It checks if the token already exists (i.e., if its address is not zero in the externalCurvesTokens mapping). If it exists, it reverts with ERC20TokenAlreadyMinted.
- Deploy ERC20: If the token doesn't exist, it calls _deployERC20 to create the token.


#### _deployERC20() Function

- This function actually deploys the new ERC20 token.
- Modify Name and Symbol for Default: If the token's symbol is the default (DEFAULT_SYMBOL), it appends a counter value to both the name and symbol. This ensures uniqueness for tokens with default symbols.
- Check for Duplicate Symbol: It checks if the symbol already exists for another token (symbolToSubject mapping). If it does, it reverts with InvalidERC20Metadata.
- Deploy Token: Calls the deploy function of the CurvesERC20Factory contract, creating a new CurvesERC20 token with the given name, symbol, and setting the contract itself as the owner.
- Update Mappings: Updates various mappings (externalCurvesTokens, externalCurvesToSubject, symbolToSubject) with the new token's details.


#### CurvesERC20Factory()  Contract's deploy Function

- Used to create a new CurvesERC20 token contract.
- Parameters: Takes name, symbol, and owner (address) as parameters.
- Create Token: Instantiates a new CurvesERC20 contract with the provided name, symbol, and owner.
- Return Token Address: Returns the address of the newly created CurvesERC20 token contract.

</br>
</br>
</br>
</br>
</br>

<img width="1668" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/512c5f45-576e-4bce-b107-bac1ae3fe27e">

- Initiate Withdrawal: The process starts with the withdraw function being called by a user.

- Balance Verification: The function checks if the user has enough tokens to withdraw.

- Token Existence and Deployment: If the token doesn't exist, it's deployed using _deployERC20.

- Token Transfer: The _transfer function moves the specified amount of tokens.

- Token Minting: The CurvesERC20 contract mints the equivalent amount of tokens to the user.


</br>
</br>
</br>
</br>
</br>


<img width="1667" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/fc855a4a-89e5-4100-b86d-ec704191963f">


</br>
</br>
</br>
</br>
</br>

## FeeSplitter.sol

<img width="1662" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/526f2b6d-97f1-467d-88f0-ad63c7b0e282">

This script manages the distribution of fees. It is responsible for the fair and accurate division of transaction fees amongst token holders, in line with the protocol's incentive structure.


## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.


-   2) Overall line coverage percentage provided by your tests : 100




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
</br>


-  2) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


<img width="780" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/1cad1505-dc78-4af4-be93-091fe3dba05e">


Ref:https://xin-xia.github.io/publication/icse194.pdf

</br>
</br>

-  3) Tests are written in Ts with Hardhat, I recommend tests written in Solidity with Foundry to increase coverage






-  5) Test Tools/Technologies analysis of the project;

<img width="986" alt="image" src="https://github.com/code-423n4/2024-01-curves/assets/104318932/f6e34315-13d1-414f-bf20-c583d5f0715d">



</br>
</br>
</br>
</br>

## d) Security Approach of the Project

### Successful current security understanding of the project;

1- They manage the 1nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.

7- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

8- ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise

9- We also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

10 - As the project team, you can consider applying the multi-stage audit model.

<img width="498" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/7ad49e46-4998-4bf2-830e-711039ea97a8">

Read more about the MPA model;
https://mpa.solodit.xyz/

11 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19

</br>
</br>

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-01-curves/blob/main/bot-report.md


</br>
</br>
</br>
</br>

## Centralization Risk

#### Owner Role:

Centrality Risk High: The Owner role is highly centralized since it is singular and possesses significant administrative powers. This includes critical functions like assigning the FeeSplitter and TokenFactory, setting protocol fees, and granting the Manager role.

Single Point of Failure: If the Owner's credentials are compromised, the entire system could be at risk. Similarly, if the Owner acts maliciously or negligently, it could have a substantial negative impact on the protocol.
Lack of Checks and Balances: With no apparent system for overseeing or limiting the Owner's decisions, there's a risk of unchecked power.
</br>

#### Manager Role:

Lower Centrality Risk (Current State): The risk is somewhat mitigated by the fact that there can be multiple Managers. This distribution of power can help prevent a single point of failure and can lead to more balanced decision-making.
Future Vision Reduces Risk Further: The plan to transition these responsibilities to a DAO indicates a move towards decentralization. If implemented, this would significantly reduce centrality risk by distributing governance power across a wider community, aligning more with the principles of decentralized systems.


Overall, the Owner role presents a significant centrality risk due to its high concentration of power and lack of oversight mechanisms. The Manager role, while currently more decentralized, still operates under the purview of the Owner. 

The transition to DAO-based governance for the Manager role is a positive step towards mitigating centrality risks and fostering a more resilient, community-driven governance structure.


</br>
</br>
</br>
</br>


##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)                         | A lot of OZ contracts |-  Version `4.9.3` is used by the project, it is recommended to use the newest version `5.0.1`                                                                                                                  |


</br>
</br>




## g) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ All staff accounts for the project should have control policies that require 2FA and must use 2FA wherever possible.
100% comprehensive security cannot be achieved based on smart contract codes alone.
Implement a more comprehensive policy to enforce non-SMS 2FA.
You can find the latest Simswap attack on Code4rena and details about it in this article: 
https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d


✅ I recommend you to set up a `system.sol` basic architecture where all contracts are registered.
The entire system can revolve around a single contract, like SystemRegistry. This is the contract that ties all the other contracts together, and from this contract we should be able to list all the other contracts in the system. It's almost like a registry. 



✅ Analyze the gas usage of each function under various conditions in tests to ensure efficiency and identify potential optimizations.	





✅  I recommend that you classify functions such as Public, External, Internal as follows, this is the most effective method to classify functions ; 
<img width="467" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/f6661c98-7595-4ae2-bc5b-b086dc4f3018">

https://x.com/PaulRBerg/status/1657098652987318292?s=20

</br>
</br>

✅  The `deploy()` function is triggered by `_deployERC20()` in the `Curves.sol` contract, but since there is no protection in the `deploy()` function, it can also be deployed by anyone, it is safer to have it run only by `Curves.sol`

When the `deploy()` function is called directly, a new token can be created without any preflight. This could cause the system to generate tokens in an uncontrolled manner, potentially disrupting the token economy.

If the `deploy()` function is run without sufficient symbol checking, multiple tokens using the same symbols may be created. This may mislead users and lead to the creation of fake or duplicate tokens.


```solidity
contracts/CurvesERC20Factory.sol:
   5  
   6: contract CurvesERC20Factory {
   7:     function deploy(string memory name, string memory symbol, address owner) public returns (address) {
   8:         CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);
   9:         return address(tokenContract);
  10:     }
  11: }
```

Recommended Mitigation Steps

Change the visibility of the deploy function from public to internal if it's only meant to be called from within the contract or its derivatives. This prevents external unauthorized access.

```solidity
contract CurvesERC20Factory {

    function deploy(string memory name, string memory symbol, address owner) internal returns (address) {
        CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);
        return address(tokenContract);
    }
}
```
</br>
</br>

✅  The `FeeSplitter.setCurves()` function can be run without any restrictions and the `curves` address can be updated, access control must be added, and during the execution of functions such as `balanceOf()` and `totalSupply()`, their working logic can be disrupted by running them from the front. 


 ```solidity
contracts/FeeSplitter.sol:
  34  
  35:     function setCurves(Curves curves_) public {
  36:         curves = curves_;
  37:     }
  38: 
  39:     function balanceOf(address token, address account) public view returns (uint256) {
  40:         return curves.curvesTokenBalance(token, account) * PRECISION;
  41:     }
  42: 
  43:     function totalSupply(address token) public view returns (uint256) {
  44:         //@dev: this is the amount of tokens that are not locked in the contract. The locked tokens are in the ERC20 contract
  45:         return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
  46:     }
 ```


Recommended Mitigation Steps

Add to `onlyManager` modifier;

```solidity
    function setCurves(Curves curves_) public onlyManager {
        curves = curves_;
    }
```



### Time spent:
24 hours