**1** **LOW** - In `FeeSplitter` user will lose fees if he `buyCurvesToken` or `sellCurvesToken` if he hasn't claimed it beforehand.

**For example**
In `FeeSplitter`:

`data.userFeeOffset[account]` = 100;
`data.cumulativeFeePerToken` = 200;

The moment user calls `buyCurvesToken` or `sellCurvesToken` it will invoke `onBalanceChange`. Which: 

```
function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }
```

Will set the `data.userFeeOffset` to the current `data.cumulativeFeePerToken`. So if a user doesn't call `claimFees` before buying more tokens he will lose fees.


**2** **LOW** - It's possible that no buying of tokens can occur in `Curves.sol` for one block. That occurs because of the `if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();` in `buyCurvesToken` and

 ```
if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();
```

 in `buyCurvesTokenWhitelisted`.

As you can see both require `startTime` to not be in a state where it is == block.timestamp.


