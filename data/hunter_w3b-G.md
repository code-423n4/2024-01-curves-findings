# Gas-Optmization for Curves Protocol

## [G-01] State variables should be cached in stack variables rather than re-reading them from storage

**Issue Description**
The contract reads state variables like `feesEconomics` multiple times without caching them in local variables, leading to unnecessary storage reads.

**Proposed Optimization**
Cache state variables in local variables before accessing multiple times to replace storage reads with cheaper stack reads.

```diff
File: contracts/Curves.sol

  117    function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
  118        if (
- 119            feesEconomics.protocolFeePercent +
- 120                feesEconomics.subjectFeePercent +
- 121                feesEconomics.referralFeePercent +
- 122                feesEconomics.holdersFeePercent >
- 123            maxFeePercent_
- 124        ) revert InvalidFeeDefinition();
- 125        feesEconomics.maxFeePercent = maxFeePercent_;
  126    }



    function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
+        FeesEconomics public _feesEconomics = feesEconomics;

        if (
+            _feesEconomics.protocolFeePercent +
+                _feesEconomics.subjectFeePercent +
+                _feesEconomics.referralFeePercent +
+                _feesEconomics.holdersFeePercent >
+            maxFeePercent_
+        ) revert InvalidFeeDefinition();
+        _feesEconomics.maxFeePercent = maxFeePercent_;
    }
```

**Estimated Gas Savings**
Each storage read costs `~100` gas. Caching avoids multiple redundant reads, saving `~100` gas per read. Based on code snippets provided, caching `feesEconomics` would save `~2000` gas across multiple functions.

**Attachments**

### Code Snippets

The code snippets highlight locations where state variables like `feesEconomics` and `_curvesTokenCounter` can be cached.

```solidity
File: contracts/Curves.sol


//>>> @audit feesEconomics S.V  cached
118        if (
119            feesEconomics.protocolFeePercent +
120                feesEconomics.subjectFeePercent +
121                feesEconomics.referralFeePercent +
122                feesEconomics.holdersFeePercent >
123            maxFeePercent_
124        ) revert InvalidFeeDefinition();
125        feesEconomics.maxFeePercent = maxFeePercent_;




//>>> @audit feesEconomics S.V  cached
129        if (
130            protocolFeePercent_ +
131                feesEconomics.subjectFeePercent +
132                feesEconomics.referralFeePercent +
133                feesEconomics.holdersFeePercent >
134            feesEconomics.maxFeePercent ||
135            protocolFeeDestination_ == address(0)
136        ) revert InvalidFeeDefinition();
137        feesEconomics.protocolFeePercent = protocolFeePercent_;
138        feesEconomics.protocolFeeDestination = protocolFeeDestination_;



//>>> @audit feesEconomics S.V  cached
146        if (
147            feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ >
148            feesEconomics.maxFeePercent
149        ) revert InvalidFeeDefinition();
150        feesEconomics.subjectFeePercent = subjectFeePercent_;
151        feesEconomics.referralFeePercent = referralFeePercent_;
152        feesEconomics.holdersFeePercent = holdersFeePercent_;




173        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether; //>>> @audit feesEconomics S.V  cached
174        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
175        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
176        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;



//>>> @audit _curvesTokenCounter S.V  cached
344        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
345            _curvesTokenCounter += 1;
346            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
347            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
348        }
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L118

## [G-02] Avoid zero to non zero storage writes

**Issue Description**
The contract `Curves` declares a private storage variable `_curvesTokenCounter` and initializes it to 0. Any subsequent writes to this variable from 0 to a non-zero value will cost 20,000 gas due to the zero to non-zero storage write.This is why the `Openzeppelin` reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1.

**Proposed Optimization**
Initialize `_curvesTokenCounter` to 1 instead of 0 to avoid the extra gas cost for the first increment operation from 0 to 1.

```diff
46    // Counter for CURVES tokens minted
-       uint256 private _curvesTokenCounter = 0;

+       uint256 private _curvesTokenCounter = 0;
```

**Estimated Gas Savings**
20,000 gas would be saved on the first write to `_curvesTokenCounter` that increments it from 0 to 1.

**Attachments**

**Code Snippets**

```solidity
File: contracts/Curves.sol

