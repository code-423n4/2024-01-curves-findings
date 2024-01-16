## Gas Findings

|    | Issue | Instances | Total Gas Saved |
|----|-------|:---------:|:---------:|
| [G-01] | `do`-`while` is cheaper than `for`-loops when the initial check can be skipped | 4 | 1020 |
| [G-02] | Avoid Explicit Initialization to Zero for Non-Constant/Non-Immutable State Variables | 1 | 2900 |
| [G-03] | Stack variable is only used once | 6 | 54 |
| [G-04] | Use `assembly` to write mutable storage values | 7 | 100 |
| [G-05] | Optimize Ether Transfers with `receive()` Function | 2 | 50 |
| [G-06] | Use `revert()` to gain maximum gas savings | 35 | 1750 |
| [G-07] | `x.a = x.a + b` always cheaper than `x.a += b` for Structs | 1 | 5 |
| [G-08] | Cache `abi.encodePacked` for Efficiency | 6 | - |
| [G-09] | Refactor duplicated require()/revert() checks to save gas | 8 | - |
| [G-10] | Consider Using += for Mappings | 1 | 40 |
| [G-11] | Use Cached Contracts for Multiple External Calls | 4 | 672 |
| [G-12] | Use `uint256(1)`/`uint256(2)` instead of `true`/`false` to save gas for changes | 1 | 17000 |
| [G-13] | Avoid contract existence checks by using low-level calls | 7 | 700 |
| [G-14] | Optimize Gas by Using Only Named Returns | 12 | 528 |
| [G-15] | State variables should be cached in stack rather than re-reading them from storage | 2 | 291 |
| [G-16] | Avoid updating storage when the value hasn't changed | 4 | 3200 |


## Gas Findings Details

### [G-01] `do`-`while` is cheaper than `for`-loops when the initial check can be skipped

Using `do-while` loops instead of `for` loops can be more gas-efficient. 
Even if you add an `if` condition to account for the case where the loop doesn't execute at all, a `do-while` loop can still be cheaper in terms of gas.

Example:
```solidity
/// 774 gas cost
function forLoop() public pure {
    for (uint256 i; i < 10;) {
        unchecked {
            ++i;
        }
    }
}
/// 519 gas cost
function doWhileLoop() public pure {
    uint256 i;
    do {
        unchecked {
            ++i;
        }
    } while (i < 10);
}
```

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Curves.sol

305: for (uint256 i = 0; i < subjects.length; i++)
330: for (uint256 i = 0; i < subjects.length; i++)
```
[305](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305) | [330](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L330)

```solidity
File: contracts/FeeSplitter.sol

55: for (uint256 i = 0; i < tokens.length; i++)
105: for (uint256 i = 0; i < tokenList.length; i++)
```
[55](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55) | [105](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L105)
</details>

### [G-02] Avoid Explicit Initialization to Zero for Non-Constant/Non-Immutable State Variables

Explicitly initializing non-constant/non-immutable state variables to zero is not efficient from a gas perspective. 
Letting Solidity use the default zero for uninitialized storage variables avoids a Gsreset, which costs 2900 gas, during deployment.
Consider removing these unnecessary initializations to optimize gas usage.
```solidity
    uint256 public foo = 0; // tx 69107 gas | exec 14669 gas
    uint256 public foo;     // tx 66854 gas | exec 12464 gas
``` 

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

47: uint256 private _curvesTokenCounter = 0
```
[47](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L47)
</details>

### [G-03] Stack variable is only used once

If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the 3 gas the extra stack assignment would spend

<details>
<summary><i>6 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

/// @audit - `summation` variable
185: uint256 summation = sum2 - sum1
/// @audit - `supply` variable
370: uint256 supply = curvesTokenSupply[curvesTokenSubject]
/// @audit - `supply` variable
385: uint256 supply = curvesTokenSupply[curvesTokenSubject]
/// @audit - `supply` variable
395: uint256 supply = curvesTokenSupply[msg.sender]
/// @audit - `tokenBought` variable
415: uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender]
/// @audit - `leaf` variable
424: bytes32 leaf = keccak256(abi.encodePacked(caller))
```
[185](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L185) | [370](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L370) | [385](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L385) | [395](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L395) | [415](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L415) | [424](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L424)
</details>

### [G-04] Use `assembly` to write mutable storage values

Writing to storage using `assembly` is more gas efficient.
```solidity
    function writeStorage() external {
        // storageNumber = 10; // 2358 gas
        // assembly {
        //     sstore(storageNumber.slot, 10) // 2350 gas
        // }
        // storageAddr = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc3; // 2411 gas
        // assembly {
        //     sstore(storageAddr.slot, 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc3) // 2350 gas
        // }
    }
