## [QA-1] `buyCurvesToken()` contains an off-by-one issue.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L213

In the following check:
```solidity
if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
```
if the `startTime <= block.timestamp` the sale should be considered open.

**Recommendation:**
Change the check as follows:
```solidity
if (startTime != 0 && startTime > block.timestamp) revert SaleNotOpen();
```

## [QA-2] `buyCurvesTokenForPresale()` contains an off-by-one issue.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L384

In the following check:
```solidity
 if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
```
if the `startTime == block.timestamp` the Presale start time should be valid.

**Recommendation:**
Change the check as follows:
```solidity
if (startTime < block.timestamp) revert InvalidPresaleStartTime();
```

## [QA-3] `buyCurvesTokenWhitelisted()` contains an off-by-one issue.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L411

In the following check:
```solidity
 if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();
```
if the `startTime == block.timestamp` the Presale should be available.

**Recommendation:**
Change the check as follows:
```solidity
if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime < block.timestamp
        ) revert PresaleUnavailable();
```