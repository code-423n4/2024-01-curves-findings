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