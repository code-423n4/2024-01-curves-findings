# [L-01] - `ownedCurvesTokenSubjects` not updated if user sells all her tokens and owns nothing. 

in the event a user sells all of its curve balance for eth, the array of curveTokenSubjects mapped in `ownedCurvesTokenSubjects` to the user ought to be updated to reflect that user no longer owns any of that curveTokenSubject tokens. 

It ought to do this to ensure robust accounting because in [_buyCurvesToken()](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L276C1-L279C10) the `ownedCurvesTokenSubjects` mapping is updated to reflect ownership of that curvesTokenSubject by user if the user makes its first buy. 

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L282C1-L293C6
```
    function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply <= amount) revert LastTokenCannotBeSold();
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

        uint256 price = getPrice(supply - amount, amount);

        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
        curvesTokenSupply[curvesTokenSubject] = supply - amount;

        _transferFees(curvesTokenSubject, false, price, amount, supply);
    }
```

above we can see that sellCurvesToken() doesn't do any update to the `ownedCurvesTokenSubjects` mapping. User can sell all its `curvesTokenSubject` tokens and will still have its `ownedCurvesTokenSubjects` mapping value showing ownership of that `curvesTokenSubject` token.