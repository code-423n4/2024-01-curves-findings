# Advanced Analysis Report for [Curves](https://github.com/code-423n4/2024-01-curves)


## 1. Overview
####  The Curves protocol represents a groundbreaking extension of the Friend.tech ecosystem, ushering in a new era of innovation in the SocialFi space. Similar to its predecessor, Curves is reshaping the decentralized social networking landscape with a suite of pioneering features, all geared towards enhancing the user experience. Notably, Curves distinguishes itself by offering enhanced compatibility with decentralized finance (DeFi)(through external CurvesERC20 tokens), setting it apart from Friend.tech and further enriching the SocialFi ecosystem.





## 2. Architecture High Overview

![Alt text](https://i.imgur.com/1MMpSya.png)




## 3. The approach I followed when reviewing the code


1. **Friend Tech Review:**
   I initiated the review process by thoroughly examining the **Friend.tech** protocol. Given that **Curves** is a fork or extension of **Friend.tech**, understanding the foundational protocol was crucial. This involved a comprehensive review of the resources provided in the README page by the sponsors, including [Friend Tech Smart Contracts](https://basescan.org/address/0xcf205808ed36593aa40a44f10c7f7c2f67d4a4d4#code), [Friend Tech Smart Contract Breakdown](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf), and [Understanding Friend Tech through Smart Contracts](https://ada-d.medium.com/understanding-friend-tech-through-smart-contracts).
 

2. **Setup the Codebase:**
   The next step involved setting up the codebase, which included compiling the code, installing dependencies, and running all available tests.

3. **Evaluating the Codebase:**
   A meticulous examination of the entire codebase followed, with a line-by-line review of each contract within scope. The focus was on understanding the contract hierarchy and overall structure to identify key components of the protocol. Particular attention was given to discerning the distinctions between Curves and Friend.tech.

4. **Functionality Understanding:**
   The review delved into comprehending the functionalities embedded in each contract. Key functions such as `buyCurvesToken`, `sellCurvesToken`, `withdraw`, and `deposit` were scrutinized for their significance. Additionally, the functionality of the presale mechanism (`buyCurvesTokenForPresale`) and the process for creating new subjectTokens, including the interaction with the `CurvesERC20` contract, were thoroughly examined.


5. Analysis of the Main Components

### 1. `Curves.sol`:

The `Curves.sol` contract serves as the core component of the protocol, responsible for crucial functionalities such as creating new subjectTokens, trading existing tokens, locking external tokens (ERC20 subjectTokens), unlocking subjectTokens, and managing the curve formula. The schematic representation of its structure is illustrated below:

![Curves.sol Structure](https://i.imgur.com/H5LO5Ku.png)

### 2. `FeeSplitter.sol`:

The `FeeSplitter.sol` contract plays a pivotal role in the protocol by managing the distribution of holder fees among token holders. The schematic representation of its structure is illustrated below:

![FeeSplitter.sol Structure](https://i.imgur.com/SYlwJZg.png)




## 4. Mechanism Review

One of the critical functionalities within the protocol is the issuance of new subjectTokens, which includes the option to buy existing tokens. There are multiple options for subjects to create new subjectTokens:

**a.** Create a new locked subjectToken (internal subjectToken) with a presale, where the merkle root must be provided. This is achieved using the `buyCurvesTokenForPresale` function. Subsequently, normal users can purchase these subjectTokens through the `buyCurvesTokenWhitelisted` function.

**b.** Create a new locked subjectToken (internal subjectToken) without a presale using the `buyCurvesToken` function.

![Option a and b](https://i.imgur.com/efN45em.png)

**c.** Alternatively, another approach to create external subjectTokens (CurvesERC20 tokens) is available if needed. This is accomplished using the `buyCurvesTokenWithName` function.

![Option c](https://i.imgur.com/0uLdgqY.png)



### Comparison between Curves and Friend.tech Protocols
| Feature                  | Curves Protocol                                      | Friend.tech Protocol                      |
|--------------------------|------------------------------------------------------|--------------------------------------------|
| **Overview**             | extension (fork) of Friend.tech               | Base protocol                              |
| **Referral Fee Implementation** | Enables protocols to earn a percentage of all user transaction fees, benefiting both the base protocol and its derivative platforms | Feature not present                      |
| **Presale Feature**      | Incorporates a presale phase for controlled and equitable token distribution | Token launches without a presale         |
| **Price Formula**     | the same formula for both |the same formula for both                      |
| **Token Holder Fee**     | Introduces fee distribution model for token holders, encouraging long-term holding | Feature not present                      |


## 5. Architecture Recommendations

### Enhance Documentation Quality

during my review, I identified a few areas where documentation enhancements could significantly improve the overall quality and clarity of the codebase. It's important to note that comprehensive documentation, including code comments (Natspec), plays a crucial role in not only providing clarity but also facilitating a smoother onboarding process for new developers(and auditors).

I understand that establishing external documentation and code comments might be a significant undertaking, but these enhancements can greatly benefit the Curves protocol community.

### Interface Standardize

Standardizing interfaces across different Contracts can simplify integrations and enhance interoperability (like in case of external referral integrations). By creating a common interface for each contract, we can ensure consistency in function names and parameters.

### Test improvments 



The existing test structure for Curves Protocol is of high quality, showcasing the developers proficiency. 
- Strengthen the test suite for Curves Protocol to cover various scenarios, including Fuzz testing, invariant testing,

- To enhance clarity and effectiveness,consider add end to end test for each scenario (sell external tokens starting the test from create new external Tokens).


### State Variables isolation
I would like to suggest a refinement in the storage management of external tokens,to enhance clarity and maintainability by isolating the state variables related to external tokens.

### Function Naming for Token Issuance:
The `buyCurvesToken` functions are currently responsible for both issuing new subject tokens and buying existing tokens. To avoid ambiguity and enhance clarity, consider differentiating the function names based on their primary purpose. For example:

`buyNewCurvesToken` for issuing new subject tokens.

`buyExistingCurvesToken` for buying existing tokens.

This distinction will make the code more explicit and reduce the potential for confusion.



## Time Spent: 
~ 25 hours 



### Time spent:
25 hours