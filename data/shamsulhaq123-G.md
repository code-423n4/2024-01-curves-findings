## Gas Optimization

## Summary

|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use bitmap to save gas|2|
|[G-02]|Inline modifiers used only once|2|
|[G-03]|Use nested if statements instead of &&|2|
|[G-04]|Use calldata instead of memory for function parameters|9|
|[G-05]|Use hardcode address instead address(this)|6|
|[G-06]|Use function instead of modifiers |1|
|[G-07]|Use do while loops instead of for loops|4|
|[G-08]|Use constants instead of type(uintx).max|1|
|[G-09]|Use uint256(1)/uint256(2) instead for true and false boolean states|4|
|[G-10]|+= costs more gas than = + for state variables|6|
|[G-11]|Use assembly for loops|4|
|[G-12]|Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|19|
|[G-13]|`do`-`while` is cheaper than `for`-loops when the initial check can be skipped |4|
|[G-14]|Using `private` rather than `public` for constants, saves gas|1|
|[G-15]|Remove or replace unused state variables|3|
|[G-16]|Use shift Right instead of division if possible to save gas|2|
|[G-17]|Use shift Right instead of division if possible to save gas|1|
|[G-18]|Do not cache constants to save gas|3|
|[G-19]|Avoid transferring amounts of zero in order to save gas|4|
|[G-20]|Using mappings instead of arrays to avoid length checks save gas|2|


## [G-01] Use bitmap to save gas
Bitmaps in Solidity are essentially a way of representing a set of boolean values within an integer type variable such as uint256. Each bit in the integer represents a true or false value (1 or 0), thus allowing efficient storage of multiple boolean values.

Bitmaps can save gas in the Ethereum network because they condense a lot of information into a small amount of storage. In Ethereum, storage is one of the most significant costs in terms of gas usage. By reducing the amount of storage space needed, you can potentially save on gas fees.

Here's a quick comparison:

If you were to represent 256 different boolean values in the traditional way, you would have to declare 256 different bool variables. Given that each bool occupies a storage slot and each storage slot costs 20,000 gas to initialize, you would end up paying a considerable amount of gas.

On the other hand, if you were to use a bitmap, you could store these 256 boolean values within a single uint256 variable. In other words, you'd only pay for a single storage slot, resulting in significant gas savings.

However, it's important to note that while bitmaps can provide gas efficiencies, they do add complexity to the code, making it harder to read and maintain. Also, using bitmaps is efficient only when dealing with a large number of boolean variables that are frequently changed or accessed together.

In contrast, the straightforward counterpart to bitmaps would be using arrays or mappings to store boolean values, with each bool value occupying its own storage slot. This approach is simpler and more readable but could potentially be more expensive in terms of gas usage.

```solidity

file: contracts/Security.sol


14   managers[msg.sender] == true;

20  managers[msg.sender] = true;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L14
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L20

## [G-02] Inline modifiers used only once

```solidity

file: /contracts/Curves.sol

103  modifier onlyTokenSubject(address curvesTokenSubject) {
        if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();
        _;
    }

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L103

```solidity

file: contracts/Security.sol

8 
    modifier onlyOwner() {
        msg.sender == owner;
        _;
    }

    modifier onlyManager() {
        managers[msg.sender] == true;
        _;
16    }
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L8


## [G-03] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: contracts/Curves.sol

213    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

246   if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0))
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L213
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L246

## [G-04] Use calldata instead of memory for function parameters
If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract.

```solidity

file:contracts/Curves.sol

340    string memory name,
341    string memory symbol

367 string memory name,
368  string memory symbol

407 bytes32[] memory proof

422    function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view 

430 string memory name,
431 string memory symbol

458   string memory name,
459   string memory symbol
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L340
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L367
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L407
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L422
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L430
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L458

```solidity

file: contracts/FeeSplitter.sol

48   function getUserTokens(address user) public view returns (address[] memory) 

52   function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) 
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L48
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L52

```solidity

file: contracts/CurvesERC20Factory.sol

7  function deploy(string memory name, string memory symbol, address owner) public returns (address)

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L7

## [G-05] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

```solidity

file: contracts/Curves.sol

297  if (to == address(this)) revert ContractCannotReceiveTransfer();

303   if (to == address(this)) revert ContractCannotReceiveTransfer();

352  address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

486  _transfer(curvesTokenSubject, msg.sender, address(this), amount);

498   if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

501 _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
``` 
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L297
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L303
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L352
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L486
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L498
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L501

## [G-06] Use function instead of modifiers 

```solidity

file: contracts/Curves.sol

103  modifier onlyTokenSubject(address curvesTokenSubject)

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L103

## [G-07] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

file:contracts/Curves.sol

305  for (uint256 i = 0; i < subjects.length; i++) 

330   for (uint256 i = 0; i < subjects.length; i++) 
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L330

```solidity

file:main/contracts/FeeSplitter.sol

55      for (uint256 i = 0; i < tokens.length; i++) 

105     for (uint256 i = 0; i < tokenList.length; i++)
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L105

## [G-08] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity

file: contracts/Curves.sol

389  presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L389

## [G-09]  Use uint256(1)/uint256(2) instead for true and false boolean states
If you don’t use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

```solidity

file: contracts/Curves.sol

242   : (true, bytes(""));

274 _transferFees(curvesTokenSubject, true, price, amount, supply);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L242
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L274

```solidity

file: contracts/Security.sol

14  managers[msg.sender] == true;

20   managers[msg.sender] = true;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L14
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L20

## [G-10] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: contracts/Curves.sol

272   curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;

345  _curvesTokenCounter += 1;

414  presalesBuys[curvesTokenSubject][msg.sender] += amount;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L272
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L345
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L414

