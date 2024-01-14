QA1. The ``maxFeePercent_`` should be less than 1e18. Otherwise, it is not reasonable. 

[https://github.com/code-423n4/2024-01-curves/blame/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117-L126])https://github.com/code-423n4/2024-01-curves/blame/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117-L126)

QA2. addFees(): The following numerator uses scaling of ``PRECISION`` for the denominator in line ``data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_;``. This is actually not sufficient since it will be canceled by the 
precision in the denominator: totalsupply_ has ``PRECISION`` too. A new scaling is needed to avoid loss of PRECISION. 

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L89-L94](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L89-L94)

Mitigation: use SCALING = 1e36 instead. 

QA3. UpdateFeeCredit() will not update ``data.userFeeOffset[account]`` by ``data.cumulativeFeePerToken``. Meanwhile, the rewards is based on the the difference between ``data.userFeeOffset[account]`` and``data.cumulativeFeePerToken``. That means, even when the balance of an account is zero, one can still RETROACTIVELY claim rewards for the PERIOD when the balance is zero since during that period, ``data.cumulativeFeePerToken`` is advanced, but not ``data.userFeeOffset[account]``.  

```javascript
   function updateFeeCredit(address token, address account) internal {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
        if (balance > 0) {
            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
            data.unclaimedFees[account] += owed / PRECISION;
            data.userFeeOffset[account] = data.cumulativeFeePerToken;
        }
    }
```

Mitigation: 
The update ``data.userFeeOffset[account] = data.cumulativeFeePerToken;`` still needs to be performed even when ``balance == 0``. 

QA4. A referral contract might DOS the trading of a particular CurveTokens. 
This is because the trading of a CurvesToken will send referral fee to a referal account, which might be an EOA or a contract. In the case of later, a referral can set the contract so that it will reject the receiving of native tokens. As a result, the following _transferFees will always abort. As a result, all trading for that CurveTokens will fail. 

```javascript
function _transferFees(
        address curvesTokenSubject,
        bool isBuy,
        uint256 price,
        uint256 amount,
        uint256 supply
    ) internal {
        (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
        {
            bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
            {
                address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
                uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
                uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
                (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
                if (!success1) revert CannotSendFunds();
            }
            {
                (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
                if (!success2) revert CannotSendFunds();
            }
            {
                (bool success3, ) = referralDefined
L241                  ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                    : (true, bytes(""));
                if (!success3) revert CannotSendFunds();
            }

            if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
            }
        }
        emit Trade(
            msg.sender,
            curvesTokenSubject,
            isBuy,
            amount,
            price,
            protocolFee,
            subjectFee,
            isBuy ? supply + amount : supply - amount
        );
    }
```

In summary, L241 might be failed by the referral contract. 

QA5. For the Curves contract, there are several mappings that map from subjects to other components. They can be combined into one mapping that maps subjects to a struct. 

```javascript
struct InternalTokenMetadata{
   ExternalTokenMeta externalTokenMeta;
   PresalesMeta presalesMeta;
   mapping(address => uint256) presalesBuys;
   address referralFeeDestination, 
   mapping(address => uint256) curvesTokenBalance;
   uint256 curvesTokenSupply
}
```

QA6. Curves._transferFees() fails to send the protocol fee to feesEconomics.protocolFeeDestination the sell case. The protocol fee will remain in the contract for this case. 

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218-L261](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218-L261)

Mitigation: calculate the protocol fee for both cases, and make sure the protocol fee is sent to feesEconomics.protocolFeeDestination for each case. 