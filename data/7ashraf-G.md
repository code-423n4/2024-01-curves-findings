# Gas Efficiency Report
## Summary

The gas efficiency report highlights two instances of potential gas optimization within the codebase. Below is a summary table outlining the issues and their respective instances:

| **Issue Number** | **Issue Title**         | **Number of Instances** |
|------------------|-------------------------|-------------------------|
| G-01             | Unnecessary Casting     | 1                       |
| G-02             | Unnecessary Check       | 1                       |

These gas efficiency issues point to areas where gas consumption can be optimized for improved performance. Addressing these instances will contribute to reducing unnecessary gas costs in smart contract operations.
## [G-01] Unnecessary casting 
The variable already declared with a variable type address
### Instances
* [Curves.sol #361](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L361)
```solidity
        return address(tokenContract);

```
## [G-02] Unnecessary check
### Instances
The first check `presalesMeta[curvesTokenSubject].startTime == 0` is redundant as `block.timestamp` is already > 0, making it enough to check if it is smaller than `block.timestamp`.
* [Curves.sol #410](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L410)
```solidity
        if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();

```