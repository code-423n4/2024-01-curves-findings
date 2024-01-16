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

### Known bug in the contract
According to medium docs, there is a bug in getPrice()
```
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        uint256 sum1 = supply == 0 ? 0 : (supply - 1 )* (supply) * (2 * (supply - 1) + 1) / 6;
        uint256 sum2 = supply == 0 && amount == 1 ? 0 : (supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1) / 6;
        uint256 summation = sum2 - sum1;
        return summation * 1 ether / 16000;
    }
```
Pay attention to the second line of code
```
uint256 sum2 = supply == 0 && amount == 1 ? 0 : (supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1) / 6;

```
When the market opens, the supply is 0. If the purchase quantity is not 1, it will execute supply — 1 + amount, causing supply — 1 to be -1, and uint256 is unsigned and there is no negative, it will trigger a runtime error. Cause transaction error.
