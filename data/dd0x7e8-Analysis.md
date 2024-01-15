# Overview
The Curves project consists of a set of smart contracts designed to work together to create a token ecosystem with unique economic and trading features. 
Here's a summary of each contract's primary purpose and functionality:

1. **Curves**: This is the main contract of the ecosystem. It handles the creation, buying, and selling of Curves tokens which are subject to a unique pricing curve. It also manages presales with whitelist functionality via Merkle proofs and enforces fee structures for transactions. The contract allows for the transfer of Curves tokens between users and supports the minting of ERC20 tokens through a factory contract.

2. **CurvesERC20Factory**: This contract is a factory for creating new instances of the `CurvesERC20` token contract. It encapsulates the logic for deploying new ERC20 tokens with specified names and symbols.

3. **CurvesERC20**: Not provided in the given code snippets, but it would typically be an ERC20 token contract that includes additional functionality or restrictions as required by the Curves ecosystem.

4. **FeeSplitter**: This contract splits fees from Curves token transactions among various parties, including a protocol fee, subject fee, referral fee, and holders fee. The `FeeSplitter` contract also handles claims by token holders to receive their share of transaction fees collected.

5. **Security**: This contract provides basic access control functionality, designating an owner and managers who are granted permission to perform certain administrative tasks.

Key Features:

- **Trading and Fees**: The main contract allows users to trade Curves tokens, with prices dynamically calculated based on a bonding curve formula. Fees are collected on each transaction and can be configured by the owner/manager of the contract.

- **Presale Mechanics**: The contract supports presale events, where tokens can be bought before a public sale, with conditions enforced through Merkle proofs to verify whitelist entries.

- **ERC20 Token Minting**: Curves tokens can be converted into ERC20 tokens, allowing for greater interoperability within the Ethereum ecosystem.

- **Fee Redistribution**: Fees collected from token trading can be claimed by token holders, incentivizing participation and investment in the Curves tokens.

- **Access Control**: Through the `Security` contract, there is a distinction between the owner (with the highest level of control) and managers (who can perform specific administrative functions).

Overall, these contracts work together to create a Curves token economy with unique features such as presales, fee collection and redistribution, and custom ERC20 token minting, all governed by an access-controlled administrative system.

# Curves finding: Lack of Access Control in `setCurves()` Function

## Risk Rating
**High Risk**

# Lines of code

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L35

# Vulnerability details

## Impact
The `setCurves()` function in the `FeeSplitter` contract is designed to update the reference to the `Curves` contract, which is used for calculating balances and total supply for tokens. However, since `setCurves()` is a public function without any access control modifiers, any external entity can call it. This poses a significant security risk because if the `curves` reference is set to a malicious contract, it could lead to incorrect fee distributions or enable other harmful activities.

The lack of access control in the `setCurves()` function means that anyone can change the contract to which `FeeSplitter` refers for critical data related to fee calculations. If compromised, the malicious contract could return falsified balances or total supplies, which would directly impact the computation of `cumulativeFeePerToken`, the amount of fees owed to each user (`unclaimedFees`), and the token holder list (`userTokens`). Such manipulations could result in users being able to claim more fees than they are entitled to, or not being able to claim fees they have rightfully earned.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

### Code
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.7;

import "forge-std/Test.sol";
import {CurvesERC20Factory} from "../../contracts/CurvesERC20Factory.sol";
import {FeeSplitter} from "../../contracts/FeeSplitter.sol";
import {Curves} from "../../contracts/Curves.sol";

contract CurvesBaseTest is Test {
    CurvesERC20Factory public factory;
    FeeSplitter public feeSplitter;
    Curves public curves;
    address public Owner = makeAddr("Owner");
    address public Bob = makeAddr("Bob");
    address public Tom = makeAddr("Tom");
    address public Eva = makeAddr("Eva");

    function setUp() public virtual {
        factory = new CurvesERC20Factory();
        feeSplitter = new FeeSplitter();
        curves = new Curves(address(factory), address(feeSplitter));
    }
}

// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.7;

import "./CurvesBase.t.sol";

contract FeeSplitterTest is CurvesBaseTest {

    function setUp() public override {
        super.setUp();
    }

    function test_POC_LackAccessControl_setCurves() public {
        vm.startPrank(Eva);
        FeeSplitter otherSplitter = new FeeSplitter();
        Curves otherCurves = new Curves(address(factory), address(otherSplitter));
        feeSplitter.setCurves(otherCurves);
        vm.stopPrank();
    }
}
```

### Test Result
```log
$ forge test --mt test_POC_LackAccessControl_setCurves -vvv
[⠒] Compiling...
[⠢] Compiling 1 files with 0.8.7
[⠒] Solc 0.8.7 finished in 13.55s
Compiler run successful!

Running 1 test for FeeSplitter.t.sol:FeeSplitterTest
[PASS] test_POC_LackAccessControl_setCurves() (gas: 3605952)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 2.89ms

Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

## Tools Used
Foundry

## Recommended Mitigation Steps
To mitigate this issue, the `setCurves()` function should be protected by an access control mechanism, such as the `onlyOwner` modifier that is utilized in other functions within the contract. 
This would ensure that only authorized addresses, likely those associated with the contract's owner, have the ability to modify the `curves` reference. 
Implementing this change would prevent unauthorized and potentially harmful reassignment of the `Curves` contract, thus safeguarding the integrity of the fee distribution process.

# Curves finding: Protocol Fee NOT Transferred to Protocol Fee Destination During Selling And LOCKED in `Cruves` Contract

## Risk Rating
**High Risk**

# Lines of code

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218-L250

## Impact
The issue within the `_transferFees()` function of the `Curves` contract pertains to the incorrect handling of fees distribution during the sale of Curves tokens. The function is intended to transfer different types of fees to their respective destinations during both buy and sell operations. However, there is a flaw in the logic that incorrectly handles the distribution of the protocol fee when selling tokens.

Here's the segment of the `_transferFees()` function where the issue lies:

```solidity
		{
			address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
			uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
			uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
			(bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
			if (!success1) revert CannotSendFunds();
		}
```

