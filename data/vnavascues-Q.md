## LOW-001 Transferring ownership is not consistent with roles

File: [`contracts/Security.sol`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol)

On deployment, the deployer address becomes these roles:

- Owner
- Manager

However, when the [ownership is transferred](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L27) (via `transferOwnership` function) the new Owner does not become a Manager (but the previous Owner still does).

### Recommended Mitigation Steps

Be consistent with what the Owner role implies and accordingly set the Manager role to the previous and new owner when transferring ownership.

```solidity
    function transferOwnership(address owner_) public onlyOwner {
        managers[owner] = false;
        managers[_owner] = true;
        owner = owner_;
    }
```

Further Manager roles can be set by calling `setManager` function.

## LOW-002 Tokens subject could be bought without paying protocol and/or external fees

File: [`contracts/Security.sol`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol)

The [`Curves` constructor](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L108) does not set protocol fees neither external fees, which means that after its deployment the following set-up calls must happen:

1. `setMaxFeePercent`.
2. `setProtocolFeePercent`.
3. `setExternalFeePercent`.

If a Curves token subject buys its first amount before any of these transactions happen, any other address will be able to front-run the set-up calls and trade tokens without paying protcol fees and/or external fees.

### Recommended Mitigation Steps

Allow the constructor to set these fees:

```solidity
constructor(
    address curvesERC20Factory_,
    address feeRedistributor_,
    address protocolFeeDestination_,
    uint256 maxFeePercent_,
    uint256 protocolFeePercent_,
    uint256 subjectFeePercent_,
    uint256 referralFeePercent_,
    uint256 holdersFeePercent_,
    uint256 maxFeePercent_,
    uint256 maxFeePercent_
) Security() {
    ...

```

Otherwise, create a deployment factory contract that handles the `Curves` deployment and set-up in a single transaction.

## LOW-003 `ownedCurvesTokenSubjects` data is not properly updated

File: [`contracts/Security.sol`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol)

Given a buyer address, the purpose of the [`ownedCurvesTokenSubjects`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L101) mapping is to keep track of the owned Curves subject tokens; this is why the token address is mapped to the buyer address on the first buy (via [`_buyCurvesToken`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L278) function calling [`_addOwnedCurvesTokenSubject`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L328) function):

```solidity
if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
    _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
}
```

This is functionality is allegedly useful on a future migration of tokens (via [`transferAllCurvesTokens`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L302) function). However, `_addOwnedCurvesTokenSubject` logic presents few problems:

- Its ownership data is not reliable as the array of token addresses is not properly handled (i.e. token address removed from `address[]`) when and address balance becomes 0 by either transferring or selling tokens.
- It incurs an innecessary gas overhead each time an address balance changes from 0 to a non-zero `uint256` due to `_addOwnedCurvesTokenSubject` looping through all the token addresses the buyer has ever owned. This has a cost of _O(n)_ when it could be _O(1)_.

### Recommended Mitigation Steps

In case of not removing this functionality, the recommendation is to:

1. Make `ownedCurvesTokenSubjects` a private [Iterable mapping](https://solidity-by-example.org/app/iterable-mapping/) or [EnumerableSet](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/EnumerableSet.sol). For instance:

```solidity
library IterableMapping {
    struct Map {
        address[] keys;
        mapping(address => bool) isOwned; // HERE, true means the address currently owns the token
        mapping(address => uint) indexOf;
        mapping(address => bool) inserted;
    }
...
```

2. On buys, sells and transfers (which also involves deposits and withdrawals) keep track of the resulting address balance and properly update the `isOwned` property of the iterable mapping. Amending `_addOwnedCurvesTokenSubject` will make its complexity _O(1)_ instead of _O(n)_.
3. Implement an external function that give addresses visibility of which tokens they own (current `ownedCurvesTokenSubjects` is private).