```solidity

file: contracts/FeeSplitter.sol

68   data.unclaimedFees[account] += owed / PRECISION;

93   data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_;

111   totalClaimable += claimable;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L68
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L93
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L111

## [G-11] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops.

```solidity  

file: contracts/Curves.sol

305  for (uint256 i = 0; i < subjects.length; i++)

330  for (uint256 i = 0; i < subjects.length; i++) 
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L330

```solidity

file: contracts/FeeSplitter.sol

55  for (uint256 i = 0; i < tokens.length; i++) 

105   for (uint256 i = 0; i < tokenList.length; i++) 
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L105

## [G‑12] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity

file: contracts/Curves.sol

104     if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();

229    address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;

247   feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);

     emit Trade(
 25  msg.sender,

 265    if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

 272 
        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
278            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);

285   if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

289  curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;

298 _transfer(curvesTokenSubject, msg.sender, to, amount);

304   address[] storage subjects = ownedCurvesTokenSubjects[msg.sender];
        for (uint256 i = 0; i < subjects.length; i++) {
            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
            if (amount > 0) {
308          _transfer(subjects[i], msg.sender, to, amount);

395    uint256 supply = curvesTokenSupply[msg.sender];
        if (supply > 1) revert CurveAlreadyExists();

        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
            presalesMeta[msg.sender].merkleRoot = merkleRoot;
400            emit WhitelistUpdated(msg.sender, merkleRoot);

414      presalesBuys[curvesTokenSubject][msg.sender] += amount;
        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

418        verifyMerkle(curvesTokenSubject, msg.sender, proof);

466  if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

486   _transfer(curvesTokenSubject, msg.sender, address(this), amount);
487        CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);

497    if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

        CurvesERC20(externalToken).burn(msg.sender, amount);
501        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L104
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L229
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L247
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L252
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L265
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L272-L278
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L285
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L289
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L298
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L304-L308 
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L395-L400
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L414-L418
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L466
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L486-L487
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L497-L501

```solidity

file: contracts/FeeSplitter.sol

81   updateFeeCredit(token, msg.sender);
        uint256 claimable = getClaimableFees(token, msg.sender);
        if (claimable == 0) revert NoFeesToClaim();
        tokensData[token].unclaimedFees[msg.sender] = 0;
        payable(msg.sender).transfer(claimable);
86        emit FeesClaimed(token, msg.sender, claimable);

107  updateFeeCredit(token, msg.sender);
            uint256 claimable = getClaimableFees(token, msg.sender);
            if (claimable > 0) {
                tokensData[token].unclaimedFees[msg.sender] = 0;
                totalClaimable += claimable;
                emit FeesClaimed(token, msg.sender, claimable);
            }
        }
        if (totalClaimable == 0) revert NoFeesToClaim();
116        payable(msg.sender).transfer(totalClaimable);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L81-L86
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L107-L116

```solidity

file: contracts/Security.sol

9  msg.sender == owner;

14      managers[msg.sender] == true;
        _;
    }

    constructor() {
        owner = msg.sender;
20        managers[msg.sender] = true;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L9
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L14-L20

## [[G-13] `do`-`while` is cheaper than `for`-loops when the initial check can be skipped 

```solidity

file: contracts/Curves.sol

305  for (uint256 i = 0; i < subjects.length; i++) 

330   for (uint256 i = 0; i < subjects.length; i++)
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L330

```solidity

file: contracts/FeeSplitter.sol

55   for (uint256 i = 0; i < tokens.length; i++) 

105    for (uint256 i = 0; i < tokenList.length; i++) 
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L105

## [G-14] Using `private` rather than `public` for constants, saves gas

```solidity

file: contracts/Curves.sol

44  string public constant DEFAULT_NAME = "Curves";
45  string public constant DEFAULT_SYMBOL = "CURVES"
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L44-L45

## [G-15] Remove or replace unused state variables

```solidity

file: contracts/Curves.sol

99   mapping(address => uint256) public curvesTokenSupply;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L99

```solidity

file: contracts/FeeSplitter.sol

19  mapping(address => uint256) userFeeOffset;
20  mapping(address => uint256) unclaimedFees;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L19-L20

```solidity

file: contracts/Security.sol

6  mapping(address => bool) public managers;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L6

## [G-16] Use shift Right instead of division if possible to save gas

```solidity

file: contracts/Curves.sol

181   uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;

184   : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L181
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L184

## [G-17] Use shift Left instead of multiplication if possible to save gas

```solidity

file: contracts/Curves.sol

181    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
186        return (summation * 1 ether) / 16000;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L181-L186
## [G-18]Do not cache constants to save gas

```solidity

file: contracts/Curves.sol

264  uint256 supply = curvesTokenSupply[curvesTokenSubject];

283     uint256 supply = curvesTokenSupply[curvesTokenSubject];
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L264
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L283

```solidity

file: contracts/FeeSplitter.sol

90   uint256 totalSupply_ = totalSupply(token);

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L90

## [G-19] Avoid transferring amounts of zero in order to save gas
Skipping the external call when nothing will be transferred, will save at least **100 gas**

```solidity

file: contracts/Curves.sol

308 _transfer(subjects[i], msg.sender, to, amount);

324   emit Transfer(curvesTokenSubject, from, to, amount);

486 _transfer(curvesTokenSubject, msg.sender, address(this), amount);

501  _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L308
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L324
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L486
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L501

## [G-20] Using mappings instead of arrays to avoid length checks save gas
Just by using a mapping, we get a gas saving of 2102 gas. When you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index 

```solidity

file: contracts/Curves.sol

101  mapping(address => address[]) private ownedCurvesTokenSubjects;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L101

```solidity

file: contracts/FeeSplitter.sol

29  mapping(address => address[]) internal userTokens;
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L29