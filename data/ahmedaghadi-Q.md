## [NC-01] Multiple `transferCurvesToken` of zero `amount` would make it difficult to execute `transferAllCurvesTokens` function.

The `transferCurvesToken` function accepts `amount` equals to zero ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L296 ):

```solidity
// Transfers tokens from current owner to receiver. Can be used for gifting or distributing tokens.
function transferCurvesToken(address curvesTokenSubject, address to, uint256 amount) external {
    if (to == address(this)) revert ContractCannotReceiveTransfer();
@->    _transfer(curvesTokenSubject, msg.sender, to, amount);
}
```

It executes `_transfer` function which also allows `amount` to be zero and add the `curvesTokenSubject` to `ownedCurvesTokenSubjects` array as it executes `_addOwnedCurvesTokenSubject` function. The `_transfer` function is as follows ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L313 ):

```solidity
function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
    if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

    // If transferring from oneself, skip adding to the list
    if (from != to) {
@->        _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
    }

    curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
    curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;

    emit Transfer(curvesTokenSubject, from, to, amount);
}
```

The `_addOwnedCurvesTokenSubject` function is as follows ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L328 ):

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

If attacker keeps on transferring multiple tokens of zero `amount` to a user, then it would be difficult for the user to execute `transferAllCurvesTokens` function as it would revert due to gas limit. The `transferAllCurvesTokens` function is as follows ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L302 ):

```solidity
// Transfer the total balance of all my tokens to another address. Can be used for migrating tokens.
function transferAllCurvesTokens(address to) external {
    if (to == address(this)) revert ContractCannotReceiveTransfer();
    address[] storage subjects = ownedCurvesTokenSubjects[msg.sender];
    for (uint256 i = 0; i < subjects.length; i++) {
        uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
        if (amount > 0) {
            _transfer(subjects[i], msg.sender, to, amount);
        }
    }
}
```

## Recommended Mitigation Steps

Adding a check for `amount` to be greater than zero in `_transfer` function will fix the issue.

## [NC-02] No need to wrap `tokenContract` in `address` as it's already an `address` for the `_deployERC20` function.

The `tokenContract` is already an address variable so there's no need to wrap it in `address` in the return value of `_deployERC20` function ( https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L338 ):

```solidity
function _deployERC20(
    address curvesTokenSubject,
    string memory name,
    string memory symbol
) internal returns (address) {
    // If the token's symbol is CURVES, append a counter value
    if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
        _curvesTokenCounter += 1;
        name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
        symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
    }

    if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

    address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

    externalCurvesTokens[curvesTokenSubject].token = tokenContract;
    externalCurvesTokens[curvesTokenSubject].name = name;
    externalCurvesTokens[curvesTokenSubject].symbol = symbol;
    externalCurvesToSubject[tokenContract] = curvesTokenSubject;
    symbolToSubject[symbol] = curvesTokenSubject;

    emit TokenDeployed(curvesTokenSubject, tokenContract, name, symbol);
@->    return address(tokenContract);
}
```
