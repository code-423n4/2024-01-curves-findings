A. Gas Limit Exceedance in `batchClaiming` Function
[Link](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L103-L120)
The `batchClaiming` function iterates through the provided `tokenList` and attempts to update fee credits and claim fees for each token in a single transaction. The gas cost associated with processing a large number of tokens in a loop may surpass the block's gas limit, resulting in transaction revertals.

## Mitigation:
consider implementing a mechanism that allows users to claim fees for a subset of tokens in separate transactions. 