In the case of selling (`isBuy` is `false`), the value being sent (`sellValue`) to the `firstDestination` (which is the `msg.sender` in this case) is calculated by subtracting all the fees from the `price`. However, the protocol fee is not being sent to the `protocolFeeDestination` as it should be. Instead, the entire net amount (after subtracting all fees) is being returned to the seller.

As a result, the protocol fee is not moving anywhere—it remains within the `Curves` contract. This means that the intended recipient, which is the `protocolFeeDestination`, does not receive the protocol fees from sales transactions, leading to funds being locked within the contract and not being distributed as per the designed economic model.

This constitutes a significant problem because certain fees are not allocated and end up being permanently trapped within the contract.

## Proof of Concept
The proof of concept implemented with Foundry indicates that even after the total sale of shares, a quantity of ETH remains in the `Curves` contract without any method available for withdrawal.

### Code
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "forge-std/Test.sol";
import {CurvesERC20Factory} from "../../contracts/CurvesERC20Factory.sol";
import {FeeSplitter} from "../../contracts/FeeSplitter.sol";
import {Curves} from "../../contracts/Curves.sol";
import "solpretty/SolPrettyTools.sol";

contract CurvesBaseTest is Test, SolPrettyTools {
    CurvesERC20Factory public factory;
    FeeSplitter public feeSplitter;
    Curves public curves;

    address public FeeDestination = makeAddr("FeeDestination");
    address public ReferralFeeDestination = makeAddr("ReferralFeeDestination");
    address public Owner = makeAddr("Owner");
    address public Bob = makeAddr("Bob");
    address public Tom = makeAddr("Tom");
    address public Eva = makeAddr("Eva");


    function setUp() public virtual {
        vm.startPrank(Owner);
        factory = new CurvesERC20Factory();
        feeSplitter = new FeeSplitter();
        curves = new Curves(address(factory), address(feeSplitter));
        feeSplitter.setManager(address(curves), true);
        feeSplitter.setCurves(curves);
        curves.setManager(Owner, true);
        curves.setERC20Factory(address(factory));
        curves.setFeeRedistributor(address(feeSplitter));
        console2.log("Setup: Max Fee Percent is 0.1 ether (10%)");
        curves.setMaxFeePercent(0.1 ether);//10%
        console2.log("Setup: Protocol Fee Percent is 0.05 ether (5%)");
        curves.setProtocolFeePercent(0.05 ether, FeeDestination);
        console2.log("Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)");
        curves.setExternalFeePercent(0.02 ether, 0.02 ether, 0.01 ether);
        vm.stopPrank();

        //init funds
        deal(Bob, 1e5 ether);
        deal(Tom, 1e5 ether);

        vm.label(address(curves), "Curves");
        vm.label(address(feeSplitter), "FeeSplitter");
    }

    function userBuyCurvesTokenWithName(
        address subject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) internal {
        vm.startPrank(subject);
        console2.log("%s Buys Curves %s Token with Amount %d", vm.getLabel(subject), symbol, amount);
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        curves.buyCurvesTokenWithName{value: totalPrice}(subject, amount, name, symbol);
        vm.stopPrank();
    }

    function userBuyCurvesToken(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        console2.log("%s Buys %s shares of subject %s with payment:", vm.getLabel(user), amount, vm.getLabel(subject));
        pp(totalPrice, 18, 5, " ether");
        curves.buyCurvesToken{value: totalPrice}(subject, amount);
        vm.stopPrank();
    }

    function userBuyCurvesTokenWithExtraPayment(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        console2.log("%s Buys %s shares of subject %s with 1 ether ETH extra payment", vm.getLabel(user), amount, vm.getLabel(subject));
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        curves.buyCurvesToken{value: totalPrice + 1 ether}(subject, amount);
        vm.stopPrank();
    }

    function userSellCurvesToken(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        console2.log("%s Sells %d shares of subject %s", vm.getLabel(user), amount, vm.getLabel(subject));
        curves.sellCurvesToken(subject, amount);
        vm.stopPrank();
    }

    function showBalance(address _addr) internal view {
        uint256 balance = payable(_addr).balance;
        console2.log("%s's ETH balance is ", vm.getLabel(_addr));
        pp(balance, 18, 5, " ether");
    }

}

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "./CurvesBase.t.sol";

contract CruvesTest is CurvesBaseTest {

    bytes32 public merkleRoot;
    bytes32[] public merkleProof;

    function setUp() public override {
        super.setUp();
        vm.prank(Bob);
        curves.setReferralFeeDestination(Bob, ReferralFeeDestination);
        vm.prank(Tom);
        curves.setReferralFeeDestination(Tom, ReferralFeeDestination);
    }

 
    function test_ProtocolFeeLocked_buyToken_sellToken() public {
        userBuyCurvesToken(Bob, Bob, 1);
        showBalance(address(curves));
        showBalance(FeeDestination);
        showBalance(address(feeSplitter));
        showBalance(ReferralFeeDestination);
        console2.log("--------------------------------------------");
        userBuyCurvesToken(Tom, Bob, 100);
        showBalance(address(curves));
        showBalance(FeeDestination);
        showBalance(address(feeSplitter));
        showBalance(ReferralFeeDestination);
        console2.log("--------------------------------------------");
        userSellCurvesToken(Tom, Bob, 100);
        showBalance(address(curves));
        showBalance(FeeDestination);
        showBalance(address(feeSplitter));
        showBalance(ReferralFeeDestination);
    }

}
```

### Test Result
```log
$ forge test --mt test_ProtocolFeeLocked_buyToken_sellToken -vvv
[⠃] Compiling...
[⠰] Compiling 5 files with 0.8.7
[⠒] Solc 0.8.7 finished in 17.83s
Compiler run successful!

