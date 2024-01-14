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

G3. claimFees() can be reimplemented to save gas - eliminate the call of getClaimableFees() as follows:

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

batchClaiming() can be reimplemented to save gas - eliminate the call of getClaimableFees() as follows:

```diff
function batchClaiming(address[] calldata tokenList) external {
        uint256 totalClaimable = 0;
        for (uint256 i = 0; i < tokenList.length; i++) {
            address token = tokenList[i];
            updateFeeCredit(token, msg.sender);
-            uint256 claimable = getClaimableFees(token, msg.sender);
+            uint256 claimable = tokensData[token].unclaimedFees[msg.sender]

            if (claimable > 0) {
                tokensData[token].unclaimedFees[msg.sender] = 0;
                totalClaimable += claimable;
                emit FeesClaimed(token, msg.sender, claimable);
            }
        }
        if (totalClaimable == 0) revert NoFeesToClaim();
        payable(msg.sender).transfer(totalClaimable);
    }
```

G4. _transfer() needs to check whether the target ``to`` has zero balance on the curvesToken, Only when it is zero there is a need to call _addOwnedCurvesTokenSubject(). Gas can be saved by adding this check. Also add a short circuit that if ``from == to`` then the function can return immediately. 

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L317-L319](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L317-L319)

G5. Curves.sellExternalCurvesToken(): the first line to check token zero address can be removed since it will be checked again inside function deposit(). 

```javascript
 function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public {
        if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();

        deposit(curvesTokenSubject, amount);
        sellCurvesToken(curvesTokenSubject, amount / 1 ether);
    }
```