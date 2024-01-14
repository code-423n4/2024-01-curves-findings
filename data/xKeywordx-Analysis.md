### [L-1] `FeeSplitter::claimFees` and `FeeSplitter::batchClaiming` functions can revert.

**Description:** If the caller of these functions is a smart contract that has a `receive()` or `fallback()` function that `reverts()` then the transaction will fail and the user will lose gas.

**Impact:** User will not be able to withdraw funds from the contract. Assets will not be lost though. The user can send the funds to a different address using one of the following functions `Curves::transferCurvesToken` or `Curves::transferAllCurvesTokens` and then withdraw the funds from the new account, but this can lead to some bad user experience.

**Recommended Mitigation:** Pull over push. Implementing a withdraw pattern where users pull their funds, rather than the contract pushing funds to them, can mitigate this issues.

### [L-2] Potential griefing attack by reverting `FeeSplitter::claimFees` and `FeeSplitter::batchClaiming` functions

**Description:** The `FeeSplitter::claimFees` and `FeeSplitter::batchClaiming` functions in the `FeeSplitter` contract are vulnerable to a potential griefing attack. This attack can occur if an external contract, designed to call these functions, has a receive function that deliberately reverts. This will causes the Ether transfer in `FeeSplitter::claimFees` or `FeeSplitter::batchClaiming` to fail, leading to a failed transaction.

**Impact:** While this issue does not directly risk the loss of assets or funds within the `FeeSplitter` contract, it can result in failed transactions for the specific contract initiating the call. This could lead to loss of gas, potentially impacting the user experience. However, the overall functionality of the `FeeSplitter` contract and other users' interactions remain unaffected.

**Recommended Mitigation:** Pull over push. Implementing a withdraw pattern where users pull their funds, rather than the contract pushing funds to them, can mitigate this issues.

### [NC-1] Function `Curves::buyCurvesToken` may fail

**Description:** If `startTime` is set to the exact time the presale is intended to start, there's a possibility of a race condition due to block time variability. Ethereum blocks are targeted to be mined approximately every 15 seconds, but this is not exact. The actual block time can vary slightly.

**Impact:** If a transaction is sent to call `buyCurvesToken` right at startTime, it might get included in a block slightly before and the transaction will fail, causing the user to lose gas and not buy the tokens that he wants.

**Recommended Mitigation:** Change the `startTime` to be strictly greater than block.timestamp. Like this you're allowing transactions to proceed in any block mined after the `startTime` has passed. This approach is more forgiving of the slight variability in block times and should provide a smoother experience for users.

```diff
    function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
        uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
-       if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
+       if (startTime != 0 && startTime > block.timestamp) revert SaleNotOpen();
        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

### [NC-2] Missing events in functions that change state variables

**Description:** Functions that change the state of the contract are not emitting events.

**Impact:** Hard to track smart contract activity, reducing transparency.

**Recommended Mitigation:** Add events emission within these functions: `setFeeRedistributor`, `setProtocolFeePercent`, `setExternalFeePercent`, `setReferralFeeDestination`.


### Time spent:
8 hours