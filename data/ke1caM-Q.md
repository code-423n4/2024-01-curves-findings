[01] User can prevent `curvesTokenSubject` from setting desired name and symbol, by deploying erc20, using `withdraw` function.

```solidity
function withdraw(address curvesTokenSubject, uint256 amount) public {
    if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

    address externalToken = externalCurvesTokens[curvesTokenSubject].token;
    if (externalToken == address(0)) {
        if (
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
            keccak256(abi.encodePacked("")) ||
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
            keccak256(abi.encodePacked(""))
        ) {
            externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
            externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
        }
        _deployERC20(
            curvesTokenSubject,
            externalCurvesTokens[curvesTokenSubject].name,
            externalCurvesTokens[curvesTokenSubject].symbol
        );
        externalToken = externalCurvesTokens[curvesTokenSubject].token;
    }
    _transfer(curvesTokenSubject, msg.sender, address(this), amount);
    CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
}
```

```solidity
function setNameAndSymbol(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) external onlyTokenSubject(curvesTokenSubject) {
        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
        externalCurvesTokens[curvesTokenSubject].name = name;
        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
    }
```

User can deploy non-existing token using `curvesTokenSubject` address and `0` amount. This will set name and symbol to default values. This will prevent `curvesTokenSubject` from creating token because token was already deployed by someone else on their behalf. This can be done even when token was not even created and has `0` total supply.

Don't allow `0` as a parameter or validate that token has valid totat supply to allow withdrawals.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L465-L488
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L428-L437

[02] Same token will be added multiple times into `userTokens` array.

```solidity
function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }
```

`if (balanceOf(token, account) > 0) userTokens[account].push(token)`

`onBalanceChange` function adds token to `userTokens` array when user has balance greater than 0. This will work correctly on the first buy as a user will have this token added once. However, with every next buy, the token will be added again since user's balance will still be greater than zero.

Before adding new token to `userTokens` check if this token was previously added.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96-L100

[03] Max fee percent can be set to 100% or more. State variables are not capped at reasonable values.

When fee values are not capped at reasonable values there is a risk that these values will be set in a way that harm user. It can be for example unexpected, greater fee forcing user to pay higher price for same tokens.

```solidity
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
        if (
            feesEconomics.protocolFeePercent +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            maxFeePercent_
        ) revert InvalidFeeDefinition();
        feesEconomics.maxFeePercent = maxFeePercent_;
    }

    function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
        if (
            protocolFeePercent_ +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            feesEconomics.maxFeePercent ||
            protocolFeeDestination_ == address(0)
        ) revert InvalidFeeDefinition();
        feesEconomics.protocolFeePercent = protocolFeePercent_;
        feesEconomics.protocolFeeDestination = protocolFeeDestination_;
    }

    function setExternalFeePercent(
        uint256 subjectFeePercent_,
        uint256 referralFeePercent_,
        uint256 holdersFeePercent_
    ) external onlyManager {
        if (
            feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ >
            feesEconomics.maxFeePercent
        ) revert InvalidFeeDefinition();
        feesEconomics.subjectFeePercent = subjectFeePercent_;
        feesEconomics.referralFeePercent = referralFeePercent_;
        feesEconomics.holdersFeePercent = holdersFeePercent_;
    }

```

Create checks that do not allow for unexpected values to be used as these state variables.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117-L153

[04] User can prevent Curves Tokens with `DEFAULT` name and symbol from being created.

```solidity
function _deployERC20(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal returns (address) {
        // If the token's symbol is CURVES, append a counter value
        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
            _curvesTokenCounter += 1;
            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
        }

        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

        externalCurvesTokens[curvesTokenSubject].token = tokenContract;
        externalCurvesTokens[curvesTokenSubject].name = name;
        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
        externalCurvesToSubject[tokenContract] = curvesTokenSubject;
        symbolToSubject[symbol] = curvesTokenSubject;

        emit TokenDeployed(curvesTokenSubject, tokenContract, name, symbol);
        return address(tokenContract);
    }
```

User can create Curves Token with `symbol` equal to `DEFAULT_SYMBOL` + next values of `_curvesTokenCounter`. After that, whenever someone else will try to create a Token with `DEFAULT_SYMBOL`, function will revert because of this line:

`if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();` as token with the next `_curvesTokenCounter` value is already created. This will freeze creating of Curves Token with `DEFAULT_SYMBOL` as `_deployERC20` is the only function in which `_curvesTokenCounter` is incremented.

To fix this, don't revert in these scenarios. Create try/catch and increment when `symbolToSubject[symbol]` already exists.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L344-L348

[05] Presale period does not have MAX values. It can last indefinitely.

```solidity
function buyCurvesTokenForPresale(
        address curvesTokenSubject,
        uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
    ) public payable onlyTokenSubject(curvesTokenSubject) {
        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply != 0) revert CurveAlreadyExists();
        presalesMeta[curvesTokenSubject].startTime = startTime;
        presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);

        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

`presalesMeta[curvesTokenSubject].startTime = startTime` once this value is set it can not be changed. It can last indefinitely.

Create max value that user can set as a period of presale. For example one week or one month.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L387