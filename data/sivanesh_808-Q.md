

| **Section ID** | **Issue Description**                                 | **Function Name**           |
|----------------|-------------------------------------------------------|-----------------------------|
| L-01           | Inaccurate Fee Distribution Due to Division Rounding  | `getFees`                   |
| L-02           | Incorrect Withdrawal Amount in `withdraw` Function    | `withdraw`                  |
| L-03           | Potential Token Supply Inconsistency                  | `sellCurvesToken`           |
| L-04           | Insufficient Balance Check in `transferCurvesToken`   | `transferCurvesToken`       |
| L-05           | Miscalculation of Buy Price in `getBuyPrice` Function | `getBuyPrice`               |
| L-06           | Flawed Fee Calculation Logic                          | `getFees`, `setMaxFeePercent` |

### [L-01] Inaccurate Fee Distribution Due to Division Rounding in `getFees`


### Contract : Curves.sol

### Description:
The contract's `getFees` function calculates various fees based on transaction price using integer division, which can result in rounding down of fractional amounts. Since Solidity does not handle fractional numbers and rounds down the results of integer divisions, this can lead to minor discrepancies in the fee distribution. Over multiple transactions, these discrepancies can accumulate, potentially leading to a significant undistributed amount of Ether.

### Code Snippet:
```solidity
function getFees(
    uint256 price
)
    public
    view
    returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
{
    protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
    subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
    referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
    holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
    totalFee = protocolFee + subjectFee + referralFee + holdersFee;
}
```

### Detailed Calculation Example for Inaccurate Fee Distribution in `getFees` Function

- Protocol Fee Percent: 2.3%
- Subject Fee Percent: 1.7%
- Referral Fee Percent: 1.1%
- Holders Fee Percent: 0.4%

### Calculations:
1. **Protocol Fee**:
   - Percent: 2.3%
   - Ideal Calculation: `123 wei * 2.3 / 100 = 2.829 wei`
   - Solidity Calculation: Truncates to `2 wei`
   - Loss: `0.829 wei`

2. **Subject Fee**:
   - Percent: 1.7%
   - Ideal Calculation: `123 wei * 1.7 / 100 = 2.091 wei`
   - Solidity Calculation: Truncates to `2 wei`
   - Loss: `0.091 wei`

3. **Referral Fee**:
   - Percent: 1.1%
   - Ideal Calculation: `123 wei * 1.1 / 100 = 1.353 wei`
   - Solidity Calculation: Truncates to `1 wei`
   - Loss: `0.353 wei`

4. **Holders Fee**:
   - Percent: 0.4%
   - Ideal Calculation: `123 wei * 0.4 / 100 = 0.492 wei`
   - Solidity Calculation: Truncates to `0 wei`
   - Loss: `0.492 wei`

### Total Loss Calculation:
- Total Ideal Fees: `2.829 wei + 2.091 wei + 1.353 wei + 0.492 wei = 6.765 wei`
- Total Fees as per Solidity: `2 wei + 2 wei + 1 wei + 0 wei = 5 wei`
- Total Loss: `6.765 wei - 5 wei = 1.765 wei`

### Impact:
In this single transaction, the contract loses `1.765 wei` due to rounding. While this seems negligible, in a high-volume contract with thousands or millions of transactions, the cumulative loss could become significant. This loss represents funds that are either unintentionally retained within the contract or not properly distributed to the intended parties, potentially leading to discrepancies and trust issues over time.



----------------------------------------------------------------------------------------------------

### [L-02]: Incorrect Withdrawal Amount in `withdraw` Function

### Contract : Curves.sol

### Description:
The contract's `withdraw` function allows users to exchange their internal token balance for ERC20 tokens. However, the function directly uses the `amount` parameter (intended to represent ERC20 tokens) as the amount for internal token balance deduction. Given that ERC20 tokens and internal tokens might not have a 1:1 value ratio, this could lead to incorrect amounts being withdrawn, potentially causing fund loss for users.

### Code Snippet:
```solidity
function withdraw(address curvesTokenSubject, uint256 amount) public {
    if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

    address externalToken = externalCurvesTokens[curvesTokenSubject].token;
    ...
    _transfer(curvesTokenSubject, msg.sender, address(this), amount);
    CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
}
```

### Detailed Calculation Example:
Let's break down the issue in the `withdraw` function with a detailed, step-by-step example to clarify how it can lead to a loss of funds.

### Initial Assumptions for Clarity:

1. **Conversion Ratio**: 1 Internal Token = 0.1 ERC20 Tokens.
2. **User's Initial Balances**:
   - Internal Tokens: 1000.
   - ERC20 Tokens: 0.

### User's Withdrawal Request:

- The user requests to withdraw ERC20 tokens equivalent to 50 Internal Tokens.

### Correct Calculation (Ideal Scenario):

