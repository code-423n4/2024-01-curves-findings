# Curve Gas Optimizations



## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."




## [G-01] Calculations should be memoized rather than re-calculating them 
In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls to pure functions and returning the cached result when the same inputs occur again.

### 2 Instances
1. ####  We can memoize the computations of `Strings.toString(_curvesTokenCounter))`
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L346

We can reduce the gas usage of the `_deployERC20()` function by memoizing the computations of `Strings.toString(_curvesTokenCounter))` i.e cache the results of the computation so that for subsequent computations we can use the cached values rather than having to recompute the value of `Strings.toString(_curvesTokenCounter))`. The diff below shows how the code could be refactored:

```solidity
file: contracts/Curves.sol

338:    function _deployERC20(
339:        address curvesTokenSubject,
340:        string memory name,
341:        string memory symbol
342:    ) internal returns (address) {
343:        // If the token's symbol is CURVES, append a counter value
344:        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
345:            _curvesTokenCounter += 1;
346:            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));  //@audit memoize Strings.toString(_curvesTokenCounter)
347:            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
348:        }
349:
.
.
.
362:    }
```
```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..3b3fc3f 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -343,8 +343,9 @@ contract Curves is CurvesErrors, Security {
         // If the token's symbol is CURVES, append a counter value
         if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
             _curvesTokenCounter += 1;
-            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
-            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
+            string memory curvesTokenCounterString = Strings.toString(_curvesTokenCounter);
+            name = string(abi.encodePacked(name, " ", curvesTokenCounterString));
+            symbol = string(abi.encodePacked(symbol, curvesTokenCounterString));
         }

         if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
```


2. #### We can memoize the computations of `supply - 1`
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L180-#L187

We can reduce the gas usage of the `getPrice()` function by memoizing the computations of `supply - 1` i.e cache the results of the computation so that for subsequent computations we can use the cached values rather than having to recompute the values. The diff below shows how the code could be refactored:

```solidity
file: contracts/Curves.sol

180:    function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
181:        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
182:        uint256 sum2 = supply == 0 && amount == 1
183:            ? 0
184:            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
185:        uint256 summation = sum2 - sum1;
186:        return (summation * 1 ether) / 16000;
187:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..19d86a1 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -178,10 +178,11 @@ contract Curves is CurvesErrors, Security {
     }

     function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
-        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
+        uint256 cachedVal = supply - 1;
+        uint256 sum1 = supply == 0 ? 0 : ((cachedVal) * (supply) * (2 * (cachedVal) + 1)) / 6;
         uint256 sum2 = supply == 0 && amount == 1
             ? 0
-            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
+            : ((supply - 1 + amount) * (supply + amount) * (2 * (cachedVal + amount) + 1)) / 6;
         uint256 summation = sum2 - sum1;
         return (summation * 1 ether) / 16000;
     }
```




## [G-02] Non efficient zero initialization
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

### Proof of concept
Using a simple Remix test

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;


contract InitializeDefaultValue {
    
    uint256 num = 0;

}
```
```
Deployment gas cost: 14669
```


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract NoInitializeDefaultValue {
    
    uint256 num;

}
```
```
Deployment gas cost: 12464
```

### Instances
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L47
```solidity
file: contracts/Curves.sol

47:    uint256 private _curvesTokenCounter = 0;
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..1a225d8 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -44,7 +44,7 @@ contract Curves is CurvesErrors, Security {
     string public constant DEFAULT_NAME = "Curves";
     string public constant DEFAULT_SYMBOL = "CURVES";
     // Counter for CURVES tokens minted
-    uint256 private _curvesTokenCounter = 0;
+    uint256 private _curvesTokenCounter;
```
```
Estimated gas saved: 2205 gas units
```




## [G-03]  State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### Instances
1. #### Cache `feeRedistributor` in a stack variable
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L246-#L248

We can reduce the gas cost of the `_transferFees()` function by reducing the number of storage reads (`SLOAD`) for the state variable `feeRedistributor`. The value of the `feeRedistributor` state variable should be cached in a stack variable then the stack variable be used for subsequent reads of the `feeRedistributor` state variable. In implementing this we replace 2 `SLOAD`s(cold acess) `200` gas units with  much cheaper 2 `MLOAD` `6` gas units: The diff below shows how the code could be refactored:

```solidity
file: contracts/Curves.sol

218:    function _transferFees(
219:        address curvesTokenSubject,
220:        bool isBuy,
221:        uint256 price,
222:        uint256 amount,
223:        uint256 supply
224:    ) internal {
225:        (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
.
.
.
246:            if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {   //@audit 1st feeRedistributor SLOAD
247:                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);   //@audit 2nd feeRedistributor SLOAD
248:                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);   //@audit 3rd feeRedistributor SLOAD
249:            }
250:        }
251:        emit Trade(
252:            msg.sender,
253:            curvesTokenSubject,
254:            isBuy,
255:            amount,
256:            price,
257:            protocolFee,
258:            subjectFee,
259:            isBuy ? supply + amount : supply - amount
260:        );
261:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..8b4ae66 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -242,10 +242,10 @@ contract Curves is CurvesErrors, Security {
                     : (true, bytes(""));
                 if (!success3) revert CannotSendFunds();
             }
-
-            if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
-                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
-                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
+            address _feeRedistributor = feeRedistributor;
+            if (feesEconomics.holdersFeePercent > 0 && address(_feeRedistributor) != address(0)) {
+                _feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
+                _feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
             }
         }
         emit Trade(
```
```
Estimated gas saved: 194 gas units
```

