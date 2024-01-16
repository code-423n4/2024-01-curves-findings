# withdraw: can be called with amount = 0, can deploy ERC20 first for subjects not yet signed up, prevent subject from setting ERC20 name/symbol

## Links to affected code

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L466](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L466)

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L479](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L479)

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L314](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L314)

## Impact

Griefing attack to user. Deploy ERC20 of subject who has not yet participated in the protocol, preventing the user from deploying ERC20 with desired settings.

## Proof of Concept

If `amount` is 0, `withdraw` can be called without owning any tokens. By calling `withdraw`, attacker can deploy arbitrary subject's ERC20. The subject is unable to call the `buyCurvesTokenWithName`, `mint`, and `setNameAndSymbol` functions, thereby preventing them from deploying the contract with their desired name and symbol.

```solidity
function withdraw(address curvesTokenSubject, uint256 amount) public {
@>  if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

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
@>      _deployERC20(
            curvesTokenSubject,
            externalCurvesTokens[curvesTokenSubject].name,
            externalCurvesTokens[curvesTokenSubject].symbol
        );
        externalToken = externalCurvesTokens[curvesTokenSubject].token;
    }
@>  _transfer(curvesTokenSubject, msg.sender, address(this), amount);
    CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
}

function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
@>  if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

    // If transferring from oneself, skip adding to the list
    if (from != to) {
        _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
    }

    curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
    curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;

    emit Transfer(curvesTokenSubject, from, to, amount);
}
```

This is PoC. Add it under the "Curves ERC20 test" describe in curves-erc20.ts and run it. Use `yarn hardhat test --grep "PoC withdraw with amount 0 and deploy"`. 

```jsx
it("PoC withdraw with amount 0 and deploy", async () => {
  const attacker = addrs[addrs.length -1];

  let tx = testContract.connect(attacker).withdraw(owner.address, 0);
  await expect(tx).to.emit(testContract, "TokenDeployed");

  const keyTokenAddress = (await testContract.externalCurvesTokens(owner.address)).token;
  const keyToken = ERC20__factory.connect(keyTokenAddress, owner);

  expect(await keyToken.name()).to.equal("Curves 1");
  expect(await keyToken.symbol()).to.equal("CURVES1");
  expect(await keyToken.totalSupply()).to.equal(0);

  // owner fail to set/deploy erc20
  tx = testContract.connect(owner).buyCurvesTokenWithName(owner.address, 1, "TestERC20", "TEST");
  expect(tx).to.be.revertedWith("ERC20TokenAlreadyMinted()");

  tx = testContract.connect(owner).mint(owner.address);
  expect(tx).to.be.revertedWith("ERC20TokenAlreadyMinted()");

  tx = testContract.connect(owner).setNameAndSymbol(owner.address, "TestERC20", "TEST");
  expect(tx).to.be.revertedWith("ERC20TokenAlreadyMinted()");
});
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Prevent withdrawal with amount 0. By blocking it in `_transfer`, other issues using amount 0 can also be prevented.

```diff
function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
-   if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
+   if (amount > curvesTokenBalance[curvesTokenSubject][from] || amount == 0) revert InsufficientBalance();

    // If transferring from oneself, skip adding to the list
    if (from != to) {
        _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
    }

    curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
    curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;

    emit Transfer(curvesTokenSubject, from, to, amount);
}
```

# buyCurvesTokenWithName: Can set empty string at name, symbol

## Links to affected code

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L374](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L374)

## Impact

Unlike other functions, `buyCurvesTokenWithName` allows setting an empty string as the name and symbol.

## Proof of Concept

The `mint` and `withdraw` functions that request ERC20 deployment prevent setting an empty string as the name and symbol. Even if an empty string is set using `setNameAndSymbol`, the functions will change it to the default name and symbol if an empty string is detected.

```solidity
function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
  if (
@>    keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
      keccak256(abi.encodePacked("")) ||
@>    keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
      keccak256(abi.encodePacked(""))
  ) {
@>    externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
@>    externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
  }
  _mint(
      curvesTokenSubject,
      externalCurvesTokens[curvesTokenSubject].name,
      externalCurvesTokens[curvesTokenSubject].symbol
  );
}