```

<details>
<summary><i>37 issue instances in 3 files:</i></summary>

```solidity
File: contracts/Curves.sol

109: curvesERC20Factory = curvesERC20Factory_
110: feeRedistributor = FeeSplitter(payable(feeRedistributor_))
114: feeRedistributor = FeeSplitter(payable(feeRedistributor_))
163: curvesERC20Factory = factory_
```
[109](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L109) | [110](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L110) | [114](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L114) | [163](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L163)

```solidity
File: contracts/FeeSplitter.sol

36: curves = curves_
```
[36](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L36)

```solidity
File: contracts/Security.sol

19: owner = msg.sender
28: owner = owner_
```
[19](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L19) | [28](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L28)
</details>

### [G-05] Optimize Ether Transfers with `receive()` Function

Consider using `receive()` function instead of a specific `deposit()` (or similar) function.
If there are several functions in the contract that can receive Ether, it is recommended to use `receive()` for the most frequently used function.
```solidity
function deposit() external payable { // 5401 gas
    // your logic
}

receive() external payable {  // 5356 gas
    // your logic
}
```

The `receive()` or `fallback()` function can handle incoming Ether transfers directly, providing more gas-efficient way to manage deposits.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Curves.sol

211: function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable
```
[211](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L211) 

```solidity
File: contracts/FeeSplitter.sol

89: function addFees(address token) public payable onlyManager
```
[89](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L89)
</details>

### [G-06] Use `revert()` to gain maximum gas savings

If you dont need Error messages, or you want gain maximum gas savings - `revert()` is a cheapest way to revert transaction in terms of gas.
```solidity
    revert(); // 117 gas 
    require(false); // 132 gas
    revert CustomError(); // 157 gas
    assert(false); // 164 gas
    revert("Custom Error"); // 406 gas
    require(false, "Custom Error"); // 421 gas
```


<details>
<summary><i>35 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Curves.sol

104: revert UnauthorizedCurvesTokenSubject()
124: revert InvalidFeeDefinition()
136: revert InvalidFeeDefinition()
149: revert InvalidFeeDefinition()
213: revert SaleNotOpen()
233: revert CannotSendFunds()
237: revert CannotSendFunds()
243: revert CannotSendFunds()
265: revert UnauthorizedCurvesTokenSubject()
270: revert InsufficientPayment()
284: revert LastTokenCannotBeSold()
285: revert InsufficientBalance()
297: revert ContractCannotReceiveTransfer()
303: revert ContractCannotReceiveTransfer()
314: revert InsufficientBalance()
350: revert InvalidERC20Metadata()
371: revert CurveAlreadyExists()
384: revert InvalidPresaleStartTime()
386: revert CurveAlreadyExists()
396: revert CurveAlreadyExists()
412: revert PresaleUnavailable()
416: revert ExceededMaxBuyAmount()
425: revert UnverifiedProof()
433: revert ERC20TokenAlreadyMinted()
434: revert InvalidERC20Metadata()
461: revert ERC20TokenAlreadyMinted()
466: revert InsufficientBalance()
491: revert NonIntegerDepositAmount()
496: revert TokenAbsentForCurvesTokenSubject()
497: revert InsufficientBalance()
498: revert InsufficientBalance()
505: revert TokenAbsentForCurvesTokenSubject()
```
[104](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L104) | [124](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L124) | [136](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L136) | [149](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L149) | [213](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L213) | [233](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L233) | [237](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L237) | [243](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L243) | [265](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L265) | [270](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L270) | [284](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L284) | [285](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L285) | [297](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L297) | [303](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L303) | [314](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L314) | [350](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L350) | [371](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L371) | [384](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L384) | [386](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L386) | [396](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L396) | [412](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L412) | [416](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L416) | [425](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L425) | [433](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L433) | [434](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L434) | [461](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L461) | [466](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L466) | [491](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L491) | [496](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L496) | [497](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L497) | [498](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L498) | [505](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L505)

```solidity
File: contracts/FeeSplitter.sol