2. #### Cache `_curvesTokenCounter` in a stack variable
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L346-#L347

We can reduce the gas cost of the `_deployERC20()` function by reducing the number of storage reads (`SLOAD`) for the state variable `_curvesTokenCounter`. The value of the `_curvesTokenCounter` state variable should be cached in a stack variable then the stack variable be used for subsequent reads of the `_curvesTokenCounter` state variable. In implementing this we replace 2 `SLOAD`s(cold acess) `200` gas units with  much cheaper 2 `MLOAD` `6` gas units: The diff below shows how the code could be refactored:

```solidity
file: contracts/Curves.sol

338:    function _deployERC20(
339:        address curvesTokenSubject,
340:        string memory name,
341:        string memory symbol
342:    ) internal returns (address) {
343:        // If the token's symbol is CURVES, append a counter value
344:        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
345:            _curvesTokenCounter += 1;
346:            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));  //@audit _curvesTokenCounter 1st SLOAD
347:            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));  //@audit _curvesTokenCounter 2nd SLOAD
348:        }
349:
.
.
.
362:    }
```
```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..d10aaf3 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -342,9 +342,9 @@ contract Curves is CurvesErrors, Security {
     ) internal returns (address) {
         // If the token's symbol is CURVES, append a counter value
         if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
-            _curvesTokenCounter += 1;
-            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
-            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
+            uint256 _curvesTokenCounterIncrement = ++_curvesTokenCounter
+            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounterIncrement)));
+            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounterIncrement)));
         }
```
```
Estimated gas saved: 194 gas units
```


3. #### Cache `curves` in a stack variable
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L45

We can reduce the gas cost of the `totalSupply()` function by reducing the number of storage reads (`SLOAD`) for the state variable `curves`. The value of the `curves` state variable should be cached in a stack variable then the stack variable be used for subsequent reads of the `curves` state variable. In implementing this we replace 2 `SLOAD`s(cold acess) `200` gas units with  much cheaper 2 `MLOAD` `6` gas units: The diff below shows how the code could be refactored:

```solidity
file: contracts/FeeSplitter.sol

43:    function totalSupply(address token) public view returns (uint256) {
44:        //@dev: this is the amount of tokens that are not locked in the contract. The locked tokens are in the ERC20 contract
45:        return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
46:    }
```

```diff
diff --git a/contracts/FeeSplitter.sol b/contracts/FeeSplitter.sol
index bb24f02..a034a6a 100644
--- a/contracts/FeeSplitter.sol
+++ b/contracts/FeeSplitter.sol
@@ -42,7 +42,8 @@ contract FeeSplitter is Security {

     function totalSupply(address token) public view returns (uint256) {
         //@dev: this is the amount of tokens that are not locked in the contract. The locked tokens are in the ERC20 contract
-        return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
+        Curves _curves = curves;
+        return (_curves.curvesTokenSupply(token) - _curves.curvesTokenBalance(token, address(_curves))) * PRECISION;
     }

     function getUserTokens(address user) public view returns (address[] memory) {
```
```
Estimated gas saved: 194 gas units
```


4. #### Cache `data.cumulativeFeePerToken` in a stack variable
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L67-#L69

We can reduce the gas cost of the `updateFeeCredit()` function by reducing the number of storage reads (`SLOAD`) for the state variable `data.cumulativeFeePerToken`. The value of the `data.cumulativeFeePerToken` state variable should be cached in a stack variable then the stack variable be used for subsequent reads of the `data.cumulativeFeePerToken` state variable. In implementing this we replace 1 `SLOAD`s(cold acess) `100` gas units with  much cheaper 1 `MLOAD` `3` gas units: The diff below shows how the code could be refactored:

```solidity
file: contracts/FeeSplitter.sol

63:    function updateFeeCredit(address token, address account) internal {
64:        TokenData storage data = tokensData[token];
65:        uint256 balance = balanceOf(token, account);
66:        if (balance > 0) {
67:            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
68:            data.unclaimedFees[account] += owed / PRECISION;
69:            data.userFeeOffset[account] = data.cumulativeFeePerToken;
70:        }
71:    }
```