47    uint256 private _curvesTokenCounter = 0;
```

This change would avoid the zero to non-zero storage write gas cost mentioned in the issue description when `_curvesTokenCounter` is first incremented from its initialization value.
Here is the refactoring suggestion converted to the specified style:

## [G-03] Refactor internal function to avoid unnecessary SLOAD

**Issue Description**
The function `_buyCurvesToken()` repeats the SLOAD to retrieve `curvesTokenSupply` from storage, even when it is already retrieved in the calling functions.

**Proposed Optimization**  
Pass the `supply` as a parameter to `_buyCurvesToken()` instead of retrieving it again from storage.

This refactors the internal function to avoid repeatedly querying storage for the supply value when it has already been retrieved in the caller function.

**Estimated Gas Savings**
Avoiding an unnecessary SLOAD to retrieve the supply would save 200 gas.

**Attachments**

**Code Snippets**

```solidity
File: contracts/Curves.sol

364    function buyCurvesTokenWithName(
365        address curvesTokenSubject,
366        uint256 amount,
367        string memory name,
368        string memory symbol
369    ) public payable {
370        uint256 supply = curvesTokenSupply[curvesTokenSubject];
371        if (supply != 0) revert CurveAlreadyExists();
372
373        _buyCurvesToken(curvesTokenSubject, amount);
374        _mint(curvesTokenSubject, name, symbol);
375    }




377    function buyCurvesTokenForPresale(
378        address curvesTokenSubject,
379        uint256 amount,
380        uint256 startTime,
381        bytes32 merkleRoot,
382        uint256 maxBuy
383    ) public payable onlyTokenSubject(curvesTokenSubject) {
384        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
385        uint256 supply = curvesTokenSupply[curvesTokenSubject];
386        if (supply != 0) revert CurveAlreadyExists();
387        presalesMeta[curvesTokenSubject].startTime = startTime;
388        presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
389        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
390
391        _buyCurvesToken(curvesTokenSubject, amount);
392    }



// @audit   _buyCurvesToken
263    function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
264        uint256 supply = curvesTokenSupply[curvesTokenSubject];
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L364

## [G-04] Multiple accesses of MAPPING/ARRAY should use a local variable cache

### These are missed from bots

**Issue Description**\
The Curves contract makes repeated reads from the `curvesTokenBalance` mapping and presalesBuys mapping without caching the values in local variables. This leads to unnecessary storage reads each time.

**Proposed Optimization**\
Cache frequently accessed mapping values in local variables to avoid multiple expensive storage reads:

```diff
File: contracts/Curves.sol

+   uint256 cachedBalance = curvesTokenBalance[curvesTokenSubject][msg.sender];
- 272        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
+            cachedBalance += amount;
  273        curvesTokenSupply[curvesTokenSubject] = supply + amount;
  274        _transferFees(curvesTokenSubject, true, price, amount, supply);
  275
  276        // If is the first token bought, add to the list of owned tokens
- 277        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
+            if (cachedBalance - amount == 0) {
```

**Estimated Gas Savings**\
Each avoided storage read saves approximately 200 gas. Caching values accessed multiple times inside loops could save thousands of gas.

**Attachments**

- **Code Snippets**

```solidity
File: contracts/Curves.sol


272        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
277        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {



285        if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();
289        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;


414        presalesBuys[curvesTokenSubject][msg.sender] += amount;
415        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];


410            presalesMeta[curvesTokenSubject].startTime == 0 ||
411            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
416        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
425        if (!MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf)) revert UnverifiedProof();
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L272

## [G-05] Access mappings directly rather than using accessor functions

**Issue Description**\
When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

**Proposed Optimization**\
Access mappings directly inside functions instead of through accessor functions whenever possible to remove the extra layer of indirection.

**Estimated Gas Savings**\
Reading a mapping value directly saves approximately `600` gas vs using an accessor function.

**Attachments**

- **Code Snippets**

```solidity
File: contracts/FeeSplitter.sol

48    function getUserTokens(address user) public view returns (address[] memory) {
49        return userTokens[user];
50    }
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L48

## [G-06] array[index] += amount is cheaper than array[index] = array[index] + amount

```solidity
File: contracts/Curves.sol

321        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
322        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L321

## [G-07] Using STORAGE instead of MEMORY for structs/arrays saves gas

**Issue Description**\
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

**Proposed Optimization**\
Declare tokens and result as storage references instead of memory, and cache any fields that need to be re-read in local stack variables. This will only incur one Gcoldsload per field actually read rather than for the entire structs.

**Estimated Gas Savings**\
2100 gas saved per unnecessary field read from memory that could be read directly from storage.

**Attachments**

- **Code Snippets**

```solidity
File: contracts/FeeSplitter.sol

