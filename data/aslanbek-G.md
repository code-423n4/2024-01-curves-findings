# G-01 remove onlyTokenSubject in favor of msg.sender

The existence of the modifier makes the code both expensive and harder to follow. Consider replacing it with `msg.sender`:

1. Remove completely - [Curves.sol#L103-L106](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L103-L106)

2. [Curves.sol#L155-L160](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L155-L160)
```diff
    function setReferralFeeDestination(
-       address curvesTokenSubject,
        address referralFeeDestination_
-   ) public onlyTokenSubject(curvesTokenSubject) {
+   ) public {
-       referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
+       referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
    }
```

3. [Curves.sol#L377-L392](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L377-L392)

```diff
    function buyCurvesTokenForPresale(
-       address curvesTokenSubject,
        uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
-   ) public payable onlyTokenSubject(curvesTokenSubject) {
+   ) public payable {
        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
-       uint256 supply = curvesTokenSupply[curvesTokenSubject];
+       uint256 supply = curvesTokenSupply[msg.sender];
        if (supply != 0) revert CurveAlreadyExists();
-       presalesMeta[curvesTokenSubject].startTime = startTime;
-       presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
-       presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
+       presalesMeta[msg.sender].startTime = startTime;
+       presalesMeta[msg.sender].merkleRoot = merkleRoot;
+       presalesMeta[msg.sender].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

4. [Curves.sol#L428-L437](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L428-L437)
```diff
    function setNameAndSymbol(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
-   ) external onlyTokenSubject(curvesTokenSubject) {
+   ) external {
-       if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
+       if (externalCurvesTokens[msg.sender].token != address(0)) revert ERC20TokenAlreadyMinted();
        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
-       externalCurvesTokens[curvesTokenSubject].name = name;
-       externalCurvesTokens[curvesTokenSubject].symbol = symbol;
+       externalCurvesTokens[msg.sender].name = name;
+       externalCurvesTokens[msg.sender].symbol = symbol;
    }
```

5. [Curves.sol#L439-L454](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L439-L454)
```diff
-   function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
+   function mint() external {
        if (
-           keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
+           keccak256(abi.encodePacked(externalCurvesTokens[msg.sender].name)) ==
            keccak256(abi.encodePacked("")) ||
-           keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
+           keccak256(abi.encodePacked(externalCurvesTokens[msg.sender].symbol)) ==
            keccak256(abi.encodePacked(""))
        ) {
-           externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
-           externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
+           externalCurvesTokens[msg.sender].name = DEFAULT_NAME;
+           externalCurvesTokens[msg.sender].symbol = DEFAULT_SYMBOL;
        }
        _mint(
            curvesTokenSubject,
-           externalCurvesTokens[curvesTokenSubject].name,
-           externalCurvesTokens[curvesTokenSubject].symbol
+           externalCurvesTokens[msg.sender].name,
+           externalCurvesTokens[msg.sender].symbol
        );
    }
```
6. [Curves.sol#L364-L375](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L364-L375)

`buyCurvesTokenWithName` reverts if `supply > 0`. When `supply == 0`, `_buyCurvesToken` reverts if called by non-subject:
```solidity
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();
```
Thus, `curvesTokenSubject` can be safely replaced with `msg.sender`:

```diff
    function buyCurvesTokenWithName(
-       address curvesTokenSubject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) public payable {
-       uint256 supply = curvesTokenSupply[curvesTokenSubject];
+       uint256 supply = curvesTokenSupply[msg.sender];
```
# G-02 Redundant checks
1. [Curves.sol#L497-L501](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L497-L501)
```diff
    function deposit(address curvesTokenSubject, uint256 amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
        uint256 tokenAmount = amount / 1 ether;
        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
-       if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
-       if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();
        CurvesERC20(externalToken).burn(msg.sender, amount);
        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
    }
```
`burn` and `_transfer` revert if `amount > balance` anyway - no reason to check the same condition twice.

2. [Curves.sol#L313-L325](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L313-L325)
```diff
    function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
-       if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
        // If transferring from oneself, skip adding to the list
-       if (from != to) {
            _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
-       }
        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;
        emit Transfer(curvesTokenSubject, from, to, amount);
    }
```
`amount > balance` check is not needed - attempt to transfer more than balance will revert due to integer underflow.
`from != to` check will be net negative for users, as they will rarely transfer to themselves.

3. [Curves.sol#L213](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L213)
```diff
-       if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
+       if (startTime >= block.timestamp) revert SaleNotOpen();
```
If `startTime` is zero, it will always be smaller than `block.timestamp`. However, it might make sense to leave it as it is for short-circuiting, if most users are expected to not use the whitelist feature.

# G-03 Pack structs more tightly

1. [Curves.sol#L68-L75](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L68-L75)

```solidity
    struct FeesEconomics {
        address protocolFeeDestination;
        uint256 protocolFeePercent;
        uint256 subjectFeePercent;
        uint256 referralFeePercent;
        uint256 holdersFeePercent;
        uint256 maxFeePercent;
    }
```
uint256 is unnecessarily high for the fee variables. Consider using a smaller uint (ideally, `uint64`) to save on SLOAD's.

2. [Curves.sol#L55-L59](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L55-L59)
```diff
    struct PresaleMeta {
-       uint256 startTime;
        bytes32 merkleRoot;
-       uint256 maxBuy;
+       uint128 startTime;
+       uint128 maxBuy;
    }
```
