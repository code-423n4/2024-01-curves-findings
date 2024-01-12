# Overview

This protocol is a fork of friend.tech with added functionalities, including presale features, holder fee, and referral fee. It also added to enable conversion of purchased tokens into ERC20 for trading on exchanges. The share (or key) of friend.tech has been renamed to curve or curve token. The protocol is deployed on the L2 [form](https://docs.form.network/) chain, which is based on OPStack.

friend.tech is a social platform that allows users to follow influencers by purchasing their tokens. The individuals being followed are called subjects. By purchasing a subject's token, users can interact with them through chat rooms.

The price of these tokens is calculated individually for each subject, increasing as the supply of a subject's token increases. Early buyers can purchase tokens at a lower price and can sell them back to the protocol once the price has risen. The price of a subject's token falls as the supply decreases when tokens are sold.

There are protocol, subject, holder, and referral fees. The protocol fee goes to the protocol operator, the subject fee to the subject, and the holder fee to users who own the subject's curve token. If the subject has set a referral fee destination, the referral fee is sent there, but if not, it is added to the protocol fee.

![Contract Architecture](https://gist.github.com/assets/70058709/4506b117-3032-4300-9bb6-26e8b638c659)

If ERC20 does not exist when a subject requests an ERC20 deployment or a user requests a withdrawal, an ERC20 contract is deployed and allocated for each subject. Once the ERC20 contract deployed, user can call the `withdraw` function to change the curve token into ERC20. The converted curve tokens are locked in the Curves contract. User can call `deposit` to burn the ERC20 and get back the curve tokens that were locked in the Curves contract. Only curve token holders(not ERC20 holder) can receive holder fees.

# Approach taken in evaluating the codebase

| Stage | Detail |
| --- | --- |
| Compile and run test | Clone the repository, install dependencies, set environment and run test. |
| Check scope and sort | List the scopes, sorting them in ascending order. Identify core and peripheral logic. |
| Read docs | Read code4rena README and docs. Research on who’s sponsors and what’s their goal. Review the sponsor's existing projects, if any. |
| Understand core logic | Skim through the code to understand the core logic. |
| Manuel Code Review | Read the code line by line to understand the logic. Find bugs due to mistakes. |
| Organize logic with writing/drawing | Organize business logic and look for logic bugs. |
| Write report, PoC | Write the report and make PoC by using test code. |

# Architecture recommendations

Since most of the core logic is taken directly from a fork of friend.tech, it shares the same weaknesses. friend.tech's key price formation is based on a very steep bonding curve model. This structure maximizes key price volatility due to buying and selling. This makes them vulnerable to market manipulation.

![price graph](https://gist.github.com/assets/70058709/5b301437-1820-48fa-8868-57e510f946eb)

Source: [https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf)

# Centralization risks

Instead of using libraries like Ownable or AccessControl, a Security contract is written to handle access control. However, the implementation of the Security contract is flawed, failing to check permissions in `onlyOwner` and `onlyManager`. This makes the protocol's overall permission verification feature inoperative so this needs to be fixed as a priority.

```solidity
modifier onlyOwner() {
@>  msg.sender == owner;
    _;
}

modifier onlyManager() {
@>  managers[msg.sender] == true;
    _;
}
```

The owner is the protocol's administrator who can set managers, protocol fee, protocol fee recipients, etc. It is recommended to use a multi-sig wallet for the owner account to mitigate centralization issues. Also, using [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.3/contracts/access/Ownable2Step.sol) can prevent misconfiguring the owner.

The Manager can determine the maximum fee rate and the subject, referral, and holder fees. Currently, the owner can add or remove manager, so the owner is considered to have the same authority as the manager. It is recommended to set the manager as governance and to prevent the owner from changing the manager to mitigate centralization issues.

# Mechanism review

## Curves

This is the core logic that allows users to parchase, sell, transfer, withdraw to ERC20, or deposit ERC20. When a token is bought/sold, protocol, subject, holder, and referral fees are distributed. Each subject can request the deployment of their ERC20, and also, if there is no ERC20 at the time of withdrawal, the user can deploy instead of subject.

The first token is purchased by the subject, and the first one can be purchased for free. The price of each token is `(k-1)^2 * 1e18 / 16000`, (k is the purchase order), which increases as purchases are made later. When a holder sells a token, the totalSupply decreases and the k moves forward, leading to a relative price drop.

## CurvesERC20

This is the ERC20 contract matching each subject's curve token. Each subject is assigned one ERC20 contract. Only the Curves contract can mint and burn, and the decimal is fixed at 18.

## CurvesERC20Factory

This is the contract that deploys the CurvesERC20 contract. Anyone can call `deploy`, but to register as a subject's ERC20, it must be called through the Curves contract.

## FeeSplitter

This is the contract that distributes the holder fee to the holders. It is implemented using the accumulator pattern. Users have to call the `claimFees` function to receive tokens. To receive a fee, user must be held in the form of curve token inside the Curves contract, not in ERC20 form.

## Security

This is the contract responsible for access control. It is a simple version of Ownable and AccessControl. There is only one owner, and multiple managers can be registered. The owner can add or remove managers.

### Time spent:
17 hours