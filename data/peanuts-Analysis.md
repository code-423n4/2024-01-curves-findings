### Summary and Overview of the protocol

This code is a fork of friend.tech with added functionalities.

There are three types of users

1. Protocol owner
2. Curve token subject
3. Normal users

- A user can buy another subject's share. As more and more shares are bought from the same subject, the price of the share will increase.
- A user can also buy his own share.
- When a share is bought, the user has to pay four types of fees: the protocol fee, the subject fee, the referral fee and the holder's fee.

Protocol fee: Paid to the protocol
Subject fee: Paid to the subject
Referral fee: Paid to the referral of the subject (The subject can set the referral address)
Holders fee: Paid to those that holds the curve token bought from a subject  

- When the share is sold, the user also has to pay four types of fees.
- Once a curve token is bought, it can be converted to an ERC20 token. The token can then be traded on external market places and sold there.

- There is also a presale option, where the user can set a whitelist to allow whitelisted users to buy their shares first, before other users are allowed to buy

- Note that curve token does not mean an actual ERC20 token. It refers to a share. There is a function to convert the curve token to an ERC20 token and back
- Note that anyone's shares can be bought. No one needs to accumulate shares before selling them.
- Note that the first share must be bought by the subject himself.

### Codebase quality analysis and comments

In this section, I will be laying out all the functions that I have checked. If the contract name has been mentioned, that means the modifiers, constructors, and inheritance are also checked as well, even if its not written down explicitly.

The mechanisms of each contract will be discussed in the next section.

##### Curves.sol

| Contract | Function                     | Explanation                                                             | Comments                                                                             |
| -------- | ---------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Curves   | setFeeRedistributor          | Set the destination of FeeSplitter                                      | onlyOwner, can be changed                                                            |
| Curves   | setMaxFeePercent             | Sets the max fee percentage                                             | Should have upper limit of 50% (max 50% fee), otherwise can charge over 100% fees    |
| Curves   | setProtocolFeePercent        | Sets the fee percentage of protocol (subject,referral,protocol,holders) | onlyOwner, Have an upper limit also with all the fee percentage                      |
| Curves   | setExternalFeePercent        | Sets the different fee percentage (subject,referral,holders)            | onlyManager, Make sure address(this) cannot be used in any capacity                  |
| Curves   | setReferralFeeDestination    | Sets the referral fee address                                           | onlySubject, can cause potential DoS by not allowing users to sell                   |
| Curves   | setERC20Factory              | Sets the ERC20 Factory address                                          | onlyOwner                                                                            |
| Curves   | getFees                      | Get the total fees payable                                              | Denominator is 1e18                                                                  |
| Curves   | getPrice                     | Get the price of one curve token                                        | Same calculation as friend.tech                                                      |
| Curves   | getBuyPrice                  | Gets the buy price of one curve token from a subject                    | -                                                                                    |
| Curves   | getSellPrice                 | Gets the sell price of one curve token from a subject                   | -                                                                                    |
| Curves   | getBuyPriceAfterFee          | Gets the buy price of one curve token from a subject after fees         | -                                                                                    |
| Curves   | getSellPriceAfterFee         | Gets the sell price of one curve token from a subject before fees       | -                                                                                    |
| Curves   | buyCurvesToken               | Purchase a curve token                                                  | This is the user's start point                                                       |
| Curves   | \_transferFees               | Transfer fees to different addresses                                    | Updates the cumulativeFeePerToken in FeeSplitter                                     |
| Curves   | \_buyCurvesToken             | Buys a curve token                                                      | Checks that msg.value is more than price + fee, but does not refund excess msg.value |
| Curves   | sellCurvesToken              | Sells a curve token                                                     | Can potentially be frontrunned                                                       |
| Curves   | transferCurvesToken          | Transfer curve tokens to another address                                | only can transfer own tokens                                                         |
| Curves   | transferAllCurvesTokens      | Transfer all curve tokens to another address                            | only can transfer own tokens capacity                                                |
| Curves   | \_transfer                   | Transfer curve tokens to another address                                | Use to transfer curve tokens, and for tokenization                                   |
| Curves   | \_addOwnedCurvesTokenSubject | For the normal user, adds a curvesTokenSubject to a list                | Will it reach OOG if too many curvesTokenSubject is added?                           |
| Curves   | \_deployERC20                | Deploys an ERC20 contract using a Factory                               | Will it be too confusing? So many ERC20 contract with same name+symbol               |
| Curves   | buyCurvesTokenWithName       | Similar to mint() but can set the name                                  | Only the curveTokenSubject can use                                                   |
| Curves   | buyCurvesTokenForPresale     | The curves token owner buys the first token                             | Only curve token owner can use, sets the first supply.                               |
| Curves   | setWhitelist                 | Sets a whitelist for the presale                                        | Only the curvesToken owner can change, merkleRoot can set to 0?                      |
| Curves   | buyCurvesTokenWhitelisted    | Buy the curve token, only for whitelisted users                         | Anyone verified in the merkle proof can buy token                                    |
| Curves   | verifyMerkle                 | Verify whether the name is whitelisted or not                           | Uses OZ MerkleProof, the leaf is not 64 bytes long                                   |
| Curves   | setNameAndSymbol             | Sets name and symbol of the token                                       | Not really sure if this works because the token must not be created yet              |
| Curves   | mint                         | Creates the ERC20 version of the curve token                            | Only the token subject can call, use DEFAULT names and symbol                        |
| Curves   | withdraw                     | Converts the curve token to a ERC20 token                               | Anyone can use, provided they have the curve token in the first place                |
| Curves   | deposit                      | Converts the ERC20 token to a curve token                               | Anyone can use, provided they have the ERC20 token in the first place                |
| Curves   | sellExternalCurvesToken      | Converts the ERC20 token to a curve token and sells the curve token     | A QoL function that combines deposit() and sellCurvesToken()                         |

