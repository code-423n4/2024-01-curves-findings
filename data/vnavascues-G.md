## GAS-001 First token buy innecessary executes \_transferFees

When a token subject buys its token for the first time the transaction incurs an unnecessary gas overhead due to executing [`_transferFees`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L218:L261) logic despite price was 0 and there were no protocol and holder fees.

### Recommended Mitigation Steps

Do not execute `_transferFees` if `curvesTokenSupply` for the given token subject address is zero.