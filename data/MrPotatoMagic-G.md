# Gas Optimizations Report

| ID     | Optimization                                                                                             |
|--------|----------------------------------------------------------------------------------------------------------|
| [G-01](#g-01-public-variables-used-in-current-contract-only-can-be-marked-private) | Public variables used in current contract only can be marked private                                     |
| [G-02](#g-02-use-uint96-for-claimablefees-to-pack-userclaimdata-struct-members-into-1-slot) | Use uint96 for claimableFees to pack UserClaimData struct members into 1 slot                            |
| [G-03](#g-03-cache-array-length-to-save-gas) | Cache array length to save gas                                                                           |
| [G-04](#g-04-use-do-while-loops-instead-of-for-loops-to-save-gas) | Use do-while loops instead of for loops to save gas                                                      |
| [G-05](#g-05-use-named-returns-to-save-gas) | Use named returns to save gas                                                                            |
| [G-06](#g-06-inline-variable-owed-to-avoid-creating-unnecessary-memory-variable) | Inline variable `owed` to avoid creating unnecessary memory variable                                     |
| [G-07](#g-07-use-calldata-instead-of-memory-for-parameters-to-save-gas) | Use calldata instead of memory for parameters to save gas                                                |
| [G-08](#g-08-no-need-to-initialize-_curvestokencounter-to-0) | No need to initialize `_curvesTokenCounter` to 0                                                         |
| [G-09](#g-09-use-msgsender-directly-to-remove-parameter-input-curvestokensubject-and-modifier-onlytokensubject) | Use `msg.sender` directly to remove parameter input `curvesTokenSubject` and modifier `onlyTokenSubject` |
| [G-10](#g-10-use-holderfee-memory-variable-instead-of-accessing-feeseconomicsholdersfeepercent-from-storage) | Use `holderFee` memory variable instead of accessing `feesEconomics.holdersFeePercent` from storage      |
| [G-11](#g-11-cache-supply---amount-in-function-sellcurvestoken) | Cache `supply - amount` in function `sellCurvesToken()`                                                  |
| [G-12](#g-12-use-binary-search-instead-of-linear-searching-in-function-_addownedcurvestokensubject) | Use binary search instead of linear searching in function `_addOwnedCurvesTokenSubject()`                |
| [G-13](#g-13-remove-unnecessary-memory-variable-tokenbought-from-function-buycurvestokenwhitelisted)  | Remove unnecessary memory variable `tokenBought` from function `buyCurvesTokenWhitelisted()`             |
| [G-14](#g-14-remove-balance-check-from-withdraw-function) | Remove balance check from `withdraw()` function                                                          |
| [G-15](#g-15-remove-token-existence-check-from-sellexternalcurvestoken-function) | Remove token existence check from `sellExternalCurvesToken()` function                                   |

## [G-01] Public variables used in current contract only can be marked private

Marking public variables private saves gas since a getter function is not created.

There are 2 instances of this:

```solidity
File: Security.sol
5:     address public owner; 
```

```solidity
File: FeeSplitter.sol
10:     Curves public curves; 
```

## [G-02] Use uint96 for claimableFees to pack UserClaimData struct members into 1 slot

Instead of this:
```solidity
File: FeeSplitter.sol
25:     struct UserClaimData {
26:         uint256 claimableFees;
27:         address token;
28:     }
```
Use this:
```solidity
File: FeeSplitter.sol
25:     struct UserClaimData {
26:         uint96 claimableFees; // 12 bytes
27:         address token; // 20 bytes
28:     }
```

## [G-03] Cache array length to save gas

Caching array length to a memory variable save gas when used across multiple instances or for loop iterations.

Cache `tokens.length` used on Line 57 and 58. 
```solidity
File: FeeSplitter.sol
57:         UserClaimData[] memory result = new UserClaimData[](tokens.length); 
58:         for (uint256 i = 0; i < tokens.length; i++) {
59:             address token = tokens[i];
60:             uint256 claimable = getClaimableFees(token, user);
61:             result[i] = UserClaimData(claimable, token);
62:         }
```

## [G-04] Use do-while loops instead of for loops to save gas

Instead of this:
```solidity
File: FeeSplitter.sol
58:         for (uint256 i = 0; i < tokens.length; i++) {
59:             address token = tokens[i];
60:             uint256 claimable = getClaimableFees(token, user);
61:             result[i] = UserClaimData(claimable, token);
62:         }
```
Use this:
```solidity
File: FeeSplitter.sol
57:         uint256 i;
58:         do {
59:             address token = tokens[i];
60:             uint256 claimable = getClaimableFees(token, user);
61:             result[i] = UserClaimData(claimable, token);
62:             unchecked {
63:                ++i;
64:             }
65:         } while (i < tokens.length);
```

## [G-05] Use named returns to save gas

The solidity compiler outputs more efficient code when the variable is declared in the return statement. 

Instead of this:
```solidity
File: FeeSplitter.sol
55:     function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
56:         address[] memory tokens = getUserTokens(user);
57:         UserClaimData[] memory result = new UserClaimData[](tokens.length); 
58:         for (uint256 i = 0; i < tokens.length; i++) {
59:             address token = tokens[i];
60:             uint256 claimable = getClaimableFees(token, user);
61:             result[i] = UserClaimData(claimable, token);
62:         }
63:         return result;
64:     }
```
Use this:
```solidity
File: FeeSplitter.sol
55:     function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory result) {
56:         address[] memory tokens = getUserTokens(user);
57:         result = new UserClaimData[](tokens.length); 
58:         for (uint256 i = 0; i < tokens.length; i++) {
59:             address token = tokens[i];
60:             uint256 claimable = getClaimableFees(token, user);
61:             result[i] = UserClaimData(claimable, token);
62:         }
63:
64:     }
```

## [G-06] Inline variable `owed` to avoid creating unnecessary memory variable 

Instead of this:
```solidity
File: FeeSplitter.sol
70:             uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance; 
71:             data.unclaimedFees[account] += owed / PRECISION; 
72:             data.userFeeOffset[account] = data.cumulativeFeePerToken;
```
Use this:
```solidity
File: FeeSplitter.sol
71:             data.unclaimedFees[account] += ((data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance) / PRECISION; 
72:             data.userFeeOffset[account] = data.cumulativeFeePerToken;
```

## [G-07] Use calldata instead of memory for parameters to save gas

Parameters that are only read and not modified can be marked calldata to avoid unnecessary copying from calldata to memory.

Variables `name` and `symbol` can be marked calldata instead of memory.
```solidity
File: CurvesERC20Factory.sol
10:     function deploy(string memory name, string memory symbol, address owner) public returns (address) {
11:         CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);
12:         return address(tokenContract);
13:     }
```

## [G-08] No need to initialize `_curvesTokenCounter` to 0 

Variable `_curvesTokenCounter` is an uint256 which has its default value as 0.

```solidity
File: Curves.sol
48:     uint256 private _curvesTokenCounter = 0; 
```

## [G-09] Use `msg.sender` directly to remove parameter input `curvesTokenSubject` and modifier `onlyTokenSubject`

Using msg.sender directly would remove the need of taking `curvesTokenSubject` as a parameter and an unnecessary check in the modifier onlyTokenSubject to see if curvesTokenSubject is equal to msg.sender.

Instead of this:
```solidity
File: Curves.sol
158:     function setReferralFeeDestination(
159:         address curvesTokenSubject,
160:         address referralFeeDestination_
161:     ) public onlyTokenSubject(curvesTokenSubject) {
162:         referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
163:     }
```
Use this:
```solidity
File: Curves.sol
158:     function setReferralFeeDestination(
160:         address referralFeeDestination_
161:     ) public {
162:         referralFeeDestination[msg.sender] = referralFeeDestination_;
163:     }
```

## [G-10] Use `holderFee` memory variable instead of accessing `feesEconomics.holdersFeePercent` from storage

This will save gas since the SLOAD opcode (100 gas) is replaced by the MLOAD opcode (3 gas).

Instead of this:
```solidity
File: Curves.sol
263:             if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
```
Use this:
```solidity
File: Curves.sol
263:             if (holderFee > 0 && address(feeRedistributor) != address(0)) {
```

## [G-11] Cache `supply - amount` in function `sellCurvesToken()`

In the code below, Lines 306 and 309 perform the same `supply - amount` operation. Caching this value into a memory variable will help save some gas.
```solidity
File: Curves.sol
306:         uint256 price = getPrice(supply - amount, amount);
307: 
308:         curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount; 
309:         curvesTokenSupply[curvesTokenSubject] = supply - amount;
```

## [G-12] Use binary search instead of linear searching in function `_addOwnedCurvesTokenSubject()`

Binary search takes O(log n) which is faster than linear searching a value in an array.

The following code block searches if the curvesTokenSubject already exists in a user's list. The bigger the user's list, the longer it will take for the linear search.
```solidity
File: Curves.sol
352:         for (uint256 i = 0; i < subjects.length; i++) {
353:             if (subjects[i] == curvesTokenSubject) {
354:                 return;
355:             }
356:         }
```

## [G-13] Remove unnecessary memory variable `tokenBought` from function `buyCurvesTokenWhitelisted()`

Creating memory variable `tokenBought` is not required since it is used only once on Line 444. Consider directly using storage value directly.

Instead of this:
```solidity
File: Curves.sol
443:         uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender]; 444:         if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
```
Use this:
```solidity
File: Curves.sol
444:         if (presalesBuys[curvesTokenSubject][msg.sender] > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
```

## [G-14] Remove balance check from `withdraw()` function

The check below is already implemented in the _transfer() function [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L314), which is internally called in the withdraw() function [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L486).

```solidity
File: Curves.sol
496:         if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance(); 
```

## [G-15] Remove token existence check from `sellExternalCurvesToken()` function

The check below is not required since it is already implemented in the deposit() function [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L496), which is internally called in the sellExternalCurvesToken() function [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L507).

```solidity
File: Curves.sol
536:         if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject(); 
```