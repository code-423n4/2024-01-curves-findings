# Analysis Report

## Preface

This audit report should be approached with the following points in mind:

 - The report does not include repetitive documentation that the team is already aware of. It does include suggestions to provide more clarity on certain aspects in the documentation.
 - The report is crafted towards providing the sponsors with value such as unknown edge case scenarios, faulty developer assumptions and unnoticed architecture-level weak spots.
 - If there exists repetitive documentation (mainly in Mechanism Review), it is to provide the judge with more context on a specific high-level or in-depth scenario for ease of understandability.

## Approach taken in evaluating the codebase

### Time spent on this audit: 7 days

**Day 1:**
 - Getting a grasp over the friend.tech documentation
 - Reviewing smaller nSLOC contracts
 - Adding inline bookmarks for surface level issues

**Day 2-4:**
 - Reviewing Curves.sol and FeeSpliter.sol
 - Adding inline bookmarks to make notes, issues and imaginary attack scenarios
 - Reviewing added features in current codebase over friend.tech

**Day 5-7:**
 - Writing reports for simpler issues
 - Writing gas optimizations report
 - Filtering out inline bookmarks for invalid and QA level issues
 - Writing QA report
 - Writing reports for issues and testing their feasibility
 - Writing analysis report

## Architecture Recommendations

### What's unique?
  - Exporting ERC20 tokens - The SocialFi sector is not huge as of yet. By creating a market not only with shares but also exportable custom ERC20 tokens, the protocol provides users flexibility to use their funds elsewhere. For example: Uniswap pools or bridge lockers or yield bearing vaults
 - Presale/Opensale system - The presale and opensale system is quite common in the NFT and ERC20 token launch sectors. But implementing them for obtaining shares of a token subject allows a user to strengthen their relationship with followers. Additionally, it creates social value and reputation for a person based on social credit.
 - Re-integration of ERC20 tokens - Exporting tokens from the protocol is one half of the coin. The other half of reintegration is crucial since it allows users to leverage their strategies by earning ETH fees in the Curves protocol itself.

### How this codebase compares to Friend.tech
 - The first difference between both codebases is that there are two new fee percentages included, namely, the referralFeePercent and holderFeePercent.
 - The second difference is that the current protocol has optimized the friend tech code by creating logical functions such as _transferFees() to avoid repetitive code ([see how friend tech implements this](https://basescan.org/address/0xcf205808ed36593aa40a44f10c7f7c2f67d4a4d4#code)).
 - The third difference is the inclusion of fee rewards to token holders. Friend tech's daily active users plummeted from around 5000 to 500 in a span of few months. This is bad for friend tech considering that it is a socialFi protocol. The Curves team has resolved this issue by allowing users to earn fees on the shares they hold. This avoids creating Curves into a market for MEV extractors and tilts it towards a long term investment opportunity for users.

### What ideas could be incorporated?
 - Currently only integer units are allowed to be deposited through the deposit() function. This will cause the reintegration to fail if a user holds non-integer amounts of tokens. Since we cannot go break the conversion from the 18 decimals format in the deposit() function, a good straightforward solution to this would be to introduce a vault for the users by the Curves team. This vault can allow users to deposit their ERC20 curves tokens. Once the number of tokens for a token subject in the vault reaches a whole integer number, an administrator can call [sellExternalCurveTokens()](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L504) on the Curves contract which will give the vault ETH, that can then be distributed among the users who deposited their ERC20 tokens in it. The ETH would obviously need to be distributed based on the ratio or weight the users provide. There can be better solutions than this but in this simple solution above I've tried to ensure no new component other than a vault is introduced to aggregate the tokens to a whole integer number. 
 - Another possible idea that could be incorporated is to lock the first token as a NFT for the token subject who initiates the curve. Since the token is free to mint, it could use generative art to determine a protocol pfp or some kind of NFT to welcome the user to Curves.
 - A possible integration would that of ERC20 tokens. Currently payments are only accepted in ETH. If new tokens are added to the protocol, it can improve the user onboarding and provide users with more options to buy shares.
 - Introducing yield-bearing strategies would be a good feature as well since once a token is exported, there might not be much a user could do with it. Implementing some kind of liquidity pools and strategies can solve this issue for users though this requires considerable development effort.

## Centralization Risks

There are two centralization risks in the protocol:
1. The owner has control over the protocol fees and managers. This owner address should use a multisig to avoid malicious percentage changes in the fee mechanism especially.
2. The managers have control over how much fees the subject and referral must earn as well as the token holder fees.

## Resources used to gain deeper context on this codebase

1. [Form network resources](https://info.form.network/)
2. [Friend tech contract breakdown](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf)
3. [Friend tech contract itself to diff with existing Curves contract](https://basescan.org/address/0xcf205808ed36593aa40a44f10c7f7c2f67d4a4d4#code)

## Mechanism Review

### High Level System Overview

![Imgur](https://i.imgur.com/dSrQsRE.png)

### Inheritance Structure

Since there are only 5 contracts in scope, it is easy to get a grasp of the overall inheritance structure in the codebase. The contracts marked other than blue are out of scope.

![Imgur](https://i.imgur.com/J80GkZH.png)

### Features of Core contracts

![Imgur](https://i.imgur.com/ftlW4g2.png)

## Systemic Risks/Architecture weak spots and how they can be mitigated
 - Preventing users from buying - Currently a token subject can block users from selling shares and withdrawing their ETH. This is because the contract uses a push over pull mechanism that allows the subject to revert in its fallback.
 - Frontrunning/Sandwiching - The codebase is currently prone to MEV attacks due to missing slippage protection. This gives the user a worse execution price.
 - Address poisoning - Currently in the protocol, 0 value transfers are allowed. This means that when the frontend pulls in the recent transfers, an user might send tokens to the wrong address.
 - DOSing custom token names and symbol - The withdraw() functions allows passing 0 as a amount which allows an attacker to deploy an ERC20 token for the token subject but with default name and symbol. This might be against what the token subject wanted and should be fixed by disabling 0 value withdrawals.
 - Phishing attacks - The deploy() function in the CurvesERC20TokenFactory is public currently. This allows an attacker to create similar tokens with names and symbols but with different owners. This could be used to fool users through social engineering tactics in public communities.
 - Another important systemic risk is that the getFees() function correctly uses incorrect denominator of 1e18. This should be replaced with maxFeePercent since each of the protocol, subject, referral and holder fees are scaled according to the order of magnitude the maxFeePercent resides in. Therefore, if maxFeePercent is ever set to a value other than 1e18, it would cause users to be charged more or less fees.

### Time spent:
60 hours