83: revert NoFeesToClaim()
91: revert NoTokenHolders()
115: revert NoFeesToClaim()
```
[83](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L83) | [91](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L91) | [115](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L115)
</details>

### [G-07] `x.a = x.a + b` always cheaper than `x.a += b` for Structs

Using direct assignment operations (e.g., `x.a = x.a + b`) is more gas-efficient compared to compound assignment operations (e.g., `x.a += b`).
Examples:
```solidity
    // direct write to storage   -> 5337 gas
    // struct storage pointer    -> 5341 gas
    // memory struct             -> 4628 gas
    // memory function param     -> 1109 gas
    x.a += b;

    // direct write to storage   -> 5330 gas
    // struct storage pointer    -> 5334 gas
    // memory struct             -> 4625 gas
    // memory function param     -> 1106 gas
    x.a = struct.a + b;
```


<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/FeeSplitter.sol

93: data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_
```
[93](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L93)
</details>

### [G-08] Cache `abi.encodePacked` for Efficiency

Function calls, especially external function calls, can be quite expensive in terms of gas. 
If you need to retrieve the same data more than once in a function, it is more efficient to call 
the function once, store the result in a local variable, and then use that variable later in the 
code instead of making the function call again.

<details>
<summary><i>6 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

442: keccak256(abi.encodePacked("")) ||
444: keccak256(abi.encodePacked(""))
472: keccak256(abi.encodePacked("")) ||
474: keccak256(abi.encodePacked(""))
```
[442](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L442) | [444](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L444) | [472](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L472) | [474](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L474)
</details>

### [G-09] Refactor duplicated require()/revert() checks to save gas

Duplicate require()/revert() checks can be refactored into a modifier or function, saving deployment costs.

<details>
<summary><i>8 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

297: if (to == address(this)) revert ContractCannotReceiveTransfer();
303: if (to == address(this)) revert ContractCannotReceiveTransfer();
350: if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
434: if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
371: if (supply != 0) revert CurveAlreadyExists();
386: if (supply != 0) revert CurveAlreadyExists();
433: if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
461: if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
```
[297](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L297) | [303](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L303) | [350](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L350) | [434](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L434) | [371](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L371) | [386](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L386) | [433](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L433) | [461](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L461)
</details>

### [G-10] Consider Using += for Mappings

Using the += operator for mappings can save around 40 gas due to avoiding the recalculation of the mapping value's hash. 
Consider refactoring the code to use this more efficient approach.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

273: curvesTokenSupply[curvesTokenSubject] = supply + amount;
```
[273](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L273)
</details>

### [G-11] Use Cached Contracts for Multiple External Calls

When function makes multiple calls to the same external contract, it is more gas-efficient to use a local copy of the contract.
This is because the EVM will cache the contract in memory, and subsequent calls will be cheaper.
It's especially true for contracts that are large and/or have many functions.
```solidity
    // local cache -> 6561 gas
    IToken localCache = storageContract;
    localCache.externalCall();
    localCache.externalCall();

    // direct call 6683 gas
    storageContract.externalCall();
    storageContract.externalCall();
```

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Curves.sol

/// @audit function _transferFees() make external call of `feeRedistributor` - 2 times
247: feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
248: feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
```
[247](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L247) | [248](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L248)

```solidity
File: contracts/FeeSplitter.sol

/// @audit function totalSupply() make external call of `curves` - 2 times
45: return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
45: return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
```
[45](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L45) | [45](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L45)
</details>

### [G-12] Use `uint256(1)`/`uint256(2)` instead of `true`/`false` to save gas for changes

Boolean variables in Solidity are more expensive than `uint256` or any type that takes up a full word, due to additional gas costs associated with write operations.
When using boolean variables, each write operation emits an extra SLOAD to read the slot's contents, replace the bits taken up by the boolean, and then write back.
This process cannot be disabled and leads to extra gas consumption.