Running 1 test for test/audit/Curves.t.sol:CruvesTest
[PASS] test_ProtocolFeeLocked_buyToken_sellToken() (gas: 667663)
Logs:
  Setup: Max Fee Percent is 0.1 ether (10%)
  Setup: Protocol Fee Percent is 0.05 ether (5%)
  Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)
  Bob Buys 1 shares of subject Bob with payment:
  0.00000  ether
  Curves's ETH balance is 
  0.00000  ether
  FeeDestination's ETH balance is 
  0.00000  ether
  FeeSplitter's ETH balance is 
  0.00000  ether
  ReferralFeeDestination's ETH balance is
  0.00000  ether
  --------------------------------------------
  Tom Buys 100 shares of subject Bob with payment:
  23.26156  ether
  Curves's ETH balance is
  21.14687  ether
  FeeDestination's ETH balance is
  1.05734  ether
  FeeSplitter's ETH balance is
  0.21146  ether
  ReferralFeeDestination's ETH balance is
  0.42293  ether
  --------------------------------------------
  Tom Sells 100 shares of subject Bob
  Curves's ETH balance is
  1.05734  ether
  FeeDestination's ETH balance is
  1.05734  ether
  FeeSplitter's ETH balance is
  0.42293  ether
  ReferralFeeDestination's ETH balance is
  0.84587  ether

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 8.10ms

Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

The provided log details the output of a test case executed using the Foundry framework for smart contract development in Solidity. The test, named `test_ProtocolFeeLocked_buyToken_sellToken`, is part of a suite designed to test the behavior of `Curves` smart contract, which manages the buying and selling of shares according to a pricing bond curve.

The log outlines the following actions and results:

- Initially, a setup process is described where various fee percentages are defined, including max fee, protocol fee, subject fee, referral fee, and holder fee.

- The first transaction shows "Bob" buying 1 share of "subject Bob" but without any Ether changing hands, as indicated by the zero balances across all accounts.

- The second transaction involves "Tom" purchasing 100 shares of "subject Bob" for a payment of 23.26156 Ether. After this transaction, the `Curves` contract holds 21.14687 Ether, and various fee destinations and splitters receive their respective portions of Ether as fees.

- The third transaction has "Tom" selling his 100 shares of "subject Bob". After the sale, the balances of the fee destinations and splitters are updated. However, it is noted that the `Curves` contract still contains 1.05734 Ether.

According to the expected behavior outlined in the test description—presumably based on the specified pricing bond curve—the `Curves` contract should have a zero balance after all shares are sold. Instead, the test reveals a critical issue: there is an amount of 1.05734 Ether that remains locked in the `Curves` contract with no available mechanism to withdraw it.

This indicates a potential flaw in the contract logic or the fee distribution system, which results in funds being permanently trapped within the contract. This issue would likely require a revision of the smart contract code to ensure that all Ether can be properly withdrawn after transactions, thereby avoiding permanent loss of funds.


## Tools Used
Foundry

## Recommended Mitigation Steps
To resolve this issue, the protocol fee should be explicitly transferred to the `protocolFeeDestination` during the sell operation, similar to how it's done during the buy operation. The corrected code would look something like this:

```solidity=228
		{
			if (isBuy) {
				uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
				(bool success, ) = feesEconomics.protocolFeeDestination.call{value: buyValue}("");
				if (!success) revert CannotSendFunds();
			} else {
				uint256 sellValue = price - subjectFee - referralFee - holderFee;
				(bool success, ) = msg.sender.call{value: sellValue}("");
				if (!success) revert CannotSendFunds();

				// Transfer protocol fee to protocolFeeDestination
				(success, ) = feesEconomics.protocolFeeDestination.call{value: protocolFee}("");
				if (!success) revert CannotSendFunds();
			}
		}
```

With this change, the protocol fee is correctly sent to `protocolFeeDestination` on sales of tokens, ensuring that all fees are distributed appropriately according to the contract's economic rules.

# Curves finding: Normal Users Potentially Cannot Sell All Purchased Curves Token

## Risk Rating
**High Risk**

# Lines of code

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L284

## Impact
The issue in the `sellCurvesToken()` function arises from a constraint that prevents the sale of the last remaining token in the total supply of a given subject token. The intention behind this constraint is likely to maintain the existence of the token and prevent the subject from selling a token that was originally obtained for free, as per the bonding curve pricing mechanism. However, the current implementation of this constraint is overly restrictive and does not differentiate between tokens owned by the subject itself and those owned by other users.

The problematic code is as follows:

```solidity
		if (supply <= amount) revert LastTokenCannotBeSold();
```

This line of code prevents any sale transaction that would result in the total supply of the subject token being reduced to zero. The issue is that it indiscriminately applies to all users, including those who did not receive any free tokens and bought their shares at a price set by the bonding curve.

The unintended consequence is that regular users who have purchased tokens are unable to sell their entire holdings if their holdings match the total remaining supply, even though they should logically be able to liquidate their investment.

## Proof of Concept
The POC shows that normal users cannot sell all purchased tokens as expected.

### Code
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "forge-std/Test.sol";
import {CurvesERC20Factory} from "../../contracts/CurvesERC20Factory.sol";
import {FeeSplitter} from "../../contracts/FeeSplitter.sol";
import {Curves} from "../../contracts/Curves.sol";
import "solpretty/SolPrettyTools.sol";

