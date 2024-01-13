
 |no. |Issue
 |-|:-
| [[L-01](#l-01)] | Curves contract doesn't refund the excess ether on buyCurvesToken transaction|
| [[L-02](#l-02)] | A manager has the power to make protocol earn zero fees|
| [[L-03](#l-03)] | Duplicate pushes to the storage of `FeesSplitter::userTokens` |


### [L-01]<a name="l-01"></a> Curves contract doesn't refund the excess ether on [Curves::buyCurvesToken](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263) transaction

### Impact
- When a user buys a curve with [Curves::buyCurvesToken](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263), it is possible to call with excess etehr, and the check [if (msg.value < price + totalFee) revert InsufficientPayment();](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L270) validates only if sent lowe than price, it will revert.
- But no where in the transaction, the excess etehr is refunded. 
- And the excess ether is jsut lost permanently inside the contract.

### Recommendation

- Modify [Curves::buyCurvesToken](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263) as shown below.

```diff
    function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
        uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
        if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

        _buyCurvesToken(curvesTokenSubject, amount);
    }

    function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);

        if (msg.value < price + totalFee) revert InsufficientPayment();
+       uint refund = msg.value - (price + totalFee);

        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }

+       if(refund > 0) payable(msg.sender).transfer(refund);
    }

```


### [L-02]<a name="l-02"></a> A manager has the power to make protocol earn zero fees

### Impact
- A manager can make te protocol fee zero, by doing the following actions
    1. if current protocol fees is 0.5ether (5 %), then call [Curves::setMaxFeePercent] with max value as 10e18 instead of previous value 1e18.
    2. Now all the fees are cut by 10 times, because we increases the precision from 1e18 to 10e18.
    3. Now the manager should also increase the subject and referral/holder fees by 10 times. But doesnt have the access to set protocol fees.
    4. This way, by changing the max value, the currently set protocol fees can be made 0 or negligable dust.

### code

```solidity
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
        if (
            feesEconomics.protocolFeePercent +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            maxFeePercent_
        ) revert InvalidFeeDefinition();
        feesEconomics.maxFeePercent = maxFeePercent_;
    }

    function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
        if (
            protocolFeePercent_ +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            feesEconomics.maxFeePercent ||
            protocolFeeDestination_ == address(0)
        ) revert InvalidFeeDefinition();
        feesEconomics.protocolFeePercent = protocolFeePercent_;
        feesEconomics.protocolFeeDestination = protocolFeeDestination_;
    }

    function setExternalFeePercent( 
        uint256 subjectFeePercent_,  uint256 referralFeePercent_,  uint256 holdersFeePercent_
    ) external onlyManager {
        if (
            feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ >
            feesEconomics.maxFeePercent
        ) revert InvalidFeeDefinition();
        feesEconomics.subjectFeePercent = subjectFeePercent_;
        feesEconomics.referralFeePercent = referralFeePercent_;
        feesEconomics.holdersFeePercent = holdersFeePercent_;
    }
```

### Recommendation
- Do not allow the manager to set max fee percent value.

```diff
-   function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
+   function setMaxFeePercent(uint256 maxFeePercent_) external onlyOwner {
        if (
            feesEconomics.protocolFeePercent +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            maxFeePercent_
        ) revert InvalidFeeDefinition();
        feesEconomics.maxFeePercent = maxFeePercent_;
    }
```




### [L-03]<a name="l-03"></a> Duplicate pushes to the storage of [FeesSplitter::userTokens](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L29)

### Impact

```solidity
    function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }
```

- This function [FeesSplitter::onBalanceChange](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L99) is called on every buy/sell of the curveTokens.
- But pushed to the array storage [`mapping(address => address[]) internal userTokens;`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L29)
- But its not validated for duplicates, just checks if balance is > 0, which mostly be true.

### Recommendation

```diff

    function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;

-       if (balanceOf(token, account) > 0) userTokens[account].push(token);
+      if (balanceOf(token, account) > 0) {
+           address[] storage tokens = userTokens[account];
+           for (uint256 i = 0; i < tokens.length; i++) {
+               if (tokens[i] == token) {
+                   return;
+               }
+           }
+           userTokens[account].push(token);
+       } 
    }

```