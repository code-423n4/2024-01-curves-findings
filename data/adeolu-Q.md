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


# [L-02] - startTime != 0 check not necessary 
the `startTime != 0` check is not necessary because the check `startTime >= block.timestamp` is present. block.timestamp will never be 0 and startTime is checked to be greater than or eq to block.timestamp. a zero value `startTime` is not possible because `startTime` is checked to be greater than `block.timestamp` in [buyCurvesTokenForPresale()](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L384) when user is creating the presale. Remove the check to reduce computation. 
```
    function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
        uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
        if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();   

        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

# [L-03] dont need to cast an address to an address again  

in `_deployERC20()` the address returned by curve factory is already of type address, no need to cast the variable `tokenContract` to type address again in a return statement 

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L352C1-L361C39
```
        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));


        externalCurvesTokens[curvesTokenSubject].token = tokenContract;
        externalCurvesTokens[curvesTokenSubject].name = name;
        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
        externalCurvesToSubject[tokenContract] = curvesTokenSubject;
        symbolToSubject[symbol] = curvesTokenSubject;


        emit TokenDeployed(curvesTokenSubject, tokenContract, name, symbol);
        return address(tokenContract);
```