contract CurvesBaseTest is Test, SolPrettyTools {
    CurvesERC20Factory public factory;
    FeeSplitter public feeSplitter;
    Curves public curves;

    address public FeeDestination = makeAddr("FeeDestination");
    address public ReferralFeeDestination = makeAddr("ReferralFeeDestination");
    address public Owner = makeAddr("Owner");
    address public Bob = makeAddr("Bob");
    address public Tom = makeAddr("Tom");
    address public Eva = makeAddr("Eva");


    function setUp() public virtual {
        vm.startPrank(Owner);
        factory = new CurvesERC20Factory();
        feeSplitter = new FeeSplitter();
        curves = new Curves(address(factory), address(feeSplitter));
        feeSplitter.setManager(address(curves), true);
        feeSplitter.setCurves(curves);
        curves.setManager(Owner, true);
        curves.setERC20Factory(address(factory));
        curves.setFeeRedistributor(address(feeSplitter));
        console2.log("Setup: Max Fee Percent is 0.1 ether (10%)");
        curves.setMaxFeePercent(0.1 ether);//10%
        console2.log("Setup: Protocol Fee Percent is 0.05 ether (5%)");
        curves.setProtocolFeePercent(0.05 ether, FeeDestination);
        console2.log("Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)");
        curves.setExternalFeePercent(0.02 ether, 0.02 ether, 0.01 ether);
        vm.stopPrank();

        //init funds
        deal(Bob, 1e5 ether);
        deal(Tom, 1e5 ether);

        vm.label(address(curves), "Curves");
        vm.label(address(feeSplitter), "FeeSplitter");
    }

    function userBuyCurvesTokenWithName(
        address subject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) internal {
        vm.startPrank(subject);
        console2.log("%s Buys Curves %s Token with Amount %d", vm.getLabel(subject), symbol, amount);
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        curves.buyCurvesTokenWithName{value: totalPrice}(subject, amount, name, symbol);
        vm.stopPrank();
    }

    function userBuyCurvesToken(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        console2.log("%s Buys %s shares of subject %s with payment:", vm.getLabel(user), amount, vm.getLabel(subject));
        pp(totalPrice, 18, 5, " ether");
        curves.buyCurvesToken{value: totalPrice}(subject, amount);
        vm.stopPrank();
    }

    function userBuyCurvesTokenWithExtraPayment(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        console2.log("%s Buys %s shares of subject %s with 1 ether ETH extra payment", vm.getLabel(user), amount, vm.getLabel(subject));
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        curves.buyCurvesToken{value: totalPrice + 1 ether}(subject, amount);
        vm.stopPrank();
    }

    function userSellCurvesToken(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        console2.log("%s Sells %d shares of subject %s", vm.getLabel(user), amount, vm.getLabel(subject));
        curves.sellCurvesToken(subject, amount);
        vm.stopPrank();
    }

    function showBalance(address _addr) internal view {
        uint256 balance = payable(_addr).balance;
        console2.log("%s's ETH balance is ", vm.getLabel(_addr));
        pp(balance, 18, 5, " ether");
    }

}


// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "./CurvesBase.t.sol";

contract CruvesTest is CurvesBaseTest {

    bytes32 public merkleRoot;
    bytes32[] public merkleProof;

    function setUp() public override {
        super.setUp();
        vm.prank(Bob);
        curves.setReferralFeeDestination(Bob, ReferralFeeDestination);
        vm.prank(Tom);
        curves.setReferralFeeDestination(Tom, ReferralFeeDestination);
    }

    function test_UserCannotSellAllTokens_buyToken_sellToken() public {
        userBuyCurvesToken(Bob, Bob, 1);
        showBalance(Bob);
        showBalance(Tom);
        userBuyCurvesToken(Tom, Bob, 100);
        userSellCurvesToken(Bob, Bob, 1);
        showBalance(Bob);
        userSellCurvesToken(Tom, Bob, 100);
        showBalance(Tom);
    }
}

```

### Result
```log
$ forge test --mt test_UserCannotSellAllTokens_buyToken_sellToken -vv
[⠘] Compiling...
No files changed, compilation skipped

Running 1 test for test/audit/Curves.t.sol:CruvesTest
[FAIL. Reason: LastTokenCannotBeSold()] test_UserCannotSellAllTokens_buyToken_sellToken() (gas: 596029)
Logs:
  Setup: Max Fee Percent is 0.1 ether (10%)
  Setup: Protocol Fee Percent is 0.05 ether (5%)
  Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)
  Bob Buys 1 shares of subject Bob with payment:
  0.00000  ether
  Bob's ETH balance is
  100,000.00000  ether
  Tom's ETH balance is
  100,000.00000  ether
  Tom Buys 100 shares of subject Bob with payment:
  23.26156  ether
  Bob Sells 1 shares of subject Bob
  Bob's ETH balance is
  100,000.99793  ether
  Tom Sells 100 shares of subject Bob

Test result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 4.61ms

Ran 1 test suites: 0 tests passed, 1 failed, 0 skipped (1 total tests)

Failing tests:
Encountered 1 failing test in test/audit/Curves.t.sol:CruvesTest
[FAIL. Reason: LastTokenCannotBeSold()] test_UserCannotSellAllTokens_buyToken_sellToken() (gas: 596029)

Encountered a total of 1 failing tests, 0 tests succeeded
```

## Tools Used
Foundry

## Recommended Mitigation Steps

Enforce the constraint that the subject cannot sell its initial free token while allowing other users to sell all their purchased tokens. This would likely involve additional state tracking within the contract to differentiate between free and purchased tokens.

A potential implementation for the second solution could be:

```solidity
		if (curvesTokenSubject == msg.sender && curvesTokenBalance[curvesTokenSubject][curvesTokenSubject] == amount) revert LastTokenCannotBeSold();
```

This revised condition checks if the caller (`msg.sender`) is the subject itself and if the transaction would result in the total supply reaching zero, which would only revert if the subject is trying to sell their last token (presumably the one they received for free).

# Curves finding: Superfluous ETH NOT Refund and Locked in Contract

## Risk Rating
**Medium Risk**

# Lines of code
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L270

## Impact
The issue in the `_buyCurvesToken()` function is related to the lack of a mechanism to handle situations where a buyer sends more Ether (ETH) than what is required for the purchase of Curves tokens, including the associated fees. The current implementation checks only if the sent ETH is less than the required amount and reverts the transaction if so:

```solidity
		if (msg.value < price + totalFee) revert InsufficientPayment();
