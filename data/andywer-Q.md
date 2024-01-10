# QA Report

## Security Contract
### Lack of Event Emission upon Ownership Transfer
The `transferOwnership` function does not emit an event when ownership is transferred. This is a poor practice as it makes tracking ownership changes difficult and reduces transparency of the contract's operation.

See <https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L27-L29>

## CurvesERC20 Contract
### Lack of Event Emission on Mint Operation
The `mint` function does not emit an event. This omission makes tracking of mint operations more difficult for off-chain services, potentially leading to discrepancies in off-chain tracking systems.

See <https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/CurvesERC20.sol#L12-L14>

## FeeSplitter
### No Event Emission After Critical State Change
There's no event emission after the `curves` state variable is updated in the `setCurves` function. This omission hampers transparency and makes it difficult to track changes to the contract's points of interaction.

See <https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L35-L37>

### Potential Duplicated Entries in userTokens Array
The `onBalanceChange` function pushes the token into the userTokens array if the balance of the token on the account is greater than zero. This could result in duplicated entries if the function is called multiple times for the same token, causing unnecessary storage expansion and possible gas issues.

See <https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96-L100>

## Curves Contract
### Lack of Event Logging for setERC20Factory function
There is no event emitted when the curvesERC20Factory address is changed by the `setERC20Factory` function. The lack of event logging prevents tracking of changes to the factory address through transaction logs, which is important for transparency and security monitoring.

See <https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L162-L164>

### Unlimited max fee
There is no sanity check in `setMaxFeePercent` to make sure that the maximum fee is actually < 100% or better still less than that, like < 50%.

See <https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117-L126>
