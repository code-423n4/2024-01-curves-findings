### Summary

* \[L-01\] Creators who bought more than 1 token for themselves in the first transaction can not change the whitelist
    
* \[L-02\] `startTime` for buying curves token should be inclusive
    
* \[L-03\] `ownedCurvesTokenSubjects` always added but never removed even if the user sells all of their tokens.
    
* \[L-04\] `onBalanceChange` function pushes `userTokens` array without checking if it already has the same token
    

---

### \[L-01\] Creators who bought more than 1 token for themselves in the first transaction can not change the whitelist

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394C1-L402C6](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394C1-L402C6)

```solidity
    function setWhitelist(bytes32 merkleRoot) external {
        uint256 supply = curvesTokenSupply[msg.sender];
-->     if (supply > 1) revert CurveAlreadyExists(); //@audit I assume it is done this way to consider the creator's initial share. It will revert if the creator bought more than 1 in the first transaction. 

        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
            presalesMeta[msg.sender].merkleRoot = merkleRoot;
            emit WhitelistUpdated(msg.sender, merkleRoot);
        }
    }
```

This protocol has a presale and whitelist feature for subject creators. As we can see above, the `setWhitelist` function checks whether the `supply > 1` or not. Allowing supply to be `1` before setting the whitelist is probably done to let the creator to buy his/her subject for the first time.

For a subject to be on sale, the account itself must buy the first token, and this check is done [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L265). However, there is no restriction in terms of how many tokens can the creator buy.

So, if the creator bought more than 1 token in the first transaction, he/she can not set the whitelist even if he/she is the only owner at that moment.

This function should check the balance of the creator instead of `1`.

**Recommendation**:

```diff
-    if (supply > 1) revert CurveAlreadyExists();
+    // subject is himself/herself. 
+    if (supply > curvesTokenBalance[msg.sender][msg.sender]) revert CurveAlreadyExists();
```

---

### \[L-02\] `startTime` for buying curves token should be inclusive

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L213](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L213)

```solidity
    function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
        uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
-->     if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen(); //@audit QA - it should be ">" instead of ">=" 

        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

As we can see above if `startTime == block.timestamp`, the function reverts. However, the buying should be started at that moment.

Besides, in the [`buyCurvesTokenWhitelisted`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L411) function, the presale also reverts if the `startTime == block.timeStamp`

```solidity
        if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();
```

So at the `startTime`, both of the presale and the regular sale reverts. One of them should not be reverted to provide continuity. An unlucky user's transaction might be mined in that perfect timestamp and reverted, which will cause gas and opportunity to buy cheap for that user.

---

### \[L-03\] `ownedCurvesTokenSubjects` always added but never removed even if the user sells all of their tokens

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L328C1-L336C6](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L328C1-L336C6)

```solidity
    function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
        address[] storage subjects = ownedCurvesTokenSubjects[owner_];
        for (uint256 i = 0; i < subjects.length; i++) {
            if (subjects[i] == curvesTokenSubject) {
                return;
            }
        }
        subjects.push(curvesTokenSubject);
    }
```

When a user buys a Curves token or someone transfers a Curves token to a user, that users `ownedCurvesTokenSubjects` is updated. However, that array is never updated when users sell their tokens or transfer all of them to another address.

This will create incorrect querying results for external users and it should be removed if the user doesn't have that token anymore. It may also cause `transferAllCurvesTokens` function to spend much more gas or even return with out of gas error.

---

### \[L-04\] `onBalanceChange` function pushes `userTokens` array without checking if it already has the same token

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96C1-L100C6](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96C1-L100C6)

```solidity
    function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
-->     if (balanceOf(token, account) > 0) userTokens[account].push(token);  //@audit it pushes without checkhing if it is already there
    }
```

This function is called when buying or selling. It directly pushes `userTokens` array without checking if that array already has the same token, which will cause the array to become enormous in time.

Only check is whether the balance is greater than 0. If a user buys the same token again or partially sells an already owned token, array will be pushed unnecessarily.