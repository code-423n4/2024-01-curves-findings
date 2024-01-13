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