```diff
diff --git a/contracts/FeeSplitter.sol b/contracts/FeeSplitter.sol
index bb24f02..0f3db3a 100644
--- a/contracts/FeeSplitter.sol
+++ b/contracts/FeeSplitter.sol
@@ -64,9 +64,10 @@ contract FeeSplitter is Security {
         TokenData storage data = tokensData[token];
         uint256 balance = balanceOf(token, account);
         if (balance > 0) {
-            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
+            uint256 dataCumulativeFeePerToken = data.cumulativeFeePerToken
+            uint256 owed = (dataCumulativeFeePerToken - data.userFeeOffset[account]) * balance;
             data.unclaimedFees[account] += owed / PRECISION;
-            data.userFeeOffset[account] = data.cumulativeFeePerToken;
+            data.userFeeOffset[account] = dataCumulativeFeePerToken;
         }
     }
```
```
Estimated gas saved: 97 gas units
```




## [G-04] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.
the recommended Mitigation Steps is that Functions guaranteed to revert when called by normal users can be marked payable.

#### Please note these instances were not included in the bots reports

### Instances

1. #### Make `FeeSplitter.onBalanceChange()` function payable
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L96-#L100

```solidity
file: contracts/FeeSplitter.sol

96:     function onBalanceChange(address token, address account) public onlyManager {
97:         TokenData storage data = tokensData[token];
98:         data.userFeeOffset[account] = data.cumulativeFeePerToken;
99:         if (balanceOf(token, account) > 0) userTokens[account].push(token);
100:    }
```

```diff
diff --git a/contracts/FeeSplitter.sol b/contracts/FeeSplitter.sol
index bb24f02..b5a830f 100644
--- a/contracts/FeeSplitter.sol
+++ b/contracts/FeeSplitter.sol
@@ -93,7 +93,7 @@ contract FeeSplitter is Security {
         data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_;
     }

-    function onBalanceChange(address token, address account) public onlyManager {
+    function onBalanceChange(address token, address account) public payable onlyManager {
         TokenData storage data = tokensData[token];
         data.userFeeOffset[account] = data.cumulativeFeePerToken;
         if (balanceOf(token, account) > 0) userTokens[account].push(token);
```
```
Estimated gas saved: 21 gas units
```


2. #### Make `Security.transferOwnership()` function payable
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L27-#L30

```solidity
file: 

27:    function transferOwnership(address owner_) public onlyOwner {
28:        owner = owner_;
29    }
```
```diff
diff --git a/contracts/Security.sol b/contracts/Security.sol
index b1dfee5..8d9894d 100644
--- a/contracts/Security.sol
+++ b/contracts/Security.sol
@@ -24,7 +24,7 @@ contract Security {
         managers[manager_] = value;
     }

-    function transferOwnership(address owner_) public onlyOwner {
+    function transferOwnership(address owner_) public payable onlyOwner {
         owner = owner_;
     }
 }
```
```
Estimated gas saved: 21 gas units
```




## [G-05] array.length should not be looked up in every loop of a for-loop
For an array A, `A.length` should not be looked up for every iteration of the loop

#### Please note these instances were not included in the bots reports

### 1 Instance
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L330

The for-loop on [Line 330](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L330) has to look-up the value of `subjects.length` on every iteration of the loop but this value is not dependent on the iteration of the loop rather it is constant through the loops iteration so the value should be saved to a variable and the variable be used in the for-loop. In implementing this we would avoid 1 `SLOAD` (`100` gas units) and and replace it with a much cheaper `MLOAD` (`3` gas units) thereby saving `97` gas units per iteration of the loop the diff below shows how the code could be refactored:

```solidity
file: contracts/Curves.sol

328:    function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
329:        address[] storage subjects = ownedCurvesTokenSubjects[owner_];
330:        for (uint256 i = 0; i < subjects.length; i++) { //@audit cache subjects.length
331:            if (subjects[i] == curvesTokenSubject) {
332:                return;
333:            }
334:        }
335:        subjects.push(curvesTokenSubject);
336:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..d576565 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -327,7 +327,8 @@ contract Curves is CurvesErrors, Security {
     // Internal function to add a curvesTokenSubject to the list if not already present
     function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
         address[] storage subjects = ownedCurvesTokenSubjects[owner_];
-        for (uint256 i = 0; i < subjects.length; i++) {
+        uint256 subjectsLength = subjects.length
+        for (uint256 i = 0; i < subjectsLength; i++) {
             if (subjects[i] == curvesTokenSubject) {
                 return;
             }
```
```
Estimated gas saved: 97 gas per iteration
```




## [G-06] Multiple accesses of a array should use a local variable cache
The instances below point to the second+ access of a value inside an array, within a function. Caching an array's struct avoids re-calculating the array offsets into memory

### Proof of concept 
```solidity
struct Person {
    string name;
    uint age;
    uint id;
}

contract NoCacheArrayElement {

    Person[] students;

    function createStudents() external  {
        Person[] memory arrayOfPersons = new Person[](3); 
        Person memory newPerson1 = Person("Emmanuel", 15,1);
        Person memory newPerson2 = Person("Faustina", 16,2);
        Person memory newPerson3 = Person("Emmanuela", 14,3);

        arrayOfPersons[0] = newPerson1;
        arrayOfPersons[1] = newPerson2;
        arrayOfPersons[2] = newPerson3;

        _addNewSet(arrayOfPersons);
    }

    function _addNewSet(Person[] memory _persons) internal {
        uint len = _persons.length;
        unchecked {
            for(uint i; i < len; ++i) {
                Person memory newStudent = Person(_persons[i].name, _persons[i].age, _persons[i].id);
                students.push(newStudent);
            }
        }

    }
}
```
```
test for test/NoCacheArrayElement.t.sol:NoCacheArrayElementTest
[PASS] test_createStudents() (gas: 230357)
```

