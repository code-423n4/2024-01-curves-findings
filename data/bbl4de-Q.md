## Low

### [L-1] In `Curves::buyCurvesTokenForPresale` the `maxBuy` parameter if set to zero hard coded to max uint256 value

The user creating a token might consider using `buyCurvesTokenForPresale`, but then decide that they want `maxBuy` to be set to 0. Then, out of the user's control `maxBuy` is automatically set to `type(uint256).max` allowing anyone to buy as much tokens as possible.

User calls `buyCurvesTokenForPresale` with `maxBuy` set to 0, thinking they disallowed anyone from buying their tokens for now.
`maxBuy` is set to max uint, not working as the user expected it to work.

Explicitly forbit setting `maxBuy` to 0 by including if-revert statement or notify the user that it was set to uint256 using an event.

### [L-2] The `Curves::withdraw` function allows anyone to deploy the ERC20 for the subject token

Although `mint` and therefore `_mint` functions have the `onlyTokenSubject` modifier, the `withdraw` function allows anyone, even with balance 0 of the corresponing token to deploy an ERC20 with the default name and symbol + counter.

Due to the require statement linked below being a ">" check and not ">=", user with balance 0 of the subject token can call this function and deploy an ERC20. It won't fail in the call to `_transfer` function either, as there the if statement also checks for amount being ">" than the balance.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L466

If allowing to deploy the ERC20 token is intended, at least checking if the caller has some balance of this subject token should be considered.

## Informational

### [I-1] Consider adding a check for `amount` parameter being 0 in `Curves::deposit()`,`Curves::sellExternalCurvesToken` and other public or external functions using `amount` as parameter
