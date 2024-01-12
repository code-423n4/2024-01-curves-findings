##### [G-01] `Curves::sellCurvesToken` function can be optimized by changing the place of the balance check

```diff
    function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
++      if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply <= amount) revert LastTokenCannotBeSold();
--      if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

        uint256 price = getPrice(supply - amount, amount);

        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
        curvesTokenSupply[curvesTokenSubject] = supply - amount;

        _transferFees(curvesTokenSubject, false, price, amount, supply);
    }
```

There is no need to initialize the supply, and do the `(supply <= amount) revert` check if the msg.sender has an insufficient balance.

##### [G-02] `Curves::buyCurvesTokenWhitelisted` function can be optimized. 

The `verifyMerkle` function should be called way sooner, if the `proof` input is wrong, there's no need to check anything else, or perform any other calculation or variable initialization.

```
    function buyCurvesTokenWhitelisted(
        address curvesTokenSubject,
        uint256 amount,
        bytes32[] memory proof
    ) public payable {
++      verifyMerkle(curvesTokenSubject, msg.sender, proof);
        if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();

        presalesBuys[curvesTokenSubject][msg.sender] += amount;
        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

--      verifyMerkle(curvesTokenSubject, msg.sender, proof);
        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

##### [G-03] `Curves::deposit` function can be optimized.

The two checks related to `InsufficientBalance()` and `TokenAbsentForCurvesTokenSubject()` should be placed before the `tokenAmount`  initialization.

```

    function deposit(address curvesTokenSubject, uint256 amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
++      if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject(); 
++      if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
        uint256 tokenAmount = amount / 1 ether;

--      if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject(); 
--      if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

        CurvesERC20(externalToken).burn(msg.sender, amount);
        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
    }
```