```solidity

struct Person {
    string name;
    uint age;
    uint id;
}

contract CacheArrayElement {

    Person[] students;

    function createStudents() external  {
        Person[] memory arrayOfPersons = new Person[](3); 
        Person memory newPerson1 = Person("Emmanuel", 15,1);
        Person memory newPerson2 = Person("Faustina", 16,2);
        Person memory newPerson3 = Person("Emmanuela", 14,3);

        arrayOfPersons[0] = newPerson1;
        arrayOfPersons[1] = newPerson2;
        arrayOfPersons[2] = newPerson3;

        _addNewSet(arrayOfPersons);
    }

    function _addNewSet(Person[] memory _persons) internal {
        uint len = _persons.length;
        unchecked {
            for(uint i; i < len; ++i) {
                Person memory myPerson = _persons[i];
                Person memory newStudent = Person(myPerson.name, myPerson.age, myPerson.id);
                students.push(newStudent);
            }
        }

    }
}
```
```
test for test/Counter.t.sol:CacheArrayElementTest
[PASS] test_createStudents() (gas: 230096)
```

### 1 Instance

1. #### Cache `subjects[i]`
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L306

We can reduce the gas usage of the `transferAllCurvesTokens()` function by cache the value of `subjects[i]` so as to avoid reading the storage array `subjects[i]` twice during the loops iterations thereby avoiding 1 `SLOAD` (`100` gas units) and and replace it with a much cheaper `MLOAD` (`3` gas units) thereby saving `97` gas units per iteration of the loop. The diff below shows how the code could be refactored:

```solidity
file: contracts/Curves.sol

302:    function transferAllCurvesTokens(address to) external {
303:        if (to == address(this)) revert ContractCannotReceiveTransfer();
304:        address[] storage subjects = ownedCurvesTokenSubjects[msg.sender];
305:        for (uint256 i = 0; i < subjects.length; i++) {
306:            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
307:            if (amount > 0) {
308:                _transfer(subjects[i], msg.sender, to, amount); //@audit cache subjects[i];
309:            }
310:        }
311:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..4dd70d1 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -302,10 +302,12 @@ contract Curves is CurvesErrors, Security {
     function transferAllCurvesTokens(address to) external {
         if (to == address(this)) revert ContractCannotReceiveTransfer();
         address[] storage subjects = ownedCurvesTokenSubjects[msg.sender];
+        address subject;
         for (uint256 i = 0; i < subjects.length; i++) {
-            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
+            subject = subjects[i];
+            uint256 amount = curvesTokenBalance[subject][msg.sender];
             if (amount > 0) {
-                _transfer(subjects[i], msg.sender, to, amount);
+                _transfer(subject, msg.sender, to, amount);
             }
         }
     }
```
```
Estmated gas saved: 97 gas units per iteration
```




## [G-07] Use += for mappings
Using += for mappings saves 40 gas due to not having to recalculate the mapping's value's hash.

### Instances
1. #### `curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount` use += for mappings
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L322

We can reduce the gas cost of `_transfer()` function if we use `+=` in the `curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount` calculation as it help avoid having to recalculate the mapping's value's hash.

```solidity
file: contracts/Curves.sol

313:    function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
314:        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
315:
316:        // If transferring from oneself, skip adding to the list
317:        if (from != to) {
318:            _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
319:        }
320:
321:        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
322:        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount; //@audit use += for mappings
323:
324:        emit Transfer(curvesTokenSubject, from, to, amount);
325:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..f8da248 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -319,7 +319,7 @@ contract Curves is CurvesErrors, Security {
         }

         curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
-        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;
+        curvesTokenBalance[curvesTokenSubject][to] += amount;

         emit Transfer(curvesTokenSubject, from, to, amount);
     }
```
```
Estimated gas saved: 80 gas units
```


## [G-08] Multiple accesses of a mapping should use a local variable cache
The instances below point to the second+ access of a value inside a mapping within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

#### Please note these instances were not included in the bots reports

### Instances

1. #### Cache the `referralFeeDestination[curvesTokenSubject]` mapping
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L241

We can reduce the gas usage of the `_transferFees()` function by cache the value of `referralFeeDestination[curvesTokenSubject]` mapping so as to avoid reading from storage and having to recalculate the key's keccak256 hash and the calculation's associated stack operations for subsequent reads. In implementing this we would avoid 1 `SLOAD` (coldaccess) `100` gas units and `Gkeccak256` `30` gas units replace it with a much cheaper `MLOAD` (`3` gas units) thereby saving `130` gas units. The diff below shows how the code should be refactored:

