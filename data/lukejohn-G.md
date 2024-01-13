G1. Curves.mint(): There is no need to assign the DEFAULT value to ``externalCurvesTokens`` since they will be updated again inside _mint(). One can save gas by simply passing the two default name and symbol to _mint without saving them to the ``externalCurvesTokens`` state variable first. 

```javascript
    function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
        if (
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
            keccak256(abi.encodePacked("")) ||
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
            keccak256(abi.encodePacked(""))
        ) {
            _mint(curvesTokenSubject, DEFAULT_NAME, DEFAULT_SYMBOL);
            return;
        }

        _mint(
            curvesTokenSubject,
            externalCurvesTokens[curvesTokenSubject].name,
            externalCurvesTokens[curvesTokenSubject].symbol
        );
    }
```

G2. We can short circuit for the case of balance = 0 to save gas below: 

```diff
 function getClaimableFees(address token, address account) public view returns (uint256) {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
+       if(balance == 0) return 0;

        uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
        return (owed / PRECISION) + data.unclaimedFees[account];
    }

```

Q3. claimFees() can be reimplemented to save gas - eliminate the call of getClaimableFees() as follows:

```diff
    function claimFees(address token) external {
        updateFeeCredit(token, msg.sender);
-        uint256 claimable = getClaimableFees(token, msg.sender);
+       uint256 claimable = tokensData[token].unclaimedFees[msg.sender]
        if (claimable == 0) revert NoFeesToClaim();

        tokensData[token].unclaimedFees[msg.sender] = 0;
        payable(msg.sender).transfer(claimable);
        emit FeesClaimed(token, msg.sender, claimable);
    }

```