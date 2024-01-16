## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls | 5 | - |
| [G-02] | keccak256() should only need to be called on a specific string literal once | 2 | - |
| [G-03] | Multiplication/division by two should use bit shifting | 3 | - |
| [G-04] | Not using the named return variable when a function returns, wastes deployment gas | 1 | - |
| [G-05] | Empty blocks should be removed or emit something | 2 | - |
| [G-06] | With assembly, .call (bool success)  transfer can be done gas-optimized | 6 | - |
| [G-07] | se constants instead of type(uintx).max | 1 | - |
| [G-08] | A modifier used only once and not being inherited should be inlined to save gas | 1 | - |
| [G-09] | Use hardcode address instead address(this) | 1 | - |
| [G-10] | array[index] += amount is cheaper than array[index] = array[index] + amount (or related variants) | 2 | - |

## Gas Optimizations  

## [G-01] Avoid contract existence checks by using low level calls		

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls.
In more recent solidity versions, the compiler will not insert these checks if the external call has a return value.
Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.


```solidity
file: /contracts/Curves.sol

247                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);

248                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);

346            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));

347            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));

352        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L247C1-L248C80


## [G-02] keccak256() should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead.
If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once

```solidity
file: /contracts/Curves.sol
///@audit  found Line : 471-474
441            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
442            keccak256(abi.encodePacked("")) ||
443            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
444            keccak256(abi.encodePacked(""))

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L441C12-L444C44


## [G-03]  Multiplication/division by two should use bit shifting

<x> * 2 is the same as <x> << 1. While the compiler uses the SHL opcode to accomplish both, the version that uses multiplication incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two.

<x> / 2 is the same as <x> >> 1. 

While the compiler uses the SHR opcode to accomplish both, the version that uses division incurs an overhead of   20 gas due  to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two.


```solidity
file: /contracts/Curves.sol

181        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;

184            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;

186        return (summation * 1 ether) / 16000;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L181C1-L181C97


## [G-04] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.


```solidity
file: /contracts/Curves.sol

361        return address(tokenContract);

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L361C1-L361C39


## [G-05]  Empty blocks should be removed or emit something

The gas cost of an empty constructor block in Solidity is typically in the range of 200-500 gas units. 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.


```solidity
file: /contracts/FeeSplitter.sol

33    constructor() Security() {}

119     receive() external payable {}

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L119


## [G-06] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), 
this storage disappears and gas optimization is provided.

```solidity
file: /contracts/Curves.sol

232                (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");

236                (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L232C1-L232C100



## [G-07] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc.
it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /contracts/Curves.sol

389        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L389C1-L389C94


## [G-08]  A modifier used only once and not being inherited should be inlined to save gas

```solidity
file: /contracts/Curves.sol#

103    modifier onlyTokenSubject(address curvesTokenSubject) {

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L103C1-L103C60


## [G-09] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

```solidity
file: /contracts/Curves.sol

297        if (to == address(this)) revert ContractCannotReceiveTransfer();

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L297C1-L297C73


## [G-10] array[index] += amount is cheaper than array[index] = array[index] + amount (or related variants)

When updating a value in an array with arithmetic, using array[index] += amount is cheaper than array[index] = array[index] + amount. This is because you avoid an additonal mload when the array is stored in memory, and an sload when the array is stored in storage. This can be applied for any arithmetic operation including +=, -=,/=,*=,^=,&=, %=, <<=,>>=, and >>>=. This optimization can be particularly significant if the pattern occurs during a loop.

```solidity
file: /contracts/Curves.sol

321         curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;

322        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L321C9-L321C53

