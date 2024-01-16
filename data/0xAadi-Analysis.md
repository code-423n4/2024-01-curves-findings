# Analysis Report - Curves

## Overview
The `Curves` protocol represents an enhanced iteration of `friend.tech`, featuring upgrades that significantly amplify its capabilities. Notable enhancements include the ability to export tokens to the ERC20 format, facilitating external trading beyond the Curves platform. 

Moreover, the implementation of a referral fee system empowers users to earn fees, reminiscent of royalties, whenever their tokens are traded within the Curves ecosystem. The introduction of a Presale feature serves as a strategic measure to combat front-running attacks during token launches, ensuring a secure and equitable distribution. 

Additionally, the incorporation of a token holder fee mechanism ensures that holders receive a portion of transaction fees, fostering a symbiotic relationship between the platform's success and the interests of its community.

## Contracts
The pivotal contracts in this Audit for the protocol are:
- **contracts/Curves.sol:** `Curves.sol` serves as the cornerstone contract for the entire ecosystem. Leveraging `friend.tech`'s foundational functions, it encompasses crucial aspects such as price calculation and forms the bedrock of the main business logic. The contract seamlessly integrates FeesEconomics, orchestrates the intricacies of buying, selling, presales, conversion to ERC20 (external curve tokens), and their subsequent conversion back. Moreover, it facilitates the selling, transferring, and migration of all tokens to another address.
- **contracts/CurvesERC20.sol:** This ERC20 contract is specifically crafted to issue external curve tokens. Token issuers can employ this contract via the Curves contract, enabling users to engage in trading activities with these ERC20 tokens. As an Ownable contract deployed by the Curves contract, it facilitates the minting or burning of tokens in response to users depositing or withdrawing external curve tokens.
- **contracts/CurvesERC20Factory.sol:** This contract deploys instances of CurvesERC20 contract.
- **contracts/FeeSplitter.sol:** This contract plays a pivotal role as the second essential component, dedicated to distributing the holder fee among curve token holders. It efficiently allocates the holder fee, a contribution provided by the Curve contract during user trades on the curve contract. Importantly, this contract facilitates token holders in claiming their entitled fee in the native currency.
- **contracts/Security.sol:** This contract, collaboratively implemented by the Curves contract and the FeeSplitter contract, introduces fundamental security features such as contract owner setups and accessible manager setups. Essential functionalities embedded in this contract include actions like transferring ownership, adding managers, and incorporating basic access modifiers.