function withdraw(address curvesTokenSubject, uint256 amount) public {
  if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

  address externalToken = externalCurvesTokens[curvesTokenSubject].token;
  if (externalToken == address(0)) {
      if (
@>        keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
          keccak256(abi.encodePacked("")) ||
@>        keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
          keccak256(abi.encodePacked(""))
      ) {
@>        externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
@>        externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
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

 

However, when using `buyCurvesTokenWithName`, the `_mint` function is called without checking if the received `name` and `symbol` parameters are empty strings. Therefore, it is possible to deploy an ERC20 with empty string as the name and symbol.

```solidity
function buyCurvesTokenWithName(
    address curvesTokenSubject,
    uint256 amount,
@>  string memory name,
@>  string memory symbol
) public payable {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply != 0) revert CurveAlreadyExists();

    _buyCurvesToken(curvesTokenSubject, amount);
@>  _mint(curvesTokenSubject, name, symbol);
}
```

This is PoC. Add it under the "Curves ERC20 test" describe in curves-erc20.ts and run it. Use `yarn hardhat test --grep "PoC Should be able to mint with empty string"`. 

```jsx
it("PoC Should be able to mint with empty string", async () => {
  await testContract.buyCurvesTokenWithName(owner.address, 1, "", "");
  await buyToken(testContract, owner, owner, 2);

  await testContract.withdraw(owner.address, 2);

  const keyTokenAddress = (await testContract.externalCurvesTokens(owner.address)).token;
  const keyToken = ERC20__factory.connect(keyTokenAddress, owner);

  expect(await keyToken.name()).to.equal("");
  expect(await keyToken.symbol()).to.equal("");
  expect(await keyToken.totalSupply()).to.equal(parseEther("2"));
});
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Use default name/symbol when the parameter is an empty string.

```diff
function buyCurvesTokenWithName(
    address curvesTokenSubject,
    uint256 amount,
    string memory name,
    string memory symbol
) public payable {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply != 0) revert CurveAlreadyExists();

+   if (
+       keccak256(abi.encodePacked(name)) ==
+       keccak256(abi.encodePacked("")) ||
+       keccak256(abi.encodePacked(symbol)) ==
+       keccak256(abi.encodePacked(""))
+   ) {
+       name = DEFAULT_NAME;
+       symbol = DEFAULT_SYMBOL;
+   }

    _buyCurvesToken(curvesTokenSubject, amount);
    _mint(curvesTokenSubject, name, symbol);
}
```

# Trade event: Emit with wrong value when no referral fee definition defined

## Links to affected code

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L257](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L257)

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L230](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L230)

## Impact

Emit Trade event with incorrect information

## Proof of Concept

If the subject did not define referral fee destination, the referral fee is included in the protocol fee. However, when emitting a trade event, the protocol fee does not include the combined referral fee.

```solidity
function _transferFees(
    address curvesTokenSubject,
    bool isBuy,
    uint256 price,
    uint256 amount,
    uint256 supply
) internal {
    (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
    {
        bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
        {
@>          address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
@>          uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
            uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
@>          (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
            if (!success1) revert CannotSendFunds();
        }
        ...
    }
    emit Trade(
        msg.sender,
        curvesTokenSubject,
        isBuy,
        amount,
        price,
@>      protocolFee,
        subjectFee,
        isBuy ? supply + amount : supply - amount
    );
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
    emit Trade(
        msg.sender,
        curvesTokenSubject,
        isBuy,
        amount,
        price,
-       protocolFee,
+       referralDefined ? protocolFee : protocolFee + referralFee,
        subjectFee,
        isBuy ? supply + amount : supply - amount
    );
```

# setMaxFeePercent: Should check if maxFeePercent_ ≤ 1e18

## Links to affected code

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L125](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L125)

## Impact

maxFeePercent can be set over 100%

## Proof of Concept

In this protocol, 1e18 means 100%. So the sum of percent variables sould less than 1e18. `setMaxFeePercent` does not check the maximum value.

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
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
  if (
      feesEconomics.protocolFeePercent +
          feesEconomics.subjectFeePercent +
          feesEconomics.referralFeePercent +
          feesEconomics.holdersFeePercent >
      maxFeePercent_
  ) revert InvalidFeeDefinition();
+ require(maxFeePercent_ <= 1e18, "exceed maximum");
  feesEconomics.maxFeePercent = maxFeePercent_;
}
```

# onBalanceChange: Updating userTokens incorrectly and will cause DoS, getUserTokensAndClaimable will not work properly

## Links to affected code

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L99](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L99)

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L52](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L52)

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L48](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L48)

## Impact

It can cause a DoS by storing duplicate values in `userTokens`.

## Proof of Concept

It does not check for duplicates when updating `userTokens`. The previously registered subject is registered again every time user buys or sells tokens. The more active the user, the longer the array `userTokens` gets.

```jsx
mapping(address => address[]) internal userTokens;

function onBalanceChange(address token, address account) public onlyManager {
    TokenData storage data = tokensData[token];
    data.userFeeOffset[account] = data.cumulativeFeePerToken;
@>  if (balanceOf(token, account) > 0) userTokens[account].push(token);
}
```

If the length of the `userTokens` array gets too long, `getUserTokens` or `getUserTokensAndClaimable` may not work. `batchClaiming`, that uses the results from `getUserTokens`, can also have an issue.

```jsx
function getUserTokens(address user) public view returns (address[] memory) {
    return userTokens[user];
}

function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
    address[] memory tokens = getUserTokens(user);
    UserClaimData[] memory result = new UserClaimData[](tokens.length);
    for (uint256 i = 0; i < tokens.length; i++) {
        address token = tokens[i];
        uint256 claimable = getClaimableFees(token, user);
        result[i] = UserClaimData(claimable, token);
    }
    return result;
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Use the EnumerableSet data type instead of an array. [https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.3/contracts/utils/structs/EnumerableSet.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.3/contracts/utils/structs/EnumerableSet.sol)

# Cannot start presale/sale if subject buys 0 tokens

## Links to affected code

[https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L265](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L265)

## Impact

User cannot purchase during the Presale period.

## Proof of Concept

It does not check if the `amount`  is more than 0 when purchasing tokens. Therefore, if subject set the amount to 0 when calling `buyCurvesTokenForPresale` and do not purchase the first token, subject cannot sell tokens during the presale period. Also, the subject cannot sell after the presale period.

```solidity
function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
@>  if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

    uint256 price = getPrice(supply, amount);
    (, , , , uint256 totalFee) = getFees(price);

    if (msg.value < price + totalFee) revert InsufficientPayment();

    curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
    curvesTokenSupply[curvesTokenSubject] = supply + amount;
    _transferFees(curvesTokenSubject, true, price, amount, supply);

    // If is the first token bought, add to the list of owned tokens
    if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
        _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
    }
}

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
@>  _buyCurvesToken(curvesTokenSubject, amount);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Enforce the amount to 1 or check if it is more than 1 when calling `buyCurvesTokenForPresale` or `buyCurvesTokenWithName` .

Or, check if the amount is > 0 in `_buyCurvesToken`.

```diff
function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
+   require(amount > 0, "wrong amount");
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

    uint256 price = getPrice(supply, amount);
    (, , , , uint256 totalFee) = getFees(price);

    if (msg.value < price + totalFee) revert InsufficientPayment();

    curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
    curvesTokenSupply[curvesTokenSubject] = supply + amount;
    _transferFees(curvesTokenSubject, true, price, amount, supply);

    // If is the first token bought, add to the list of owned tokens
    if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
        _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
    }
}
```