```

However, there's no corresponding logic to handle the case where `msg.value` is greater than `price + totalFee`. In such a scenario, the excess ETH sent by the buyer would not be used for the token purchase and would remain in the contract’s balance, effectively locking these funds and preventing their return to the buyer.

This can lead to two problems:

1. **Locked Funds**: Buyers who accidentally send too much ETH will have the excess amount locked in the contract, with no straightforward way to retrieve it.

2. **User Experience**: The lack of a refund for overpayment may frustrate users and erode trust in the platform, as they may feel penalized for making a mistake.

## Proof of Concept
The POC shows that there is no refund while buying tokens.

### Code
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "forge-std/Test.sol";
import {CurvesERC20Factory} from "../../contracts/CurvesERC20Factory.sol";
import {FeeSplitter} from "../../contracts/FeeSplitter.sol";
import {Curves} from "../../contracts/Curves.sol";
import "solpretty/SolPrettyTools.sol";

contract CurvesBaseTest is Test, SolPrettyTools {
    CurvesERC20Factory public factory;
    FeeSplitter public feeSplitter;
    Curves public curves;

    address public FeeDestination = makeAddr("FeeDestination");
    address public ReferralFeeDestination = makeAddr("ReferralFeeDestination");
    address public Owner = makeAddr("Owner");
    address public Bob = makeAddr("Bob");
    address public Tom = makeAddr("Tom");
    address public Eva = makeAddr("Eva");


    function setUp() public virtual {
        vm.startPrank(Owner);
        factory = new CurvesERC20Factory();
        feeSplitter = new FeeSplitter();
        curves = new Curves(address(factory), address(feeSplitter));
        feeSplitter.setManager(address(curves), true);
        feeSplitter.setCurves(curves);
        curves.setManager(Owner, true);
        curves.setERC20Factory(address(factory));
        curves.setFeeRedistributor(address(feeSplitter));
        console2.log("Setup: Max Fee Percent is 0.1 ether (10%)");
        curves.setMaxFeePercent(0.1 ether);//10%
        console2.log("Setup: Protocol Fee Percent is 0.05 ether (5%)");
        curves.setProtocolFeePercent(0.05 ether, FeeDestination);
        console2.log("Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)");
        curves.setExternalFeePercent(0.02 ether, 0.02 ether, 0.01 ether);
        vm.stopPrank();

        //init funds
        deal(Bob, 1e5 ether);
        deal(Tom, 1e5 ether);

        vm.label(address(curves), "Curves");
        vm.label(address(feeSplitter), "FeeSplitter");
    }

    function userBuyCurvesTokenWithName(
        address subject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) internal {
        vm.startPrank(subject);
        console2.log("%s Buys Curves %s Token with Amount %d", vm.getLabel(subject), symbol, amount);
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        curves.buyCurvesTokenWithName{value: totalPrice}(subject, amount, name, symbol);
        vm.stopPrank();
    }

    function userBuyCurvesToken(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        console2.log("%s Buys %s shares of subject %s with payment:", vm.getLabel(user), amount, vm.getLabel(subject));
        pp(totalPrice, 18, 5, " ether");
        curves.buyCurvesToken{value: totalPrice}(subject, amount);
        vm.stopPrank();
    }

    function userBuyCurvesTokenWithExtraPayment(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        console2.log("%s Buys %s shares of subject %s with 1 ether ETH extra payment", vm.getLabel(user), amount, vm.getLabel(subject));
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        curves.buyCurvesToken{value: totalPrice + 1 ether}(subject, amount);
        vm.stopPrank();
    }

    function userSellCurvesToken(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        console2.log("%s Sells %d shares of subject %s", vm.getLabel(user), amount, vm.getLabel(subject));
        curves.sellCurvesToken(subject, amount);
        vm.stopPrank();
    }

    function showBalance(address _addr) internal view {
        uint256 balance = payable(_addr).balance;
        console2.log("%s's ETH balance is ", vm.getLabel(_addr));
        pp(balance, 18, 5, " ether");
    }
}

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "./CurvesBase.t.sol";

contract CruvesTest is CurvesBaseTest {

    bytes32 public merkleRoot;
    bytes32[] public merkleProof;

    function setUp() public override {
        super.setUp();
        vm.prank(Bob);
        curves.setReferralFeeDestination(Bob, ReferralFeeDestination);
        vm.prank(Tom);
        curves.setReferralFeeDestination(Tom, ReferralFeeDestination);
    }

    function test_NoRefund_buyToken_sellToken() public {
        showBalance(address(curves));
        userBuyCurvesTokenWithExtraPayment(Bob, Bob, 1);
        showBalance(address(curves));
        showBalance(FeeDestination);
        showBalance(address(feeSplitter));
        showBalance(ReferralFeeDestination);
    }
}


```
### Test Result
```log
$ forge test --mt test_NoRefund_buyToken_sellToken -vvv
[⠘] Compiling...
[⠒] Compiling 5 files with 0.8.7
[⠑] Solc 0.8.7 finished in 17.70s
Compiler run successful!

Running 1 test for test/audit/Curves.t.sol:CruvesTest
[PASS] test_NoRefund_buyToken_sellToken() (gas: 281544)
Logs:
  Setup: Max Fee Percent is 0.1 ether (10%)
  Setup: Protocol Fee Percent is 0.05 ether (5%)
  Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)
  Curves's ETH balance is 
  0.00000  ether
  Bob Buys 1 shares of subject Bob with 1 ether ETH extra payment
  Curves's ETH balance is 
  1.00000  ether
  FeeDestination's ETH balance is 
  0.00000  ether
  FeeSplitter's ETH balance is 
  0.00000  ether
  ReferralFeeDestination's ETH balance is
  0.00000  ether

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 3.66ms

Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)

```

## Tools Used
Foundry

## Recommended Mitigation Steps
To resolve this issue and enhance the user experience, the contract should implement a refund mechanism that sends back any excess ETH to the buyer. This can be achieved by adding a few lines of code at the end of the `_buyCurvesToken()` function:

```solidity
		// After processing the token purchase and fee transfers
		if (msg.value > price + totalFee) {
			uint256 excessAmount = msg.value - (price + totalFee);
			(bool refundSuccess, ) = msg.sender.call{value: excessAmount}("");
			require(refundSuccess, "Refund of excess ETH failed");
		}
```

This additional code calculates the excess amount of ETH by subtracting the total cost (`price + totalFee`) from the `msg.value` sent by the buyer. It then attempts to transfer this excess amount back to the buyer's address. If the transfer fails, the transaction reverts with an appropriate error message. This ensures that buyers only pay the exact amount required for their purchase and receive an automatic refund for any overpayment.
```
### Test Result
```log
$ forge test --mt test_NoRefund_buyToken_sellToken -vvv
[⠘] Compiling...
[⠒] Compiling 5 files with 0.8.7
[⠑] Solc 0.8.7 finished in 17.70s
Compiler run successful!

