## [QA-1] wrong modifier on a function

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L247

The `onBalanceChange` function in  the `feeSplitter` contract is protected by the onlyManager modifier, restricting access to addresses with the manager role. The function is called from `Curves.sol` which implys that the contract should have Manager role for it to execute. It is possible to set any address as Manager in the `Security` contract, However, assigning the manager role to external contracts like Curves.sol might lead to vulns
```
 function onBalanceChange(address token, address account) public onlyManager
```

Contracts with the manager role gain access to sensitive functions. Malicious actors could exploit bugs in the contracts and take advantage of  the manager role to further escalate privilages. 

Since `onBalanceChange` is only called from the Curve contract, the access should be restricted to the curve contract. Manager roles should be restricted to only trusted accounts (EOAs).

## [QA-2] Gas Limitations Not Adequately Addressed in batchClaiming

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L103

The `batchClaiming` function iterates through a provided token list, potentially leading to excessive gas consumption if the list is long.
A developer comment suggests using `getUserTokens` to estimate and prepare the batch, which isn't implemented within the function itself. Long token lists could cause the transaction to fail due to gas limitations, preventing users from claiming fees and negatively impacting user experience.