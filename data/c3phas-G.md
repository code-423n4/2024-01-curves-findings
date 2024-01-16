# Gas Report

## Table of Contents

- [Gas Report](#gas-report)
  - [Table of Contents](#table-of-contents)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Codebase impressions](#codebase-impressions)
  - [Avoid making similar state reads due to how functions are called(Save 3320 Gas on average)](#avoid-making-similar-state-reads-due-to-how-functions-are-calledsave-3320-gas-on-average)
  - [Refactor function mint to only do sstores if token is not minted already](#refactor-function-mint-to-only-do-sstores-if-token-is-not-minted-already)
  - [First token bought should be added directly to the list of owned tokens when using function `_buyCurvesToken`](#first-token-bought-should-be-added-directly-to-the-list-of-owned-tokens-when-using-function-_buycurvestoken)
  - [We can save some SLOADS by refactoring the function `buyCurvesTokenWhitelisted`(Save 113 Gas on average)](#we-can-save-some-sloads-by-refactoring-the-function-buycurvestokenwhitelistedsave-113-gas-on-average)
  - [Function might consume too much which might cause a DOS](#function-might-consume-too-much-which-might-cause-a-dos)
    - [Recommendation](#recommendation)
  - [We should set a limit for the length of the tokens when Batching to avoid consuming too much gas](#we-should-set-a-limit-for-the-length-of-the-tokens-when-batching-to-avoid-consuming-too-much-gas)
  - [Refactor the function `sellExternalCurvesToken` to avoid making unnecessary state loads(SLOADS) - Save 304 Gas on average](#refactor-the-function-sellexternalcurvestoken-to-avoid-making-unnecessary-state-loadssloads---save-304-gas-on-average)
  - [Only Make SLOADS when necessary](#only-make-sloads-when-necessary)
  - [Reorder  the checks to have cheaper checks first](#reorder--the-checks-to-have-cheaper-checks-first)
  - [Usual gas findings that should have been found by bots but were missed](#usual-gas-findings-that-should-have-been-found-by-bots-but-were-missed)
  - [Cache storage values in memory to minimize SLOADs](#cache-storage-values-in-memory-to-minimize-sloads)
    - [We can cache the variable `_curvesTokenCounter` in memory and then assign it(Saves 144 Gas on average)](#we-can-cache-the-variable-_curvestokencounter-in-memory-and-then-assign-itsaves-144-gas-on-average)
    - [Save 1 SLOAD by caching `data.cumulativeFeePerToken`(Save 74 Gas on average)](#save-1-sload-by-caching-datacumulativefeepertokensave-74-gas-on-average)
  - [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
  - [No need to cache state reads  if using once](#no-need-to-cache-state-reads--if-using-once)
    - [Variable `supply` is only used therefore no need to cache `curvesTokenSupply[msg.sender]`](#variable-supply-is-only-used-therefore-no-need-to-cache-curvestokensupplymsgsender)
    - [Do not cache `curvesTokenSupply[curvesTokenSubject]` as it's only required once](#do-not-cache-curvestokensupplycurvestokensubject-as-its-only-required-once)
    - [Supply is only used once, no need to cache `curvesTokenSupply[curvesTokenSubject]`](#supply-is-only-used-once-no-need-to-cache-curvestokensupplycurvestokensubject)
  - [Conclusion](#conclusion)


## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Codebase impressions

The developers have done an excellent job of identifying and implementing some of the most evident optimizations.

However, we have identified additional areas where optimization is possible, some of which may not be immediately apparent.

Most of this optimizations are not the usually pattern matching findings, the are very specific to this codebase.


## Avoid making similar state reads due to how functions are called(Save 3320 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L80-L87
|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | ----- |
| Before | -    | -   | 77744 | 
| After  | -    | -   | 74424 | 

```solidity
File: /contracts/FeeSplitter.sol
80:    function claimFees(address token) external {
81:        updateFeeCredit(token, msg.sender);
82:        uint256 claimable = getClaimableFees(token, msg.sender);
83:        if (claimable == 0) revert NoFeesToClaim();
84:        tokensData[token].unclaimedFees[msg.sender] = 0;
85:        payable(msg.sender).transfer(claimable);
86:        emit FeesClaimed(token, msg.sender, claimable);
87:    }
```

The function `claimFees` makes two function calls to `updateFeeCredit(token, msg.sender)`  and `getClaimableFees(token, msg.sender)` that are implemented as follows

**updateFeeCredit()**
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L71
```solidity
File: /contracts/FeeSplitter.sol
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

**getClaimableFees()**
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L73-L78
```solidity
File: /contracts/FeeSplitter.sol
73:    function getClaimableFees(address token, address account) public view returns (uint256) {
74:        TokenData storage data = tokensData[token];
75:        uint256 balance = balanceOf(token, account);
76:        uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
77:        return (owed / PRECISION) + data.unclaimedFees[account];
78:    }
```

The two functions are making two similar  calls, one being a state read `TokenData storage data = tokensData[token]` and the other a function call to `balanceOf(token, account)`
This essentially means, when the function `claimFees()` is called, we end up making unnecessary calls . We can avoid making this similar calls by combining the implementations into one and using it directly inside the `claimFees()`  function


```diff
     function claimFees(address token) external {
-        updateFeeCredit(token, msg.sender);
-        uint256 claimable = getClaimableFees(token, msg.sender);
+        TokenData storage data = tokensData[token];
+        uint256 balance = balanceOf(token, msg.sender);
+        if (balance > 0) {
+            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[msg.sender]) * balance;
+            data.unclaimedFees[msg.sender] += owed / PRECISION;
+            data.userFeeOffset[msg.sender] = data.cumulativeFeePerToken;
+        }
+        uint256 owed_ = (data.cumulativeFeePerToken - data.userFeeOffset[msg.sender]) * balance;
+        uint256 claimable = (owed_ / PRECISION) + data.unclaimedFees[msg.sender];
         if (claimable == 0) revert NoFeesToClaim();
         tokensData[token].unclaimedFees[msg.sender] = 0;
         payable(msg.sender).transfer(claimable);
```


## Refactor function mint to only do sstores if token is not minted already

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L439-L454

```solidity
File: /contracts/Curves.sol

439:    function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
440:        if (
441:        keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
442:            keccak256(abi.encodePacked("")) ||        443:keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
444:            keccak256(abi.encodePacked(""))
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
```

The function `mint()` starts of by checking for empty name and symbol and assigning them  if they are empty. We then call `_mint()` passing the args from the function `mint`
Function `_mint()`  first checks if the token we intend to mint is already minted and reverts if so. We check this by checking `externalCurvesTokens[curvesTokenSubject].token != address(0)` see the implementation of `_mint()`

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L456-L463

```solidity
    function _mint(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal onlyTokenSubject(curvesTokenSubject) {
        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
        _deployERC20(curvesTokenSubject, name, symbol);
    }
```

Instead of doing all those SSTORES and end up reverting on the check in `_mint` ,we can move the check for `alreadyMinted` to the function `mint()` and just to avoid messing with any other function that might require to use `_mint` we can move the `_deployERC20(curvesTokenSubject, name, symbol)` call to the main function
The function `mint()` should first check if the token is minted. 


```diff
     function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
+        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
+
         if (
             keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
             keccak256(abi.encodePacked("")) ||
@@ -446,11 +448,7 @@ contract Curves is CurvesErrors, Security {
             externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
             externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
         }
-        _mint(
-            curvesTokenSubject,
-            externalCurvesTokens[curvesTokenSubject].name,
-            externalCurvesTokens[curvesTokenSubject].symbol
-        );
+        _deployERC20(curvesTokenSubject, externalCurvesTokens[curvesTokenSubject].name, externalCurvesTokens[curvesTokenSubject].symbol);
     }
```

## First token bought should be added directly to the list of owned tokens when using function `_buyCurvesToken` 

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263-L280
```solidity
File: /contracts/Curves.sol
263:    function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
264:        uint256 supply = curvesTokenSupply[curvesTokenSubject];
265:        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();


272:        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;

276:        // If is the first token bought, add to the list of owned tokens
277:        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
278:            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
279:        }
280:    }
```

When someone buys the token, we check if it's the first token they have bought and if so we add it to the list of owned tokens. We do this by calling an internal function `_addOwnedCurvesTokenSubject()`  which is implemented as follows

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L327-L336
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
The internal function will check if the token already exists and only add it if it does not exists(this involves reading the state variable `ownedCurvesTokenSubjects[owner_]` and looping on the list (**very gas intensive**))

However , iteratating on this list is not necessary when we call the function from the function `_buyCurvesToken()`  since , we only call the internal function when we are guaranteed that this is the first token being bought by the user

```solidity
        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
```

All we need to do is simply add the token directly

```diff
         // If is the first token bought, add to the list of owned tokens
         if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
-            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
+            ownedCurvesTokenSubjects[msg.sender].push(curvesTokenSubject);
         }
     }
```

Also, note the comment, this only happens if this is the first token bought, which means it can't be on the list.
The check `curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0` ensures no other tokens exists on this users account.
This ensures that we handle the case of tokens having been transfered to us


## We can save some SLOADS by refactoring the function `buyCurvesTokenWhitelisted`(Save 113 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L404-L420
|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | ----- |
| Before | 99264    | 163498   | 126028 | 
| After  | 99115    | 163385   | 125915 | 

```solidity
File: /contracts/Curves.sol
404:    function buyCurvesTokenWhitelisted(
405:        address curvesTokenSubject,
406:        uint256 amount,
407:        bytes32[] memory proof
408:    ) public payable {
409:        if (
410:            presalesMeta[curvesTokenSubject].startTime == 0 ||
411:            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
412:        ) revert PresaleUnavailable();


414:        presalesBuys[curvesTokenSubject][msg.sender] += amount;
415:        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
416:        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();


418:        verifyMerkle(curvesTokenSubject, msg.sender, proof);
419:        _buyCurvesToken(curvesTokenSubject, amount);
420:    }
```

On Line 414, we write to storage(SSTORE + SLOAD)  then make another SLOAD when trying to validate that `tokenBought` won't exceed the `maxBuyAmount`.
A more efficient way would be to validate this first then write to storage


```diff
-        presalesBuys[curvesTokenSubject][msg.sender] += amount;
-        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
+        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender] + amount;
         if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

+        presalesBuys[curvesTokenSubject][msg.sender] = tokenBought;
+
         verifyMerkle(curvesTokenSubject, msg.sender, proof);
         _buyCurvesToken(curvesTokenSubject, amount);
     }
```


## Function might consume too much which might cause a DOS

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L52-L61
```solidity
File: /contracts/FeeSplitter.sol
52:    function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
53:        address[] memory tokens = getUserTokens(user);
54:        UserClaimData[] memory result = new UserClaimData[](tokens.length);
55:        for (uint256 i = 0; i < tokens.length; i++) {
56:            address token = tokens[i];
57:            uint256 claimable = getClaimableFees(token, user);
58:            result[i] = UserClaimData(claimable, token);
59:        }
60:        return result;
61:    }
```

Calling `getUserTokens(user)`  basically tries to retrieve all userTokens which are tracked by the mapping `userTokens` and return an array of tokens which we store as `tokens`. 
If the list is too big, iterating on it would consume too much gas , given we also call a function `getClaimableFees` which does alot of state loads

The function `onBalanceChange`  is used by the manager to push some more items on the list of userTokens.
If the manager pushes too many items, looping on them would be too expensive
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96-L100
```solidity
    function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }
```

### Recommendation
We might want to have a limit set for the length 


## We should set a limit for the length of the tokens when Batching to avoid consuming too much gas

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L103-L117
```solidity
File: /contracts/FeeSplitter.sol
    function batchClaiming(address[] calldata tokenList) external {
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
        payable(msg.sender).transfer(totalClaimable);
    }
```

As noted by the comment on the code, the batching will fail if the list is too long, utilising the function `getUserTokens()` we can get the number of tokens and then only batch if the code can be executed to completion

Since the loop is calling functions that read and write to state the gas consumption might be too much.


## Refactor the function `sellExternalCurvesToken` to avoid making unnecessary state loads(SLOADS) - Save 304 Gas on average

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L504-L509
|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | ----- |
| Before | -    | -   | 83967 | 
| After  | -    | -   | 83663 | 

```solidity
File: /contracts/Curves.sol
504:    function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public {
505:        if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();

507:        deposit(curvesTokenSubject, amount);
508:        sellCurvesToken(curvesTokenSubject, amount / 1 ether);
509:    }
```
The function `sellExternalCurvesToken` makes a call to function `deposit` which has the following implementation

```solidity
490:    function deposit(address curvesTokenSubject, uint256 amount) public {
491:        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

493:        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
494:        uint256 tokenAmount = amount / 1 ether;

496:        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
497:        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
498:        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

500:        CurvesERC20(externalToken).burn(msg.sender, amount);
501:        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
502:    }
```

On Line 493, we are caching the value of `externalCurvesTokens[curvesTokenSubject].token` but if we go back to the calling function, we make another state read to the same state variable
We can inline the function deposit to avoid making state reads twice

```diff

     function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public {
-        if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();
+        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
+        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
+        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
+        uint256 tokenAmount = amount / 1 ether;

-        deposit(curvesTokenSubject, amount);
+        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
+        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();
+
+        CurvesERC20(externalToken).burn(msg.sender, amount);
+        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
         sellCurvesToken(curvesTokenSubject, amount / 1 ether);
     }
 }
```

## Only Make SLOADS when necessary

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L71
```solidity
File: /contracts/FeeSplitter.sol
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

In function , `updateFeeCredit` the variable `data` is only used when `balance > 0`  , therefore there's no need to  make a state read if we are not going to make use of it

```diff
     function updateFeeCredit(address token, address account) internal {
-        TokenData storage data = tokensData[token];
         uint256 balance = balanceOf(token, account);
         if (balance > 0) {
+            TokenData storage data = tokensData[token];
             uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
             data.unclaimedFees[account] += owed / PRECISION;
             data.userFeeOffset[account] = data.cumulativeFeePerToken;
```

If balance is not greater than 0, then we save ourselves one entire SLOAD



## Reorder  the checks to have cheaper checks first

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L490-L502
```solidity
File: /contracts/Curves.sol
490:    function deposit(address curvesTokenSubject, uint256 amount) public {
491:        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

493:        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
494:        uint256 tokenAmount = amount / 1 ether;

496:        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
497:        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
498:        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

500:        CurvesERC20(externalToken).burn(msg.sender, amount);
501:        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
502:    }
```


```diff
     function deposit(address curvesTokenSubject, uint256 amount) public {
-        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
-
+        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();//@audit Ensure users deposit an amount that will be divisible by 1 ether
+
+        uint256 tokenAmount = amount / 1 ether;//@audit-ok safe because of the earlier check
+        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();//@audit is this more expensive?
+
         address externalToken = externalCurvesTokens[curvesTokenSubject].token;
-        uint256 tokenAmount = amount / 1 ether;
-
         if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
+
         if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
-        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

         CurvesERC20(externalToken).burn(msg.sender, amount);
         _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
```

## Usual gas findings that should have been found by bots but were missed

## Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

### We can cache the variable `_curvesTokenCounter` in memory and then assign it(Saves 144 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L338-L348

**Gas benchmarks based on function `withdraw()`

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | ----- |
| Before | 153428    | 1813418   | 1689234 | 
| After  | 153428    | 1813218   | 1689088 | 

```solidity
File: /contracts/Curves.sol
338    function _deployERC20(

344:        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
345:            _curvesTokenCounter += 1;
346:            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
347:            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
348:        }
```

We can do some caching for `_curvesTokenCounter`. 

```diff
     function _deployERC20(
         address curvesTokenSubject,
@@ -342,9 +346,10 @@ contract Curves is CurvesErrors, Security {
     ) internal returns (address) {
         // If the token's symbol is CURVES, append a counter value
         if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
-            _curvesTokenCounter += 1;
-            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
-            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
+            uint256 counter = _curvesTokenCounter + 1;
+            name = string(abi.encodePacked(name, " ", Strings.toString(counter)));
+            symbol = string(abi.encodePacked(symbol, Strings.toString(counter)));
+            _curvesTokenCounter = counter;
         }
```


### Save 1 SLOAD by caching `data.cumulativeFeePerToken`(Save 74 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L71
|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | ----- |
| Before | -    | -   | 77744 | 
| After  | -    | -   | 77670 | 

```solidity
File: /contracts/FeeSplitter.sol
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
@@ -64,15 +64,16 @@ contract FeeSplitter is Security {
         TokenData storage data = tokensData[token];
         uint256 balance = balanceOf(token, account);
         if (balance > 0) {
-            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
+            uint256 _cumulativeFeePerToken = cumulativeFeePerToken;
+            uint256 owed = (_cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
             data.unclaimedFees[account] += owed / PRECISION;
-            data.userFeeOffset[account] = data.cumulativeFeePerToken;
+            data.userFeeOffset[account] = _cumulativeFeePerToken;
         }
     }
```


## Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L277-L279
```solidity
File: /contracts/Curves.sol
277:        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
278:            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
279:        }
```
The operation `curvesTokenBalance[curvesTokenSubject][msg.sender] - amount` cannot underflow. The current value of `curvesTokenBalance[curvesTokenSubject][msg.sender]` is simply a result of adding `amount` to the `curvesTokenBalance[curvesTokenSubject][msg.sender]` therefore subtracting again should be safe

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L287
```solidity
File: /contracts/Curves.sol
287:        uint256 price = getPrice(supply - amount, amount);
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L289
```solidity
File: /contracts/Curves.sol
289:        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
```
Before the operation is performed, a check exists that ensures that ` curvesTokenBalance[curvesTokenSubject][msg.sender]` is greater than `amount`


## No need to cache state reads  if using once

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394-L402
### Variable `supply` is only used therefore no need to cache `curvesTokenSupply[msg.sender]`

```solidity
File: /contracts/Curves.sol
394:    function setWhitelist(bytes32 merkleRoot) external {
395:        uint256 supply = curvesTokenSupply[msg.sender];
396:        if (supply > 1) revert CurveAlreadyExists();

398:        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
399:            presalesMeta[msg.sender].merkleRoot = merkleRoot;
400:            emit WhitelistUpdated(msg.sender, merkleRoot);
401:        }
402:    }
```
 The variable `supply` is only used once in the function. As such there was no need to cache the value of `curvesTokenSupply[msg.sender]`. Caching only adds to the gas consumption rather than save us some gas. 
 
```diff
     function setWhitelist(bytes32 merkleRoot) external {
-        uint256 supply = curvesTokenSupply[msg.sender];
-        if (supply > 1) revert CurveAlreadyExists();
+        if (curvesTokenSupply[msg.sender] > 1) revert CurveAlreadyExists();

         if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
             presalesMeta[msg.sender].merkleRoot = merkleRoot;
```


https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L377-L392
### Do not cache `curvesTokenSupply[curvesTokenSubject]` as it's only required once

```solidity
File: /contracts/Curves.sol
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

391:        _buyCurvesToken(curvesTokenSubject, amount);
392:    }
```


```diff
     ) public payable onlyTokenSubject(curvesTokenSubject) {
         if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
-        uint256 supply = curvesTokenSupply[curvesTokenSubject];
-        if (supply != 0) revert CurveAlreadyExists();
+        if (curvesTokenSupply[curvesTokenSubject] != 0) revert CurveAlreadyExists();
         presalesMeta[curvesTokenSubject].startTime = startTime;
         presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
         presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
```

### Supply is only used once, no need to cache `curvesTokenSupply[curvesTokenSubject]`

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L370-L371
```solidity
File: /contracts/Curves.sol
370:        uint256 supply = curvesTokenSupply[curvesTokenSubject];
371:        if (supply != 0) revert CurveAlreadyExists();
```

## Conclusion
It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
