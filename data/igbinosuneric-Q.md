## Title 
Unused External Contract Interface (IERC20) in FeeSplitter.sol

## Description
The IERC20 interface is imported, but there are no explicit interactions with external ERC-20 tokens within the contract.
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L7
```

## Recommendation
If the contract intends to interact with external ERC-20 tokens, ensure that the interactions are implemented correctly. If not needed, consider removing the unnecessary import.