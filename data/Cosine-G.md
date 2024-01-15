## The whole `onlyTokenSubject` modifier token is a waste of gas and can be replaced with the direct use of msg.sender

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L103-L106

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L155-L160

The `Curves` contract uses a modifier called `onlyTokenSubject` which looks like this:

```solidity
modifier onlyTokenSubject(address curvesTokenSubject) {
    if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();
    _;
}
```

It checks if the given `curvesTokenSubject` address equals the msg.sender.

This modifier is used multiple times in the system and wastes gas as modifying the given mapping for msg.sender has the same effect as passing in a custom address and checking if it equals msg.sender. Here is one example:

```solidity
function setReferralFeeDestination(
    address curvesTokenSubject,
    address referralFeeDestination_
) public onlyTokenSubject(curvesTokenSubject) {
    referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
}
```

Therefore the whole modifier can be removed and instead working with msg.sender instead of a custom address checked to equal msg.sender will save gas on every function that uses this modifier as one if statement and parameter can be avoided.

## The `amount` parameter in the `buyCurvesTokenWithName` and `buyCurvesTokenForPresale` functions can be removed as it must always equal 1

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L364-L375

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L377-L392

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L267

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L180-L187

The `buyCurvesTokenWithName` function can be used by the owner of a token to mint the first one and therefore start the sale and mint the external ERC-20 version of it in one call. It has a parameter named `amount` which is used to specify the amount of tokens to buy. But the function will revert if the supply is not zero and the first mint must always be only one otherwise the price calculation will revert. The `amount` parameter can be removed and replaced with 1 instead as any other value will revert.

Here we can see the flow of this function:

```solidity
function buyCurvesTokenWithName(
    address curvesTokenSubject,
    uint256 amount,
    string memory name,
    string memory symbol
) public payable {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply != 0) revert CurveAlreadyExists();

    _buyCurvesToken(curvesTokenSubject, amount);
    _mint(curvesTokenSubject, name, symbol);
}

function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
    ...

    uint256 price = getPrice(supply, amount);
    ...
}

function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}
```

The same holds true for the `buyCurvesTokenForPresale` function which is used to start a presale. It also has an `amount` parameter which can be removed as it must always be 1:

```solidity
function buyCurvesTokenForPresale(
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
}
```

## The `sellExternalCurvesToken` function checks the same condition twice

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L505

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L496

The `sellExternalCurvesToken` function checks if the given external curves token exists and calls the `deposit` function right after that which checks the same condition again.

Here we can see that the same condition is checked twice:

```solidity
function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public {
    if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();

    deposit(curvesTokenSubject, amount);
    sellCurvesToken(curvesTokenSubject, amount / 1 ether);
}

function deposit(address curvesTokenSubject, uint256 amount) public {
    ...

    address externalToken = externalCurvesTokens[curvesTokenSubject].token;
    ...

    if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
    ...
}
```

Remove this line `if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();` inside the `sellExternalCurvesToken` function.

## The constructor in the `CurvesERC20` contract transfers the ownership to the msg.sender twice

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/CurvesERC20.sol#L9

When any user initializes the external ERC-20 version of a curve token the constructor of the `CurvesERC20` contract is called.

Here we can see the constructor and that it calls the `transferOwnership` function of the `Ownable` contract from OpenZeppelin:

```solidity
contract CurvesERC20 is ERC20, Ownable {
    constructor(string memory name_, string memory symbol_, address owner) ERC20(name_, symbol_) {
        transferOwnership(owner);
    }

    ...
}
```

However, this is already done inside the constructor of the `Ownable` contract:

```solidity
abstract contract Ownable is Context {
    address private _owner;

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    /**
     * @dev Initializes the contract setting the deployer as the initial owner.
     */
    constructor() {
        _transferOwnership(_msgSender());
    }

    ...
}
```

Therefore this line `transferOwnership(owner);` can be removed from the `CurvesERC20` constructor.

## Redundant calculation inside the `claimFees` function

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L87

The `claimFees` function can be called to claim fees. It first calls the `updateFeeCredit`, then the `getClaimableFees` function, and then updates the `unclaimedFees` of the user to zero and transfers the fees to the user.

Here we can see these three functions:

```solidity
function updateFeeCredit(address token, address account) internal {
    TokenData storage data = tokensData[token];
    uint256 balance = balanceOf(token, account);
    if (balance > 0) {
        uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
        data.unclaimedFees[account] += owed / PRECISION;
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
    }
}

function getClaimableFees(address token, address account) public view returns (uint256) {
    TokenData storage data = tokensData[token];
    uint256 balance = balanceOf(token, account);
    uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
    return (owed / PRECISION) + data.unclaimedFees[account];
}

function claimFees(address token) external {
    updateFeeCredit(token, msg.sender);
    uint256 claimable = getClaimableFees(token, msg.sender);
    if (claimable == 0) revert NoFeesToClaim();
    tokensData[token].unclaimedFees[msg.sender] = 0;
    payable(msg.sender).transfer(claimable);
    emit FeesClaimed(token, msg.sender, claimable);
}
```

As we can see the `unclaimedFees` amount is calculated by subtracting the `userFeeOffset` from the `cumulativeFeePerToken`. This is done twice first in the `updateFeeCredit` and then in the `getClaimableFees` function. But as the `updateFeeCredit` sets the `userFeeOffset` to the `cumulativeFeePerToken`. Subtracting the `userFeeOffset` from the `cumulativeFeePerToken` is redundant as it will always be zero as they are equal now. Therefore the `claimFees` should just work with the `unclaimedFees` to save gas.
