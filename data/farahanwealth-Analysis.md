## **Analysis Report for Curves.sol**

### Overview:
The provided Solidity contract appears to be a part of a token ecosystem that involves the creation and management of custom tokens (referred to as CURVES tokens) and a decentralized exchange mechanism for trading these tokens. The contract is relatively complex and incorporates various features, including token creation, presale functionalities, fee distribution, and more.

### High-Level Observations:
1. **Contract Architecture:**
   - The contract follows the SPDX-License-Identifier standard and uses Solidity version 0.8.7.
   - It imports several OpenZeppelin contracts, indicating reliance on established libraries.
   - The contract extends another contract called `Security`.

2. **Structs and Enums:**
   - Several structs (`ExternalTokenMeta`, `PresaleMeta`, `FeesEconomics`, and `UserClaimData`) are defined to organize data.
   - An enum `CurvesErrors` is declared for custom error handling.

3. **State Variables:**
   - The contract defines various state variables to store information about tokens, presales, fees, etc.
   - `curvesERC20Factory` and `feeRedistributor` are critical addresses initialized in the constructor.

4. **Modifiers and Events:**
   - Modifiers such as `onlyTokenSubject` and `onlyManager` are used to restrict access.
   - Several events, including `Trade`, `Transfer`, and `WhitelistUpdated`, are emitted.

5. **Functions:**
   - The contract contains numerous functions for setting parameters, buying/selling tokens, fee management, and more.
   - Notable functions include `buyCurvesToken`, `sellCurvesToken`, `transferCurvesToken`, `buyCurvesTokenForPresale`, and `setWhitelist`.

### Identified Issues:

1. **Fallback Function:**
   - The contract lacks a fallback function, and it might be beneficial to include a payable fallback function for handling unexpected Ether transfers.
   - Recommendation: Implement a payable fallback function with proper handling and logging.

2. **Potential Reentrancy Vulnerability:**
   - The contract uses external calls (e.g., `call`, `transfer`) within loops and external functions, which may expose it to reentrancy attacks.
   - Recommendation: Use the reentrancyGuard pattern or ensure that state changes precede external calls.

3. **Presale Mechanics:**
   - The `buyCurvesTokenForPresale` function seems to lack proper validation of presale conditions, potentially leading to undesired token purchases.
   - Recommendation: Review and enhance the presale-related logic to ensure security.

4. **Gas Limit Issues:**
   - Gas usage can be unpredictable, especially in functions involving loops or external contract interactions.
   - Recommendation: Perform gas testing to ensure the contract remains within reasonable gas limits under various conditions.

### Conclusion:
The contract appears to provide a comprehensive set of functionalities for managing custom tokens and supporting presales.

## **Analysis Report for FeeSplitter.sol**

### Overview:
The provided Solidity contract, `FeeSplitter.sol`, appears to be a component of a larger system for fee distribution among token holders. It is designed to work with the `Curves` contract and employs a mechanism for claiming and distributing fees among users holding various tokens.

### High-Level Observations:

1. **Inheritance and Dependencies:**
   - The contract inherits from `Security`, indicating a dependency on security-related functionalities.
   - It imports the `Curves.sol` contract and relies on it for various token-related operations.

2. **State Variables:**
   - `curves`: A state variable representing the instance of the `Curves` contract, indicating the integration between the two contracts.
   - `PRECISION`: A constant representing the precision factor used in fee calculations.

3. **Structs and Enums:**
   - Two structs (`TokenData` and `UserClaimData`) are defined to organize and store fee-related data.

4. **Mapping and Event:**
   - `tokensData`: A mapping storing information about cumulative fees, user offsets, and unclaimed fees for each token.
   - `userTokens`: A mapping associating users with the tokens they hold.
   - `FeesClaimed` event: Emitted when a user claims fees.

5. **Functions:**
   - `setCurves`: Allows setting the `Curves` contract address.
   - `balanceOf`, `totalSupply`: View functions providing information about token balances and total supplies.
   - `getUserTokens`, `getUserTokensAndClaimable`: Functions for retrieving user-held tokens and associated claimable fees.
   - `updateFeeCredit`: Internal function to update user fee credits.
   - `getClaimableFees`: View function to calculate claimable fees for a user.
   - `claimFees`: Allows users to claim their fees.
   - `addFees`: Allows the addition of fees to the contract.
   - `onBalanceChange`: Called when a user's balance changes, updating user fee offsets.
   - `batchClaiming`: Allows batch claiming of fees for multiple tokens.

6. **Fallback Function:**
   - The contract includes a receive function to handle incoming Ether transfers.

### Identified Issues:

1. **Potential Gas Limit Issues:**
   - The `batchClaiming` function could face gas limit issues when processing a large number of tokens, resulting in transactions failing.
   - Recommendation: Consider batch processing with a limit or optimize the function to handle a larger number of tokens efficiently.

2. **Fallback Function Handling:**
   - The contract handles Ether transfers in the `receive` function, but there is no explicit handling of unexpected Ether transfers in the fallback function.
   - Recommendation: Implement a payable fallback function with proper handling and logging.

### Conclusion:
The `FeeSplitter.sol` contract plays a crucial role in managing fee distribution among token holders, particularly in conjunction with the `Curves` contract. While the contract design is generally sound, addressing the identified issues is recommended to enhance the contract's robustness and usability.



### Time spent:
12 hours