## Description
Event reentrancy spotted in `FeeSplitter::claimFees` function 


## Impact
Making payments in a function like claimFees before emitting an event provides several disadvantages. It prevents external observers, such as a dApp, the ability to track and react to specific occurrences within the smart contract.


## Tools Used
Manual review and Slither


## Recommended Mitigation Steps
Emit the FeeSplitter::FeesClaimed` event before making payments
```diff

 function claimFees(address token) external {
        updateFeeCredit(token, msg.sender);
        uint256 claimable = getClaimableFees(token, msg.sender);
        if (claimable == 0) revert NoFeesToClaim();
        tokensData[token].unclaimedFees[msg.sender] = 0;
+       emit FeesClaimed(token, msg.sender, claimable);
+       payable(msg.sender).transfer(claimable);

-        payable(msg.sender).transfer(claimable);
-        emit FeesClaimed(token, msg.sender, claimable);
}
```
