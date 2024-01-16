## [L-01]

## Title
Lack of a double step transferownership pattern

## Links
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L27-L29

## Impact
The `Security::transferOwnership` function is used to change the address of owner. But this is done in one step. If the new address is not correct the ownership will be lost. It is a best practice to transfer the ownership in two-step procedure.

## Recommendation
It is recommended to implement a two-step process where the owner nominates an account and the nominated account needs to call an `acceptOwnership()` function for the transfer of the ownership to fully succeed. This can be easily achieved by using `OpenZeppelin’s Ownable2Step` contract.


## [L-02]

## Title
The size of `Curves` contract is close to the contract size limit on Optimism Superchain

## Links
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L41

## Impact
The size limit for smart contracts deployed on the Optimism platform, is 24KB (24,576 bytes). This limit is set to prevent denial-of-service (DoS) attacks. The size of the `Curves` contract is `23.046` which is very close to the defined size limit. The size is retrieved by the command: `npx hardhat size-contracts`. If the contract size is larger than the size limit, the contract can't be deployed.

## Recommendation
Optimize the size of the contract and take the size into account when modify or add new functionality to the contract.


## [L-03]

## Title
Precision loss in `fees` in `FeeSplitter` contract

## Links
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L73-L78

## Impact
Let's consider a scenario where a user has a very small balance relative to the `PRECISION` factor used in the `FeeSplitter` contract. Due to the way the contract calculates the `owed` fees in `FeeSplitter::getClaimableFees`, if the user's balance is too small, the multiplication by their balance might not be enough to reach the `PRECISION` threshold, resulting in a division by `PRECISION` that yields zero.

```javascript
    function getClaimableFees(address token, address account) public view returns (uint256) {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
        uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
@>      return (owed / PRECISION) + data.unclaimedFees[account];
    }

```
Let's say a user has an unclaimedFees balance of `100 wei` from previous transactions. In a new transaction, the user is supposed to receive an additional `0.5 wei` of fees (which is less than `PRECISION`). The calculation of new `owed fees` would be `0.5 wei / 1e18`, which results in zero due to rounding. However, the user's total claimable fees would still be `0 + 100 wei = 100 wei`. So, while the user wouldn't lose their previously unclaimed fees, they would not be able to claim the additional `0.5 wei` due to the rounding down to zero in the division by `PRECISION`.

## [L-04]

## Title
The length of the `name` and `symbol` on `CurvesERC20` token can be arbitrary

## Links
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L364-L375
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L428-L437

Functions `Curves::buyCurvesTokenWithName` and `Curves::setNameAndSymbol` which receive as arguments and set the `name` and `symbol` for the new deployed `CurvesERC20` token do not check if their length is not too long. That can lead to problems with deploing the tokens or inconsistencies between the tokens (every token can have different length of `name` and `symbol`).
Add a restriction of the length of the `name` and `symbol` for the `CurvesERC20` tokens.

## [L-05]

## Title
Missing zero check for `merkleRoot`

## Links
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394

In the `Curves::setWhitelist` the input parameter `merkleRoot` is not checked if it is not `0`. It is assigned without any check:

```javascript

function setWhitelist(bytes32 merkleRoot) external {
        uint256 supply = curvesTokenSupply[msg.sender];
        if (supply > 1) revert CurveAlreadyExists();

        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
@>          presalesMeta[msg.sender].merkleRoot = merkleRoot;
            emit WhitelistUpdated(msg.sender, merkleRoot);
        }
    }

```