|ID|Title|Category|Severity|Instances|
|-|:-|:-:|:-:|:-:|
| [[G-1](#g-1--use-assembly-to-write-address-storage-values)] | Use assembly to write address storage values  | Gas Optimization | Informational | 1 |
| [[G-2](#g-2--use-do-while)] | Use do while | Gas Optimization | Informational | 2 |
| [[G-3](#g-3--use-assembly-to-update-mapping)] | Use assembly to update mapping  | Gas Optimization | Informational | 1 |
| [[G-4](#g-4--arrayindex--amount-is-cheaper-than-arrayindex--arrayindex--amount)] |  array[index] += amount is cheaper than array[index] = array[index] + amount  | Gas Optimization | Informational | 1 |
| [[G-5](#g-5-possible-optimizations-in-sellcurvestoken-function)] |  Possible Optimizations in `sellCurvesToken` function   | Gas Optimization | Informational | 1 |
| [[G-6](#g-6--state-variable-read-once)] |  State variable read once   | Gas Optimization | Informational | 3 |
| [[G-7](#g-7--with-assembly-call-bool-success-transfer-can-be-done-gas-optimized)] | With assembly, ``.call` (bool success) transfer can be done gas-optimized | Gas Optimization | Informational | 2 |

## **G-1** | Use assembly to write address storage values 

By using assembly, we can directly interact with the storage slot of the storedAddress variable, allowing us to efficiently write and read address values in storage.

```diff
diff --git a/Curves.sol b/aCurves.sol
index 817c79b..a134bf1 100644
--- a/Curves.sol
+++ b/aCurves.sol
@@ -160,7 +160,10 @@ contract Curves is CurvesErrors, Security {
     }
 
     function setERC20Factory(address factory_) external onlyOwner {
-        curvesERC20Factory = factory_;
+        assembly{
+            sstore(curvesERC20Factory.slot,factory_)
+        }
+        //curvesERC20Factory = factory_;
     }

```
> Gas diff
| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Curves.setERC20Factory` | 29171 | 29110 | -61 | -0.21% |


## **G-2** | Use do while 

```diff
diff --git a/Curves.sol b/aCurves.sol
index 817c79b..3462340 100644
--- a/Curves.sol
+++ b/aCurves.sol
@@ -302,12 +302,16 @@ contract Curves is CurvesErrors, Security {
     function transferAllCurvesTokens(address to) external {
         if (to == address(this)) revert ContractCannotReceiveTransfer();
         address[] storage subjects = ownedCurvesTokenSubjects[msg.sender];
-        for (uint256 i = 0; i < subjects.length; i++) {
+        uint i;
+        do{//for (uint256 i = 0; i < subjects.length; i++) {
             uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
             if (amount > 0) {
                 _transfer(subjects[i], msg.sender, to, amount);
             }
-        }
+            unchecked{
+                ++i;
+            }
+        }while(i < subjects.length);
     }

```

```diff
diff --git a/FeeSplitter.sol b/aFeeSplitter.sol
index bb24f02..39f0e29 100644
--- a/FeeSplitter.sol
+++ b/aFeeSplitter.sol
@@ -102,7 +102,8 @@ contract FeeSplitter is Security {
     //@dev: this may fail if the the list is long. Get first the list with getUserTokens to estimate and prepare the batch
     function batchClaiming(address[] calldata tokenList) external {
         uint256 totalClaimable = 0;
-        for (uint256 i = 0; i < tokenList.length; i++) {
+        uint256 i;
+        do{//for (uint256 i = 0; i < tokenList.length; i++) {
             address token = tokenList[i];
             updateFeeCredit(token, msg.sender);
             uint256 claimable = getClaimableFees(token, msg.sender);
@@ -111,7 +112,10 @@ contract FeeSplitter is Security {
                 totalClaimable += claimable;
                 emit FeesClaimed(token, msg.sender, claimable);
             }
-        }
+            unchecked {
+                ++i;
+            }
+        } while(i < tokenList.length);
         if (totalClaimable == 0) revert NoFeesToClaim();
         payable(msg.sender).transfer(totalClaimable);
     }

```
> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Curves.transferAllCurvesTokens` | 258653 | 257950 | -703 | -0.27% |
| `FeeSplitter.batchClaiming` | 130572 | 130317 | -255 | -0.20% |
| `Curves` | 5232829 | 5229815 | -3014 | -0.06% |
| `FeeSplitter` | 1558996 | 1555982 | -3014 | -0.19% |

## **G-3** | Use assembly to update mapping 

```diff
diff --git a/Curves.sol b/aCurves.sol
index df8ed0a..5eaaf3c 100644
--- a/Curves.sol
+++ b/aCurves.sol
@@ -156,7 +156,13 @@ contract Curves is CurvesErrors, Security {
         address curvesTokenSubject,
         address referralFeeDestination_
     ) public onlyTokenSubject(curvesTokenSubject) {
-        referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
+        //referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
+        assembly {
+        mstore(0, curvesTokenSubject)
+        mstore(32, referralFeeDestination.slot)
+        let ref := keccak256(0, 64)
+        sstore(ref, referralFeeDestination_)
+     }  
     }
```
| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Curves.setReferralFeeDestination` | 44906 | 44823 | -83 | -0.18% |
| `Curves` | 5232829 | 5209469 | -23360 | -0.45% |

## **G-4** array[index] += amount is cheaper than array[index] = array[index] + amount

```diff
diff --git a/Curves.sol b/aCurves.sol
index 817c79b..bd79aaa 100644
--- a/Curves.sol
+++ b/aCurves.sol
@@ -318,8 +318,8 @@ contract Curves is CurvesErrors, Security {
             _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
         }
 
-        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
-        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;
+        curvesTokenBalance[curvesTokenSubject][from] -= amount;
+        curvesTokenBalance[curvesTokenSubject][to] += amount;
 
         emit Transfer(curvesTokenSubject, from, to, amount);
     }
```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Curves.deposit` | 51290 | 51023 | -267 | -0.52% |
| `Curves.sellExternalCurvesToken` | 83967 | 83633 | -334 | -0.40% |
| `Curves.transferAllCurvesTokens` | 258653 | 257317 | -1336 | -0.52% |
| `Curves.transferCurvesToken` | 95688 | 95354 | -334 | -0.35% |
| `Curves.withdraw` | 1689234 | 1688900 | -334 | -0.02% |
| `Curves` | 5232829 | 5182245 | -50584 | -0.97% |


## **G-5** | Possible Optimizations in `sellCurvesToken` function

```diff
diff --git a/Curves.sol b/aCurves.sol
index 817c79b..c05284e 100644
--- a/Curves.sol
+++ b/aCurves.sol
@@ -283,11 +283,14 @@ contract Curves is CurvesErrors, Security {
         uint256 supply = curvesTokenSupply[curvesTokenSubject];
         if (supply <= amount) revert LastTokenCannotBeSold();
         if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();
-
-        uint256 price = getPrice(supply - amount, amount);
+        uint256 fsub;
+        assembly{
+            fsub := sub(supply,amount)
+        }
+        uint256 price = getPrice(fsub, amount);
 
         curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
-        curvesTokenSupply[curvesTokenSubject] = supply - amount;
+        curvesTokenSupply[curvesTokenSubject] = fsub;
 
         _transferFees(curvesTokenSubject, false, price, amount, supply);
     }
```

> Gas diff
| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Curves.sellCurvesToken` | 59894 | 59537 | -357 | -0.60% |
| `Curves.sellExternalCurvesToken` | 83967 | 83610 | -357 | -0.43% |
| `Curves` | 5232829 | 5229815 | -3014 | -0.06% |

## **G-6** | State variable read once 

```diff
diff --git a/Curves.sol b/aCurves.sol
index 817c79b..c5b9181 100644
--- a/Curves.sol
+++ b/aCurves.sol
@@ -367,8 +367,9 @@ contract Curves is CurvesErrors, Security {
         string memory name,
         string memory symbol
     ) public payable {
-        uint256 supply = curvesTokenSupply[curvesTokenSubject];
-        if (supply != 0) revert CurveAlreadyExists();
+        //uint256 supply = curvesTokenSupply[curvesTokenSubject];
+        //@audit Gas State variable read once
+        if (curvesTokenSupply[curvesTokenSubject] != 0) revert CurveAlreadyExists();
 
         _buyCurvesToken(curvesTokenSubject, amount);
         _mint(curvesTokenSubject, name, symbol);
@@ -382,8 +383,9 @@ contract Curves is CurvesErrors, Security {
         uint256 maxBuy
     ) public payable onlyTokenSubject(curvesTokenSubject) {
         if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
-        uint256 supply = curvesTokenSupply[curvesTokenSubject];
-        if (supply != 0) revert CurveAlreadyExists();
+        //@audit Gas State variable read once
+        //uint256 supply = curvesTokenSupply[curvesTokenSubject];
+        if (curvesTokenSupply[curvesTokenSubject] != 0) revert CurveAlreadyExists();
         presalesMeta[curvesTokenSubject].startTime = startTime;
         presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
         presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
@@ -392,8 +394,9 @@ contract Curves is CurvesErrors, Security {
     }
 
     function setWhitelist(bytes32 merkleRoot) external {
-        uint256 supply = curvesTokenSupply[msg.sender];
-        if (supply > 1) revert CurveAlreadyExists();
+        //@audit Gas State variable read once
+        //uint256 supply = curvesTokenSupply[msg.sender];
+        if (curvesTokenSupply[msg.sender] > 1) revert CurveAlreadyExists();
 
         if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
             presalesMeta[msg.sender].merkleRoot = merkleRoot;
@@ -508,3 +511,4 @@ contract Curves is CurvesErrors, Security {
         sellCurvesToken(curvesTokenSubject, amount / 1 ether);
     }
 }

```

```diff
diff --git a/FeeSplitter.sol b/aFeeSplitter.sol
index bb24f02..54499f2 100644
--- a/FeeSplitter.sol
+++ b/aFeeSplitter.sol
@@ -87,10 +87,10 @@ contract FeeSplitter is Security {
     }
 
     function addFees(address token) public payable onlyManager {
-        uint256 totalSupply_ = totalSupply(token);
-        if (totalSupply_ == 0) revert NoTokenHolders();
+        //uint256 totalSupply_ = totalSupply(token);
+        //if (totalSupply_ == 0) revert NoTokenHolders();
         TokenData storage data = tokensData[token];
-        data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_;
+        data.cumulativeFeePerToken = data.cumulativeFeePerToken + (msg.value * PRECISION) / totalSupply(token);
     }

```
> Gas diff
| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `Curves.buyCurvesToken` | 124108 | 124106 | -2 | -0.00% |
| `Curves.buyCurvesTokenForPresale` | 205341 | 205328 | -13 | -0.01% |
| `Curves.buyCurvesTokenWithName` | 1774847 | 1774834 | -13 | -0.00% |
| `Curves.setWhitelist` | 48121 | 48108 | -13 | -0.03% |
| `FeeSplitter.addFees` | 61587 | 61541 | -46 | -0.07% |
| `Curves` | 5232829 | 5228978 | -3851 | -0.07% |
| `FeeSplitter` | 1558996 | 1544651 | -14345 | -0.92% |

## **G-7** | With assembly, ``.call` (bool success) transfer can be done gas-optimized

```diff
diff --git a/FeeSplitter.sol b/aFeeSplitter.sol
index bb24f02..911a74a 100644
--- a/FeeSplitter.sol
+++ b/aFeeSplitter.sol
@@ -82,7 +82,11 @@ contract FeeSplitter is Security {
         uint256 claimable = getClaimableFees(token, msg.sender);
         if (claimable == 0) revert NoFeesToClaim();
         tokensData[token].unclaimedFees[msg.sender] = 0;
-        payable(msg.sender).transfer(claimable);
+        //payable(msg.sender).transfer(claimable);
+        bool success;
+        assembly {                                    
+                    success := call(gas(), caller(), claimable, 0, 0, 0, 0)
+                }
         emit FeesClaimed(token, msg.sender, claimable);
     }
 
@@ -113,8 +117,13 @@ contract FeeSplitter is Security {
             }
         }
         if (totalClaimable == 0) revert NoFeesToClaim();
-        payable(msg.sender).transfer(totalClaimable);
+        //payable(msg.sender).transfer(totalClaimable);
+        bool success;
+        assembly {                                    
+                    success := call(gas(), caller(), totalClaimable, 0, 0, 0, 0)
+                }
     }
```

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `FeeSplitter.batchClaiming` | 130572 | 130513 | -59 | -0.05% |
| `FeeSplitter.claimFees` | 77744 | 77685 | -59 | -0.08% |
| `FeeSplitter` | 1558996 | 1534764 | -24232 | -1.55% |