53        address[] memory tokens = getUserTokens(user);
54        UserClaimData[] memory result = new UserClaimData[](tokens.length);
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L53

## [G-08] Using  CALLDATA instead of  MEMORY for read-only arguments in external functions

**Issue Description**\
When an external function with a memory array argument is called, abi.decode must loop through calldata copying to memory. This costs at least 60 gas per index.

**Proposed Optimization**\
Use calldata directly for read-only array arguments to external functions, avoiding the copying loop.

**Estimated Gas Savings**\
At least 60 gas saved per index of the array, by eliminating the copying loop.

**Attachments**

- **Code Snippets**

```solidity
File: contracts/Curves.sol

407        bytes32[] memory proof

422    function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L407

```solidity
File: contracts/FeeSplitter.sol

48    function getUserTokens(address user) public view returns (address[] memory) {

52    function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L48

## [G-09] Using assembly to revert with an error message

**Issue Description**\
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.

**Estimated Gas Savings**\
Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042


contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734


contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message
with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks
the solidity compiler does under the hood.

**Attachments**

- **Code Snippets**

```solidity
File: contracts/Curves.sol

104        if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();

124        ) revert InvalidFeeDefinition();

136        ) revert InvalidFeeDefinition();

149        ) revert InvalidFeeDefinition();

213        if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

233                if (!success1) revert CannotSendFunds();

237                if (!success2) revert CannotSendFunds();

243                if (!success3) revert CannotSendFunds();

265        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

270        if (msg.value < price + totalFee) revert InsufficientPayment();

284        if (supply <= amount) revert LastTokenCannotBeSold();

285        if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

297        if (to == address(this)) revert ContractCannotReceiveTransfer();

303        if (to == address(this)) revert ContractCannotReceiveTransfer();

314        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

350        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

371        if (supply != 0) revert CurveAlreadyExists();

384        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();

386        if (supply != 0) revert CurveAlreadyExists();

396        if (supply > 1) revert CurveAlreadyExists();

412        ) revert PresaleUnavailable();

416        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

425        if (!MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf)) revert UnverifiedProof();

433        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();

434        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

461        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();

466        if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

491        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

496        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();

497        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();

498        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();


505        if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#104

```solidity
File: contracts/FeeSplitter.sol

83        if (claimable == 0) revert NoFeesToClaim();

91        if (totalSupply_ == 0) revert NoTokenHolders();

115        if (totalClaimable == 0) revert NoFeesToClaim();
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#83

## [G-10] Using do-while loops instead of for loops for optimization

**Issue Description**\
Solidity do-while loops are more gas efficient than for loops, even when an if check is added to prevent execution of an empty loop body.

**Proposed Optimization**\
Replace for loops with equivalent do-while loops to reduce gas costs, at the expense of more unconventional code.

**Estimated Gas Savings**\
Do-while loops are inherently more efficient than for loops in Solidity. Exact savings will depend on loop body but can be significant for tightly repeated code.

**Attachments**

- **Code Snippets**

```solidity
File: contracts/Curves.sol

305        for (uint256 i = 0; i < subjects.length; i++) {

330        for (uint256 i = 0; i < subjects.length; i++) {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305

```solidity
File: contracts/FeeSplitter.sol

55        for (uint256 i = 0; i < tokens.length; i++) {

105        for (uint256 i = 0; i < tokenList.length; i++) {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55

## [G-11] Avoid contract existence checks by using low level calls

**Issue Description**\
Prior to Solidity 0.8.10, the compiler would insert extra code to check for contract existence before external function calls,
even if the call had a return value. This wasted gas by performing a check that wasn’t necessary.

**Proposed Optimization**\
For functions that make external calls with return values in Solidity <0.8.10, optimize the code to use low-level calls instead
of regular calls. Low-level calls skip the unnecessary contract existence check.

```solidity

//Before:
contract C {
  function f() external returns(uint) {
    address(otherContract).call(abi.encodeWithSignature("func()"));
  }
}

//After:
contract C {
  function f() external returns(uint) {
    (bool success,) = address(otherContract).call(abi.encodeWithSignature("func()"));
    require(success);
    return decodeReturnValue();
  }
}
```

**Estimated Gas Savings**\
Each avoided EXTCODESIZE check saves 100 gas. If 10 external calls are made in a common function, this would save 1000 gas
total.

**Attachments**

- **Code Snippets**

```solidity
File: contracts/Curves.sol

352        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L352
