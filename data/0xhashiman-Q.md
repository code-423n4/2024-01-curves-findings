## Impact
The `claimFees()` and `batchClaiming()` functions in `FeeSplitter`.sol have two issues. One, which is not valid according to C4 BotRace, involves the use of transfer instead of call due to gas . The second issue, not addressed in the race, is that these functions do not check the return value of the low-level call. Consequently, if the call fails, the entire transaction will not revert.

## Proof of Concept

I assume that the project instead of using transfer they uses the other way which is call but the problem is that this low level call will return a boolean value indicating whether the call was successful. However, it is important to note that this return value is not being checked in the current implementation.

There's a chance that the call might not succeed, and the transaction proceeds without reverting. I'm categorizing this as a low-level concern, leaving it to the judges to determine whether it qualifies as a medium-level issue or not.

Code Reference:

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L80-L87
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L103-L117

## Tools Used
Manual Review

## Recommended Mitigation Steps

It is recommended to ensure that the return value of the low-level call is checked. If the call is unsuccessful, the protocol should revert the entire transaction. The suggested modification is as follows:

```solidity
(bool success, ) = payable(recipient()).call{value: amount}("");
require(success,"Low level call failed");
```
