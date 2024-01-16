
# Curves::_buyCurvesToken() excess value is not returned to buyer __

The function [`Curves::_buyCurvesToken()`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263C14-L263C24) accepts a `msg.value > price`, but the excess value is not returned to the buyer, but stays in the contract. 

# Curves::transferAllCurvesTokens() unbounded gas loop, because tokens are not removed from ownedCurvesTokenSubjects array. Consider removing them when selling or transferring
The function [Curves::transferAllCurvesTokens()](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L302) has an unbounded loop, which turns into an unpredictable amount of gas. Moreover, the iterated array `ownedCurvesTokenSubjects`, does not remove elements once they are unnecessary, so it only increases with the amount of `subjectTokens` that an account has ever held in history.

## Mitigation

Consider removing the `tokenSubject` from the `ownedCurvesTokenSubjects` when the balance turns 0 (either in `Curves::sellCurvesToken()` or in `transferAllCurvesTokens()` or `transferCurvesToken()`)


# Duplicatd elements inside FeeSplitter::userTokens mapping as FeeSplitter::onBalanceChange() does not check if elements are present in the array before adding

The function `FeeSplitter::onBalanceChange()` adds the token address to the `userTokens` array if the user has positive balance, regardless if the token has already been added, [see here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L99).

## Impact:

    `FeeSplitter::getUserTokens()` will show duplicated tokens
    `FeeSplitter::getUserTokensAndClaimable()` will show duplicated claimable amounts, as tokens are duplicated

# Curves::getSellPrice() price is calculated on the last item sold. Selling multiple tokens at once gives a far worse price than selling one by one
