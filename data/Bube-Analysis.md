# ****🛠️ Curves Analysis

---

## Overview

---

The Curves is the first interoperable SocialFi protocol. Buy, sell and trade SocialFi tokens across a universe of dapps. It is an extension of `friend.tech`. The new feature of Curves is the possibility the users to transfer their tokens from the Curves protocol to the ERC20 format. Curves introduces a presale phase. This allows creators to manage and stabilize their tokens prior to public trading, ensuring a more controlled and equitable distribution. Curves also implements a referral fee and a fee distribution model designed to promote long-term holding over short-term trading. This model rewards token holders proportionally, thereby encouraging sustained investments in the ecosystem.

**The key contracts of Curves protocol for this Audit are**:

- Curves: This contract includes functionality for creating and trading ERC20 tokens with a bonding curve pricing mechanism, managing fees, and conducting presales with whitelisting through Merkle proofs.
- CurvesERC20: This contract inherits from OpenZeppelin's ERC20 and Ownable contracts. It is an ERC20 token with additional minting and burning functionalities that are restricted to the contract owner.
- CurvesERC20Factory: This contract is intended to deploy instances of another contract - CurvesERC20 contract. 
- FeeSplitter: This contract is designed to manage and distribute fees among token holders. The contract uses a Curves contract to determine the balances and total supply of tokens. The FeeSplitter contract inherits from a Security contract.
- Security: The role of this contract is to manage ownership and permissions for managers within the contract.

## Trusted Roles

- Owner: The role of the Owner is to designate the FeeSplitter and TokenFactory within Curves. Furthermore, the Owner holds exclusive rights to establish the protocol fee and is the sole entity authorized to confer the Manager role.
- Manager: Managers are granted the power to modify various fees, except for the protocol fee. Currently, the protocol accommodates multiple managers.

## Approach Taken-in Evaluating The Curves Protocol

---

| Number | Stage | Details | Information |
| --- | --- | --- | --- |
| 1 | Compile and Run Test | Installation | Well written and easy to setup. |
| 2 | Architecture Review | [friend.tech](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf) | Good description of the friend.tech system, which is the ground of the Curves protocol|
| 3 | Graphical Analysis | Graphical Analysis with [Solidity-Metrics](https://github.com/ConsenSys/solidity-metrics) | A visual view helps a lot with understanding the structure of the code in contracts of the project. |
| 4 | Slither Analysis | [Slither Report](https://github.com/crytic/slither) | Slither tool helped to mark the areas of the code which require more attention |
| 5 | Test Suits | Tests | The scope and content of the tests of the project are analyzed. The project uses Hardhat framework. Therefore, integration with Foundry was performed. And some tests in Foundry were written. |
| 6 | Manual Code Review | [Scope](https://github.com/code-423n4/2024-01-curves/tree/main?tab=readme-ov-file#scope) | Reviewed all the contracts in scope. |
| 7 | Known issues | [Known Issues](https://github.com/code-423n4/2024-01-curves/tree/main?tab=readme-ov-file#automated-findings--publicly-known-issues) | Observing the known issues and bot report. |

## New insights and learning from this audit

---

Learned about: 

- Idea behind friend.tech and Curves
- Different ERC20 vulnerabilities
- MerkleProof verification and the possible vulnerabilities

## Conclusion

---

The Curves protocol is well structured. If the codebase has more comments, it will be better. Also, there are some issues related to access control which can be critical for the protocol. The idea behind the protocol is interesting and it was pleasure to work on this audit. 


### Time spent:
30 hours

### Time spent:
30 hours