Running 1 test for test/audit/Curves.t.sol:CruvesTest
[PASS] test_NoRefund_buyToken_sellToken() (gas: 281544)
Logs:
  Setup: Max Fee Percent is 0.1 ether (10%)
  Setup: Protocol Fee Percent is 0.05 ether (5%)
  Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)
  Curves's ETH balance is 
  0.00000  ether
  Bob Buys 1 shares of subject Bob with 1 ether ETH extra payment
  Curves's ETH balance is 
  1.00000  ether
  FeeDestination's ETH balance is 
  0.00000  ether
  FeeSplitter's ETH balance is 
  0.00000  ether
  ReferralFeeDestination's ETH balance is
  0.00000  ether

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 3.66ms

Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)

```

## Tools Used
Foundry

## Recommended Mitigation Steps
To resolve this issue and enhance the user experience, the contract should implement a refund mechanism that sends back any excess ETH to the buyer. This can be achieved by adding a few lines of code at the end of the `_buyCurvesToken()` function:

```solidity
		// After processing the token purchase and fee transfers
		if (msg.value > price + totalFee) {
			uint256 excessAmount = msg.value - (price + totalFee);
			(bool refundSuccess, ) = msg.sender.call{value: excessAmount}("");
			require(refundSuccess, "Refund of excess ETH failed");
		}
```

This additional code calculates the excess amount of ETH by subtracting the total cost (`price + totalFee`) from the `msg.value` sent by the buyer. It then attempts to transfer this excess amount back to the buyer's address. If the transfer fails, the transaction reverts with an appropriate error message. This ensures that buyers only pay the exact amount required for their purchase and receive an automatic refund for any overpayment.

# Curves finding: Ensuring Accurate Fee Distribution Before Token Sale in the `FeeSplitter` Contract

## Risk Rating
**Medium Risk**

# Lines of code
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L71

## Impact
The issue described is related to how the `FeeSplitter` contract manages the distribution and claiming of holder fees. Each holder's claimable fees are calculated based on their current balance of Curves tokens at the time of the claim. This approach creates a scenario where a token holder who sells their tokens without first claiming their accumulated fees will lose a portion of the fees they were entitled to.

Here's why this problem occurs:

When a holder sells their tokens, their balance decreases. Since the claimable fees are proportional to the holder's balance, reducing the balance before claiming means they will be able to claim less in fees afterward. The fees that would have been claimed based on the higher balance before the sale remain unclaimed and thus become locked in the contract, as there's no mechanism to distribute these remaining fees to the rightful owner.

The following function illustrates where the claimable fees are updated:

```solidity
function updateFeeCredit(address token, address account) internal {
    TokenData storage data = tokensData[token];
    uint256 balance = balanceOf(token, account);
    if (balance > 0) {
        uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
        data.unclaimedFees[account] += owed / PRECISION;
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
    }
}
```

The `updateFeeCredit` function calculates the fees owed based on the difference between the `cumulativeFeePerToken` and the user's `userFeeOffset`, then multiplies this by the user's current token balance. The fees are then added to `unclaimedFees[account]`, and the `userFeeOffset` is updated to the current `cumulativeFeePerToken`.

The flaw lies in the fact that if a user's balance changes (due to selling tokens) before they claim their fees, the fees calculated by the `owed` variable would be based on the new, lower balance.

## Proof of Concept
### Code
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "forge-std/Test.sol";
import {CurvesERC20Factory} from "../../contracts/CurvesERC20Factory.sol";
import {FeeSplitter} from "../../contracts/FeeSplitter.sol";
import {Curves} from "../../contracts/Curves.sol";
import "solpretty/SolPrettyTools.sol";

contract CurvesBaseTest is Test, SolPrettyTools {
    CurvesERC20Factory public factory;
    FeeSplitter public feeSplitter;
    Curves public curves;

    address public FeeDestination = makeAddr("FeeDestination");
    address public ReferralFeeDestination = makeAddr("ReferralFeeDestination");
    address public Owner = makeAddr("Owner");
    address public Bob = makeAddr("Bob");
    address public Tom = makeAddr("Tom");
    address public Eva = makeAddr("Eva");


    function setUp() public virtual {
        vm.warp(1705240800);
        vm.startPrank(Owner);
        factory = new CurvesERC20Factory();
        feeSplitter = new FeeSplitter();
        curves = new Curves(address(factory), address(feeSplitter));
        feeSplitter.setManager(address(curves), true);
        feeSplitter.setCurves(curves);
        curves.setManager(Owner, true);
        curves.setERC20Factory(address(factory));
        curves.setFeeRedistributor(address(feeSplitter));
        console2.log("Setup: Max Fee Percent is 0.1 ether (10%)");
        curves.setMaxFeePercent(0.1 ether);//10%
        console2.log("Setup: Protocol Fee Percent is 0.05 ether (5%)");
        curves.setProtocolFeePercent(0.05 ether, FeeDestination);
        console2.log("Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)");
        curves.setExternalFeePercent(0.02 ether, 0.02 ether, 0.01 ether);
        vm.stopPrank();

        //init funds
        deal(Bob, 1e5 ether);
        deal(Tom, 1e5 ether);
        deal(Eva, 1e5 ether);

        vm.label(address(curves), "Curves");
        vm.label(address(feeSplitter), "FeeSplitter");
    }

    function userBuyCurvesTokenWithName(
        address subject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) internal {
        vm.startPrank(subject);
        console2.log("%s Buys Curves %s Token with Amount %d", vm.getLabel(subject), symbol, amount);
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        curves.buyCurvesTokenWithName{value: totalPrice}(subject, amount, name, symbol);
        vm.stopPrank();
    }

    function userBuyCurvesToken(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        console2.log("%s Buys %s shares of subject %s with payment:", vm.getLabel(user), amount, vm.getLabel(subject));
        pp(totalPrice, 18, 5, " ether");
        curves.buyCurvesToken{value: totalPrice}(subject, amount);
        vm.stopPrank();
    }

    function userBuyCurvesTokenWithExtraPayment(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        console2.log("%s Buys %s shares of subject %s with 1 ether ETH extra payment", vm.getLabel(user), amount, vm.getLabel(subject));
        uint256 totalPrice = curves.getBuyPriceAfterFee(subject, amount);
        curves.buyCurvesToken{value: totalPrice + 1 ether}(subject, amount);
        vm.stopPrank();
    }

    function userSellCurvesToken(address user, address subject, uint256 amount) internal {
        vm.startPrank(user);
        console2.log("%s Sells %d shares of subject %s", vm.getLabel(user), amount, vm.getLabel(subject));
        curves.sellCurvesToken(subject, amount);
        vm.stopPrank();
    }

    function userMint(address subject) internal {
        vm.startPrank(subject);
        console2.log("%s Mint ERC20 Tokens", vm.getLabel(subject));
        curves.mint(subject);
        vm.stopPrank();
    }

    function userClaimFees(address user, address subject) internal {
        vm.startPrank(user);
        console2.log("%s claims fees from subject %s", vm.getLabel(user), vm.getLabel(subject));
        feeSplitter.claimFees(subject);
        vm.stopPrank();
    }

    function showBalance(address _addr) internal view {
        uint256 balance = payable(_addr).balance;
        console2.log("%s's ETH balance is ", vm.getLabel(_addr));
        pp(balance, 18, 5, " ether");
    }

    function showTokenMete(address subject) internal view {
        (string memory name, string memory symbol, address token)
        = curves.externalCurvesTokens(subject);
        console2.log("Token Meta: name is %s, symbol is %s, address is %s", name, symbol, token);
        address subject_ = curves.symbolToSubject(symbol);
        console2.log("Subject of symbol `%s` is %s", symbol, vm.getLabel(subject_));
    }

    function showClaimableFees(address subject, address user) internal view {
        uint256 claimableFees = feeSplitter.getClaimableFees(subject, user);
        console2.log("%s's claimable fees of subject %s is ", vm.getLabel(user), vm.getLabel(subject));
        pp(claimableFees, 18, 5, " ether");
    }

}

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "./CurvesBase.t.sol";

contract FeeSplitterTest is CurvesBaseTest {

    function setUp() public override {
        super.setUp();
    }

    function test_POC_LackAccessControl_setCurves() public {
        vm.startPrank(Eva);
        FeeSplitter otherSplitter = new FeeSplitter();
        Curves otherCurves = new Curves(address(factory), address(otherSplitter));
        feeSplitter.setCurves(otherCurves);
        vm.stopPrank();
    }

    function test_DuplicateData_userTokens() public {
        userBuyCurvesToken(Bob, Bob, 1);
        userBuyCurvesToken(Tom, Bob, 1);
        userBuyCurvesToken(Tom, Bob, 1);
        userBuyCurvesToken(Tom, Bob, 1);
        address[] memory subjects = feeSplitter.getUserTokens(Tom);
        console2.log("length of subjects is %d", subjects.length);
        for(uint256 i; i < subjects.length; i++) {
            console2.log("Current subject is %s", vm.getLabel(subjects[i]));
        }
    }

    function test_claimFees() public {
        userBuyCurvesToken(Bob, Bob, 1);
        userMint(Bob);
        userBuyCurvesToken(Tom, Bob, 100);
        showBalance(address(feeSplitter));
        showClaimableFees(Bob, Tom);
        showClaimableFees(Bob, Bob);
        userSellCurvesToken(Tom, Bob, 50);
        showBalance(address(feeSplitter));
        showClaimableFees(Bob, Tom);
        showClaimableFees(Bob, Bob);
        userClaimFees(Tom, Bob);
        userClaimFees(Bob, Bob);
        showClaimableFees(Bob, Tom);
        showClaimableFees(Bob, Bob);
        showBalance(address(feeSplitter));
    }

}
```

