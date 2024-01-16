# Gas report

## Table Of Contents

- [Gas report](#gas-report)
  - [Table Of Contents](#table-of-contents)
  - [Refactor the `getFees` function to save gas](#g-01-refactor-the-getfees-function-to-save-gas)
  - [No need to initialize variable `_curvesTokenCounter` to 0](#g-02-no-need-to-initialize-variable-_curvestokencounter-to-0)
  - [Unchecked arithmetic operations that cannot underflow/overflow(The bot missed these)](#g-03-unchecked-arithmetic-operations-that-cannot-underflowoverflowthe-bot-missed-these)
  - [Consider using solady's optimize library](#g-04-consider-using-soladys-optimize-library)
  - [Assembly: Use scratch space when building emitted events with two data arguments](#g-05-assembly-use-scratch-space-when-building-emitted-events-with-two-data-arguments)
  - [Don’t cache state variable if used once](#g-06-dont-cache-state-variable-if-used-once)
  - [Use calldata instead of memory for function parameters](#g-07-use-calldata-instead-of-memory-for-function-parameters)
  - [Refactor the `batchClaming` Function for better gas optimization](#g-08-refactor-the-batchclaming-function-for-better-gas-optimization)
  - [Cache state variables in memory(not found by bots)](#g-09-cache-state-variables-in-memorynot-found-by-bots)
  - [Due to how short circuit works, we can save an entire SLOAD by performing sloads only when needed(Save 2100 Gas)](#g-10-due-to-how-short-circuit-works-we-can-save-an-entire-sload-by-performing-sloads-only-when-neededsave-2100-gas)
  - [Optimizing check order for cost-efficient function execution](#g-11-optimizing-check-order-for-cost-efficient-function-execution)
    - [In the deposit() function change order of  `tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this` condition](#in-the-deposit-function-there-is-a-check-for-tokenamount--curvestokenbalancecurvestokensubjectaddressthis-which-is-state-read-and-the-second-check-involves-an-external-function-check-which-high-gas-consume-compare-to-the-state-read-so-its-recommended-that-change-the-order-of-if-condition)
  - [Multiple accesses of the same mapping/array key/index should be cached- Not cached by bot](#g-12-multiple-accesses-of-the-same-mappingarray-keyindex-should-be-cached--not-cached-by-bot)

## [G-01] Refactor the `getFees` function to save gas

### We can refactor the return statement to save 3 Mul opcodes and 3 DIV opcodes which is going to save 15, 15 gas respectively.

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L165C1-L179C1

```solidity
File: contracts/Curves.sol
166: function getFees(
        uint256 price
    )
        public
        view
        returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
    {
        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether; //@audit Refactor this Calculation
        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
    }
```

The above code first calculates the total of all fees and then multiplies it with the price and finally divides the result by 1 Ether and then all results and get the final value of total fees.

This reduces the number of division operations to one, potentially optimizing gas usage in the contract execution.

**Optimized code**

```diff
File: contracts/Curves.sol
166: function getFees(
        uint256 price
    )
        public
        view
-        returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
+        returns (uint256 totalFee)
    {
-        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether; //@audit Refactor this Calculation
-        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
-        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
-        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
-        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
-   }
+        total_fee = feesEconomics.protocolFeePercent + feesEconomics.subjectFeePercent + feesEconomics.referralFeePercent + feesEconomics.holdersFeePercent ;
+        totalFee = (price * totalFee)/1 ether;
+       }
```

## [G-02] No need to initialize variable `_curvesTokenCounter` to 0

0 is the default value for an unsigned integer, thus setting it to 0 again is not required.

If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). If you explicitly initialize it with its default value, you increase deployment cost and size

Initializing variables with their default values requires additional bytecode instructions to set those values explicitly during deployment. By excluding these instructions, the overall size of the compiled contract's bytecode can be reduced.

The gas cost difference (remix and foundry) is 2206 gas per variable

We get 3 extra opcodes when we initialize it.

```
PUSH1 00 --> 3 gas
DUP1     --> 3 gas
SSTORE   --> 2200 gas

```

The above explains the `2206 gas` difference.

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L47

**Optimized code**

```diff
File: contracts/Curves.sol
-47: uint256 private _curvesTokenCounter = 0;
+47: uint256 private _curvesTokenCounter;
```

## [G-03] **Unchecked arithmetic operations that cannot underflow/overflow(The bot missed these)**

The operation `supply - amount` cannot overflow due to the check on Line 289 and L 288

From the `sellCurvesToken()` function below the if statement on l 289 ensures that the value of `curvesTokenBalance[curvesTokenSubject][msg.sender]` is greater or equals the value of `amount` before the calculation can occur thereby making it impossible that the calculation would underflow. Therefore we can save up to `40` gas units if we do this calculation in an `unchecked{}` block as this would save some gas from the unnecessary internal over/underflow checks. The diff below shows how the code could be refactored:

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L289C5-L290C65

**Optimized code**

```diff
File: contracts/CurvesERC20.sol
286: function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply <= amount) revert LastTokenCannotBeSold();
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance(); 

        uint256 price = getPrice(supply - amount, amount);

-        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount; //@audit unchecked here
-        curvesTokenSupply[curvesTokenSubject] = supply - amount;

+ unchecked{
+        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
+        curvesTokenSupply[curvesTokenSubject] = supply - amount;
+   }

        _transferFees(curvesTokenSubject, false, price, amount, supply);
    }
```

## [G-04] Consider using solady's optimize library

`FixedPointMathLib`

Saves gas, and works to avoid unnecessary [overflows](https://github.com/Vectorized/solady/blob/6cce088e69d6e46671f2f622318102bd5db77a65/src/utils/FixedPointMathLib.sol#L267).

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L179C1-L188C1

**Optimized code**

```solidity
File: contracts/Curves.sol
184: function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) { 
        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
        return (summation * 1 ether) / 16000; //@audit
    }
```

## [G-05] Assembly: Use scratch space when building emitted events with two data arguments

Using the [scratch space](https://gist.github.com/IllIllI000/87c4f03139fa03780fa548b8e4b02b5b) for more than one, but at most two words worth of data (non-indexed arguments) will save gas over needing Solidity's abi memory expansion used for emitting normally.

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L394C1-L402C6

**Optimized code**

```solidity
File: contracts/Curves.sol
398: function setWhitelist(bytes32 merkleRoot) external {
        uint256 supply = curvesTokenSupply[msg.sender];
        //n the setWhitelist function, there is a check for if (supply > 1) revert CurveAlreadyExists();. This condition checks if the supply is greater than 1. It's essential to understand the context of this check and whether it aligns with the intended logic. Typically, for a new whitelist setting, you might expect the supply to be zero, not necessarily 1.
        if (supply > 1) revert CurveAlreadyExists(); 

        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
            presalesMeta[msg.sender].merkleRoot = merkleRoot;
            emit WhitelistUpdated(msg.sender, merkleRoot); //@audit Use scratch space
        }
    }
```

## [G-06] Don’t cache state variable if used once

Caching here will simply increase gas cost, since it doesn't affect readability, we should not cache since we only reference the cached variables only once

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L404C2-L414C46

**Optimized code**

```diff
File: contracts/Curves.sol
409: function buyCurvesTokenWhitelisted(
        address curvesTokenSubject,
        uint256 amount,
        bytes32[] memory proof
    ) public payable {
        if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();

        presalesBuys[curvesTokenSubject][msg.sender] += amount;
-        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender]; //@audit Don't Cache state variables only used once
-        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
+        if (presalesBuys[curvesTokenSubject][msg.sender] > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
        verifyMerkle(curvesTokenSubject, msg.sender, proof);
        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L390C1-L402C6

**Optimized code**

```diff
File: contracts/Curves.sol
398: function setWhitelist(bytes32 merkleRoot) external {
-        uint256 supply = curvesTokenSupply[msg.sender];
-        if (supply > 1) revert CurveAlreadyExists(); //@audit Don't Cache state variables only used once
+        if (curvesTokenSupply[msg.sender] > 1) revert CurveAlreadyExists();
        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
            presalesMeta[msg.sender].merkleRoot = merkleRoot;
            emit WhitelistUpdated(msg.sender, merkleRoot);
        }
    }
```

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L360C3-L376C1

**Optimized code**

```diff
File: contracts/Curves.sol
368: function buyCurvesTokenWithName(
        address curvesTokenSubject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) public payable {
-        uint256 supply = curvesTokenSupply[curvesTokenSubject];
-        if (supply != 0) revert CurveAlreadyExists(); //@audit Don't Cache state variables only used once
+        if (supply != 0) revert CurveAlreadyExists();

        _buyCurvesToken(curvesTokenSubject, amount);
        _mint(curvesTokenSubject, name, symbol);
    }
```

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L377C3-L392C6

**Optimized code**

```diff
File: contracts/Curves.sol
381: function buyCurvesTokenForPresale(
        address curvesTokenSubject,
        uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
    ) public payable onlyTokenSubject(curvesTokenSubject) {
        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
-        uint256 supply = curvesTokenSupply[curvesTokenSubject]; //@audit Don't Cache state variables only used once
-        if (supply != 0) revert CurveAlreadyExists();
+        if (curvesTokenSupply[curvesTokenSubject] != 0) revert CurveAlreadyExists();
        presalesMeta[curvesTokenSubject].startTime = startTime; 
        presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);

        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

## [G-07] Use calldata instead of memory for function parameters

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I've also flagged instances where the function is public but can be marked as external since it's not called by the contract, and cases where a constructor is involved

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L406C24-L407C4

**Optimized code**

```diff
File: contracts/curves.sol
408: function buyCurvesTokenWhitelisted( 
        address curvesTokenSubject,
        uint256 amount,
-        bytes32[] memory proof //@audit
+        bytes32[] calldata proof
    ) public payable {
        if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();

        presalesBuys[curvesTokenSubject][msg.sender] += amount;
        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender]; //@ok Don't Cache state variables only used once
        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

        verifyMerkle(curvesTokenSubject, msg.sender, proof);
        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L421C1-L426C6

**Optimized code**

```diff
File: contracts/curves.sol
-426: function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view {
+426: function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] calldata proof) public view {
        // Verify merkle proof
        bytes32 leaf = keccak256(abi.encodePacked(caller));
        if (!MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf)) revert UnverifiedProof();
    }
```

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L428C2-L432C19

**Optimized code**

```diff
File: contracts/curves.sol
432:  function setNameAndSymbol(
            address curvesTokenSubject,
-            string memory name,
-            string memory symbol
+            string calldata name,
+            string calldata symbol
        ) external onlyTokenSubject(curvesTokenSubject) {
            if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
            if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
            externalCurvesTokens[curvesTokenSubject].name = name; //@ok cache multiple array 
            externalCurvesTokens[curvesTokenSubject].symbol = symbol;
        }
```

## [G-08] Refactor the `batchClaming` Function for better gas optimization

Instead of transferring funds to msg.sender directly, use a conditional check to avoid unnecessary gas costs when transferring zero funds.

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L103C1-L117C6

```solidity
File: contracts/FeeSplitter.sol
103: function batchClaiming(address[] calldata tokenList) external {
        uint256 totalClaimable = 0;
        for (uint256 i = 0; i < tokenList.length; i++) {
            address token = tokenList[i];
            updateFeeCredit(token, msg.sender);
            uint256 claimable = getClaimableFees(token, msg.sender);
            if (claimable > 0) {
                tokensData[token].unclaimedFees[msg.sender] = 0;
                totalClaimable += claimable;
                emit FeesClaimed(token, msg.sender, claimable);
            }
        }
        if (totalClaimable == 0) revert NoFeesToClaim();
        payable(msg.sender).transfer(totalClaimable); //@audit need to refactor this
}
```

**Optimized code**

```diff
File: contracts/FeeSplitter.sol
103: function batchClaiming(address[] calldata tokenList) external {
        uint256 totalClaimable = 0;
        for (uint256 i = 0; i < tokenList.length; i++) {
            address token = tokenList[i];
            updateFeeCredit(token, msg.sender);
            uint256 claimable = getClaimableFees(token, msg.sender);
            if (claimable > 0) {
                tokensData[token].unclaimedFees[msg.sender] = 0;
                totalClaimable += claimable;
                emit FeesClaimed(token, msg.sender, claimable);
            }
        }
-        if (totalClaimable == 0) revert NoFeesToClaim();
-        payable(msg.sender).transfer(totalClaimable); 
+        if (totalClaimable > 0) {
+        payable(msg.sender).transfer(totalClaimable);
}
```

## [G-09] **Cache state variables in memory(not found by bots)**

We can cache `feesEconimocs` and `feeRedistributor` to save 4 sloads (-400 gas)

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L116C1-L126C6

**Optimized code**

```diff
File: contracts/Curves.sol
117: function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
+      feesEconomics = _feesEconomics;
        if (
-            feesEconomics.protocolFeePercent + //@audit cache state variable
-                feesEconomics.subjectFeePercent +
-                feesEconomics.referralFeePercent +
-                feesEconomics.holdersFeePercent >
+                _feesEconomics.protocolFeePercent + 
+                _feesEconomics.subjectFeePercent +
+                _feesEconomics.referralFeePercent +
+                _feesEconomics.holdersFeePercent >
            maxFeePercent_
        ) revert InvalidFeeDefinition();
        feesEconomics.maxFeePercent = maxFeePercent_; 
    }
```

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L139C1-L153C6

**Optimized code**

```diff
File: contracts/Curves.sol
141: function setExternalFeePercent(
        uint256 subjectFeePercent_,
        uint256 referralFeePercent_,
        uint256 holdersFeePercent_
    ) external onlyManager {
+       feesEconomics = _feesEconomics;
-        if (
-            feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ > //@audit cache state variable
-            feesEconomics.maxFeePercent
+        if (
+            _feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ > //@audit cache state variable
+            _feesEconomics.maxFeePercent
        ) revert InvalidFeeDefinition();
        feesEconomics.subjectFeePercent = subjectFeePercent_;
        feesEconomics.referralFeePercent = referralFeePercent_;
        feesEconomics.holdersFeePercent = holdersFeePercent_;
    }
```

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L167C4-L179C1

**Optimized code**

```diff
File: contracts/Curves.sol
166: function getFees(
        uint256 price
    )
        public
        view
        returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
    {
+       feesEconomics = _feesEconomics;
-        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether; 
-        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether; //@audit cache feeEconomics
-        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
-        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
+        protocolFee = (price * _feesEconomics.protocolFeePercent) / 1 ether; 
+        subjectFee = (price * _feesEconomics.subjectFeePercent) / 1 ether; 
+        referralFee = (price * _feesEconomics.referralFeePercent) / 1 ether;
+        holdersFee = (price * _feesEconomics.holdersFeePercent) / 1 ether;
        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
    }
/*
```

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L284C6-L285C103

**Optimized code**

```diff
FIle: contracts/Curves.sol
285: function sellCurvesToken(address curvesTokenSubject, uint256 amount) public { 
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply <= amount) revert LastTokenCannotBeSold();
+       CurvesTokenBalance memory curvesTokenBalance = _curvesTokenBalance;
-        if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance(); //@audit cache state variable see l 289
+        if (_curvesTokenBalance < amount) revert InsufficientBalance();
        uint256 price = getPrice(supply - amount, amount);

-        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
+        curvesTokenBalance[curvesTokenSubject][msg.sender] = _curvesTokenBalance - amount;
        curvesTokenSupply[curvesTokenSubject] = supply - amount;

        _transferFees(curvesTokenSubject, false, price, amount, supply); 
    }
```

## [G-10] **Due to how short circuit works, we can save an entire SLOAD by performing sloads only when needed(Save 2100 Gas)**

In `_transferFees()` function there is an if condition which is implementededed with use of && condition. We can breakdown this condition into two-part and in a result we can save some gas 

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L246C11-L250C10

```solidity
File: contrcats/Curves.sol
218: function _transferFees(
        address curvesTokenSubject,
        bool isBuy,
        uint256 price,
        uint256 amount,
        uint256 supply
    ) internal {
 ....
            if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
            }
        }
```

**Optimized code**

```diff
-249: if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) { //@audit use nested if
-                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
-	                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
+      if (feesEconomics.holdersFeePercent > 0 ) 
+      {
+       if( address(feeRedistributor) != address(0)
+       {
+                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
+	                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
            }
+      }
```

## [G-11] **Optimizing check order for cost-efficient function execution**

### **In the deposit() function there is a check for `tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]` which is state read and the second check involves an external function check which high gas consume compare to the state read. So its recommended that change the order of if condition**

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L489C1-L502C6

```solidity
File: contracts/CurvesERC20.sol
494: function deposit(address curvesTokenSubject, uint256 amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
        uint256 tokenAmount = amount / 1 ether;

        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance(); 
        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance(); //@audit revert first

        CurvesERC20(externalToken).burn(msg.sender, amount);
        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
    }
```

**Optimized code**

```diff
File: contracts/CurvesERC20.sol
494: function deposit(address curvesTokenSubject, uint256 amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
        uint256 tokenAmount = amount / 1 ether;
+   if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance(); 
        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance(); 
-        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance(); 

        CurvesERC20(externalToken).burn(msg.sender, amount);
        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
    }
```

## [G-12] Multiple accesses of the same mapping/array key/index should be cached- Not cached by bot

Caching repeated accesses to the same mapping or array key/index in smart contracts can lead to significant gas savings. In Solidity, each read operation from storage (like accessing a value in a mapping or array using a key or index) costs gas. By storing the accessed value in a local variable and reusing it within the function, you avoid multiple expensive storage read operations. This practice is particularly beneficial in loops or functions with multiple reads of the same data. Implementing this caching approach enhances efficiency and reduces transaction costs, which is crucial for optimizing smart contract performance and user experience on the blockchain.

**This instance is not include in Bot automatic finding report**

- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L313C1-L325C6

**Optimized code**

```diff
File: contracts/CurvesERC20.sol
316: function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
+        curvesTokenBalance memory _curvesTokenBalance = curvesTokenBalance[curvesTokenSubject][from];
        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

        // If transferring from oneself, skip adding to the list
        if (from != to) {
            _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
        }

-        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount; //@audit cache multiple array variable 
+        curvesTokenBalance[curvesTokenSubject][from] = _curvesTokenBalance- amount;
        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount; 
    }
```