## Auditing Approach
In the process of auditing the smart contracts, I scrutinized the deployed contracts of [friend.tech](https://basescan.org/address/0xcf205808ed36593aa40a44f10c7f7c2f67d4a4d4#code#L1) and extensively studied the code breakdown documents [1](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf) and [2](https://ada-d.medium.com/understanding-friend-tech-through-smart-contracts-edac5d98cd49). My auditing approach involved a meticulous examination of the code base, test files, conducting a comprehensive code review focusing on solidity and security. To ensure the contracts' functionality and resilience, I actively compiled the code and tested various scenarios. This detailed analysis enabled me to evaluate the contracts' performance and security measures in diverse scenarios. Additionally, I verified the main invariants, explored attack ideas presented by the protocol, and considered additional contextual information as part of my thorough audit approach.

## Codebase Quality Analysis
The codebase exhibits a moderate level of quality by delivering clarity in functionality through self-explanatory code, employing simple functionalities and logic. However, its readability is somewhat compromised due to the absence of well-documented comments. Despite handling complex calculations, the code is inherently explanatory and moderately understandable for both auditors and developers.

The absence of essential documentation and inline comments in the code has resulted in an increased learning curve and auditing time.

### Modularity
The codebase demonstrates a high level of modularity, effectively segregating complex additional logics into separate functions, particularly with the holder fee calculations. The use of distinct contracts to manage these aspects contributes to reducing overall complexity. Additionally, the presence of separate contract files for `CurvesERC20`, `CurvesERC20Factory`, and `Security` enhances the code's modularity.

### Comments
Insufficient comments throughout the contracts limit the clarity of each function, requiring auditors and developers to invest more time in understanding the functionality. Improving comments to thoroughly explain the various functionalities would greatly enhance the code's accessibility and comprehension.

### Access Controls
Access control is effectively implemented throughout the codebase, ensuring smooth and conflict-free execution of functions. However, a critical oversight is identified in the `setCurves()` function within `FeeSplitter.sol`, where a crucial access control check is missing. This vulnerability potentially allows any user to set the curve contract in FeeSplitter, posing a severe threat. 

Exploiting this loophole, an attacker could perpetually disrupt the platform and steal all fees from the FeeSplitter contract.

### Events
All user-interacting functions exposed by both the `Curves` contract and the `FeeSplitter` contract emit necessary events. These events are appropriately indexed, ensuring efficient event log searches and retrievals. However, it's important to note that the `Trade` event in the `Curves` contract does not currently have proper indexing, and this aspect should be addressed for improved event handling and analysis.

### Error Handling
The contracts demonstrate effective error handling, with particular attention to the `Curves` and `FeeSplitter` contracts. Notably, the `Curves` contract employs an interface named `CurvesErrors` to define various errors. To enhance codebase, it is recommended to use a dedicated library contract to store these errors instead of utilizing an interface within the Curves contract. This approach improves code structure and maintainability.

### Interfaces
The codebase currently lacks efficiency in the utilization of interfaces. Instead of importing interfaces, the contracts import the entire contract, thereby unnecessarily inflating the contract size. Specifically, the `Curves` contract imports `CurvesERC20`, `CurvesERC20Factory`, and `FeeSplitter` contracts. To enhance the codebase, it is advisable to use the interfaces of these contracts for improved efficiency and code cleanliness.

### Libraries
The contracts currently do not incorporate any libraries. However, it is advisable to consider using a separate library to encapsulate the errors defined in the Curves contract. This practice can enhance modularity and maintainability within the codebase.

## Mechanism Review
### Creating Curve Tokens (Curve Token Subjects)
Curve tokens are created by a token subject, which is typically the address that initiates the creation process. The creation is done through functions like `buyCurvesTokenWithName` or `buyCurvesTokenForPresale`. These functions ensure that a token can only be created once per token subject by checking the supply of the token associated with the subject's address. If the supply is non-zero, it indicates that the token has already been created.

### Price Calculation
The price for buying and selling Curve tokens is determined by a bonding curve formula, which is a mathematical curve that defines the relationship between the token's price and its supply. The `getPrice` function calculates the `price` based on the current `supply` and the `amount` being bought or sold. The price typically increases with `supply` for purchases and decreases with `supply` for sales. The `getBuyPrice` and `getSellPrice` functions provide the `price` for buying and selling tokens, respectively, and include fee calculations.

### Depositing and Withdrawing External Curve Tokens
Depositing and withdrawing external Curve tokens involve interacting with an ERC20 token contract that represents the Curve tokens on external platforms. The `deposit` function allows users to deposit their external Curve tokens into the contract by burning the ERC20 tokens and crediting their internal balance. The `withdraw` function allows users to withdraw by minting new ERC20 tokens and debiting their internal balance.

When depositing or withdrawing, the contract must accurately update the user's balance and potentially adjust the fee distribution since the user's share of the total supply has changed. The FeeSplitter contract is used to manage the distribution of fees among token holders, ensuring that fees are fairly allocated based on token ownership over time.

### Fee Distribution Among Holders
The `FeeSplitter` contract is designed to distribute fees among token holders. The mechanism for fee distribution is based on the token balances of the holders and the cumulative fees collected by the contract.

1. **Fee Collection**: Fees are collected by the `FeeSplitter` contract through the `addFees` function, which is payable and can receive Ether. The amount of Ether received is considered the fee to be distributed.

2. **Cumulative Fee Calculation**: When fees are added, the contract calculates a `cumulativeFeePerToken` by dividing the total received fees by the `totalSupply` of the tokens (excluding any tokens locked in the contract itself). This results in an average fee per token that is used for distribution.

3. **Holder Balance Tracking**: The contract maintains a mapping of `userFeeOffset` for each holder, which tracks the cumulative fee per token at the time of the user's last balance change.

4. **Unclaimed Fee Accounting**: The contract also tracks `unclaimedFees` for each holder, representing the amount of fees that have been accrued but not yet claimed by the holder.

5. **Fee Credit Updates**: When a holder's balance changes (due to a buy or sell operation), the `onBalanceChange` function is called to update the holder's fee credit. This involves calculating any owed fees based on the difference between the current `cumulativeFeePerToken` and the holder's `userFeeOffset`, and updating the `unclaimedFees`.

6. **Claiming Fees**: Holders can claim their accumulated fees through the `claimFees` function. This function updates the holder's fee credit and then transfers the claimable amount to the holder's address.

## Critical Issues

### Access Control
The `setCurves` function lacks access control, allowing any entity to change the `Curves` contract address. This critical vulnerability could disrupt fee distribution and potentially lead to a denial of service, impacting the entire platform. Implementing proper access control for the `setCurves` function is crucial to prevent unauthorized changes and ensure the security of the protocol.

### Double-Counting of Fees
The `claimFees` function may lead to double-counting of fees. This occurs due to the sequential calling of `updateFeeCredit` and `getClaimableFees`, potentially resulting in an inaccurate calculation of owed fees. Adjusting the `claimFees` function to prevent double-counting by utilizing the updated `unclaimedFees` directly after calling `updateFeeCredit` is recommended to ensure accurate fee distribution.

### Unfair Fee Distribution to Buyers
The `FeeSplitter` contract may allow buyers to receive a portion of `fees` from their current purchase. This issue arises due to the order of operations in the `_buyCurvesToken()` function in the `Curves` contract, which updates the token supply before transferring fees. Modifying the fee distribution functions in both the Curves and FeeSplitter contracts is necessary to ensure that fees are distributed based on the actual balance of the user and calculated based on the real supply rather than the inflated or reduced supply after a trade.

### Incorrect Fee Distribution to Sellers
The `sellCurvesToken()` function in the `Curves` contract incorrectly calculates fees based on the total supply after the sale, rather than before. This results in sellers receiving fewer fees than they are entitled to. Adjusting the fee distribution mechanism in the `sellCurvesToken()` function to calculate fees based on the real supply before the trade is essential for fair and accurate fee distribution to sellers.

### Loss of Unclaimed Fees while Performing Purchase or Sales
The `onBalanceChange()` function in the `FeeSplitter` contract incorrectly updates the user's fee offset to the current `cumulativeFeePerToken` value. This can lead to the loss of unclaimed fees for users who have not yet claimed their accrued fees. Modifying the `onBalanceChange()` function to calculate the cumulative fee per token based on the previous supply is necessary to prevent the loss of unclaimed fees during user transactions.

## Recommendations
- Implement proper access control for the `setCurves` function to prevent unauthorized changes to the `Curves` contract address.
- Adjust the `claimFees` function to prevent double-counting of fees by using the updated `unclaimedFees` directly after calling `updateFeeCredit`.
- Modify the fee distribution functions in both the Curves and FeeSplitter contracts to ensure that fees are distributed based on the actual balance of the user and calculated based on the real supply rather than the inflated or reduced supply after a trade.
- Adjust the fee distribution mechanism in the `sellCurvesToken()` function to calculate fees based on the real supply before the trade to ensure fair and accurate fee distribution to sellers.
- Modify the `onBalanceChange()` function in the `FeeSplitter` contract to calculate the cumulative fee per token based on the previous supply to prevent the loss of unclaimed fees during user transactions.

## Centralisation Risk

The Curves protocol relies on the Owner role in key contracts, including Curves, CurvesERC20, and FeeSplitter. This role is crucial for tasks such as setting fee distributors, configuring fee percentages, defining the factory contract in Curves, specifying the curve contract address in FeeSplitter, and managing mint and burn functions in CurvesERC20. It's noteworthy that all these contracts utilize OpenZeppelin's Ownable contract, introducing a potential vulnerability related to centralization.

### Recommendations

To mitigate this risk, it is strongly recommended to implement OpenZeppelin's Ownable2Step contract. This step enhances security by addressing centralization concerns in the protocol.

## Systemic Risk

### Lack of Upgradability Provisions

The contract lacks a structured approach to upgradability. Without provisions for upgradability, deploying updates to address vulnerabilities or introduce improvements could pose challenges. Considering the integration of upgradability features is advisable to facilitate future enhancements and fixes.

### Absence of Pausability Feature

The contract currently lacks a pausability mechanism, a critical feature for emergency halting of contract functions in response to vulnerabilities or attacks. Introducing a pausability feature would enable a swift response to protect user funds and maintain system integrity during unforeseen events.

### Incorrect Fee Distribution

As discussed earlier, if the fee distribution mechanism does not accurately account for the timing of balance changes, it could lead to systemic unfairness in fee allocation. This could undermine the integrity of the token economy and erode user trust.


### Time spent:
40 hours