- **Amount of ERC20 Tokens to be Received**: Since 1 Internal Token equals 0.1 ERC20 Tokens, for 50 Internal Tokens, the user should receive `50 * 0.1 = 5 ERC20 Tokens`.
- **Reduction in Internal Tokens**: To get 5 ERC20 Tokens, 50 Internal Tokens should be deducted from the user's balance.

### Flawed Execution in Current Contract Logic:

- **ERC20 Tokens Minted**: The contract correctly mints 5 ERC20 Tokens for the user.
- **Deduction of Internal Tokens**: Here's where the issue arises. Instead of deducting 50 Internal Tokens (the correct amount), the contract deducts only 5 Internal Tokens, mirroring the amount of ERC20 Tokens.

### Balances After the Flawed Execution:

- **User's Internal Token Balance**: Initially 1000, only 5 are deducted, resulting in 995 remaining (instead of the correct 950).
- **User's ERC20 Token Balance**: Increases by 5, as intended.

### Detailed Loss Analysis:

- **Before Withdrawal**: The user has 1000 Internal Tokens and 0 ERC20 Tokens.
- **After Withdrawal**: The user should have 950 Internal Tokens and 5 ERC20 Tokens. However, due to the flaw, they have 995 Internal Tokens and 5 ERC20 Tokens.
- **Result of the Flaw**: The user retains an extra 45 Internal Tokens (995 - 950) due to incorrect deduction.
  
### Impact:

This issue leads to a situation where users withdraw more value in ERC20 Tokens than they are giving up in Internal Tokens. Over multiple transactions, this could lead to a significant depletion of ERC20 Tokens in the contract, essentially giving away more assets than intended and causing a loss of funds.



--------------------------------------------------------------------------------------------------------------

### [L-03]: Potential Token Supply Inconsistency in `sellCurvesToken` Function

### Contract : Curves.sol

### Description:
The `sellCurvesToken` function is designed to allow users to sell their Curves tokens. However, there's a potential issue with how the function updates the total supply of tokens. The function first checks if the total supply is less than or equal to the amount being sold, then reverts with `LastTokenCannotBeSold`. But it doesn't account for scenarios where multiple simultaneous transactions could lead to a total supply being reduced to zero or a negative value, potentially causing inconsistencies in the token supply.

### Code Snippet:
```solidity
function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply <= amount) revert LastTokenCannotBeSold();
    ...
}
```

### Issue Detail:
The issue arises from the lack of synchronization when multiple `sellCurvesToken` calls are made in parallel. If two or more transactions try to sell tokens at the same time, and their combined amount equals or exceeds the total supply, it could lead to the supply going negative. This is because the check for `supply <= amount` happens before the actual deduction from the supply, allowing simultaneous transactions to pass this check.

### Potential Consequences:
- **Token Supply Underflow**: Multiple simultaneous `sell` transactions could lead to the token supply becoming negative.
- **Accounting Inconsistencies**: Negative token supply can lead to accounting issues, making it difficult to track the actual number of tokens in circulation.



--------------------------------------------------------------------------------------------------



### [L-04] Insufficient Balance Check After State Modification in `transferCurvesToken`

### Contract : Curves.sol

### Description:
The contract's `transferCurvesToken` function allows users to transfer tokens to another address. However, the function first allows for a state modification (token transfer) before performing an adequate balance check. This sequence of operations could lead to a situation where users transfer more tokens than they have, potentially causing negative balances and inconsistencies.

### Code Snippet:
```solidity
function transferCurvesToken(address curvesTokenSubject, address to, uint256 amount) external {
    if (to == address(this)) revert ContractCannotReceiveTransfer();
    _transfer(curvesTokenSubject, msg.sender, to, amount);
}

function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
    if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
    ...
}
```

### Expected Behavior:
The contract should first verify that the sender (`from`) has enough tokens to cover the transfer amount (`amount`). Only after this check passes should it proceed to transfer tokens from the sender to the recipient (`to`). This ensures that no user can transfer more tokens than they own.

### Actual Behavior:
The current implementation performs the transfer operation before it conducts the sufficient balance check in the `_transfer` function. This sequence could allow a user to initiate a transfer of tokens they do not own, potentially leading to negative balances in the sender's account.




--------------------------------------------------------------------------------

### [L-05] Miscalculation of Buy Price in `getBuyPrice` Function

### Contract : Curves.sol

### Description:
The `getBuyPrice` function in the contract is responsible for calculating the price for users to buy Curves tokens. However, there appears to be a potential miscalculation in how the price is derived, particularly in the formula used for price calculation. This miscalculation could lead to users being charged an incorrect amount for their token purchases, potentially leading to either an overcharge or undercharge, and consequently, a loss of funds for either the users or the contract.

### Code Snippet:
```solidity
function getBuyPrice(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
    return getPrice(curvesTokenSupply[curvesTokenSubject], amount);
}

function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}
```

### Expected Behavior:
The expected behavior of the `getBuyPrice` function is to accurately calculate the cost of purchasing a specified amount of Curves tokens based on the current supply. The calculation should correctly reflect the intended pricing formula, ensuring that users pay a fair and accurate price for their token purchases.