##### FeeSplitter.sol

| Contract    | Function                  | Explanation                                                                | Comments                                                               |
| ----------- | ------------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| FeeSplitter | setCurves                 | Set the Curves contract                                                    | Anyone can set it?                                                     |
| FeeSplitter | balanceOf                 | Returns the curve token balance of a subject of an account                 | Multiplied by 1e18, to follow ERC20 18decimals format                  |
| FeeSplitter | totalSupply               | Returns the number of curve tokens bought from a subject                   | Only counts those not tokenized                                        |
| FeeSplitter | getUserTokens             | Gets all the curve token subjects invested from a user                     | Function name is quite misleading                                      |
| FeeSplitter | getUserTokensAndClaimable | Gets all the curve token subjects invested from a user and claims all fees | -                                                                      |
| FeeSplitter | updateFeeCredit           | Updates the userFeeOffset of an account                                    | Called from claimFees(). Make sure token gets the correct holder's fee |
| FeeSplitter | getClaimableFees          | Gets the claimable fee of the curve token subject of a user                | -                                                                      |
| FeeSplitter | claimFees                 | Claims the appropriate amount of fees                                      | unclaimedFees is set to zero. Claim all or zero fees.                  |
| FeeSplitter | addFees                   | Add fees into the contract                                                 | Increases cumulativeFeePerToken, called from Curves.sol                |
| FeeSplitter | onBalanceChange           | Sets the userFeeOffset when balance is changed                             | userToken is updated, should be called everytime a new token is issued |
| FeeSplitter | batchClaiming             | Claims fees in batches                                                     | Same as claim fees, but with an array                                  |

##### Security.sol

(Table is not needed as contract is small)

- As pointed out by the Bot, the modifiers needs a require statement, otherwise the modifier will not work as intended.
- There can only be one owner but there can be many managers, assuming the onlyManager modifier works correctly.
- The ownership transfer should be a two step process.
- There should be a zero-address check for transferring ownership.

##### CurvesERC20.sol

(Table is not needed as contract is small)

- Only the owner can mint or burn the CurvesERC20 tokens. - Owner is Curves.sol
- There is no max mint limit on the number of ERC20 tokens. - Curves.sol controls the limit
- Owner can burn tokens from any account. - This function cannot work independently.

### Mechanism Review

##### Curves.sol

`Buying a curve token`

- Similar to friends.tech, except with 2 more fees - holders and referral.
- Referral address can be set by the curve token subject himself, open to DoS.
- `_addOwnedCurvesTokenSubject()` can potentially run out of gas if a user buys from too many subject.
- The remainder of `msg.value` is not transferred back to the owner.
- This mechanism is similar to friends.tech.

`Selling a curve token`

- Similar to friends.tech, except with 2 more fees - holders and referral.
- Users selling can get frontrunned by a large whale, netting him a lesser amount than intended. Setting a deadline and slippage would be ideal.
- This mechanism is similar to friends.tech.

