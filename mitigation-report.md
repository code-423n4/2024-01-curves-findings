# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts. 

A C4 solo mitigation review is an event where a top Code4rena community contributor, commonly referred to as a warden or a team, reviews mitigations from a preceding C4 audit in exchange for a bounty provided by sponsoring projects.

During the solo mitigation review outlined in this document, warden [hals](https://code4rena.com/@hals) reviewed the mitigations made by the Curves team following their [initial C4 audit](https://code4rena.com/audits/2024-01-curves) that took place between January 8 - 16, 2024. The mitigation review was conducted between May 8 - 11, 2024 and consisted of two rounds of review; both rounds are summarized by hals below.

## Initial Review

The purpose of this review is to confirm the **High and Medium** [findings](https://github.com/code-423n4/2024-01-curves-findings/issues?q=is%3Aopen+is%3Aissue+label%3A%223+%28High+Risk%29%22%2C%222+%28Med+Risk%29%22+label%3A%22selected+for+report%22) resulting from the Code4rena audit were resolved.

- This mitigation review was performed based on the updates pushed in the following commits in the Curves Contract:
  - Fixes: b9d575b7f6c4a140b70c2ced88efb533ec8dc8c6.
  - Code Refactor: 5fdf93e40f4a85c1951f61ebde815dd5f187074b.
  - Fixes & Groups Functionality: 730ffabc163795795495f900bc5696c52ba02296.

## Final Review

A follow up review was also conducted for the unresolved issues resulting from the initial Mitigation Review that were either noted as Partially Fixed or Not Fixed. The final status can be reviewed in the Mitigation Review Table below.

- The fixes were pushed in the following commits in the Curves Contract:

  - Fixes: 3329f0fa69e27598c202b5fc7f3334b7dbbd36db.
  - Fixes: 21c196b5e6c6504544eccf869bdf3b3f0a68be49.
  - Fixes: c320645c2eaaf7dd0a861952383b81891db4a108.

**All High & Medium issues resulting from the Code4rena audit and from the mitigation review are now fixed.**

***

## Audit Findings Mitigation Review Table

| Issue | Title  | Initial Review Status | Final Review Status |
| ---------- |-------- | ---- | ----- |
| [MR-H-01](#mr-h-01) | Whitelisted accounts can be forcefully DoSed from buying `curveTokens` during the presale | Fixed ✅ | - |
| [MR-H-02](#mr-h-02) | Unrestricted claiming of fees due to missing balance updates in `FeeSplitter` | Not Fixed ❌  | Fixed ✅ |
| [MR-H-03](#mr-h-03) | Attack to make `CurveSubject` to be a `HoneyPot` | Not Fixed ❌ | Fixed ✅ |
| [MR-H-04](#mr-h-04) | Unauthorized Access to `setCurves` Function | Fixed ✅ | - |
| [MR-M-01](#mr-m-01) | Protocol and referral fee would be permanently stuck in the Curves contract when selling a token | Partially Fixed ❌ | Fixed ✅ |
| [MR-M-02](#mr-m-02) | Theft of holder fees when `holderFeePercent` was positive and is set to zero | Fixed ✅ | - |
| [MR-M-03](#mr-m-03) | If a user sets their curve token symbol as the default one plus the next token counter instance it will render the whole default naming functionality obsolete | Fixed ✅ | - |
| [MR-M-04](#mr-m-04) | Withdrawing with amount `= 0` will forcefully set name and symbol to default and disable some functions for token subject | Not Fixed ❌ | Fixed ✅ |
| [MR-M-05](#mr-m-05) | Stuck rewards in `FeeSplitter` contract | Not Fixed ❌ | Fixed ✅ |
| [MR-M-06](#mr-m-06) | A subject creator within a single block can claim holder fees without holding due to unprotected reentrancy path | Fixed ✅ | - |
| [MR-M-07](#mr-m-07) | Selling will be bricked if all other tokens are withdrawn to ERC20 token | Fixed ✅ | - |
| [MR-M-08](#mr-m-08) | Single token purchase restriction on curve creation enables sniping | Fixed ✅ | - |
| [MR-M-09](#mr-m-09) | `Curves::_buyCurvesToken()`, Excess of Eth received is not refunded back to the user | Partially Fixed ❌ | Fixed ✅ |
| [MR-M-10](#mr-m-10) | `onBalanceChange` causes previously unclaimed rewards to be cleared | Fixed ✅ | - |
| Bot Finding: [MR-HB-01](#mr-hb-01) | Malformed equate statement | Fixed ✅ | - |

***

## [MR-H-01] Whitelisted accounts can be forcefully DoSed from buying `curveTokens` during the presale<a id="mr-h-01" ></a>

**C4 Original Issue:** [H-01](https://github.com/code-423n4/2024-01-curves-findings/issues/1068)

### Description

Anyone can fill the tokens array of any user preventing them from buying in the presale and increasing the cost of their interactions with the protocol when depositing/withdrawing/...

### Mitigation

`transferAllCurvesTokens()` is updated to remove the subject token from the sender's `ownedCurvesTokenSubjects` list.

`_transfer()` is updated to add the removed subject from the sender to the receiver `ownedCurvesTokenSubjects` list **only** if the transferred subject isn't present in the receiver's `ownedCurvesTokenSubjectsMapping` (done via `_addOwnedCurvesTokenSubject`).

### Conclusion

Fixed.

***

## [MR-H-02] Unrestricted claiming of fees due to missing balance updates in `FeeSplitter`<a id="mr-h-02" ></a>

### Initial Review

**C4 Original Issue:** [H-02](https://github.com/code-423n4/2024-01-curves-findings/issues/247)

#### Description

Anyone that has tokens via direct transfer can claim fees unrestrictedly.

#### Mitigation

A new `_updateBalance()` function is introduced and triggered during all token transfer operations (buying/selling/transfer), where it first saves the unclaimed fees in `tokensData[token].internalClaimedFees[account]`, then updates the token balance, and finally updates the `userFeeOffest` via `FeeSplitter.onBalanceChange()`.

The mitigation is done by introducing a new `internalClaimedFees` key to the `TokenData` struct, where the unclaimed fees of the user is cached before updating his offset when buying and selling subject tokens, but this introduced a new issue as users can infinitely claim rewards as per the following scenario:

  1. A user makes a first purchase of a subject token, where `feeDistributor.beforeBalanceChange()` will be invoked to save the unclaimed fees in the `tokensData[token].internalClaimedFees[account]`, but since this is the first purchase of that user, this value will be zero.
  2. The user makes a second purchase of the same subject token, and in this time the `tokensData[token].internalClaimedFees[account]` will be initiated with the unclaimed fees.
  3. When the user claims his fees, there will be always a claimable value equal to the last `tokensData[token].internalClaimedFees[account]` as it's not reset upon fees claiming.

PoC: please refer to the `testUsersCanClaimFeesInfinetly()` test in the [following gist](https://gist.github.com/DevHals/d9f6613036b614e2be001a1c411e60c5).

#### Conclusion

Not Fixed.

#### Recommendation

1. Update `FeeSplitter._claimFees()` to reset the `internalClaimedFees` of the subject token:

```diff
    function _claimFees(address token, address account) internal {
        updateFeeCredit(token, account);
        uint256 claimable = getClaimableFees(token, account);
        if (claimable == 0) revert NoFeesToClaim();
        tokensData[token].unclaimedFees[account] = 0;
+       tokensData[token].internalClaimedFees[account] = 0;
        payable(account).transfer(claimable);
        emit FeesClaimed(token, account, claimable);
    }
```

2. Update `FeeSplitter.batchClaiming()` to reset the `internalClaimedFees` of the subject token:

```diff
    function batchClaiming(address[] calldata tokenList) external nonReentrant {
        for (uint256 i = 0; i < tokenList.length; i++) {
            address token = tokenList[i];
            if (tempCheckDuplicate[token]) revert InvalidTokenList();
            tempCheckDuplicate[token] = true;
        }

        uint256 totalClaimable = 0;
        for (uint256 i = 0; i < tokenList.length; i++) {
            address token = tokenList[i];
            tempCheckDuplicate[token] = false;

            updateFeeCredit(token, msg.sender);
            uint256 claimable = getClaimableFees(token, msg.sender);
            if (claimable > 0) {
                tokensData[token].unclaimedFees[msg.sender] = 0;
+               tokensData[token].internalClaimedFees[msg.sender] = 0;
                totalClaimable += claimable;
                emit FeesClaimed(token, msg.sender, claimable);
            }
        }
        if (totalClaimable == 0) revert NoFeesToClaim();
        payable(msg.sender).transfer(totalClaimable);
    }
```

3. Recommended to use `call` method instead of `transfer` method to send claimable fees to the user.

### Final Review

The previous mitigation introduced a new vulnerability where users can claim fees infinitely as the `tokensData[token].internalClaimedFees[account]` isn't reset upon claiming.

#### Mitigation

`FeeSplitter._claimFees()` & `FeeSplitter.batchClaiming()` were updated to reset the `internalClaimedFees` of the subject token upon claiming.

`call` method is used instead of `transfer` method to send claimable fees to the user in `batchClaiming()`.

Fixed in commits:
- 21c196b5e6c6504544eccf869bdf3b3f0a68be49.
- c320645c2eaaf7dd0a861952383b81891db4a108.

#### Conclusion

Fixed.

***

## [MR-H-03] Attack to make `CurveSubject` to be a `HoneyPot`<a id="mr-h-03" ></a>

### Initial Review

**C4 Original Issue:** [H-03](https://github.com/code-423n4/2024-01-curves-findings/issues/172)

#### Description

`CurveSubject` could be a contract that doesn't accept ether transfers resulting in DoS when users try to sell their tokens (if the subject fee is set then it will receive a percentage of the sold amount).

#### Mitigation

The implemented mitigation replaced `call` method with `transfer` method to send `subjectFee`, but this doesn't resolve the issue, as any malicious `curvesTokenSubject` address could still control the transaction by reverting if it implements a heavy logic upon native token receive that exceeds the default forwarded gas of 2300.

#### Conclusion

Not Fixed.

#### Recommendation

Would recommend using solady's `forceSafeTransferETH()` method to transfer `subjectFee`:

```diff
function _transferFees(
        address curvesTokenSubject,
        bool isBuy,
        uint256 price,
        uint256 amount,
        uint256 supply
    ) internal nonReentrant {
        (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
        {
           //some code...
            {
                //@dev: subject fee
-               payable(curvesTokenSubject).transfer(subjectFee);
+                SafeTransferLib.forceSafeTransferETH(curvesTokenSubject,subjectFee,42000)//forwarded gas value is similar to the used one, could be lowered though
            }
         //some code...
    }
```

### Final Review

The previous mitigation replaced `call` method with `transfer` method to send `subjectFee`, but this doesn't resolve the issue, as any malicious `curvesTokenSubject` address could still control the transaction by reverting if it implements a heavy logic upon native token receive that exceeds the default forwarded gas of 2300.

#### Mitigation

Solady's `forceSafeTransferETH()` method is used to transfer `subjectFee`to the `curvesTokenSubject`.

Fixed in commit: 3329f0fa69e27598c202b5fc7f3334b7dbbd36db.

#### Conclusion

Fixed.

***

## [MR-H-04] Unauthorized Access to `setCurves` Function<a id="mr-h-04" ></a>

**C4 Original Issue:** [H-04](https://github.com/code-423n4/2024-01-curves-findings/issues/4)

### Description

The `setCurves` function is unrestricted and can be accessed by anyone to change the address of the Curves contract.

### Mitigation

`onlyOwner` modifier is added to the `setCurves()` function to restrict accessing it for the contract owner only.

### Conclusion

Fixed.

***

## [MR-M-01] Protocol and referral fee would be permanently stuck in the Curves contract when selling a token<a id="mr-m-01" ></a>

### Initial Review 

**C4 Original Issue:** [M-01](https://github.com/code-423n4/2024-01-curves-findings/issues/1294)

#### Description

The protocol fees will be permanently stuck in the contract when **selling** as these fees will not be transferred to any address when selling/only when buying curves tokens.

The referral fees will be permanently stuck in the contract if the referral isn't set for a subject token (as it will be deducted from the price but not sent to any address).

#### Mitigation

`_transferFees()` function is updated so that the protocol will mainly receive the `protocolFee` upon selling and buying of curves tokens, and the `referralFee` will be sent to the protocol if the subject's referral isn't defined.

#### Conclusion

Partially Fixed.

#### Recommendation

The case where `feeRedistributor` is not set yet (`== address(0)`) and the `holderFee` is `> 0`; the `holderFee` will be deducted from the price and stuck in the contract, would recommend updating the `_transferFees` to send the `holderFee` to the protocol if the `feeRedistributor` address isn't set:

```diff
function _transferFees(
        address curvesTokenSubject,
        bool isBuy,
        uint256 price,
        uint256 amount,
        uint256 supply
    ) internal nonReentrant {
        (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
        {
            //some code...

            //@dev: holders fee
            {
                if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
                    if (
                        curvesTokenSupply[curvesTokenSubject] == curvesTokenBalance[curvesTokenSubject][address(this)]
                    ) {
                        (bool success, ) = feesEconomics.protocolFeeDestination.call{value: holderFee}("");
                        if (!success) revert CannotSendFunds();
                    } else {
                        feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
                    }
+               }else if (feesEconomics.holdersFeePercent > 0 ){
+                       (bool success, ) = feesEconomics.protocolFeeDestination.call{value: holderFee}("");
+                       if (!success) revert CannotSendFunds();
+               }
            }

        //some code...
    }
```

### Final Review

The previous mitigation doesn't handle the case where `feeRedistributor` is not set yet (`== address(0)`) and the `holderFee` is `> 0`; which will result in the `holderFee` being stuck in the contract.

#### Mitigation

The aforementioned case is handled by updating `CurvesBase._transferFees()` function to send the `holderFee` to the `protocolFeeDestination` address if `feeRedistributor` address is not set.

Fixed in commit: 3329f0fa69e27598c202b5fc7f3334b7dbbd36db.

#### Conclusion

Fixed.

***

## [MR-M-02] Theft of holder fees when `holderFeePercent` was positive and is set to zero<a id="mr-m-02" ></a>

**C4 Original Issue:** [M-02](https://github.com/code-423n4/2024-01-curves-findings/issues/895)

### Description

A holder fee is imposed on all buy and sell transactions, and if it's set to zero after being non-zero it will open the door for subject holders to claim these fees.

### Mitigation

A new `_updateBalance()` function is introduced, where it first saves the unclaimed fees via `feeRedistributor.beforeBalanceChange()`, then updates curves token balance, and finally updates the `userFeeOffest` via `FeeSplitter.onBalanceChange()`, so the `userFeeOffest` will be always updated regardless of the value of `holdersFeePercent` being zero or not.

### Conclusion

Fixed.

***

## [MR-M-03] If a user sets their curve token symbol as the default one plus the next token counter instance it will render the whole default naming functionality obsolete<a id="mr-m-03" ></a>

**C4 Original Issue:** [M-03](https://github.com/code-423n4/2024-01-curves-findings/issues/647)

### Description

The default naming functionality would be DoS'd if any subject names its token symbol similar to the next default token symbol (CURVE N).

### Mitigation

Mitigated by introducing a new `startsWith()` function to check that the assigned subject's symbol before external token deployment is not similar to the default symbol.

### Conclusion

Fixed.

***

## [MR-M-04] Withdrawing with amount `= 0` will forcefully set name and symbol to default and disable some functions for token subject<a id="mr-m-04" ></a>

### Initial Review

**C4 Original Issue:** [M-04](https://github.com/code-423n4/2024-01-curves-findings/issues/608)

#### Description

`withdraw()` function doesn't check if the withdrawn amount is zero thus enabling anyone that doesn't have any amount of that subject token to deploy the token with the default name and symbol if these haven't been set by the subject creator.

#### Mitigation

This issue is not mitigated, as anyone can still invoke the `withdraw()` function with zero amount (might be invoked by a non-subject token holder) and deploying the external curves token contract with the default name and symbol if these were not set by the subject's creator.

#### Conclusion

Not Fixed.

#### Recommendation

Would recommend enforcing external subject contract deployment upon subject creation, and/or prevent calling `withdraw()` function with amount `= 0`.

### Final Review

This previous mitigation doesn't resolve the issue , as anyone can still invoke the `withdraw()` function with zero amount and deploying the external curves token contract with the default name and symbol if these were not set by the subject's creator.

#### Mitigation

`Curves.withdraw()` function is updated to prevent withdrawing with amount `=0`.

Fixed in commit: 3329f0fa69e27598c202b5fc7f3334b7dbbd36db.

#### Conclusion

Fixed.

***

## [MR-M-05] Stuck rewards in `FeeSplitter` contract<a id="mr-m-05" ></a>

### Initial Review

**C4 Original Issue:** [M-05](https://github.com/code-423n4/2024-01-curves-findings/issues/403)

#### Description

Stuck native tokens in the `FeeSplitter` contract that are resulted from rounding down in `addFees()` function.

#### Mitigation

This issue is not mitigated, as the `_updateBalance()` function that is called when users withdraw their subject tokens will result in `Curves` contract claimable fees to be accumulated and never withdrawn as the contract doesn't implement any function/method to withdraw these fees, also native tokens resulted from rounding down in `feeSplitter.addFees()` will be stuck.

PoC: please refer to the `testStuckCurvesClaimableFees()` test in the [following gist](https://gist.github.com/DevHals/d9f6613036b614e2be001a1c411e60c5).

#### Conclusion

Not Fixed.

#### Recommendation

In `Curves` contract: add a function that enables the contract owner from claiming subject fees due to external tokens withdrawals.

In `FeeSplitter` contract: add a function that enables the contract owner from withdrawing stuck fees due to rounding down.

### Final Review

The previous mitigation doesn't resolve the issue, as it doesn't introduce a function to enable claiming fees accumulated for the curves contract and a rescue function to withdraw stuck fees due to rounding down in the `FeeSplitter` contract.

#### Mitigation

A new `CurvesBase.claimFees()` function is added to enable the contract owner from withdrawing claimable fees of the curves contract.

A new mechanism is implemented in the `FeeSplitter` contract to rescue the stuck fees by adding a new `totalClaimableFees` state variable that tracks the total fees when added and withdrawn, and the difference between this variable and the contract's balance can be rescued by the owner via `FeeSplitter.rescue()` function.

Fixed in commit: 21c196b5e6c6504544eccf869bdf3b3f0a68be49.

#### Conclusion

Fixed.

***

## [MR-M-06] A subject creator within a single block can claim holder fees without holding due to unprotected reentrancy path<a id="mr-m-06" ></a>

**C4 Original Issue:** [M-06](https://github.com/code-423n4/2024-01-curves-findings/issues/386)

### Description

A subject creator can keep on claiming holder fees on every buy and sell transaction even when he doesn't hold the balance as a result of re-entrancy in `_transferFees()` function because the `feeRedistributor.onBalanceChange()` is called after the call is made to transfer the `subjectFee` for the `curvesTokenSubject` address.

### Mitigation

A `nonReentrant` modifier is added to the `_transferFess()` function that is called when buying/selling subject tokens.

`_buyCurvesToken()` & `sellCurvesToken()` functions are updated to invoke `feeRedistributor.onBalanceChange()` before transferring fees (where the external call to transfer fees for the `curvesTokenSubject` is made).

### Conclusion

Fixed.

***

## [MR-M-07] Selling will be bricked if all other tokens are withdrawn to ERC20 token<a id="mr-m-07" ></a>

**C4 Original Issue:** [M-07](https://github.com/code-423n4/2024-01-curves-findings/issues/378)

### Description

`FeeSplitter.addFees()` reverts if the `totalSupply` of the token is `0`, thus preventing withdrawing the other bought/deposited tokens.

### Mitigation

The `_transferFees` function is updated to check if the Curves contract balance matches the `totalSupply` of the subject tokens, and if so, the `holderFee` is sent to the protocol, and the `feeRedistributor.addFees()` will not be called in this case (when all subject tokens are withdrawn).

`feeRedistributor.addFees()` is updated to not revert when the totalSupply of the subject token is zero.

### Conclusion

Fixed.

***

## [MR-M-08] Single token purchase restriction on curve creation enables sniping<a id="mr-m-08" ></a>

**C4 Original Issue:** [M-08](https://github.com/code-423n4/2024-01-curves-findings/issues/243)

### Description

Subjects creators are restricted to buying only one token when initializing their subjects.

### Mitigation

The issue is mitigated by updating the `CurvesBase.getPrice()` function to allow the first purchase made by the subject owner to be `> 1`.

### Conclusion

Fixed.

***

## [MR-M-09] `Curves::_buyCurvesToken()`, Excess of Eth received is not refunded back to the user<a id="mr-m-09" ></a>

### Initial Review 

**C4 Original Issue:** [M-09](https://github.com/code-423n4/2024-01-curves-findings/issues/48)

#### Description

Users can send more than required native tokens to buy subject tokens, and they will not be refunded the extra sent amount.

#### Mitigation

`_buyCurvesToken()` function is updated to refund the extra `msg.value` to the buyer via `transfer()` method.

#### Conclusion

Partially Fixed.

#### Recommendation

Would recommend sending the refund via `call` method instead of using `transfer` method (no re-entrancy will be introduced here as the function follows the check-effect-interaction mechanism before making the external call):

```diff
  function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);

        if (msg.value < price + totalFee) revert InsufficientPayment();

        _updateBalance(curvesTokenSubject, msg.sender, curvesTokenBalance[curvesTokenSubject][msg.sender] + amount);
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        //@dev: refund the excess
        if (msg.value > price + totalFee) {
-           payable(msg.sender).transfer(msg.value - price - totalFee);
+                       (bool success, ) = msg.sender.call{value: msg.value - price - totalFee}("");
+                       if (!success) revert CannotSendFunds();
        }

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
    }
```

### Final Review

The previous mitigation uses `transfer` method to send the refund for the buyer.

#### Mitigation

`_buyCurvesToken()` function is updated to send the refund to the buyer via `call()` method.

Fixed in commit: 3329f0fa69e27598c202b5fc7f3334b7dbbd36db.

#### Conclusion

Fixed.

***

## [MR-M-10] `onBalanceChange` causes previously unclaimed rewards to be cleared<a id="mr-m-10" ></a>

**C4 Original Issue:** [M-10](https://github.com/code-423n4/2024-01-curves-findings/issues/39)

### Description

Users will lose their unclaimed rewards as the `onBalanceChange()` function does not help to claim the previous rewards, but directly resets `userFeeOffset`, causing the user's unclaimed rewards to be cleared with each buying or selling of subject tokens.

### Mitigation

A new mechanism is introduced to save the unclaimed fees by introducing a new `_updateBalance()` function that is triggered during all token transfer operations (buying/selling/transfer), where it first saves the unclaimed fees, then updates token balance, and finally updates the `userFeeOffest` via `FeeSplitter.onBalanceChange()`.

This mitigation introduced a vulnerability described in [MR-H-02](#mr-h-02), so resolving MR-H-02 will complete the mitigation of this issue, but would consider this issue as fixed since it introduced the logic to save the unclaimed fees before balance change.

### Conclusion

Fixed.

***

## [MR-HB-01] Malformed equate statement<a id="mr-hb-01" ></a>

**C4 Original Issue:** [HB-01](https://github.com/code-423n4/2024-01-curves/blob/main/bot-report.md#h-01)

### Description

Access control modifiers don't enforce access control due to a missing check, allowing anyone to break them.

### Mitigation

Access control modifiers are updated with a correct check mechanism to grant access for intended privileged addresses only.

### Conclusion

Fixed.

***

# Disclosures
C4 is an open organization governed by participants in the community.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
