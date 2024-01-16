## [L-01] If a subject first mint the externalCurvesTokens won't be able to initialize an open sale because the _mint() will revert

When the user initiates an open-sale without a presale needs to buy the first share of his curveToken (so the supply is != 0 and allows other users to buy his curveToken).
- The problem is if the user first mints first the externalToken by calling the [`Curves::mint() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L439-L454) instead of calling first the [`Curves::buyCurvesTokenWithName() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L364-L375).
    - Both functions will end up calling the [`Curves::_mint() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L456-L463) which checks if the externalCurvesToken.token is already set, (which is set in the [`Curves::_deployERC20() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L338-L362)), and if it's already set, it reverts the whole tx.

> Curves.sol
```solidity
    function buyCurvesTokenWithName(
        ...
    ) public payable {
        ...
        //@audit-info => Internally calls the _mint()
        _mint(curvesTokenSubject, name, symbol);
    }

    function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
        ...
        //@audit-info => Internally calls the _mint()
        _mint(
            curvesTokenSubject,
            externalCurvesTokens[curvesTokenSubject].name,
            externalCurvesTokens[curvesTokenSubject].symbol
        );
    }

    function _mint(
       ...
    ) internal onlyTokenSubject(curvesTokenSubject) {
        //@audit-info => If the externalCurvesTokens is already defined, reverts, the contract has already been deployed!
        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
        _deployERC20(curvesTokenSubject, name, symbol);
    }

    function _deployERC20(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal returns (address) {
        ...

        //@audit-info => Sets the address of the externalCurvesTokens with the address of the contract that was just deployed!
        //@audit-info => This is the only place where the address of the externalCurvesTokens is defined
        externalCurvesTokens[curvesTokenSubject].token = tokenContract;
        ...
    }
```

**Fix:**
- In the _mint(), instead of reverting if the externalCurvesTokens already exists, just run a `return`, so the tx can be completed while it also prevents a new externalCurveToken from being deployed!

---
## [L-02] Not checking if a merkleRoot has already been set before calling the buyCurvesTokenForPresale() and not validating if the provided merkleRoot is empty can override the previously defined merkleRoot and initialize a presale with an empty merkleRoot

If the subjectToken sets the merkleRoot for the whitelist before calling the [`Curves::buyCurvesTokenForPresale() function`], but when the buyCurvesTokenForPresale() function is finally called, and the merkleRoot is passed as empty (thinking that it has already been set), the final merkleRoot for the presale will be an empty merkleRoot

> Curves.sol
```solidity
    function buyCurvesTokenForPresale(
        address curvesTokenSubject,
        uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
    ) public payable onlyTokenSubject(curvesTokenSubject) {
        ...
        //@audit-issue => If the presalesMeta.merkleRoot has already been set by calling the setWhitelist(), and when this function is called the merkleRoot is passed as an empty merkleRoot, the presale will be initiated using an empty merkleRoot, the merkleRoot received in this function will override the previous one!
        presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
        ...

    }
```

**Fix:**
- Check if the provided merkleRoot is empty && if the `presalesMeta.merkleRoot` is also empty, if so, revert the tx, don't allow to start a presale with an empty merkleRoot

---
## [L-03] Not checking if the tokenSubject is buying more than 1 curveToken when buyingTokenForPresale will cause the tx to revert
- **When a tokenSubject initiates a presale there are a couple of requisites, one of them it is that the curveTokenSupply is 0** (which means, that the tokenSubject has not bought the first share of his curveToken).
- The [`Curves::buyCurvesTokenForPresale() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L377-L392) allows the user to define the amount of curveTokens that the tokenSubject would like to buy before the presale is started. **The problem is** that if the tokenSubject tries to buy more than 1 share of his curveToken, the whole tx will revert, the reason is that the curveTokenSupply is 0, and when the getPrice() is called, if the supply is 0, the math operations will cause an underflow that will blow up the whole tx. When buying the first share of a curveToken, the amount to be bought can be at most 1 share.

> Curves.sol
```solidity

    function buyCurvesTokenForPresale(
        address curvesTokenSubject,
        uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
    ) public payable onlyTokenSubject(curvesTokenSubject) {
        ...

        //@audit-info => If any curvesTokens of this subject have already been bought, it can't create a presale because sales have already occurred!
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply != 0) revert CurveAlreadyExists();
        
        ...
        //@audit-info => If `amount` != 1, the getPrice() will cause an underflow!!!
        _buyCurvesToken(curvesTokenSubject, amount);
    }

    function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;

        //@audit-info => The subjectToken must buy 1 share to activate the price, it can't buy more than 1 share for the first time!
        //@audit-issue => If supply is 0 and the tokenSubject attempts to purchase more than 1 share, instead of returning 0, the function will compute the math formula, which will overflow when computing (supply - 1) because supply is 0!!!

        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
        return (summation * 1 ether) / 16000;
    }
