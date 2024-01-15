## L-01: tokens are never removed from `ownedCurvesTokenSubjects`

Tokens are added to the `ownedCurvesTokenSubjects` when users buy or receive them.

However, they are not removed once the user balance becomes 0 (ie. on sells or transfers from).

```solidity
// Internal function to add a curvesTokenSubject to the list if not already present
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

### Impact

Functions that iterate over the owned tokens will consume more gas than they should.

### Recommended Mitigation Steps

Remove the tokens from the list once the user balance becomes 0, after sells or transfers from.

## L-02: confusing parameter `amount` in `buyCurvesTokenForPresale` and `buyCurvesTokenWithName`

Both `buyCurvesTokenForPresale` and `buyCurvesTokenWithName` have a useless parameter `amount` that must always be set to 1 for the transaction to not revert.

This parameter is passed over to `_buyCurvesToken` when the supply is 0; hence only one token can be bought at that point.

```solidity
function buyCurvesTokenWithName(
    address curvesTokenSubject,
    uint256 amount,
    string memory name,
    string memory symbol
) public payable {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply != 0) revert CurveAlreadyExists();

    _buyCurvesToken(curvesTokenSubject, amount);
    _mint(curvesTokenSubject, name, symbol);
}
```

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

### Impact

`buyCurvesTokenForPresale` and `buyCurvesTokenWithName` will revert when their parameter `amount` is set to something else than 1.

### Recommended Mitigation Steps

Remove the parameter and call `_buyCurvesToken` with a hardcoded value of 1, which is the only one accepted when calling `buyCurvesTokenForPresale` and `buyCurvesTokenWithName`.

## L-03: no one can buy tokens at `startTime` when presale is set

When the TokenSubject initializes a presale on their token, they must set a `startTime` in which the public sale will begin. However, this `startTime` is excluded from both the `buyCurvesTokenWhitelisted` and `buyCurvesToken` functions.

```solidity
function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
    uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

    _buyCurvesToken(curvesTokenSubject, amount);
}
```

```solidity
function buyCurvesTokenWhitelisted(
    address curvesTokenSubject,
    uint256 amount,
    bytes32[] memory proof
) public payable {
    if (
        presalesMeta[curvesTokenSubject].startTime == 0 ||
        presalesMeta[curvesTokenSubject].startTime <= block.timestamp
    ) revert PresaleUnavailable();

    presalesBuys[curvesTokenSubject][msg.sender] += amount;
    uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
    if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

    verifyMerkle(curvesTokenSubject, msg.sender, proof);
    _buyCurvesToken(curvesTokenSubject, amount);
}
```

### Impact

Neither the whitelisted addresses nor normal users will be able to buy tokens at the `startTime`.

### Recommended Mitigation Steps

Allow users to buy tokens at `startTime`.

```diff
function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
    uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
-    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
+    if (startTime != 0 && startTime > block.timestamp) revert SaleNotOpen();

    _buyCurvesToken(curvesTokenSubject, amount);
}
```

## L-04: `FeeSplitter.userTokens` will contain a lot of duplicates

`FeeSplitter::onBalanceChange` pushes the `token` address to the `userTokens` mapping value on every transaction in which the balance of the user is greater than 0.

```solidity
function onBalanceChange(address token, address account) public onlyManager {
    TokenData storage data = tokensData[token];
    data.userFeeOffset[account] = data.cumulativeFeePerToken;
    if (balanceOf(token, account) > 0) userTokens[account].push(token);
}
```

Moreover, tokens are not removed from the list once the user balance becomes 0.

### Impact

`userTokens` will contain duplicated addresses for each user, making it grow much faster than it should and impractical to work with.

### Recommended Mitigation Steps

Use an `EnumerableSet` instead, and remove the `token` address once the balance of the user is 0.

See [OpenZeppelin's implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.3/contracts/utils/structs/EnumerableSet.sol).