### Actual Behavior:
The actual behavior might involve a miscalculation due to the complexity of the formula used in the `getPrice` function. Particularly, the formula involving `sum1` and `sum2` might not correctly calculate the incremental price based on the token supply and amount being purchased. This could result in users being charged too much or too little for their purchases, depending on how the miscalculation manifests.


### Scenario for Calculation:

1. **Initial Supply of Tokens**: Assume the current supply of Curves tokens is 500.
2. **User's Purchase Amount**: The user wants to buy 50 tokens.
3. **Intended Pricing Strategy**: Assume the price per token should increase linearly with supply. For simplicity, let's say the price starts at 1 wei per token at 500 tokens and increases by 0.1 wei for each additional token.

### Expected Price Calculation (Linear):

- Price for the 501st token: 1.1 wei
- Price for the 502nd token: 1.2 wei
- ...
- Price for the 550th token: 5.0 wei
- Total expected price for 50 tokens: Sum of prices from 501st to 550th token.

### Actual Price Calculation According to the Contract:

Using the `getPrice` function in the contract:
```solidity
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    ...
    return (summation * 1 ether) / 16000;
}
```

- **Step 1: Calculate `sum1` (Initial Sum)**
  - `sum1` for a supply of 500 tokens.

- **Step 2: Calculate `sum2` (Sum after Purchase)**
  - `sum2` for a supply of 550 tokens (500 initial + 50 to be purchased).

- **Step 3: Price Calculation**
  - The price for 50 tokens is calculated by `((sum2 - sum1) * 1 ether) / 16000`.

### Calculation Outcome and Potential Discrepancies:

- **Expected Outcome**: If the price per token increases linearly, the total expected price for 50 tokens can be calculated as the sum of an arithmetic series. This could be a specific value, let's say `X wei`.
- **Actual Outcome**: The contract’s calculation might result in a different amount, say `Y wei`, due to the complex formula used in `getPrice`.

### Potential Scenarios:

- **Overcharge Scenario**: If `Y > X`, i.e., the contract's calculated price is higher than the expected linear price, users would be overcharged.
- **Undercharge Scenario**: If `Y < X`, i.e., the contract's calculated price is lower than the expected linear price, the contract would be selling tokens at a lower price, leading to revenue loss.

### Specific Example:

Let's say the expected price `X` for 50 tokens is `275 wei` (as per the linear increase), but the contract’s formula calculates `Y` as `300 wei`. In this case, the user is overcharged by `25 wei`. This discrepancy, when magnified over numerous transactions, could lead to significant losses for users or the contract.

### Impact:
A miscalculation in the buy price can lead to significant financial implications:
- **For Users**: They might be overcharged for their purchases, leading to a loss of funds.
- **For the Contract/Platform**: An undercharge would mean selling tokens at a price lower than intended, potentially leading to a loss of revenue or devaluation of the token's worth.


----------------------------------------------------------------------------------------------------------------------------------------------

### [L-06] Flawed Fee Calculation Logic in Curves Smart Contract

### Contract : Curves.sol

### Description:
A critical bug has been identified in the Curves smart contract, specifically within the logic that calculates and distributes transaction fees. This flaw lies in the possibility of setting fee percentages in a way that their combined sum exceeds 100%, leading to overcharging and misappropriation of funds.

### Code Snippet:
The bug is present in the following functions: `getFees` and `setMaxFeePercent`.

```solidity
function getFees(uint256 price) public view returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee) {
    protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
    subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
    referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
    holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
    totalFee = protocolFee + subjectFee + referralFee + holdersFee;
}

function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
    if (
        feesEconomics.protocolFeePercent +
        feesEconomics.subjectFeePercent +
        feesEconomics.referralFeePercent +
        feesEconomics.holdersFeePercent >
        maxFeePercent_
    ) revert InvalidFeeDefinition();
    feesEconomics.maxFeePercent = maxFeePercent_;
}
```

### Exact Bug Explanation:
1. **Faulty Total Fee Calculation**: The total fee is calculated as the sum of individual fees (`protocolFee`, `subjectFee`, `referralFee`, `holdersFee`). However, there's no inherent mechanism to ensure that the total does not exceed 100% of the transaction value.

2. **Inadequate Validation in `setMaxFeePercent`**: The function `setMaxFeePercent` checks if the sum of the fees is greater than `maxFeePercent_` but doesn't ensure that `maxFeePercent_` itself is within a logical bound (i.e., not exceeding 100%).

#### How the Bug Can Lead to Fund Loss:
- **Overcharging**: If the combined fee percentages exceed 100%, the total fee calculated will be more than the transaction amount, leading to overcharging users.
- **Fund Misappropriation**: Excessive fees can drain the contract balance or user funds, effectively transferring wealth from users to fee recipients in an unintended manner.

----------------------------------------------------------------------------------------------------------------------------------------------