By using `uint256(1)` and `uint256(2)` for representing true and false states, you can avoid a `Gwarmaccess` (100 gas) cost and also avoid a `Gsset` (20000 gas) cost when changing from `false` to `true`, after having been `true` in the past.
This approach helps in optimizing gas usage, making your contract more cost-effective.

[Usage in OpenZeppelin ReentrancyGuard.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27)

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Security.sol

6: mapping(address => bool) public managers;
```
[6](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L6)
</details>

### [G-13] Avoid contract existence checks by using low-level calls

Before version 0.8.10, the Solidity compiler would insert extra code, such as EXTCODESIZE (costing 100 gas), to check the existence of a contract for external function calls.
Newer versions, starting from 0.8.10, no longer insert these checks if the external call has a return value.
You can achieve similar behavior in earlier Solidity versions by using low-level calls like `call`.
This low-level call don't check for contract existence, saving gas costs.

<details>
<summary><i>11 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Curves.sol

247: feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
352: address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));
425: if (!MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf)) revert UnverifiedProof();
487: CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
500: CurvesERC20(externalToken).burn(msg.sender, amount);
```
[247](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L247) | [352](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L352) | [425](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L425) | [487](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L487) | [500](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L500)

```solidity
File: contracts/FeeSplitter.sol

40: return curves.curvesTokenBalance(token, account) * PRECISION;
45: return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
```
[40](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L40) | [45](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L45)
</details>

### [G-14] Optimize Gas by Using Only Named Returns

The Solidity compiler can generate more efficient bytecode when using named returns.
It's recommended to replace anonymous returns with named returns for potential gas savings.

Example:
```solidity
/// 985 gas cost
function add(uint256 x, uint256 y) public pure returns (uint256) {
    return x + y;
}
/// 941 gas cost
function addNamed(uint256 x, uint256 y) public pure returns (uint256 res) {
    res = x + y;
}
```

<details>
<summary><i>12 issue instances in 3 files:</i></summary>

```solidity
File: contracts/Curves.sol

179: function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
188: function getBuyPrice(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
192: function getSellPrice(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
196: function getBuyPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
203: function getSellPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
337: function _deployERC20(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal returns (address) {
```
[179](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L179) | [188](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L188) | [192](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L192) | [196](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L196) | [203](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L203) | [337](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L337)

```solidity
File: contracts/CurvesERC20Factory.sol

7: function deploy(string memory name, string memory symbol, address owner) public returns (address) {
```
[7](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L7)

```solidity
File: contracts/FeeSplitter.sol

38: function balanceOf(address token, address account) public view returns (uint256) {
42: function totalSupply(address token) public view returns (uint256) {
47: function getUserTokens(address user) public view returns (address[] memory) {
51: function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
72: function getClaimableFees(address token, address account) public view returns (uint256) {
```
[38](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L38) | [42](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L42) | [47](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L47) | [51](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L51) | [72](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L72)
</details>

### [G-15] State variables should be cached in stack rather than re-reading them from storage

Caching state variables in local variables can optimize gas usage, as accessing the stack is cheaper than accessing storage.

The instances below point to the second+ access of a state variable within a function.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

/// @audit function _deployERC20() uses state variable `_curvesTokenCounter` 2 times
346: name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
347: symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
```
[346](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L346) | [347](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L347)
</details>

### [G-16] Avoid updating storage when the value hasn't changed

A check regarding whether the current value and the new value are the same should be added.
This helps prevent unnecessary state changes and events in case the new value is the same as the current value.

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: contracts/Curves.sol

/// @audit Missing `referralFeeDestination_` check before state change
159: referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
/// @audit Missing `factory_` check before state change
163: curvesERC20Factory = factory_;
```
[159](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L159) | [163](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L163)

```solidity
File: contracts/FeeSplitter.sol

/// @audit Missing `curves_` check before state change
36: curves = curves_;
```
[36](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L36)

```solidity
File: contracts/Security.sol

/// @audit Missing `value` check before state change
24: managers[manager_] = value;
```
[24](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L24)
</details>