### Result
```log
$ forge test --mt test_claimFees -vv
[⠘] Compiling...
[⠆] Compiling 1 files with 0.8.7
[⠃] Solc 0.8.7 finished in 12.50s
Compiler run successful!

Running 1 test for test/audit/FeeSplitter.t.sol:FeeSplitterTest
[PASS] test_claimFees() (gas: 1623367)
Logs:
  Setup: Max Fee Percent is 0.1 ether (10%)
  Setup: Protocol Fee Percent is 0.05 ether (5%)
  Setup: Subject Fee Percent is 0.02 ether (2%), Referral Fee Percent is 0.02 ether (2%), Holder Fee Percent is 0.01 ether (1%)
  Bob Buys 1 shares of subject Bob with payment:
  0.00000  ether
  Bob Mint ERC20 Tokens
  Tom Buys 100 shares of subject Bob with payment:
  23.26156  ether
  FeeSplitter's ETH balance is
  0.21146  ether
  Tom's claimable fees of subject Bob is
  0.20937  ether
  Bob's claimable fees of subject Bob is
  0.00209  ether
  Tom Sells 50 shares of subject Bob
  FeeSplitter's ETH balance is
  0.39610  ether
  Tom's claimable fees of subject Bob is
  0.18102  ether
  Bob's claimable fees of subject Bob is
  0.00571  ether
  Tom claims fees from subject Bob
  Bob claims fees from subject Bob
  Tom's claimable fees of subject Bob is
  0.00000  ether
  Bob's claimable fees of subject Bob is
  0.00000  ether
  FeeSplitter's ETH balance is
  0.20937  ether

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 5.10ms

Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

## Tools Used
Foundry

## Recommended Mitigation Steps
To solve this issue, the contract could be updated with a mechanism to snapshot a user's claimable fees at the moment they sell their tokens. This would ensure that the user's right to claim fees is preserved based on their balance before the sale.

One approach to this could be:

```solidity
function sellCurvesToken(address token, uint256 amount) public {
    // First, update the fee credit to ensure owed fees are captured
    updateFeeCredit(token, msg.sender);
    // Then, perform the sale logic (not shown here)
    // ...
}
```

By calling `updateFeeCredit` before executing the sale, the contract captures the owed fees before the user's balance is reduced. This ensures the user can later claim the appropriate amount of fees that they accumulated while holding a higher balance of tokens.

Additionally, it would be prudent to encourage or enforce that holders claim their fees before selling their tokens, such as through UI prompts or additional checks in the smart contract logic.

--- 

# QA Report

## No Upper Limit for Max Fee Percent

The issue with the `setMaxFeePercent()` function in the `Curves` contract is that it lacks a validation check to ensure that the maximum fee percentage (`maxFeePercent`) does not exceed 100%, which is represented by `1 ether`.

Here's the problematic part of the `setMaxFeePercent()` function:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117C1-L126C6

```solidity
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
    if (
        feesEconomics.protocolFeePercent +
            feesEconomics.subjectFeePercent +
            feesEconomics.referralFeePercent +
            feesEconomics.holdersFeePercent >
        maxFeePercent_
    ) revert InvalidFeeDefinition();
    feesEconomics.maxFeePercent = maxFeePercent_;
}
```

The function allows an external manager to set the `maxFeePercent`, but it does not prevent the new `maxFeePercent` from being set to a value greater than `1 ether` (100%).

To resolve this issue, the `setMaxFeePercent()` function should include a check to ensure that the `maxFeePercent` value being set does not exceed `1 ether`:

```solidity
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
    require(maxFeePercent_ < 1 ether, "Invalid max fee percent");
    ...
    feesEconomics.maxFeePercent = maxFeePercent_;
}
```

## Check-Effects-Interactions Pattern Violation

The issue with the `_buyCurvesToken()` function, as described, arises from the order of operations, particularly the placement of the `_transferFees()` call before the state-modifying function `_addOwnedCurvesTokenSubject()`. This ordering could potentially make the function vulnerable to reentrancy attacks.

Here's the problematic sequence:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L274C1-L279C10

```solidity
        _transferFees(curvesTokenSubject, true, price, amount, supply);
        // ...
        _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