```

**Fix:**
- Since the first purchase can be of only 1 share, remove the `amount` parameter, and instead, when calling the `_buyCurvesToken()`, hardcode the amount of 1, in this way, when initiating a presale, the tokenSubject will attempt to only buy the first share.

---
## [L-04] If only the symbol or the name for a curveToken was set by the tokenSubject, when minting the curveToken, both variables will be overridden by the default value instead of the value defined by the tokenSubject
- TokenSubjects can set in advance the name and symbol for their curveToken, they can call the [`Curves::setNameAndSymbol() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L428-L437) and set these variables.
> Curves.sol
```solidity
    function setNameAndSymbol(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) external onlyTokenSubject(curvesTokenSubject) {
        ...
        
        //@audit-info => tokenSubjects can set name & symbol, but they are not forced to set the two of them, they could set only the name or only the symbol.
        externalCurvesTokens[curvesTokenSubject].name = name;
        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
    }
```

When minting the curveToken, if one of the two variables is not set, the two of them will be overridden to the default values, no matter if the other variable was indeed defined.
- For example, if the name was set but not the symbol, in this case, both of them will be overridden for the default values.
> Curves.sol
```solidity
    function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
        //@audit-issue => If only the name or the symbol was set, this will override the two and set them as the default value!
        if (
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
            keccak256(abi.encodePacked("")) ||
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
            keccak256(abi.encodePacked(""))
        ) {
            externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
            externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
        }
        ...
    }
```

**Fix:**
-  Check individually if the name and the symbol were defined, if not, override only the one that is not defined.

---
## [L-05] onBalanceChange() will push multiple times the same token's address to the userTokens array
When the [`FeeSplitter::onBalanceChange() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L96-L100) is called, at the end validates if the account's curveToken balance is > 0, if so, it pushes the token to an array that represents the tokens of the user.
- **The problem is** that there is not a check to validate if the token's address is already in the userTokens array, this will cause each time the onBalanceChange() is called (and the user has a balance on the curveToken) that the token's address is added again to the userTokens array. This will provoke that there are multiple registries with the same token address, and when users call the [`FeeSplitter::getUserTokens()`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L48-L50), they will get tons of duplicates for the same addresses.

> FeeSplitter.sol
```solidity
    function onBalanceChange(address token, address account) public onlyManager {
        ...

        //@audit-issue => onBalanceChange() will push multiple times the same token's address to the userTokens array
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }

```

**Fix:**
- Validate if the token's address is already in the userTokens array.
- Another option could be to use EnumerableSets and use the .contain() to validate if the token's address already exists in the variable!

---
## [L-06] tokenSubjects may not be able to deploy their externalCurveTokens with the symbol of their preference
- If the symbol that a tokenSubject would like to use is already chosen by another tokenSubject, that symbol will be set to the address of the first tokenSubject who deployed his externalToken first, and then, when a different tokenSubject tries to mint his externalCurveToken his tx will be reverted

> Curves.sol
```solidity
    function _deployERC20(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal returns (address) {

        //@audit-info => If symbolToSubject already has an address it means that the the given symbol has already been used to mint an externalCurvesToken!
        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

        ...
        ...
        ...

        //@audit-info => A symbol is set to point to the tokenSubject's address who deployed his externalCurveToken first!
        symbolToSubject[symbol] = curvesTokenSubject;

        ...
    }
```

**Fix:**
- Allow the same symbol to be used for different tokenSubjects, but only once for each tokenSubject

> Curves.sol
```solidity
    function _deployERC20(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal returns (address) {

        //@audit-info => If symbolToSubject already has an address it means that the the given symbol has already been used to mint an externalCurvesToken!
-       if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

+       if (symbolToSubject[symbol][curvesTokenSubject]) revert InvalidERC20Metadata();

        ...
        ...
        ...

        //@audit-info => A symbol is set to point to the tokenSubject's address who deployed his externalCurveToken first!
-       symbolToSubject[symbol] = curvesTokenSubject;

+       symbolToSubject[symbol][curvesTokenSubject] = true;

        ...
    }
```