```solidity
file: contracts/Curves.sol

218:    function _transferFees(
219:        address curvesTokenSubject,
220:        bool isBuy,
221:        uint256 price,
222:        uint256 amount,
223:        uint256 supply
224:    ) internal {
225:        (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
226:        {
227:            bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0); //@audit referralFeeDestination[curvesTokenSubject] read
228:            {
229:                address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
230:                uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
231:                uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
232:                (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
233:                if (!success1) revert CannotSendFunds();
234:            }
235:            {
236:                (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
237:                if (!success2) revert CannotSendFunds();
238:            }
239:            {
240:                (bool success3, ) = referralDefined
241:                    ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("") //@audit referralFeeDestination[curvesTokenSubject] read
242:                    : (true, bytes(""));
243:                if (!success3) revert CannotSendFunds();
244:            }
.
.
.
261:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..3d94e54 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -224,7 +224,8 @@ contract Curves is CurvesErrors, Security {
     ) internal {
         (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
         {
-            bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
+            address curvesTokenSubjectFeeDestination = referralFeeDestination[curvesTokenSubject];
+            bool referralDefined = curvesTokenSubjectFeeDestination != address(0);
             {
                 address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
                 uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
@@ -238,7 +239,7 @@ contract Curves is CurvesErrors, Security {
             }
             {
                 (bool success3, ) = referralDefined
-                    ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
+                    ? curvesTokenSubjectFeeDestination.call{value: referralFee}("")
                     : (true, bytes(""));
                 if (!success3) revert CannotSendFunds();
             }
```
```
Estimated gas saved: 130 gas units
```



2. #### Cache the `curvesTokenBalance[curvesTokenSubject][from]` mapping
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L314

We can reduce the gas usage of the `_transfer()` function by cache the value of `curvesTokenBalance[curvesTokenSubject][from]` mapping so as to avoid reading from storage and having to recalculate the key's keccak256 hash and the calculation's associated stack operations for subsequent reads. In implementing this we would avoid 1 `SLOAD` (coldaccess) `100` gas units and 2 `Gkeccak256` `60` gas units (since this is a mapping of a mapping) replace it with a much cheaper `MLOAD` (`3` gas units) thereby saving `160` gas units. The diff below shows how the code should be refactored:

```solidity
file: contracts/Curves.sol

313:    function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
314:        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
315:
316:        // If transferring from oneself, skip adding to the list
317:        if (from != to) {
318:            _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
319:        }
320:
321:        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
322:        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;
323:
324:        emit Transfer(curvesTokenSubject, from, to, amount);
325:    }
```
```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..f71269b 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -311,14 +311,15 @@ contract Curves is CurvesErrors, Security {
     }

     function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
-        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
+        uint256 curvesTokenSubjectTokenBalance  = curvesTokenBalance[curvesTokenSubject][from];
+        if (amount > curvesTokenSubjectTokenBalance) revert InsufficientBalance();

         // If transferring from oneself, skip adding to the list
         if (from != to) {
             _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
         }

-        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
+        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenSubjectTokenBalance - amount;
         curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;

         emit Transfer(curvesTokenSubject, from, to, amount);
```
```
Estimated gas saved: 160 gas units
```

3. #### Cache the `externalCurvesTokens[curvesTokenSubject]` mapping
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L354-#L356

Rather than having to recalculate the `externalCurvesTokens[curvesTokenSubject]` mapping key keccak256 hash every time its accessed we can cache the mapping value into a local storage `ExternalTokenMeta` variable then use this variable for subsequent reads and write to the `externalCurvesTokens[curvesTokenSubject]` mapping. In implementing this we would reduce the `_deployERC20()` function gas cost by `80` gas units. The diff below shows how the code should be refactored:

```solidity
file: contracts/Curves.sol

338:    function _deployERC20(
339:        address curvesTokenSubject,
340:        string memory name,
341:        string memory symbol
342:    ) internal returns (address) {
343:        // If the token's symbol is CURVES, append a counter value
344:        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
345:            _curvesTokenCounter += 1;
346:            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
347:            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
348:        }
349:
350:        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
351:
352:        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));
353:
354:        externalCurvesTokens[curvesTokenSubject].token = tokenContract;
355:        externalCurvesTokens[curvesTokenSubject].name = name;
356:        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
357:        externalCurvesToSubject[tokenContract] = curvesTokenSubject;
358:        symbolToSubject[symbol] = curvesTokenSubject;
359:
360:        emit TokenDeployed(curvesTokenSubject, tokenContract, name, symbol);
361:        return address(tokenContract);
362:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..6e1acf1 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -351,9 +351,11 @@ contract Curves is CurvesErrors, Security {

         address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

-        externalCurvesTokens[curvesTokenSubject].token = tokenContract;
-        externalCurvesTokens[curvesTokenSubject].name = name;
-        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
+        ExternalTokenMeta storage _externalCurvesToken = externalCurvesTokens[curvesTokenSubject];
+
+        _externalCurvesToken.token = tokenContract;
+        _externalCurvesToken.name = name;
+        _externalCurvesToken.symbol = symbol;
         externalCurvesToSubject[tokenContract] = curvesTokenSubject;
         symbolToSubject[symbol] = curvesTokenSubject;
```
```
Estimated gas saved: 80 gas units
```


