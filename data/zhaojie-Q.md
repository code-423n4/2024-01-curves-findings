
## [L1] token creation may not be possible using contract accounts/abstract accounts

When the user purchases, the token owner receives eth _transferFees and sends eth to the owner using a low level call.
To create a token, creator need to purchase one token and the creator will also receive ETH.
The problem is that if the creator is a contract account and does not implement the fallback function, the ETH cannot be accepted and therefore the token cannot be created.
Some abstract accounts will use contract implementation, if these abstract accounts do not implement fallback, they will not receive ETH, so these abstract accounts will not be able to create tokens.

## [L2] If the `fee percent` or price is too high, it will lead to arithmetic overflow

Use price * FeePercent when `getFees`, for example `protocolFee`:

```
protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
```

The problem is that `price` and `protocolFeePercent` are variables of type `uint256`. Arithmetic overflow occurs if their values are large.

It is therefore recommended that the variable types for price and fee percent be defined in a range where no overflow will occur


## [L3] zksync does not support transfer/send functions
Curves will be deployed on some Layer 2 networks. In addition, `Feesplit#claimFees` send ETH(gas token) using the `transfer` function.
zksync is a popular ZKP-based Layer 2 network. However, transfer/send is not supported on this network. If the Curves protocol needs to be deployed on this network, the sending mode of gas token (ETH) need to changed.

## [L4] Contract accounts may not be able to sell tokens
When selling tokens, _transferFees send ETH to the caller (msg.sender) using a low level call.
But the problem is that if the caller is a contract account and the fallback function is not implemented, the ETH will fail to send, resulting in the user being unable to sell the token.
Therefore, it is recommended that _transferFees send ETH in other ways.
