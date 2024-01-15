# [G-01] Consider using EnumerableSet, instead of iterating over the whole mapping in `_addOwnedCurvesTokenSubject()`

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L327)
```
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

Function `_addOwnedCurvesTokenSubject` adds a curvesTokenSubject to the list if not already present. The implementation of this functions is, however, not very gas-efficient. It iterates over a whole `ownedCurvesTokenSubjects[owner_]` and it checks if `curvesTokenSubject` is already there. If it's not (which means, that the loop needs to iterate over the whole `ownedCurvesTokenSubjects[owner_]`) - it adds that element: `subjects.push(curvesTokenSubject)`. The complexity of this solution is `O(N)`. We need to iterate over the whole `ownedCurvesTokenSubjects[owner_]`, to confirm, that `curvesTokenSubject` does not exist there. 

The much better idea is to utilize EnumerableSet, e.g.: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/EnumerableSet.sol
The complexity of adding new element to the enumerable set is: `O(1)`, which is much better than `O(N)`.
Enumerable set's properties guarantees that the set won't contain the same element twice - so it's basically the same what function `_addOwnedCurvesTokenSubject()` tries to achieve by iterating over a whole `ownedCurvesTokenSubjects[owner_]` in `O(N)`.


# [G-02] Use short-circuit in `if`/`else` conditions


Solidity uses short-circuit when evaluating conditions. E.g. in `f(x) AND g(x)` - when `f(x)` is evaluated as false, `g(x)` won't be called (because false AND whatever is false).
The same for OR: `f(x) OR g(x)` - when `f(x)` is evaluated as true, `g(x)` won't be called (because true OR whatever is true).

This suggests, that in `if`-conditions, less-costly operations should be on the left-side (evaluated first).

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L129)
```
 if (
            protocolFeePercent_ +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            feesEconomics.maxFeePercent ||
            protocolFeeDestination_ == address(0)
```

`protocolFeeDestination_ == address(0)` costs less gas than performing 3 additions and comparison, thus above code should be rewritten to:

```
 if (
            protocolFeeDestination_ == address(0) ||
            protocolFeePercent_ +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            feesEconomics.maxFeePercent
            
```

# [G-03] Do not create local variables used only once

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L185)
```
 uint256 summation = sum2 - sum1;
 return (summation * 1 ether) / 16000;
```

Variable `summation` is used only once, thus there's no need to declare it at all:

```
return ((sum2 - sum1)) * 1 ether) / 16000;
```


[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L395)
```
uint256 supply = curvesTokenSupply[msg.sender];
if (supply > 1) revert CurveAlreadyExists();
```

Variable `supply` is used only once, thus there's no need to declare it at all:

```
if (curvesTokenSupply[msg.sender] > 1) revert CurveAlreadyExists();
```

# [G-04] Multiple accesses of the same mapping/array key/index should be cached

The bot-report missed some instances, which are reported here.
According to the bot-report:

```
[G-02] Multiple accesses of the same mapping/array key/index should be cached
There are 2 instance(s) of this issue:

function mint
function withdraw
```

The instances missed by the bot-report are:

* `function _buyCurvesToken`

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L272)
```
 curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
 [...]
 if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
```

* `function sellCurvesToken`

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L285)
```
 if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();
 [...]
 curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
```

* `function _transfer`

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L314)
```
if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
[...]
curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
```


* `function buyCurvesTokenWhitelisted`

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L410)
```
presalesMeta[curvesTokenSubject].startTime == 0 ||
presalesMeta[curvesTokenSubject].startTime <= block.timestamp

 [...]
presalesBuys[curvesTokenSubject][msg.sender] += amount;
uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
```


# [G-05] Some operations can be unchecked

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L277)
```
 curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
[...]
 if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
```

Beacuase we firstly add `amount` to  `curvesTokenBalance[curvesTokenSubject][msg.sender]`, we can be sure that substracting it later won't underflow, thus `curvesTokenBalance[curvesTokenSubject][msg.sender] - amount` can be unchecked.


[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L289)
```
curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
curvesTokenSupply[curvesTokenSubject] = supply - amount;
```

When `supply <= amount`, function will revert with `LastTokenCannotBeSold()`. Because of that, `supply - amount` won't underflow and can be unchecked.

When `(curvesTokenBalance[curvesTokenSubject][msg.sender] < amount`, function will revert with `InsufficientBalance()`. Because of that, `curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount` won't underflow and can be unchecked.



[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L321)
```
 curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
```

When `amount > curvesTokenBalance[curvesTokenSubject][from]`, function will revert with `InsufficientBalance()`. Because of that, `curvesTokenBalance[curvesTokenSubject][from] - amount` won't underflow and can be unchecked.


# [G-06] Do not allow transfer 0 amount of tokens

In `Curves.sol`, function `transferAllCurvesTokens()` performs `_transfer()` whenever `amount > 0`. This check is, however, missing in `transferCurvesToken()`. 

It's possible to call `transferCurvesToken()` with `amount` set to 0. This call, will call another function, `_transfer()`, which also does not verify if `amount` is non-zero.

Transferring 0 amount of tokens does not change the state of the contract and is useless. Our recommendation is to add additional check which won't allow calling `transferCurvesToken()` when `amount == 0`, e.g.: `require(amount > 0, "Amount cannot be 0")`.

# [G-07] Move conditional check higher in `_buyCurvesToken()` to avoid redundant operation

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L272)
```
        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
```

We firstly add `amount` to `curvesTokenBalance[curvesTokenSubject][msg.sender]` (line 272), and then subtract it (line 277) in `if`-condition. This operation will be redundant when we'll perform conditional check sooner:

```
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] == 0) {  // @audit: no need for curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0 anymore, since we do comparison before adding amount.
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

       
```

Please notice, that `_addOwnedCurvesTokenSubject` does not interfere with `curvesTokenBalance` and `curvesTokenSupply`, thus this call can be done sooner as in the example above. 