4. #### Cache the `presalesMeta[curvesTokenSubject]` mapping
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L387-#L389

Rather than having to recalculate the `presalesMeta[curvesTokenSubject]` mapping key keccak256 hash every time its accessed we can cache the mapping value into a local storage `PresaleMeta` variable then use this variable for subsequent reads and write to the `presalesMeta[curvesTokenSubject]` mapping. In implementing this we would reduce the `buyCurvesTokenForPresale()` function gas cost by `80` gas units. The diff below shows how the code should be refactored:

```solidity
file: contracts/Curves.sol

377:    function buyCurvesTokenForPresale(
378:        address curvesTokenSubject,
379:        uint256 amount,
380:        uint256 startTime,
381:        bytes32 merkleRoot,
382:        uint256 maxBuy
383:    ) public payable onlyTokenSubject(curvesTokenSubject) {
384:        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
385:        uint256 supply = curvesTokenSupply[curvesTokenSubject];
386:        if (supply != 0) revert CurveAlreadyExists();
387:        presalesMeta[curvesTokenSubject].startTime = startTime;
388:        presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
389:        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
390:
391:        _buyCurvesToken(curvesTokenSubject, amount);
392:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..b1a79d2 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -384,9 +384,10 @@ contract Curves is CurvesErrors, Security {
         if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
         uint256 supply = curvesTokenSupply[curvesTokenSubject];
         if (supply != 0) revert CurveAlreadyExists();
-        presalesMeta[curvesTokenSubject].startTime = startTime;
-        presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
-        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
+        PresaleMeta storage curvesTokenSubjectPresalesMeta = presalesMeta[curvesTokenSubject];
+        curvesTokenSubjectPresalesMeta.startTime = startTime;
+        curvesTokenSubjectPresalesMeta.merkleRoot = merkleRoot;
+        curvesTokenSubjectPresalesMeta.maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);

         _buyCurvesToken(curvesTokenSubject, amount);
     }
```
```
Estimated gas saved: 80 gas units
```


5. #### Cache the `presalesMeta[msg.sender]` mapping
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L398-#L399

Rather than having to recalculate the `presalesMeta[msg.sender]` mapping key keccak256 hash every time its accessed we can cache the mapping value into a local storage `PresaleMeta` variable then use this variable for subsequent reads and write to the `presalesMeta[msg.sender]` mapping. In implementing this we would reduce the `setWhitelist` function gas cost by `40` gas units. The diff below shows how the code should be refactored:

```solidity
file: contracts/Curves.sol

394:    function setWhitelist(bytes32 merkleRoot) external {
395:        uint256 supply = curvesTokenSupply[msg.sender];
396:        if (supply > 1) revert CurveAlreadyExists();
397:
398:        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
399:            presalesMeta[msg.sender].merkleRoot = merkleRoot;
400:            emit WhitelistUpdated(msg.sender, merkleRoot);
401:        }
402:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..f895dbd 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -394,9 +394,9 @@ contract Curves is CurvesErrors, Security {
     function setWhitelist(bytes32 merkleRoot) external {
         uint256 supply = curvesTokenSupply[msg.sender];
         if (supply > 1) revert CurveAlreadyExists();
-
-        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
-            presalesMeta[msg.sender].merkleRoot = merkleRoot;
+        PresaleMeta storage userPresaleMeta = presalesMeta[msg.sender];
+        if (userPresaleMeta.merkleRoot != merkleRoot) {
+            userPresaleMeta.merkleRoot = merkleRoot;
             emit WhitelistUpdated(msg.sender, merkleRoot);
         }
     }
```
```
Estimated gas saved: 40
```

6. #### Cache the `externalCurvesTokens[curvesTokenSubject]` mapping
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L433-#L436

Rather than having to recalculate the `externalCurvesTokens[curvesTokenSubject]` mapping key keccak256 hash every time its accessed we can cache the mapping value into a local storage `ExternalTokenMeta` variable then use this variable for subsequent reads and write to the `externalCurvesTokens[curvesTokenSubject]` mapping. In implementing this we would reduce the `setNameAndSymbol()` function gas cost by `80` gas units. The diff below shows how the code should be refactored:

```solidity
file:  contracts/Curves.sol
    
428:    function setNameAndSymbol(
429:        address curvesTokenSubject,
430:        string memory name,
431:        string memory symbol
432:    ) external onlyTokenSubject(curvesTokenSubject) {
433:        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
434:        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
435:        externalCurvesTokens[curvesTokenSubject].name = name;
436:        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
437:    }
```
```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..103f1be 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -430,10 +430,11 @@ contract Curves is CurvesErrors, Security {
         string memory name,
         string memory symbol
     ) external onlyTokenSubject(curvesTokenSubject) {
-        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
+        ExternalTokenMeta storage externalCurvesToken = externalCurvesTokens[curvesTokenSubject];
+        if (externalCurvesToken.token != address(0)) revert ERC20TokenAlreadyMinted();
         if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
-        externalCurvesTokens[curvesTokenSubject].name = name;
-        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
+       externalCurvesToken.name = name;
+        externalCurvesToken.symbol = symbol;
     }
```
```
Estimated gas saved: 80 gas units
```