```

The `_transferFees()` function includes external calls to transfer ETH, which are potentially unsafe because they can be hijacked by malicious contracts to re-enter the current contract before the initial execution is completed.

```solidity
        (bool success, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
        // ... other external calls
```

According to the `Check-Effects-Interactions` pattern recommended by Solidity, state changes should be performed before any external interactions. This prevents other contracts from disrupting the expected flow of execution, which could lead to unintended outcomes, including loss of funds or corruption of the contract's state.

## Restriction on Initial Share Purchase (Self-Buy)
The issue with the `Curves` contract is that it has a design limitation or specific behavior that restricts the initial purchase (self-buy) by the token subject (presumably the creator or owner of the token) to exactly one unit of the token. This is due to how the `getPrice()` function calculates the cost for a given amount of tokens based on the current supply.

Here is the critical part of the `getPrice()` function that leads to this behavior:
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L180C1-L187C6

```solidity
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
```

If the `supply` is zero and `amount` is anything other than one, the expression `(supply - 1 + amount)` would result in an underflow because `supply - 1` is negative when `supply` is zero. Solidity underflowing will revert the transaction, preventing the purchase of more than one token when the supply is zero.

Moreover, the `setWhitelist()` function adds an additional constraint:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394C1-L396C53

```solidity
        if (supply > 1) revert CurveAlreadyExists();
```

This code indicates that once the supply exceeds one, the whitelist can no longer be updated, implying that the initial buy must be exactly one unit of the token.

The difference between the behavior of the `Curves` contract and the `Friend.tech` could lead to confusion among users. Users familiar with `Friend.tech` might expect the ability to self-buy any initial amount, and if not well documented, this discrepancy in `Curves` could result in a poor user experience.

To address this issue and improve user experience:

1. **Clarify in Documentation**: It should be clearly documented that the initial purchase by the token subject is limited to exactly one unit to avoid confusion.

2. **Validate Input**: Implement checks in functions that call `getPrice()` to ensure that if the supply is zero, the `amount` is one, for example `buyCurvesTokenForPresale()` and provide a clear revert message if not.

3. **Adjust Design**: If the intent is to allow subjects to buy more than one unit initially, the `getPrice()` function and other related logic should be adjusted to support this use case.

By implementing these measures, the `Curves` contract can provide clarity and consistency to users, aligning with their expectations and preventing unexpected reverts due to underflows in the pricing calculation.

## Redundant Token Added in Array
The issue with the `onBalanceChange()` function in the `FeeSplitter` contract is that it indiscriminately adds the `token` to the `userTokens[account]` array every time there is a balance change for an account that is non-zero. This can lead to the same `token` being added multiple times for the same account, resulting in a bloated and redundant array.

Here's the problematic part of the function:

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L99C76-L99C76

```solidity
if (balanceOf(token, account) > 0) userTokens[account].push(token);
```

This line does not check whether the `token` already exists in the `userTokens[account]` array before pushing it. As a result, if the `onBalanceChange()` function is called multiple times for the same `token` and `account` pair, which is common in a token economy with frequent transactions, the `userTokens[account]` array for that account will accumulate duplicates of the `token` address.

The consequences of this issue are:

1. **Increased Gas Costs**: Every time a token is redundantly added, it costs gas. Over time, with multiple redundant additions, this can become unnecessarily expensive.

2. **Inefficient Storage**: The Ethereum blockchain is not an ideal place for storing large amounts of redundant data due to the high cost of storage.

3. **Slower Processing**: When iterating through the `userTokens[account]` array, having a large number of duplicates can slow down processing times for functions that use this array, such as when calculating claimable fees or performing batch operations.

4. **Complication of Logic**: Having duplicates can complicate the logic of other functions that depend on the `userTokens[account]` array, potentially leading to errors or unintended behavior.

A solution to this issue is to check whether the `token` is already in the `userTokens[account]` array before adding it. This can be done by either using a mapping to keep track of which tokens are already associated with an account or by iterating through the array to check for the presence of the `token` before adding it.

Here's a potential fix using a mapping to ensure uniqueness:

```solidity
mapping(address => mapping(address => bool)) private hasToken;

function onBalanceChange(address token, address account) public onlyManager {
    TokenData storage data = tokensData[token];
    data.userFeeOffset[account] = data.cumulativeFeePerToken;
    if (balanceOf(token, account) > 0 && !hasToken[account][token]) {
        userTokens[account].push(token);
        hasToken[account][token] = true;
    }
}
```

By adding the `hasToken` mapping, the contract can quickly check if a `token` is already associated with an `account` and avoid adding duplicates to the `userTokens[account]` array. This approach maintains the integrity of the array and ensures efficient use of resources.


### Time spent:
20 hours