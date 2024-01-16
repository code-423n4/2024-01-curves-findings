
### [GAS-1] doesn't need to invoke getClaimableFees in `claimFees`

#### Impact:
It's already updated the claimableFees in `updateFeeCredit`, so it doesn't need to call `getClaimableFees`. Instead, use `uint256 claimable = tokensData[token].unclaimedFees[msg.sender]`

#### Vulnerability Detail


```solidity
File: contracts/FeeSplitter.sol:L82
    function claimFees(address token) external { 
        updateFeeCredit(token, msg.sender);
        uint256 claimable = getClaimableFees(token, msg.sender); // @audit gas claimable = tokensData[token].unclaimedFess[msg.sender]
        if (claimable == 0) revert NoFeesToClaim();
        tokensData[token].unclaimedFees[msg.sender] = 0;
        payable(msg.sender).transfer(claimable);
        emit FeesClaimed(token, msg.sender, claimable);
    }
```


#### Recommendation

Recommend using the tokensData[token].unclaimedFees[msg.sender] instead of invoking the getClaimableFees