## [G-09] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

//f(x) is a low gas cost operation \
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) \
f(x) && g(y)

### Instance

1. #### Apply short-circuiting rule to the if statement `if (protocolFeePercent_ + feesEconomics.subjectFeePercent + feesEconomics.referralFeePercent + feesEconomics.holdersFeePercent > feesEconomics.maxFeePercent || protocolFeeDestination_ == address(0))`
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L129-#L136

In the `setProtocolFeePercent()` function as shown below we can apply the concept of short-circuiting to the if statement `if (protocolFeePercent_ + feesEconomics.subjectFeePercent + feesEconomics.referralFeePercent + feesEconomics.holdersFeePercent > feesEconomics.maxFeePercent || protocolFeeDestination_ == address(0))` such that the statement `protocolFeeDestination_ == address(0)` is evaluated first (since its gas cost is lesser) before `protocolFeePercent_ + feesEconomics.subjectFeePercent + feesEconomics.referralFeePercent + feesEconomics.holdersFeePercent > feesEconomics.maxFeePercent` (which is more gas expensive as it involves multiple `SLOAD`) so that in scenarios where the statement `protocolFeeDestination_ == address(0)` results to true the functions flow would continue without having to execute the more gas consuming operation `protocolFeePercent_ + feesEconomics.subjectFeePercent + feesEconomics.referralFeePercent + feesEconomics.holdersFeePercent > feesEconomics.maxFeePercent`. The code could be refactored as shown in the diff below:

```solidity
file: contracts/Curves.sol

128:    function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
129:        if (
130:            protocolFeePercent_ +
131:                feesEconomics.subjectFeePercent +
132:                feesEconomics.referralFeePercent +
133:                feesEconomics.holdersFeePercent >
134:            feesEconomics.maxFeePercent ||
135:            protocolFeeDestination_ == address(0)
136:        ) revert InvalidFeeDefinition();
137:        feesEconomics.protocolFeePercent = protocolFeePercent_;
138:        feesEconomics.protocolFeeDestination = protocolFeeDestination_;
139:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..2cd0283 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -126,13 +126,13 @@ contract Curves is CurvesErrors, Security {
     }

     function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
-        if (
+        if (protocolFeeDestination_ == address(0) ||
             protocolFeePercent_ +
                 feesEconomics.subjectFeePercent +
                 feesEconomics.referralFeePercent +
                 feesEconomics.holdersFeePercent >
-            feesEconomics.maxFeePercent ||
-            protocolFeeDestination_ == address(0)
+            feesEconomics.maxFeePercent
+
         ) revert InvalidFeeDefinition();
         feesEconomics.protocolFeePercent = protocolFeePercent_;
         feesEconomics.protocolFeeDestination = protocolFeeDestination_;
```



## [G-10] Pre-calculate equations or computations which contain only constant values in constructor
For equations or computations that only involve constants or immutable values they could be computed in the contract's constructor and saved to an immutable variable. Using the immutable variable would be cheaper than re-computing the value every time the function is called.

### 1 Instance

1. #### Compute `keccak256(abi.encodePacked(""))` in the constructor and save value to an immutable variable
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L442
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L444
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L472
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L474

Since `abi.encodePacked("")` is a constant value, the value of `keccak256(abi.encodePacked(""))` can be calculated in the constructor. Calling `keccak256(abi.encodePacked(""))` on a known, constant value is a waste of gas rather the calcualtion should be done in the constructor then saved to an immutable variable so that the computations of `keccak256(abi.encodePacked(""))` in the `mint()` and `withdraw()` functions would be replaced with cheaper stack read. The code could be refactored as shown in the diff below:

