| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-sellcurvestoken-and-buycurvestoken-functions-can-revert-due-to-lack-of-require) | `sellCurvesToken` and `buyCurvesToken` functions can revert due to lack of require. | Low |
| [L-02](#l-02-lack-of-refund-mechanism-in-buycurvestoken-function) | Lack of refund mechanism in `_buyCurvesToken` function. | Low |
| [L-03](#l-03-buycurvestokenwithname-function-lack-of-onlytokensubject-modifier) | `buyCurvesTokenWithName` function lack of `onlyTokenSubject` modifier. | Low |
| [L-04](#l-04-onbalancechange-push-same-token-into-array-more-than-once) | `onBalanceChange` push same token into array more than once. | Low |


# [L-01] `sellCurvesToken` and `buyCurvesToken` functions can revert due to lack of require.

## Description
There is no check that input `amount` is not equal to zero in `sellCurvesToken` and `buyCurvesToken` funcitons, therefore revert can happen.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L282

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L211

## Recommendation
`sellCurvesToken` function:

```diff
function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
++  if (amount == 0) revert InvalideAmount();
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply <= amount) revert LastTokenCannotBeSold();
    ...
}
```

`buyCurvesToken` function:

```diff
function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
++  if (amount == 0) revert InvalideAmount();
    uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
    ...
}
```


# [L-02] Lack of refund mechanism in `_buyCurvesToken` function.

## Description
In `_buyCurvesToken` function there is a check that `msg.value` less than sum of `price` and `totalFee`.

```solidity
if (msg.value < price + totalFee) revert InsufficientPayment();
```

However, it's possible for user to send more than `sum` by mistake.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L270

## Recommendation
Consider to implement refund mechanism or change condition to:

```diff
--  if (msg.value < price + totalFee) revert InsufficientPayment();
++  if (msg.value != price + totalFee) revert InsufficientPayment();
```

# [L-03] `buyCurvesTokenWithName` function lack of `onlyTokenSubject` modifier.

## Description
There is a condition in `buyCurvesTokenWithName` function which means that supply should be equal to zero when it called:

```solidity
if (supply != 0) revert CurveAlreadyExists();
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L371


Then `_buyCurvesToken` function is called:

```solidity
_buyCurvesToken(curvesTokenSubject, amount);
```

inside this function there is a condition:

```solidity
if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();
```

This condition means that it only possible to call this function if `curvesTokenSubject == msg.sender`.

## Recommendation
Consider to add `onlyTokenSubject` modifier to `buyCurvesTokenWithName` function.


# [L-04] `onBalanceChange` push same token into array more than once.

## Description

`onBalanceChange` function pushes token into array despite it's already have been added.

```solidity
function onBalanceChange(address token, address account) public onlyManager {
    TokenData storage data = tokensData[token];
    data.userFeeOffset[account] = data.cumulativeFeePerToken;
    if (balanceOf(token, account) > 0) userTokens[account].push(token);
}
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L99

## Recommendation
Cosnider to use EnumerableSet by OpenZeppelin or check if token already in the array.