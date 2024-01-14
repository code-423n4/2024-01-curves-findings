## [G-01] `CurvesERC20Factory.deploy()`: Combining the two statements will save gas

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L7-L10

The function `CurvesERC20Factory.deploy()` is currently written as so

```solidity
function deploy(string memory name, string memory symbol, address owner) public returns (address) {
    CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);
    return address(tokenContract);
}
```

There are redundant typecasting, and storing of information. We can save some gas by simplifying the function like this:

```solidity
function deploy(string memory name, string memory symbol, address owner) public returns (address) {
    return address(new CurvesERC20(name, symbol, owner));
}
```

Saves **13** gas on hardhat testing.

## [G-02] `CurvesERC20`: constructor should use `_transferOwnership`, not `transferOwnership`

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L8-L10

In the constructor of `CurvesERC20`:

```solidity
constructor(string memory name_, string memory symbol_, address owner) ERC20(name_, symbol_) {
    transferOwnership(owner); // @audit this should be _transferOwnership
}
```

It transfers ownership from the factory to the deployer. [However](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol), it is using the public function `transferOwnership()`, instead of the internal function `_transferOwnership()`. The public function is as follow:

```solidity
function transferOwnership(address newOwner) public virtual onlyOwner {
    if (newOwner == address(0)) {
        revert OwnableInvalidOwner(address(0));
    }
    _transferOwnership(newOwner);
}
```

It has an access control modifier, which reads into storage to check the owner, then calls the internal function anyway. 

Therefore it is strictly more gas-efficient to use the internal `_transferOwnership()`, as opposed to the public counterpart.

Saves at least **100** gas for reducing storage reads, saves some more for reducing the number of operations. Optimizes about **500** gas in hardhat testing.

## [G-03] `Curves`: storage can be packed into fewer slots

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L42-L47

The current storage layout (top 3 variables) on `Curves` is as follow:

```solidity
address public curvesERC20Factory; // slot 1
FeeSplitter public feeRedistributor; // slot 2
uint256 private _curvesTokenCounter = 0; // slot 3
```

Based on these two facts:
- `_curvesTokenCounter` only increments by one, and is incremented every time a new `CurvesERC20` is deployed.
- Every time a token is deployed, the factory address must be read.

We can then combine `curvesERC20Factory` and `_curvesTokenCounter` into one storage slot as follow:

```solidity
address public curvesERC20Factory; 
uint96 private _curvesTokenCounter = 0; // slot 1
FeeSplitter public feeRedistributor; // slot 2
```

This will save gas for it reduces the number of storage slots to be read and write on. 

Saves **18000** gas on hardhat testing.

## [G-04] `FeesEconomics` can be packed into just two storage slots, instead of six

*Note: this finding elaborates bot finding [G-19], which was only vaguely described and does not tell how storage can be packed.*

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L68-L75

The current `FeesEconomics` struct is as follow:

```solidity
struct FeesEconomics {
    address protocolFeeDestination;
    uint256 protocolFeePercent;
    uint256 subjectFeePercent;
    uint256 referralFeePercent;
    uint256 holdersFeePercent;
    uint256 maxFeePercent;
}
```

Each variable takes up one slot. However examining `getFees()`, we see the fee percentages are denominated by $10^{18}$ = 100%. 

Therefore none of the fee percentages can exceed $10^{18}$, and this value fits comfortably in the unsigned 64-bit data type. The new storage can be as follow:

```solidity
struct FeesEconomics {
    address protocolFeeDestination;
    uint64 maxFeePercent; // slot 1
    uint64 protocolFeePercent;
    uint64 subjectFeePercent;
    uint64 referralFeePercent;
    uint64 holdersFeePercent; // slot 2
}
```

Saves at least **8000** gas on every sale, as the fee has to be calculated on each sale, and we've reduced at least 4 storage slot with this optimization.

## [G-05] `CurvesERC20Factory` contract is redundant

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol

The highlighted characteristics of the `CurvesERC20Factory` are as follow:
- There is a single function `deploy()`
- This contract is used by `Curves` to deploy new tokens.

When a new `CurvesERC20` token is deployed, the owner is initially the factory, but then the constructor transfers ownership to `Curves` anyway. 

Then there is no point to using the factory, as opposed to just let `Curves` deploy a new token by itself. This way, ownership does not have to be re-written, and it simplifies the code logic considerably.

The gas impact is that:
- Reduces storage reads by not having to read the factory address.
- Reduces one storage write, by not having to write into token owner address twice.
- Further gas savings by reducing the number of function calls.

The storage reductions will save gas by the thousands.

## [G-06] `Curves.mint()` can use `msg.sender` as the parameter directly

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L439C5-L439C92

The `mint()` function signature is as follow:

```solidity
function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
```

The `onlyTokenSubject(curvesTokenSubject)` only does the check

```solidity
modifier onlyTokenSubject(address curvesTokenSubject) {
    if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();
    _;
}
```

The purpose is only to ensure that no one can mint a token for anyone else.

Then there is no purpose to using the modifier, but rahter one can use `msg.sender` as the subject directly. This will save some gas on the condition check.

## [G-07] Redundant block timestamp check in `buyCurvesTokenWhitelisted`

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L409-L412

There is the following condition check in `buyCurvesTokenWhitelisted`:

```solidity
if (
    presalesMeta[curvesTokenSubject].startTime == 0 ||
    presalesMeta[curvesTokenSubject].startTime <= block.timestamp
) revert PresaleUnavailable();
```

Note that, since `block.timestamp` can never be zero, then if the first condition is satisfied, the second condition always satisfies. Therefore it is enough to check the second condition only.

## [G-08] The math in `getPrice()` can be slightly simplified

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L186

The price for a token can be defined as follow:
- Let one unit of price be equal to $0.0000625$ Ethers.
- The price for token $N$ is equal to $N^2$ price units.

Then it is possible to define a constant defining one price unit, and multiply `summation` by that constant (instead of multiplying by `1 ether / 16000`). The expressions can be easily proven to be mathematically equivalent.

## [G-09] There is no need to use `abi.encodePacked` on a single parameter

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L424

```solidity
bytes32 leaf = keccak256(abi.encodePacked(caller));
```

The code linked above calls `abi.encodePacked` on a single parameter: the `caller` address.

ABI encode packed simply concatenates the data together without padding zero bits, while normal abi encode pads the data so that each parameter is 256 bits.

In other words, `abi.encodePacked()` on a single parameter will just produce itself. Therefore, there is no need to encode it before hashing.
