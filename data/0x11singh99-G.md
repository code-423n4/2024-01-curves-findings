# Gas Optimizations

## Gas Optimizations

| Number                                                                                                                                                     | Issue                                                                                                                                            | Instances |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [[G-01](#g-01-pack-structs-by-putting-variables-that-can-fit-together-next-to-each-other-saves-8000-gas)]                                                  | Pack structs by putting variables that can fit together next to each other (saves 8000 Gas)                                                      |     4     |
| [[G-02](#g-02-use-holderfee-in-place-of-feeseconomicsholdersfeepercent-saves1-gcoldsload-2100-gas)]                                                        | Use `holderFee` in place of `feesEconomics.holdersFeePercent` SAVES(1 Gcoldsload 2100 GAS)                                                       |     1     |
| [[G-03](#g-03-unnecessary-modifier-checks-in-functions)]                                                                                                   | Unnecessary `modifier` checks in functions                                                                                                       |     2     |
| [[G-04](#g-04-cache-state-variablemapping-saves-600-gas)]                                                                                                  | Cache ``state variable`/`mapping` (saves 600 Gas)                                                                                                |     6     |
| [[G-05](#g-05-cache-state-variable-to-save-1-sload-on-each-iteration)]                                                                                     | Cache `state variable` to save `1 SLOAD` on each iteration                                                                                       |     1     |
| [[G-06](#g-06-avoid-writing-storage-variable-with-same-value)]                                                                                             | Avoid writing storage variable with same value                                                                                                   |     2     |
| [[G-07](#g-07-checking-the-conditions-that-are-less-computationally-expensive-or-involve-simple-comparisons-first-can-help-save-gas)]                      | Checking the `conditions` that are less computationally expensive or involve simple comparisons first can help save gas.                         |     1     |
| [[G-08](#g-08-when-the-value-is-used-multiple-times-in-an-expression-calculating-it-once-and-storing-it-in-a-variable-can-prevent-redundant-calculations)] | When the `value` is used multiple times in an expression, `calculating` it once and storing it in a variable can prevent redundant calculations. |     1     |
| [[G-09](#g-09-can-make-the-variable-outside-the-loop-to-save-gas)]                                                                                         | Can make the variable outside the `loop` to save gas                                                                                             |     5     |
| [[G-10](#g-10-use-calldata-instead-of-memory)]                                                                                                             | Use `calldata` instead of `memory`                                                                                                               |     4     |
| [[G-11](#g-11-dont-cache-state-variable-if-it-is-used-only-once)]                                                                                          | Don't cache `state variable` if it is used only once                                                                                             |     3     |

## [G-01] Pack structs by putting variables that can fit together next to each other (saves 8000 Gas)

### `SAVE: 8000 GAS, 4 SLOT`

_4 Instances_

### Note: Caught by bot but they didn't specify in how much size this should be reduced and why

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas

### `protocolFeeDestination`, `protocolFeePercent`, `subjectFeePercent`, `referralFeePercent`, `holdersFeePercent` and `maxFeePercent` can be packed in just 2 SLOT: `SAVES 8000 GAS, 4 SLOT`

Convert All FeePercent into uint64 because Fee Percent can't more than 1e18 so uint64 is more than enough to hold this number.

```solidity
File : contracts/Curves.sol

68:    struct FeesEconomics {
69:        address protocolFeeDestination;
70:        uint256 protocolFeePercent;
71:        uint256 subjectFeePercent;
72:        uint256 referralFeePercent;
73:        uint256 holdersFeePercent;
74:        uint256 maxFeePercent;
75:    }

```

[68-75](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L68C1-L75C6)

## [G-02] Use `holderFee` in place of `feesEconomics.holdersFeePercent` SAVES(1 Gcoldsload 2100 Gas)

Since if feesEconomics.holdersFeePercent is 0 then holderFee also be 0. So it is better to read from stack not from storage. Can Avoind 1 GCold slod

```solidity
File : contracts/Curves.sol

if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
            }

```

[246-249](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L246C13-L249C14)

## [G-03] Unnecessary `modifier` checks in functions

_2 Instances_

### Unnecessary `onlyTokenSubject(curvesTokenSubject)` check

Unnecessary modifier since it will only pass if `supply` is 0 and it is checked that in `_buyCurvesToken` it will only pass when `curvesTokenSubject` is `msg.sender` so no need to add extra modifier here.

### `SAVES ~38 GAS` when every time function respectively called

```solidity
File : contracts/Curves.sol

377:  function buyCurvesTokenForPresale(
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
392:    }

```

[377-392](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L377C1-L392C6)

### Unnecessary `onlyTokenSubject(curvesTokenSubject)` check

\_mint function is called in two functions one in mint function that have already that modifier check

### `SAVES ~38 GAS` when every time function respectively called

```solidity
File : contracts/Curves.sol

456: function _mint(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
       ) internal onlyTokenSubject(curvesTokenSubject) {
        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
        _deployERC20(curvesTokenSubject, name, symbol);
463:      }

```

[456-463](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L456C5-L463C6)

## [G-04] Cache `state variable`/`mapping` (saves 600 Gas)

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

### `SAVES: 600 GAS, 6 SLOAD`

**Note: Missed by bot-report**

_6 instance_

### Cache `data.cumulativeFeePerToken` SAVES(~100 GAS)

```solidity
File : contracts/FeeSplitter.sol

67:      uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
68:       data.unclaimedFees[account] += owed / PRECISION;
69:       data.userFeeOffset[account] = data.cumulativeFeePerToken;

```

[67-69](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L67C2-L69C70)

```diff
File : contracts/FeeSplitter.sol

+     uint256 _cumulativeFeePerToken = data.cumulativeFeePerToken;
-      uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
+      uint256 owed = (_cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
       data.unclaimedFees[account] += owed / PRECISION;
-       data.userFeeOffset[account] = data.cumulativeFeePerToken;
+       data.userFeeOffset[account] = _cumulativeFeePerToken;

```

### Cache `curvesTokenBalance[curvesTokenSubject][msg.sender]` first and use cached value in the if block to save 1 SLOAD and 1 checked subtraction

```solidity
File : contracts/Curves.sol

272:  curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
273:        curvesTokenSupply[curvesTokenSubject] = supply + amount;
274:        _transferFees(curvesTokenSubject, true, price, amount, supply);
275:
276:        // If is the first token bought, add to the list of owned tokens
277:        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {

```

[272-277](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L272C9-L277C80)

```diff
File : contracts/Curves.sol

function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);

        if (msg.value < price + totalFee) revert InsufficientPayment();

+       uint256 _curvesTokenBalance = curvesTokenBalance[curvesTokenSubject][msg.sender];

-       curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
+       curvesTokenBalance[curvesTokenSubject][msg.sender] = _curvesTokenBalance + amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        // If is the first token bought, add to the list of owned tokens
-       if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
+       if (_curvesTokenBalance == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
    }

```

### Cache `referralFeeDestination[curvesTokenSubject]` SAVES(~100 GAS)

```solidity
File : contracts/Curves.sol

227:    bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
            ...
240:      (bool success3, ) = referralDefined
241:         ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")

```

[227-241](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L227C13-L241C94)

```diff
+       address _referralFeeDestination = referralFeeDestination[curvesTokenSubject];

-       bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
+       bool referralDefined = _referralFeeDestination != address(0);
            {
                address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
                uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
                uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
                (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
                if (!success1) revert CannotSendFunds();
            }
            {
                (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
                if (!success2) revert CannotSendFunds();
            }
            {
                (bool success3, ) = referralDefined
-                   ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
+                   ? _referralFeeDestination.call{value: referralFee}("")

```

### Cache `curvesTokenBalance[curvesTokenSubject][msg.sender]` SAVES(~100 GAS)

```solidity
File : contracts/Curves.sol

285:    if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

287:    uint256 price = getPrice(supply - amount, amount);

288:    curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;

```

[285-288](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L285C1-L290C1)

```diff
File : contracts/Curves.sol

    function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply <= amount) revert LastTokenCannotBeSold();

+       uint256 _curvesTokenBalance = curvesTokenBalance[curvesTokenSubject][msg.sender];

-       if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();
+       if (_curvesTokenBalance < amount) revert InsufficientBalance();

        uint256 price = getPrice(supply - amount, amount);

-       curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
+       curvesTokenBalance[curvesTokenSubject][msg.sender] = _curvesTokenBalance - amount;
        curvesTokenSupply[curvesTokenSubject] = supply - amount;

        _transferFees(curvesTokenSubject, false, price, amount, supply);
    }

```

### Cache `curvesTokenBalance[curvesTokenSubject][from]` SAVES(~100 GAS)

```solidity
File : contracts/Curves.sol

314:   if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

316:   // If transferring from oneself, skip adding to the list
317:    if (from != to) {
318:      _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
319:     }
320:
321:     curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;

```

[314-321](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L314C1-L321C110)

```diff
File : contracts/Curves.sol

+       uint256 _curvesTokenBalance = curvesTokenBalance[curvesTokenSubject][from];

-       if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
+       if (amount > _curvesTokenBalance) revert InsufficientBalance();

        // If transferring from oneself, skip adding to the list
        if (from != to) {
            _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
        }

-       curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
+       curvesTokenBalance[curvesTokenSubject][from] = _curvesTokenBalance - amount;

```

### Cache `_curvesTokenCounter` SAVES(~100 GAS)

```solidity
File : contracts/Curves.sol

344:   if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
            _curvesTokenCounter += 1;
            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
348:        }

```

[344-348](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L344-L348)

```diff
File : contracts/Curves.sol

        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
            _curvesTokenCounter += 1;
+           _cachedCurveTokenCounter = _curvesTokenCounter;

-            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
-            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
+            name = string(abi.encodePacked(name, " ", Strings.toString(_cachedCurveTokenCounter)));
+            symbol = string(abi.encodePacked(symbol, Strings.toString(_cachedCurveTokenCounter)));
        }


```

## [G-05] Cache `state variable` to save `1 SLOAD` on each iteration

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

**Note: Not in bot findings**

_1 Instance_

### Cache `subjects[i]` SAVES(~100 GAS) on each iteration

```solidity
File : contracts/Curves.sol

305:        for (uint256 i = 0; i < subjects.length; i++) {
306:            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
307:            if (amount > 0) {
308:                _transfer(subjects[i], msg.sender, to, amount);
309:            }

```

[305-309](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305C1-L309C14)

```diff
File : contracts/Curves.sol

        for (uint256 i = 0; i < subjects.length; i++) {
+           address _subject =  subjects[i];
-            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
+            uint256 amount = curvesTokenBalance[_subject][msg.sender];
            if (amount > 0) {
-                _transfer(subjects[i], msg.sender, to, amount);
+                _transfer(_subject, msg.sender, to, amount);
            }

```

## [G-06] Avoid writing storage variable with same value

### Check with previous address is not similar to new to avoid duplicate writes

```solidity
File : contracts/Curves.sol

    function setReferralFeeDestination(
        address curvesTokenSubject,
        address referralFeeDestination_
     ) public onlyTokenSubject(curvesTokenSubject) {
        referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
    }

    function setERC20Factory(address factory_) external onlyOwner {
        curvesERC20Factory = factory_;
    }

```

[155-164](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L155C5-L164C6)

## [G-07] Checking the `conditions` that are less computationally expensive or involve simple comparisons first can help save gas.

_1 instance_

```solidity
File: contracts/Curves.sol

129:    if (
130:            protocolFeePercent_ +
131:                feesEconomics.subjectFeePercent +
132:                feesEconomics.referralFeePercent +
133:                feesEconomics.holdersFeePercent >
134:            feesEconomics.maxFeePercent ||
135:            protocolFeeDestination_ == address(0)
136:        ) revert InvalidFeeDefinition();

```

[129-136](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L129C9-L136C41)

```diff
File: contracts/Curves.sol

-     if (
-            protocolFeePercent_ +
-                feesEconomics.subjectFeePercent +
-                feesEconomics.referralFeePercent +
-                feesEconomics.holdersFeePercent >
-            feesEconomics.maxFeePercent ||
-            protocolFeeDestination_ == address(0)
-        ) revert InvalidFeeDefinition();
+     if (
+            protocolFeeDestination_ == address(0) ||
+            protocolFeePercent_ +
+                feesEconomics.subjectFeePercent +
+                feesEconomics.referralFeePercent +
+                feesEconomics.holdersFeePercent >
+            feesEconomics.maxFeePercent
+        ) revert InvalidFeeDefinition();

```

## [G-08] When the `value` is used multiple times in an expression, `calculating` it once and storing it in a variable can prevent redundant calculations.

_1 instance_

In this code, you have expressions that involve the term `supply - 1` multiple times. To save gas, you can calculate `supply - 1` only once and store it in a variable. This is because redundant calculations of the same value consume extra computational resources.

```solidity
File : contracts/Curves.sol

181:        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
182:        uint256 sum2 = supply == 0 && amount == 1
183:            ? 0
184:            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
185:        uint256 summation = sum2 - sum1;
186:        return (summation * 1 ether) / 16000;
187:    }

```

[181-187](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L181C1-L187C6)

```diff
File : contracts/Curves.sol

+       uint256 _supplyMinusOne = supply - 1;
-       uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
+       uint256 sum1 = supply == 0 ? 0 : ((_supplyMinusOne) * (supply) * (2 * (_supplyMinusOne) + 1)) / 6;
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
-            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
+            : ((_supplyMinusOne + amount) * (supply + amount) * (2 * (_supplyMinusOne + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
        return (summation * 1 ether) / 16000;
    }

```

## [G-09] Can make the variable outside the `loop` to save gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements. By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract.

_5 instances_

```solidity
File : contracts/Curves.sol

305:    for (uint256 i = 0; i < subjects.length; i++) {
306:            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];

```

[305-306](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305C9-L306C74)

```diff
File : contracts/Curves.sol

+       uint256 amount;
        for (uint256 i = 0; i < subjects.length; i++) {
-            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
+            amount = curvesTokenBalance[subjects[i]][msg.sender];

```

```solidity
File : contracts/FeeSplitter.sol

55:        for (uint256 i = 0; i < tokens.length; i++) {
56:            address token = tokens[i];
57:            uint256 claimable = getClaimableFees(token, user);

```

[55-57](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55C1-L57C63)

```diff
File : contracts/FeeSplitter.sol
+       address token;
+       uint256 claimable;
        for (uint256 i = 0; i < tokens.length; i++) {
-            address token = tokens[i];
-            uint256 claimable = getClaimableFees(token, user);
+            token = tokens[i];
+            claimable = getClaimableFees(token, user);

```

```solidity
File : contracts/FeeSplitter.sol

105:    for (uint256 i = 0; i < tokenList.length; i++) {
106:            address token = tokenList[i];
107:            updateFeeCredit(token, msg.sender);
108:            uint256 claimable = getClaimableFees(token, msg.sender);

```

[105-108](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L105C9-L108C69)

```diff
File : contracts/FeeSplitter.sol
+       address token;
+       uint256 claimable;
        for (uint256 i = 0; i < tokenList.length; i++) {
-            address token = tokenList[i];
+            token = tokenList[i];
             updateFeeCredit(token, msg.sender);
-            uint256 claimable = getClaimableFees(token, msg.sender);
+            claimable = getClaimableFees(token, msg.sender);

```

## [G-10] Use `calldata` instead of `memory`

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

_4 instances_

```solidity
File : contracts/Curves.sol

404:    function buyCurvesTokenWhitelisted(
405:        address curvesTokenSubject,
406:        uint256 amount,
407:        bytes32[] memory proof //@audit use calldata
408:    ) public payable {


        //@audit use calldata for `proof`
422:    function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view {

```

[404-408](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L404C1-L408C23), [422](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L422C1-L422C108)

```solidity
File : contracts/CurvesERC20Factory.sol
       //@audit use calldata for `name` and `symbol`
7: function deploy(string memory name, string memory symbol, address owner) public returns (address) {
8:        CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);
9:        return address(tokenContract);
10:    }

```

[7-10](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L7-L10)

## [G-11] Don't cache `state variable` if it is used only once

_3 instance_

### Don't cache `curvesTokenSupply[curvesTokenSubject]`

```solidity
File : contracts/Curves.sol

364: function buyCurvesTokenWithName(
        address curvesTokenSubject,
        uint256 amount,
        string memory name,
        string memory symbol
     ) public payable {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply != 0) revert CurveAlreadyExists();

        _buyCurvesToken(curvesTokenSubject, amount);
        _mint(curvesTokenSubject, name, symbol);
375:    }
```

[364-375](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L364C5-L375C6)

### Don't cache `curvesTokenSupply[msg.sender]`

```solidity
File : contracts/Curves.sol

394: function setWhitelist(bytes32 merkleRoot) external {
        uint256 supply = curvesTokenSupply[msg.sender];
        if (supply > 1) revert CurveAlreadyExists();

        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
            presalesMeta[msg.sender].merkleRoot = merkleRoot;
            emit WhitelistUpdated(msg.sender, merkleRoot);
        }
402:    }

```

[394-402](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L394C5-L402C6)

### Don't cache `presalesBuys[curvesTokenSubject][msg.sender]`

```solidity
File : contracts/Curves.sol

415:     uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];

```

[415](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L415C9-L415C76)
