# Smart Contract Analysis Report for `Curves.sol` and `FeeSplitter.sol`

## Executive Summary
The `Curves` smart contract, along with the `FeeSplitter` contract, constitutes a sophisticated system for managing diverse token-related operations such as buying, selling, and transferring tokens, as well as handling fee distributions. While the contracts demonstrate intricate mechanisms and solid security practices, they also contain certain vulnerabilities and inefficiencies that could be exploited or lead to unexpected behavior.

## System Overview
- **`Curves.sol`**: Implements core functionality for token management and trading. It includes mechanisms for setting up presales, managing ERC20 tokens, and handling token economics.
- **`FeeSplitter.sol`**: Complements `Curves.sol` by managing fee distributions among token holders. It calculates and distributes fees based on token ownership.

## Risk Identification and Management
- **Duplicate Token Names/Symbols**: The `setNameAndSymbol` function in `Curves.sol` allows for the setting of duplicated names and symbols, potentially causing confusion or token misidentification.
- **Denial of Service (DoS) in `transferAllCurvesTokens`**: The `transferAllCurvesTokens` function could be susceptible to a DoS attack if the `ownedCurvesTokenSubjects[msg.sender]` array becomes very large.

## Testing and Coverage Analysis
- **Scenario-Based Testing**: Testing should cover scenarios including token presale, fee distribution, edge cases in token transfers, and potential DoS scenarios in `transferAllCurvesTokens`.
- **Adversarial Situations**: Tests need to simulate adversarial conditions such as duplicate token registration and excessive token transfer requests to ensure robustness.

## Systemic Risks and Governance
- **Fee Calculation**: The fee calculation mechanism should be thoroughly reviewed, especially for transactions involving small amounts, to ensure fees are accurately computed and collected.
- **Token Governance**: Mechanisms for token registration and management need clear governance to prevent misuse or confusion in the token ecosystem.

## Security Process and Audit Recommendations
- **Audit for Duplicate Tokens**: Conduct an audit focusing on the `setNameAndSymbol` function to ensure the prevention of duplicate tokens.
- **Stress Testing for `transferAllCurvesTokens`**: Perform stress testing for the `transferAllCurvesTokens` function to evaluate and mitigate potential DoS risks.

## Balancing Growth and Risk
- **Mitigation for Small Transaction Fees**: Implement a minimum fee threshold or adjust fee calculation logic to address fee calculation issues for small transactions.
- **Dynamic Handling in `transferAllCurvesTokens`**: Consider implementing dynamic mechanisms to handle large arrays in `transferAllCurvesTokens` to mitigate potential DoS attacks.

## Summary and Next Steps
- Address the identified issues related to token name/symbol duplication and potential DoS in `transferAllCurvesTokens`.
- Enhance fee calculation methods to ensure accuracy and fairness in all transaction sizes.
- Conduct comprehensive testing and audits to solidify the contract's security posture and functionality.


### Time spent:
25 hours