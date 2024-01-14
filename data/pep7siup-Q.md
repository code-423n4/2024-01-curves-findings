## Unrefunded Excess ETH After Curve Token Purchase

### Impact
It allows excess ETH to remain unrecovered after the purchase of Curve tokens. While not critical, this issue could lead to financial discrepancies and affect user experience.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L280

```solidity
// File: contracts/Curves.sol
263:    function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        ...
278:            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
279:        }
280:    }// <= FOUND: excess ETH after buy Curve token not refunded
```


## Inflated `userTokens` Array in `getUserTokensAndClaimable`

### Impact
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L99

```solidity
// File: contracts/FeeSplitter.sol
96:    function onBalanceChange(address token, address account) public onlyManager {
97:        TokenData storage data = tokensData[token];
98:        data.userFeeOffset[account] = data.cumulativeFeePerToken;
99:        if (balanceOf(token, account) > 0) userTokens[account].push(token); // <= FOUND: inflated userTokens
100:    }
```

Everytime `onBalanceChange` is called, the `token` is pushed to `userTokens` without checking its existence. This issue leads to an inflated array size, potentially causing increased gas costs (for example, third parties relying on the `getUserTokensAndClaimable` function) and requiring additional steps to filter out duplicated tokens. 


### Recommended Mitigation Steps
[OpenZeppelin's EnumerableSet](https://docs.openzeppelin.com/contracts/3.x/api/utils#EnumerableSet) is suggested as a suitable alternative to efficiently manage unique tokens without the risk of inflation.


## Unfair Holders Fee Calculation After FeeRedistributor Compromise

### Impact
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L176

```solidity
// File: contracts/Curves.sol
166:    function getFees(
        ...
174:        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
175:        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
176:        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;// <= FOUND: user still has to pays holderFee if setFeeRedistributor to address zero
177:        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
178:    }
```

In the event of an emergency compromise where the `feeRedistributor` contract is set to address 0x0, the administrator may overlook resetting the `feesEconomics.holdersFeePercent` or just hasn't doing so in time. In the mean time, any buy/sell attempts would still accounting for the `holdersFee` which could leads to user spending more than they should

### Recommended Mitigation Steps
To mitigate this vulnerability and ensure fair fee calculations after the `feeRedistributor` contract compromise, it is recommended to include a check for the `feeRedistributor` address before calculating the `holdersFee`. If the `feeRedistributor` is set to address 0x0, the `holdersFee` should be set to 0 to prevent users from being unfairly charged.