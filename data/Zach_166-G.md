# 1. The `ownedCurvesTokenSubjects` map size only increases and never decreases which leads to a growing gas cost when calling the `_addOwnedCurvesTokenSubject` and `transferAllCurvesTokens` functions.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L328C1-L336C6

In the `Curves` contract, the `ownedCurvesTokenSubjects` variable is used to record the types of Curves tokens held by each user. When transferring in or buying tokens, new tokens are added by calling the `_addOwnedCurvesTokenSubject` function.

```
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
However, the `ownedCurvesTokenSubjects` map does not pop up elements when a user sells or transfers tokens. This causes the user's `ownedCurvesTokenSubjects` map to grow longer over time, increasing the gas cost of each call to `_addOwnedCurvesTokenSubject` and `transferAllCurvesTokens`. This could potentially harm the interests of users holding a significant amount of tokens.

It is recommended to implement logic to decrease the `ownedCurvesTokenSubjects` map length when a user sells or transfers tokens out.



# 2. It is unnecessary to use the `getClaimableFees` function when calling the `claimFees` or `batchClaiming` functions

In the `FeeSplitter` contract, when users call `claimFees` or `batchClaiming`, the following code is used to determine the amount of holder fees to be claimed:

```
updateFeeCredit(token, msg.sender);
uint256 claimable = getClaimableFees(token, msg.sender);
```

However, based on the function definitions:

```
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
```
The value of `data.unclaimedFees[account]` in the `updateFeeCredit` function is equivalent to the return value of the `getClaimableFees` function at that moment. Therefore, the code can be modified to reduce gas consumption as follows:
```
updateFeeCredit(token, msg.sender);
uint256 claimable = tokensData[token].unclaimedFees[msg.sender];
```