```solidity
file: contracts/Curves.sol

439:    function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
440:        if (
441:            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
442:            keccak256(abi.encodePacked("")) ||  //@audit Pre-calculate in constructor
443:            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
444:            keccak256(abi.encodePacked(""))  //@audit Pre-calculate in constructor
445:        ) {
446:            externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
447:            externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
448:        }
449:        _mint(
450:            curvesTokenSubject,
451:            externalCurvesTokens[curvesTokenSubject].name,
452:            externalCurvesTokens[curvesTokenSubject].symbol
453:        );
454:    }
.
.
.
465:    function withdraw(address curvesTokenSubject, uint256 amount) public {
466:        if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();
467:
468:        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
469:        if (externalToken == address(0)) {
470:            if (
471:                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
472:                keccak256(abi.encodePacked("")) ||  //@audit Pre-calculate in constructor
473:                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
474:                keccak256(abi.encodePacked(""))  //@audit Pre-calculate in constructor
475:            ) {
476:                externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
477:                externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
478:            }
479:            _deployERC20(
480:                curvesTokenSubject,
481:                externalCurvesTokens[curvesTokenSubject].name,
482:                externalCurvesTokens[curvesTokenSubject].symbol
483:            );
484:            externalToken = externalCurvesTokens[curvesTokenSubject].token;
485:        }
486:        _transfer(curvesTokenSubject, msg.sender, address(this), amount);
487:        CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
488:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..a9e3904 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -45,6 +45,7 @@ contract Curves is CurvesErrors, Security {
     string public constant DEFAULT_SYMBOL = "CURVES";
     // Counter for CURVES tokens minted
     uint256 private _curvesTokenCounter = 0;
+    string immutable emptyStringHash;

     struct ExternalTokenMeta {
         string name;
@@ -108,6 +109,7 @@ contract Curves is CurvesErrors, Security {
     constructor(address curvesERC20Factory_, address feeRedistributor_) Security() {
         curvesERC20Factory = curvesERC20Factory_;
         feeRedistributor = FeeSplitter(payable(feeRedistributor_));
+        emptyStringHash = keccak256(abi.encodePacked(""));
     }

     function setFeeRedistributor(address feeRedistributor_) external onlyOwner {
@@ -439,9 +441,9 @@ contract Curves is CurvesErrors, Security {
     function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
         if (
             keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
-            keccak256(abi.encodePacked("")) ||
+            emptyStringHash ||
             keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
-            keccak256(abi.encodePacked(""))
+            emptyStringHash
         ) {
             externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
             externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
@@ -469,9 +471,9 @@ contract Curves is CurvesErrors, Security {
         if (externalToken == address(0)) {
             if (
                 keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
-                keccak256(abi.encodePacked("")) ||
+                emptyStringHash ||
                 keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
-                keccak256(abi.encodePacked(""))
+                emptyStringHash
             ) {
                 externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
```




## [G-11] Remove `presalesMeta[curvesTokenSubject].startTime == 0` conditional
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L410

The conditional `presalesMeta[curvesTokenSubject].startTime == 0` in the statement `if (presalesMeta[curvesTokenSubject].startTime == 0 ||presalesMeta[curvesTokenSubject].startTime <= block.timestamp) revert PresaleUnavailable()` can be removed without any issues, as the conditional `presalesMeta[curvesTokenSubject].startTime <= block.timestamp` covers and satisfy that condition because in scenarios where `presalesMeta[curvesTokenSubject].startTime` is equal to 0 it would also be less than `block.timestamp`. In implementing this change we would save 1 `SLOAD` (100 gas unts) and Gkeccal256 (30 gas units)

```solidity
file: contracts/Curves.sol

404:    function buyCurvesTokenWhitelisted(
405:        address curvesTokenSubject,
406:        uint256 amount,
407:        bytes32[] memory proof
408:    ) public payable {
409:        if (
410:            presalesMeta[curvesTokenSubject].startTime == 0 ||      //@audit i think this is redundant
411:            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
412:        ) revert PresaleUnavailable();
413:
414:        presalesBuys[curvesTokenSubject][msg.sender] += amount;
415:        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
416:        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
417:
418:        verifyMerkle(curvesTokenSubject, msg.sender, proof);
419:        _buyCurvesToken(curvesTokenSubject, amount);
420:    }
```
```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..9c4c70f 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -407,7 +407,6 @@ contract Curves is CurvesErrors, Security {
         bytes32[] memory proof
     ) public payable {
         if (
-            presalesMeta[curvesTokenSubject].startTime == 0 ||
             presalesMeta[curvesTokenSubject].startTime <= block.timestamp
         ) revert PresaleUnavailable();
```
```
Estimated gas saved: 130 gas units
```




## [G-12] Using calldata instead of memory for read-only arguments in external functions saves gas

### Instances
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L430-#L431

```solidity
file: contracts/Curves.sol

428:    function setNameAndSymbol(
429:        address curvesTokenSubject,
430:        string memory name,     //@audit use calldata in-place memory
431:        string memory symbol     //@audit use calldata in-place memory
432:    ) external onlyTokenSubject(curvesTokenSubject) {
433:        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
434:        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
435:        externalCurvesTokens[curvesTokenSubject].name = name;
436:        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
437:    }
```

```diff
diff --git a/contracts/Curves.sol b/contracts/Curves.sol
index 817c79b..d569aa3 100644
--- a/contracts/Curves.sol
+++ b/contracts/Curves.sol
@@ -427,8 +427,8 @@ contract Curves is CurvesErrors, Security {

     function setNameAndSymbol(
         address curvesTokenSubject,
-        string memory name,
-        string memory symbol
+        string calldata name,
+        string calldata symbol
     ) external onlyTokenSubject(curvesTokenSubject) {
         if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
         if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
```



## [G-13]  Use solidity version 0.8.20 to gain gas boost

#### Please note these instances were not included in the bots report.

### Instances
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L2
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L2
- https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L2



## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.


