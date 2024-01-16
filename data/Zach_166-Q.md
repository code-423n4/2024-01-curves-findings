# Excess ETH transferred by users when calling `buyCurvesToken` will be permanently locked within the contract

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263-L280

In the `Curves` contract, when users invoke the `buyCurvesToken` function to purchase curves tokens with `ETH`, the amount of `ETH` transferred by the user needs to satisfy the condition `msg.value >= price + totalFee`. However, if `msg.value > price + totalFee`, the excess `ETH` will be permanently locked in the `Curves` contract and cannot be withdrawn.

It is recommended that users set the condition as `msg.value == price + totalFee` to prevent the loss caused by users transferring excess ETH into the contract.