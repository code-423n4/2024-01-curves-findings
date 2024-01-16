#  **Advanced Analysis Report for Curves** 

[Audit approach](#1-audit-approach)

[Scope and Architecture Overview](#2-scope-and-architecture-overview)

[Roles](#3-roles)

[Codebase Overview](#4-codebase-overview)

[Centralization Risks](#5-centralization-risks)

[Systemic Risks](#6-systemic-risks)

[Recommendations](#7-recommendations)

[Conclusions](#8-conclusions)

[Resources](#9-resources)


## **1. Audit approach**

**Phase 1**: Protocol Familiarization: The review began by analyzing the protocol documentation, including the readMe and Friend Tech blogs, followed by a preliminary examination of the relevant contracts and their tests.

**Phase 2**: Deep Contract Review: A meticulous, line-by-line review of the in-scope contracts was conducted, meticulously following the expected interaction flow.

**Phase 3**: Issue Discussion and Resolution: Potential pitfalls identified during the review were discussed with the developers to clarify whether they were genuine issues or intentional features of the protocol.

**Phase 4**: Reporting: Audit reports were finalized based on the findings.

***

## **2. Scope and Architecture Overview**

#### **[Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol)** 

- Curves.sol acts as the control center of the Curves protocol.

- It's built on a foundation of code borrowed from friend.tech but has its own unique twists and adjustments.

- They both share a common mathematical approach for certain tasks, which is conveniently stored within the `getPrice` method.

**`Curve` purchase and sale**: Users who interact with the protocol are able to buy or sell `Curves` tokens of their choice by calling the `buyCurvesToken` and the `sellCurvesToken`. The price is calculated using the `getPrice` model. Important to note that fees are charged on token trade. Users also have the option of a merkle "buy with whitlelist" or "buy for presale", to protect from malicious frontrunning during token lauches.

**`CurveERC20` deployment, deposit and withdrawal**: Through the `_deployERC20` function, the CurveERC20 can be deployed with a user specified name and symbol, or the protocol default name and symbol - Curve. These ERC20s are 18 decimals and can be exchanged at a 1:1 ratio with the Curves, through deposits or withdrawal.

**Fee distribution**: A portion of the transaction cost is resevered as fees for the protocol, the curve owner or subject, the referral fee and the holder's fee. These fees are distributed to the required addresses upon transaction completion, and the holder's fee is sent to the `FeeSplitter` contract, from which they can be claimed.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/296076912-a7130297-a325-4147-9ecf-29bbef691909.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240116T172913Z&X-Amz-Expires=300&X-Amz-Signature=cc134006656ce51404c4428bd34eafd792a35bfb5e8ef4f88bc941b03a08e944&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="Curves"> sLOC - 413
</p>

#### **[FeeSplitter.sol:](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol)**

- FeeSplitter.sol cts as the backbone of a fair and transparent mechanism that rewards token holders with a portion of the fees generated from purchase and sales of curves
 
- It encourages long-term commitment from its community through a unique fee distribution system.
 
**Fee claiming**: By calling the `claimFees` and the `batchClaiming` functions, loyal `Curve` holders can claim their portion of the holder's fee which they're entitled to. 

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/296076755-6b5b607b-2db9-4803-8522-0c93b41a5ed9.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240116T172730Z&X-Amz-Expires=300&X-Amz-Signature=0d2461ebce431c962a480a4ba7d356a21ecac66623d96dcbf7966b58caa3bf41&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="Feesplitter">
sLOC - 95
</p>


#### **[Security.sol:](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol)**
- Security.sol is the protocol's digital security guard, enforcing a strict set of rules to keep things safe and compliant. 
- The contract plays a crucial role in maintaining the integrity of the protocol and acts as a blueprint for a secure system, covering the core functions.

<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/296076998-a6c55fda-aca4-459e-b403-076dd75c4e5e.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240116T172955Z&X-Amz-Expires=300&X-Amz-Signature=f6ae922a207b6aac14355ddca42c9ba2d170f3afc0ac275c1bb295729dd013dd&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="Security"> sLOC - 23
</p>


#### **[CurvesERC20Factory.sol:](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol)**
- CurvesERC20Factory.sol centralizes ERC20 token creation logic to optimize efficiency and scalability within the protocol.
**Deployment**: The ERC20 tokens are deployed using the create opcode which computes the token address as a function of the sender’s own address and a nonce.
<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/296077426-bb7410e1-88e1-459e-8e40-df776b2edb30.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240116T173059Z&X-Amz-Expires=300&X-Amz-Signature=28f0adabc69c5ce9fefa95256f328ee6d6d1706f0dabcc8bfce5496ea9739020&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="CurvesERC20Factory"> sLOC - 8
</p>



#### **[CurvesERC20.sol:](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol)**
- CurvesERC20.sol acts as a blueprint for creating compatible ERC20 tokens when Curve tokens are exported out of the protocol.
**Minting and burning**: The tokens can only be minted/burned by users upon calling the `withdraw`/`deposit` functions in the `Curves` contract respectively. It is therefore like a swap between the internal protocol Curve and the external Curve ERC20 at a ratio of 1 to 1.  
<p align="center">
    <img width= auto src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/296077340-4a9f7554-f2eb-4ca2-85a3-fe2eb7114331.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240116T173021Z&X-Amz-Expires=300&X-Amz-Signature=94d94f06ade0e6b348765172dceef8c711f436e9bf5ca18568dcb45ea0f7e74c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="CurvesERC20">sLOC - 14
</p>


***
## **3. Roles**

**Owner** controls core Curves functions: assigning FeeSplitter/TokenFactory, setting fees, and granting Manager roles.

**Manager** handles fee adjustments(excluding the protocol fee). This centralized approach is intended as a stopgap measure, with the long-term objective being a transition to a DAO-based governance model, thereby promoting community engagement and decentralized decision-making.

**Subject** is the user who creates a new Curve Token. This is done by purchasing the first token. The internal identifier of the token is the same as the address of its creator, which is why all maps are indexed by the token subject. A Token Subject can own a unique token of their own, although they can possess any number of units of the same token. They can also buy tokens from other people.

***
## **4. Codebase Overview**

**Audit Information** - For the purpose of the security review, the Curve Protocol consists of five smart contracts totaling 660 SLoC. Its core design principle is composition, enabling efficient and flexible integration. It is scheduled to be deployed on the Form chain, an Ethereum L2 built to advance the SocialFi ecosystem, giving each user the freedom to transact in and across all platforms. It operates independently of oracles and sidechains and represents an upgrade of the tech.finance Friend contracts. The tech.finance mathematical model is used.

**Documentation and NatSpec** - The codebase has no officical documentation. It relies on the similar albeit outdated FriendsTech medium articles, which doesn't do enough in explaining the nitty gritty. The contracts are also barely commented, and basically lack NatSpec for the most part. This made the audit a bit more challinging.

**Testability** - While at first glance, the codebase appears to be well tested, it quickly falls apart upon scruitiny. The implemented tests are mosty simple unit tests. The more complex parts of the codebase are also not fuzz tested.

**Gas Optimization** - The codebase isn't very well optimized for gas usage. A number of basic gas saving techniques were ignored. No `Unchecked` in loops and subtractions, an older non gas efficient solidity version is used, state variables that could be marked internal or private were made public, via-ir is not enabled in deployment, and so on. That being said, custom errors which are more gas efficient were used in place of revert strings, which is commendable.

**Error handling and Event Emission** -  Custom errors were used in the codebase in place of require/assert errors. Events are well handled, emitted for important parameter changes, although in some cases, they seemed to not follow the checks-effects-interaction patterns. 

Other basic safety measures, two step address updates, protected fallbaack functions, checked loops, zero address checks etc were also not implemented.

***

## **5. Centralization Risks** 
As with any protocol that incorporates admin functions, the protocol is also vulnerable to actions of a malicious admin. 
 - Owners and managers can set fees extremely high to grief users;
 - The owner can mint or burn CurveERC20 tokens arbitrarily, which destabilizes the intended 1 to 1 ratio of the Curve tokens; and so on

***
## **6. Systemic Risks** 

- The system does not impose any caps on the number of curves that can be created, potentially leading to scalability challenges if a substantial number of curves are integrated into the system.
- The protocol price increase on supply increase fee model might be discouraging to potential users.
- Lack of sweep function and the existence of a fallback function in the FeeSplitter contract will cause a loss of any mistakenly sent ETH;
- There are no timelocks or pause protocols in place in case of emergency market conditions, and so on;

***
## **7. Recommendations**

- The codebase should be sanitized, and commented to make it easier to understand. In that aspect, a well written gitbook document also helps. The one currently relied on is for a different codebase and doesn't have a number of the current codebase's functions. Additionally, more information should be included about the Forms Network where the contracts will be deployed to;

- Testing should be improved, including invariant and fuzzing tests;

- The token name and symbol setter should be sanitized to prevent malicious code injections;

- A refund/sweep function should be introduced for cases in which excess tokens are sent into the contracts;

- Create2/Create3 should be used in place of basic create, as they're safer and less vulnerable to frontrunning or reorgs;

- More libraries should be imported to help basic cotract functions. For isntance, the `Security` contract can be replaced by `Ownable2Step`;

- Solidity and OpenZeppelin contract versions should be updated as they provide lots of benefits in security and optimization;
***
## **8. Conclusions**

In general, the codebase is compact, of small size and well-designed which is commendable, but identified risks need to be fixed. Recommended measures should be implemented to protect the protocol from potential attacks. Timely audits and sanitizations should be conducted to keep the codebase fresh and up to date with evolving security times.
***
## **9. Resources**

- [C4 ReadMe](https://code4rena.com/audits/2024-01-curves#top)
- [FriendTech 1](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf)
- [FriendTech 2](https://ada-d.medium.com/understanding-friend-tech-through-smart-contracts-edac5d98cd49)
- [Forms Network Chain](https://docs.form.network/)



### Time spent:
18 hours