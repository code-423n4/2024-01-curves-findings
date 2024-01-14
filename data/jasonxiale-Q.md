## [L-01] `Curves.setNameAndSymbol` can be DOSed
File:
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L428-L437
In [Curves.sol#L434](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L434), when `symbolToSubject[symbol]` exists, the function will revert. So a malicious user can DOS a normal user's `curves.setNameAndSymbol` by setting the same symbol

## [L-02] `Curves._deployERC20` can be DOSed
File:
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L338-L362
In [Curves.sol#L350](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L350) when `symbolToSubject[symbol]` exists, the function will revert. So a malicious user can DOS a normal user's `curves._deployERC20` by setting the smae symbol

## [L-03] `FeeSplitter.getUserTokens` and `FeeSplitter.getUserTokensAndClaimable` can be DoSed
File:
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L48-L50
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L52-L61
Both `FeeSplitter.getUserTokens` and `FeeSplitter.getUserTokensAndClaimable` returns an array of memory, which might cause out-of-gas

## [L-04] `FeeSplitter.onBalanceChange` shouldn't push token into `FeeSplitter.userTokens` everytime if `balanceOf(token, account) > 0`
File:
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96-L100
In current [FeeSplitter.onBalanceChange](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96-L100) implementation, everytime `onBalanceChange` is called, if `balanceOf(token, account) > 0` is met, the token will be push into [userTokens](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L99C44-L99C54) array, this is not correct, if there is a lot of buy/sell for one token, there will lots of duplicated token in `userTokens` array, and this might cause Out-of-Gas issue for `FeeSplitter.getUserTokens` and `FeeSplitter.getUserTokensAndClaimable`.

## [L-05] possible reentrancy attack in `Curves._transferFees`
File
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218-L249
In [Curves._transferFees](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218-L249), when sending ETH by calling payable.call(), reentrancy attack may happens because of:
1. [firstDestination](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L232) is untrusted when the function is called by `Curves.sellCurvesToken`
1. [referralDefined](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L240-L243) is untrusted because it's set by `curvesTokenSubject` in [setReferralFeeDestination](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L155-L160)