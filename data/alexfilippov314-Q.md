**[[1]]** The `Trade` event includes the `protocolFee` and the `subjectFee` but lacks the `referralFee` and the `holdersFee`. This appears incomplete.
```solidity
event Trade(
    address trader,
    address subject,
    bool isBuy,
    uint256 tokenAmount,
    uint256 ethAmount,
    uint256 protocolEthAmount, 
    uint256 subjectEthAmount,
    uint256 supply
);
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L80

**[[2]]** The `Curves.setMaxFeePercent` function enables modification of the `feeEconomics.maxFeePercent`. Implementing restrictions on the magnitude of this value would provide users with a clearer understanding of the potential fee levels.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L125

**[[3]]** `If block.timestamp` equals to `presalesMeta[curvesTokenSubject].startTime`, the user cannot call either `Curves.buyCurvesToken` or `Curves.buyCurvesTokenForPresale`.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L213
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L411

**[[4]]** In the `Curves._deployERC20` function, if the symbol equals "CURVES", a counter is appended to the end of the symbol. However, this approach is somewhat fragile, as anyone can create a token with a symbol like "CURVES5". Consequently, this logic becomes ineffective.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L350

**[[5]]** The `Curves.mint` function uses default name and symbol values if any of these values are empty. However, if the name is not empty while the symbol is, the current implementation will still default to the default name value. A similar issue can be found in the `Curves.withdraw` function.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L440-L448

**[[6]]** The `Curves.withdraw` function contains a block that is nearly identical to the `Curves.mint` function. Perhaps it makes sense to call the `Curves.mint` function here to simplify the code and reduce the contract size.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L470-L483

**[[7]]** The `FeeSplitter.updateFeeCredit` function is internal. Since the protocol follows the convention of using an underscore prefix for internal functions, it would be better to rename it to `_updateFeeCredit`.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63