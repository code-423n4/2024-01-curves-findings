## The parameter amount of buyCurvesToken should be checked if it is greater than 0

This can reduce invalid operations by users

```solidity
    function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
        uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
        if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

        _buyCurvesToken(curvesTokenSubject, amount);
    }
```



## `_addOwnedCurvesTokenSubject` should make a maximum number limit, otherwise it may be dos

Set a maximum number limit for subjects, otherwise unexpected situations will occur when the number is large

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



## `setWhitelist` should check supply > 0

When supply is greater than 0, it means that curveToken already exists, not 0

```solidity
    function setWhitelist(bytes32 merkleRoot) external {
        uint256 supply = curvesTokenSupply[msg.sender];
        if (supply > 1) revert CurveAlreadyExists();

        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
            presalesMeta[msg.sender].merkleRoot = merkleRoot;
            emit WhitelistUpdated(msg.sender, merkleRoot);
        }
    }
```

