# QA Report

| QA issues | Issues                                                                                            |
|-----------|---------------------------------------------------------------------------------------------------|
| [L-01]    | FeeSplitter `onBalanceChange()` function will push duplicate tokens to `userTokens[account]`.     |
| [L-02]    | `_buyCurvesToken()` function does not return excess Ether.                                        |
| [L-03]    | The tests are too simple, especially for holder fees.                                             |
| [N-01]    | Add in code comments and documentation that the initial buy amount must be 1.                     |
| [N-02]    | `buyCurvesTokenWithName` and `buyCurvesTokenForPresale` code style improvement.                   |

## [L-01] FeeSplitter `onBalanceChange()` function will push duplicate tokens to `userTokens[account]`.

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L247
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L96

In the Curve protocol, the FeeSplitter's `onBalanceChange()` function is activated with each buy or sell transaction. This function repeatedly adds the token to `userTokens[account]`, regardless of whether it's a duplicate. As a result, each transaction involving the same token will cause an increase in `userTokens[account]`, leading to additional time and space consumption.

```solidity
function onBalanceChange(address token, address account) public onlyManager {
    TokenData storage data = tokensData[token];
    data.userFeeOffset[account] = data.cumulativeFeePerToken;
    if (balanceOf(token, account) > 0) userTokens[account].push(token);
}
```

Solution: Check for token existance before pushing. Better to use a map for keeping track of token existance.

## [L-02] `_buyCurvesToken()` function does not return excess Ether.

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L263-L280

In the Curve protocol, if a user sends more Ether than needed for a token purchase, the surplus is not refunded but remains within the protocol. This can be problematic, particularly for users trying to avoid front-running by sending extra Ether.

Additionally, the token buying function requires that `msg.value >= price + totalFee`. However, there's a potential issue in fee calculation by the `getFees()` function. For instance, if `feesEconomics.holdersFeePercent` is set without a corresponding `feeRedistributor`, the fees wouldn't be distributed to holders but instead get locked in the contract.

```solidity
function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

    uint256 price = getPrice(supply, amount);
    (, , , , uint256 totalFee) = getFees(price);

    if (msg.value < price + totalFee) revert InsufficientPayment();

    curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
    curvesTokenSupply[curvesTokenSubject] = supply + amount;
    _transferFees(curvesTokenSubject, true, price, amount, supply);

    // If is the first token bought, add to the list of owned tokens
    if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
        _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
    }
}
```
```solidity
function getFees(
    uint256 price
)
    public
    view
    returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
{
    protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
    subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
    referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
    holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
    totalFee = protocolFee + subjectFee + referralFee + holdersFee;
}
```

## [L-03] The tests are too simple, especially for holder fees.

The handling of the holder fee in the system is complex and prone to errors, yet it lacks comprehensive unit tests. The existing tests use a `mockCurveToken` instead of the actual Curve contract. This has led to a high-severity issue where the holder fee implementation is flawed (submitted independently as a high severity issue).

A simple unit test to demonstrate this error is when a user makes two separate buy transactions – for example, buying 10 tokens in one and 1 token in another. In such a scenario, the user's holder fee decreases, even though they haven't claimed the fee.

The suggestion here is to write more high quality unit tests.

```js
import { expect } from "chai";
import { ethers } from "hardhat";
import { FeeSplitter__factory } from "../contracts/types";
import { deployCurveContracts } from "./test.helpers";

it("Holder fee is incorrectly implemented.", async () => {
  // Setup owners and testContract.
  const [owner, ...addrs] = await ethers.getSigners();
  const testContract = await deployCurveContracts();

  // Set holders fee to 5%.
  const holdersFee = ethers.utils.parseEther("0.05");
  await testContract.setMaxFeePercent(holdersFee);
  await testContract.setExternalFeePercent(0.0, 0.0, holdersFee);

  // Setup owner's initial token share.
  await testContract.buyCurvesToken(owner.address, 1);
  const feeSplitter = FeeSplitter__factory.connect(testContract.feeRedistributor(), owner);

  // addrs[0] buys 10 tokens
  await testContract.connect(addrs[0]).buyCurvesToken(owner.address, 10, {
    value: await testContract.getBuyPriceAfterFee(owner.address, 10)
  });
  // BigNumber { value: "1093750000000000" }
  console.log(await feeSplitter.getClaimableFees(owner.address, addrs[0].address));

  // addrs[0] buys 1 tokens
  await testContract.connect(addrs[0]).buyCurvesToken(owner.address, 1, {
    value: await testContract.getBuyPriceAfterFee(owner.address, 1)
  });
  // Holder fee has decreased.
  // BigNumber { value: "346614583333326" }
  console.log(await feeSplitter.getClaimableFees(owner.address, addrs[0].address));
});
```

## [NC-01] Add in code comments and documentation that the initial buy amount must be 1.

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L184

The `getPrice()` function in the contract has a contraint that if the current supply is zero, the maximum amount that can be purchased is 1 token, or else there would be an integer underflow error for `supply - 1 + amount`.

If this is indeed the intended behavior, it would be beneficial to clearly document it in the code comments. Such an underflow behavior might not be immediately obvious to someone reviewing the code without this context.

```solidity
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}
```

## [NC-02] `buyCurvesTokenWithName` and `buyCurvesTokenForPresale` code style improvement.

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L364-L392

These two functions are meant for token initialization, but they have some code style issues:

1. The `amount` parameter in the functions could be removed, as the initial purchase is limited to 1 token.
2. One function has `onlyTokenSubject(curvesTokenSubject)` protected while the other does not. It's better to maintain uniformity in access control.
3. The function names like `buyCurvesTokenWithName()` could be misleading, as they imply general user access. Renaming them to `initCurvesTokenWithName` and `initCurvesTokenForPresale` might be more descriptive and accurate.
4. These functions don't need to be payable, considering the first token purchase requires no payment.

```
function buyCurvesTokenWithName(
    address curvesTokenSubject,
    uint256 amount,
    string memory name,
    string memory symbol
) public payable {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply != 0) revert CurveAlreadyExists();

    _buyCurvesToken(curvesTokenSubject, amount);
    _mint(curvesTokenSubject, name, symbol);
}

function buyCurvesTokenForPresale(
    address curvesTokenSubject,
    uint256 amount,
    uint256 startTime,
    bytes32 merkleRoot,
    uint256 maxBuy
) public payable onlyTokenSubject(curvesTokenSubject) {
    if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply != 0) revert CurveAlreadyExists();
    presalesMeta[curvesTokenSubject].startTime = startTime;
    presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
    presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);

    _buyCurvesToken(curvesTokenSubject, amount);
}
```