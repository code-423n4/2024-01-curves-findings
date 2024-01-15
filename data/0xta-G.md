# Gas Optimization

## [G-01] bytes.concat() can be used in place of abi.encodePacked

```solidity
File: contracts/Curves.sol


346            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
347            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));

424        bytes32 leaf = keccak256(abi.encodePacked(caller));

441            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
442            keccak256(abi.encodePacked("")) ||
443            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
444            keccak256(abi.encodePacked(""))

471                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
472                keccak256(abi.encodePacked("")) ||
473                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
474                keccak256(abi.encodePacked(""))
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L346

## [G-02] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: contracts/Curves.sol

352        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L352

## [G-03] Access MAPPINGS directly rather than using accessor functions

When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas
cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving
the corresponding value.

```solidity
File: contracts/FeeSplitter.sol

48    function getUserTokens(address user) public view returns (address[] memory) {
        return userTokens[user];
    }
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L48

## [G-04] Use hardcode address instead address(this)

```solidity
File: contracts/Curves.sol

297        if (to == address(this)) revert ContractCannotReceiveTransfer();

303        if (to == address(this)) revert ContractCannotReceiveTransfer();

352        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

486        _transfer(curvesTokenSubject, msg.sender, address(this), amount);


498        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

501        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);

```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L297

## [G-05] Use Do-While loops instead of for loops

```solidity
File: contracts/Curves.sol

305        for (uint256 i = 0; i < subjects.length; i++) {

330        for (uint256 i = 0; i < subjects.length; i++) {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305

```solidity
File: contracts/FeeSplitter.sol

55        for (uint256 i = 0; i < tokens.length; i++) {

105        for (uint256 i = 0; i < tokenList.length; i++) {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55

## [G-06] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: contracts/Curves.sol

181        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
181        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;


// >>> -1,  +1
184            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L181

## [G-07] array[index] += amount is cheaper than array[index] = array[index] + amount

```solidity
File: contracts/Curves.sol

321        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
322        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L321

## [G-08] Using STORAGE instead of MEMORY for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: contracts/FeeSplitter.sol

53        address[] memory tokens = getUserTokens(user);
54        UserClaimData[] memory result = new UserClaimData[](tokens.length);
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L53

## [G-09] Using  calldata instead of  memory for read-only arguments in external functions

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the
memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 \* <mem_array>.length). Using calldata directly, obliviates the need for
such a loop in the contract code and runtime execution.

```solidity
File: contracts/Curves.sol

407        bytes32[] memory proof

422    function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L407

```solidity
File: contracts/FeeSplitter.sol

48    function getUserTokens(address user) public view returns (address[] memory) {

52    function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L48