#### 1) Curves::getClaimableFees
The view only function does have to load tokenData in storage. It is much more efficient to load that in memory.

```
 TokenData storage data = tokensData[token];
```

#### 2) CurvesERC20Factory::deploy
The name and symbol can be passed as callData instead of memory.