`Whitelisting`

- merkleRoot is not checked for 0 address
- merkleRoot cannot be changed because once `buyCurvesTokenForPresale()` is called, `setWhitelist` cannot be called since supply > 1.
- Owner is limited to `maxBuy`.
- This is optional, the subject can choose not to whitelist/conduct a presale.
- This mechanism allows for presale. Take note that the merkleRoot cannot be changed and the owner is limited to `maxBuy`.

`Tokenizing a curve share`

- Denomination is changed from 1 to 1e18.
- The token is now ERC20 compatible.
- Anyone can create a tokenized share by calling withdraw(). This disallows the original subject from creating their own name and symbol for the token.
- A tokenized share is not eligible for the holder's fee as it is not counted as part of the total supply in the FeeSplitter contract.
- The Curves address 'locks' the shares in the contract.
- This mechanism makes share trading flexible. Take note of changing the names and symbols. Make sure the ERC20 displayed name and the Curves.sol displayed name is the same.

`UnTokenizing a curve share`

- Denomination is changed back to 1.
- The Curves address 'unlocks' the shares in the contract.
- This mechanism makes share trading flexible, but it will affect the holders fee calculation because data.userFeeOffset[account] is not updated when untokenizing a curve share.

`Deploying ERC20 contracts using a factory`

- Deploys a very simple ERC20 contract, where the owner is Curves.sol.
- The factory (out of scope) simply deploys a new ERC20 contract.

##### FeeSplitter.sol

`Assigning and Claiming Fees`

- `addFees()` is called in Curves.sol. `data.cumulativeFeePerToken` is always increasing.
- Note that the supply only refers to untokenized shares. This can be confusing because the tokenized shares which are not part of the supply can become untokenized and vice versa, which makes accounting challenging.
- Some users can have `data.userFeeOffset[account] == 0` if they trade the ERC20 shares. Make sure that the userFeeOffset is always updated properly
- Fees are always claimed fully, no partial claiming allowed
- Use of payable.transfer can be dangerous due to the gas limit.
- `batchClaiming` is a bonus function for people that invest in many curve token subject.
- The fee distribution is fair, those with more tokens will earn a larger fee. Those who have a token early will also earn more fees.
- This mechanism gives an added incentive to those that buy shares. Take note that the fees in the contract can be stolen because `data.userFeeOffset[account] == 0` when a user untokenize his share.

### Centralization Risks


- Owner controls how much fees percentage to put. Can potentially put 100% or more as the protocol fee percentage
- Owner also controls the manager (which can be himself)
- Owner controls the change of contracts

Overall, the protocol has a high centralization risk because the owner takes cares of most of the sensitive function. The only exception is that the owner cannot withdraw funds from the main contract.

### Architecture Review

##### `Design Patterns and Best Practices:`

Established design patterns, like modifiers are not used properly. There is not many checks for zero values and upper bounds limits for sensitive functions. Usage of payable.transfer is apparent as well, which is not ideal (using low-level call is better)

Good usage of errors. Clear and concise error messages.

##### `Code Readability and Maintainability:`

Functions are understandable, code uses simple structure which is good. Comments are lacklustre; more comments for every function is welcomed to facilitating understanding. Code is compact as well, which can be easily maintained.

##### `Error Handling and Input Validation:`

Events and Error messages are easy to understand. Error messages are also written neatly at the top of the contract. Error messages are pretty intuitive.

##### `Interoperability and Standards Compliance:`

Good knowledge of the ERC20 standard and Factory usage. Something to consider is that all the ERC20 tokens will have the same name and symbol, it's hard to differentiate which contract points to which curve token subject.

##### `Testing and Documentation:`

Test has 100% coverage. There is no proper documentation, nor website. One good thing is that the medium articles are extremely informative, and since protocol is a fork of friend.tech, the design is somewhat similar. Have a good documentation (gitbook), some top-down view of the protocol, an end-to-end scenario of how a curve token subject work and how a normal user works. Also good to have a comparison with friend.tech: what is new, what is the same, what has improved.

##### `Upgradeability:`

Protocol is not meant to be upgraded at all.

##### `Dependency Management:`

Little reliance on external libraries like OpenZeppelin, which minimizes attack vectors.

Overall, architecture from the protocol needs to be improved.

- More documentation and overarching explanation of the protocol.
- More NATSPECs in the code.
- Make sure best practices are always adhered to.

### Time spent:
24 hours