### **`merkleProof` length not checked in Curves::verifyMerkle**
proof length is not checked, recommend adding a check that proof.length > 0
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L422-L426

### Missing zero address check
```solidity
Curves::setFeeRedistributor
Curves::setReferralFeeDestination
Curves::setERC20Factory
Curves::transferCurvesToken
Curves::transferAllCurvesTokens
FeeSplitter::setCurves
Security::transferOwnership
```

### Unused import
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L7

### Event is missing in important function
Security::transferOwnership

### silent overflow
```solidity
function getSellPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
        uint256 price = getSellPrice(curvesTokenSubject, amount);
        (, , , , uint256 totalFee) = getFees(price);
        // @audit check price >= totalFee, otherwise silent overflow
        return price - totalFee;
    }
```
