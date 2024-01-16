## Summary

### Gas Optimization

no |Issue| Instances||
|-|:-|:-:|:-:|
| [G-01]|Add unchecked {} for subtractions where the operands can’t underflow because of previous checks Adding an uncheck  |9|--|  | 
| [G-02] |Using assembly to revert with an error message |11|--|  | 
| [G-03] |Modifier makes it expensive since we end up reading state twice |2|--|   |
| [G-04] |We can save an entire SLOAD (2100 Gas) by short circuiting the operations|11|--|   |
| [G-05] | Cache function calls |6|--|   |
| [G-06] | Using immutable on variables that are only set in the constructor and never after (Save 14.7K Gas) |3|--|   |
| [G-07] |Avoid making external calls if we don’t have to |14|--|   |
| [G-08] |Modifier execution order can help save an entire The first modifier onlyOwner() which is also the first on |11|--|   | 
| [G-09] |State variables only set in the constructor should be declared immutable |2|--|   | 
| [G-10] |State variables that are used multiple times in a function should be cached in stack variables |5|--| |
## Gas Optimizations  

## [G-1] Add unchecked {} for subtractions where the operands can’t underflow because of previous checks  Adding an unchecked block for a subtraction that cannot underflow due to a previous check saves approximately 3 gas per subtraction.
With many such subtractions occurring across interactions, the total gas savings could be significant. 
```solidity
file:contracts/Curves.sol

  181     uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1)   +  1)) / 6;
 185       uint256 summation = sum2 - sum1;
 194        return getPrice(curvesTokenSupply[curvesTokenSubject] - amount, amount);
    }
 208      return price - totalFee;
 231      uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
 287        uint256 price = getPrice(supply - amount, amount);
 290       curvesTokenSupply[curvesTokenSubject] = supply - amount;
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L181
```solidity
File:/contracts/FeeSplitter.sol
 67     uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) *balance;
 76     uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L67


## [G-2] Using assembly to revert with an error message
 When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.
```solidity
file: /contracts/Curves.sol
 104        if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();
 213        if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
 233        if (!success1) revert CannotSendFunds();
 265        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();
 284        if (supply <= amount) revert LastTokenCannotBeSold();
 297        if (to == address(this)) revert ContractCannotReceiveTransfer();
 303        if (to == address(this)) revert ContractCannotReceiveTransfer();
 314        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
 350        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
 371         if (supply != 0) revert CurveAlreadyExists();
 434       if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L104
## [G-3]Modifier makes it expensive since we end up reading state twice (Saves 1197 Gas on average from the tests)  
```solidity
file:/contracts/Curves.so
103     modifier onlyTokenSubject(address curvesTokenSubject) {
      
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L103
 ```solidity
 file:/contracts/Security.sol
    13    modifier onlyManager() {
          managers[msg.sender] == true;
         _;
    
 ```
 https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L13-L16

## [G-4] We can save an entire SLOAD (2100 Gas) by short circuiting the operations

```solidity
file: 
  218     function _transferFees(
  224   internal {
      
  233     if (!success1) revert CannotSendFunds();        
  246     if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0))
  264       if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();
  277       if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
  285     if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();
  297       if (to == address(this)) revert ContractCannotReceiveTransfer();
  314       if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
  350       if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
  396       if (supply > 1) revert CurveAlreadyExists();
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218-L249


## [G-5] Cache function calls
The following finding is somehow related to this one, some confusion on why the devs choose to do the calls. The next finding will show an alternate optimization.

```solidity

file:
   89  function setWhitelist(bytes32 merkleRoot) external {
   90   uint256 supply = curvesTokenSupply[msg.sender];
   91   if (supply > 1) revert CurveAlreadyExists();

   93     if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
   94       presalesMeta[msg.sender].merkleRoot = merkleRoot;
   95       emit WhitelistUpdated(msg.sender, merkleRoot);
        }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99

## [G-6] Using immutable on variables that are only set in the constructor and never after (Save 14.7K Gas)
Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
```solidity
file:/contracts/Curves.sol
42       address public uniV3Pool;
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L42

```solidity
file:/contracts/Security.sol
5    address public owner;
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L5-L5
```

## [G-7]Avoid making external calls if we don’t have to
```solidity
file:/contracts/Curves.sol
  113  function setFeeRedistributor(address feeRedistributor_) external onlyOwner {
  114   feeRedistributor = FeeSplitter(payable(feeRedistributor_));
    }

   117    function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
   118     if (
   119        feesEconomics.protocolFeePercent +
   120            feesEconomics.subjectFeePercent +
   121            feesEconomics.referralFeePercent +
   122            feesEconomics.holdersFeePercent >
   123        maxFeePercent_
   124   ) revert InvalidFeeDefinition();
   128    function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
   141      function setExternalFeePercent(
   162       function setERC20Factory(address factory_) external onlyOwner {
   302      function transferAllCurvesTokens(address to) external {
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L113

## [G-8] Modifier execution order can help save an entire
The first modifier onlyOwner() which is also the first one to be executed in our function involves reading a state variable governor, Since this is the first SLOAD we consume 2100 Gas for the SLOAD.
The second modifier nonZero does not read any state variable, only parameters being passed to it, in our case , some function parameters
```solidity
file:
103    modifier onlyTokenSubject(address curvesTokenSubject) {
104    if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();   105   _;
   

```
  https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L103C5-L106C6

```solidity
file:/contracts/Security.sol
8      modifier onlyOwner() {
9      msg.sender == owner;
10        _;
11      }
13   modifier onlyManager() {
14   managers[msg.sender] == true;
15      _;
16   }
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L8
## [G-9] State variables only set in the constructor should be declared immutable
The solidity compiler will directly embed the values of immutable variables into your contract bytecode and therefore will save you from incurring a Gsset (20000 gas) when you set storage variables in the constructor, a Gcoldsload (2100 gas) when you access storage variables for the first time in a transaction,  and a Gwarmaccess (100 gas) for each subsequent access to that storage slot.

```solidity
file:/contracts/Curves.sol
42:    address public curvesERC20Factory;
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L42-L42
```solidity
file:/contracts/Security.sol
5   address public owner;
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L5

## [G-10] State variables that are used multiple times in a function should be cached in stack variables
By caching state variables in stack variables, we reduce the need to frequently access storage, thereby saving gas.

```solidity
file:/contracts/Curves.sol
42    address public curvesERC20Factory;
43    FeeSplitter public feeRedistributor;
44   string public constant DEFAULT_NAME = "Curves";
45   string public constant DEFAULT_SYMBOL = "CURVES";

```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L42C39-L42

```solidity
file:
5    address public owner;

```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L5
