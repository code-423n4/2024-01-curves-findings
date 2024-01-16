
| **Section ID** | **Optimization Description**                         | **Function Name**                    |
|----------------|-----------------------------------------------------|--------------------------------------|
| G-01           | Optimize Calculation Logic                          | `getPrice`                           |
| G-02           | Simplify Fee Calculation                            | `getFees`                            |
| G-03           | Optimize Arithmetic in Sell Price Calculation       | `getSellPriceAfterFee`               |
| G-04           | Optimize Fee Calculation in Buy Price Function      | `getBuyPriceAfterFee`                |
| G-05           | Optimize Arithmetic in Total Supply Calculation     | `calculateTotalSupply`               |
| G-06           | Optimize Loops in Token Transfer Function           | `transferAllCurvesTokens`            |
| G-07           | Optimizing `setProtocolFeePercent` Function         | `setProtocolFeePercent` / `optimizedSetProtocolFeePercent` |

   



### [G-01] Optimize Calculation Logic in `getPrice` Function

#### Contract: Curves.sol

#### Description:
The `getPrice` function in the provided code performs a series of mathematical operations that can be optimized. Specifically, the formula involves repeated arithmetic operations that can be simplified.

#### Original Code:
```solidity
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    return (sum2 - sum1) * 1 ether / 16000;
}
```

#### Optimized Code:
```solidity
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    if (supply == 0 && amount == 1) return 0;

    uint256 supplyMinusOne = supply - 1;
    uint256 sum1 = supplyMinusOne * supply * (2 * supplyMinusOne + 1) / 6;
    uint256 newSupply = supply + amount;
    uint256 newSupplyMinusOne = newSupply - 1;
    uint256 sum2 = newSupplyMinusOne * newSupply * (2 * newSupplyMinusOne + 1) / 6;

    return (sum2 - sum1) * 1 ether / 16000;
}
```

#### Explanation:
The optimized version of `getPrice` removes redundant checks and calculations, and introduces intermediate variables to hold recurring sub-expressions, making the code more efficient and readable. 

1. The initial check for `supply == 0 && amount == 1` is moved to the top for an early return, avoiding unnecessary calculations.
2. Intermediate variables `supplyMinusOne` and `newSupplyMinusOne` are used to store the repeated sub-expressions, reducing the number of arithmetic operations required.



---------------------------------------------------------------------------------------------


### [G-02] Simplify Fee Calculation in `getFees` Function

#### Contract: Curves.sol

#### Description:
The `getFees` function involves multiple calculations that are similar in nature. These calculations can be optimized by simplifying the repetitive arithmetic operations and potentially saving gas.

#### Original Code:
```solidity
function getFees(
    uint256 price
)
    public
    view
    returns (
        uint256 protocolFee,
        uint256 subjectFee,
        uint256 referralFee,
        uint256 holdersFee,
        uint256 totalFee
    )
{
    protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
    subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
    referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
    holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
    totalFee = protocolFee + subjectFee + referralFee + holdersFee;
}
```

#### Optimized Code:
```solidity
function getFees(
    uint256 price
)
    public
    view
    returns (
        uint256 protocolFee,
        uint256 subjectFee,
        uint256 referralFee,
        uint256 holdersFee,
        uint256 totalFee
    )
{
    uint256 feeBase = price / 1 ether;
    protocolFee = feeBase * feesEconomics.protocolFeePercent;
    subjectFee = feeBase * feesEconomics.subjectFeePercent;
    referralFee = feeBase * feesEconomics.referralFeePercent;
    holdersFee = feeBase * feesEconomics.holdersFeePercent;
    totalFee = protocolFee + subjectFee + referralFee + holdersFee;
}
```

#### Explanation:
In the optimized code, the repetitive operation `(price / 1 ether)` is extracted out as a common factor `feeBase`, which is then used to calculate the individual fees. This reduces the number of division operations, which are relatively costly in terms of gas. The optimization assumes that the precision loss due to the change in order of operations is acceptable in the context of the contract's functionality. This change is likely to save gas due to the reduced number of arithmetic operations, especially when the function is called frequently.


-------------------------------------------------------------------------------------------------------------------------------------------

### [G-03] Optimize Arithmetic in `getSellPriceAfterFee` Function

#### Contract: Curves.sol

#### Description:
The `getSellPriceAfterFee` function performs calculations to determine the selling price of tokens after deducting fees. This calculation can be optimized to reduce the computational overhead and potentially save gas.

#### Original Code:
```solidity
function getSellPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
    uint256 price = getSellPrice(curvesTokenSubject, amount);
    (, , , , uint256 totalFee) = getFees(price);

    return price - totalFee;
}
```

#### Optimized Code:
```solidity
function getSellPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
    uint256 price = getSellPrice(curvesTokenSubject, amount);
    uint256 feeBase = price / 1 ether;
    uint256 totalFee = feeBase * (feesEconomics.protocolFeePercent + feesEconomics.subjectFeePercent + feesEconomics.referralFeePercent + feesEconomics.holdersFeePercent);

    return price - totalFee;
}
```

#### Explanation:
In the optimized version, the calculation of the total fee is simplified:
1. A single division operation (`feeBase = price / 1 ether`) is performed first, which is more gas-efficient than multiple divisions.
2. The total fee percentage is calculated by summing up the individual fee percentages, followed by a single multiplication with the `feeBase`. This reduces the number of multiplication and division operations.

This optimization reduces the computational complexity of the fee calculation, which can lead to lower gas costs when the function is called frequently. It's particularly effective in scenarios where the price is a significant value, as it reduces the number of arithmetic operations involved.



-----------------------------------------------------------------------------------------------------------------------------------

### [G-04] Optimize Fee Calculation in `getBuyPriceAfterFee` Function

#### Contract: Curves.sol

#### Description:
In the `getBuyPriceAfterFee` function, the original implementation calculates the total fee by calling `getFees`, which involves multiple arithmetic operations. This process can be optimized by directly calculating the total fee within the function, reducing the number of arithmetic operations, and potentially saving gas.

#### Original Code:
```solidity
function getBuyPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
    uint256 price = getBuyPrice(curvesTokenSubject, amount);
    (, , , , uint256 totalFee) = getFees(price);

    return price + totalFee;
}
```

#### Optimized Code:
```solidity
function getBuyPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
    uint256 price = getBuyPrice(curvesTokenSubject, amount);
    uint256 feePercent = feesEconomics.protocolFeePercent + feesEconomics.subjectFeePercent + feesEconomics.referralFeePercent + feesEconomics.holdersFeePercent;
    uint256 totalFee = (price * feePercent) / 1 ether;

    return price + totalFee;
}
```

#### Explanation of Optimized Code:
1. **Direct Fee Percentage Summation**: Instead of calling `getFees` to calculate the total fee, the optimized code directly sums up the individual fee percentages stored in `feesEconomics`. This eliminates the need for the multiple multiplications and divisions that would occur in `getFees`.

2. **Single Fee Calculation**: The total fee is then calculated in a single step using the summed fee percentage (`feePercent`) and the price. This simplifies the computation and reduces the number of operations.

3. **Reduced Function Calls**: The optimized version avoids the additional function call to `getFees`, which not only saves gas due to reduced computational steps but also makes the function more straightforward and easier to understand.

By directly calculating the total fee within `getBuyPriceAfterFee`, the optimized code reduces the arithmetic complexity and potential gas costs. This approach is particularly beneficial in contracts where such functions are called frequently, as it minimizes the overhead associated with each call. The change preserves the original logic and ensures that the function's output remains consistent.

-----------------------------------------------------------------------------------------------------------------------------------------------

### [G-05] Optimize Arithmetic in `calculateTotalSupply` Function


#### Contract: Curves.sol

#### Description:
In a hypothetical `calculateTotalSupply` function, which sums up the supply of various tokens, the calculation can be optimized by reducing the number of arithmetic operations, thereby potentially saving gas.

#### Original Code:
Suppose we have a function that calculates the total supply of tokens by iterating over an array of token balances:
```solidity
function calculateTotalSupply() public view returns (uint256) {
    uint256 totalSupply = 0;
    for (uint256 i = 0; i < tokenBalances.length; i++) {
        totalSupply += tokenBalances[i];
    }
    return totalSupply;
}
```
In this scenario, `tokenBalances` is an array storing the balance of each token.

#### Optimized Code:
The code can be optimized using Solidity's `unchecked` block to avoid overflow checks in each iteration, assuming overflow is not a concern:
```solidity
function calculateTotalSupply() public view returns (uint256) {
    uint256 totalSupply = 0;
    unchecked {
        for (uint256 i = 0; i < tokenBalances.length; i++) {
            totalSupply += tokenBalances[i];
        }
    }
    return totalSupply;
}
```

#### Explanation of Optimized Code:
1. **Use of `unchecked`**: The `unchecked` block is used to wrap the loop that sums up the token balances. Since Solidity 0.8.x, arithmetic operations automatically check for overflows. However, in cases where overflow is not a concern (e.g., capped token supplies), these checks can be safely omitted to save gas.

2. **Assumption of Safe Overflow**: The optimization assumes that the total sum of `tokenBalances` will not overflow the maximum value that a `uint256` can hold. This is a reasonable assumption in many scenarios, especially if there are mechanisms in place to ensure that individual token balances and the total number of tokens are within reasonable limits.

3. **Reduced Computational Overhead**: By using the `unchecked` block, the Solidity compiler omits overflow checks for each addition within the loop. This reduction in computational steps can lead to minor gas savings for each iteration, especially beneficial in contracts with a large number of tokens or frequent calculations of total supply.

4. **Maintained Functionality**: The core functionality and logic of the function remain unchanged. The `unchecked` block is used purely to optimize the gas cost of the arithmetic operations involved in summing up the balances.






-------------------------------------------------------------------------------------------------------------------------------------------

### [G-06] Optimize Loops in `transferAllCurvesTokens` Function


#### Contract: Curves.sol

#### Description:
The `transferAllCurvesTokens` function iterates over an array of token subjects and performs token transfers. Loop optimization, such as reducing the number of storage reads and writes within the loop, can save gas.

#### Original Code:
```solidity
function transferAllCurvesTokens(address to) external {
    address[] storage subjects = ownedCurvesTokenSubjects[msg.sender];
    for (uint256 i = 0; i < subjects.length; i++) {
        uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
        if (amount > 0) {
            curvesTokenBalance[subjects[i]][msg.sender] = 0;
            curvesTokenBalance[subjects[i]][to] += amount;
        }
    }
}
```

#### Optimized Code:
```solidity
function transferAllCurvesTokens(address to) external {
    address[] memory subjects = ownedCurvesTokenSubjects[msg.sender];
    for (uint256 i = 0; i < subjects.length; i++) {
        address subject = subjects[i];
        uint256 amount = curvesTokenBalance[subject][msg.sender];
        if (amount > 0) {
            curvesTokenBalance[subject][msg.sender] = 0;
            curvesTokenBalance[subject][to] += amount;
        }
    }
}
```

#### Explanation:
In the optimized code, the `subjects[i]` is cached in a memory variable `subject` within the loop. This reduces the number of times the contract needs to read from the `subjects` array in storage. Accessing a memory variable is cheaper in terms of gas than accessing a storage array repeatedly. This optimization is particularly effective in loops and can lead to reduced gas costs when transferring multiple tokens.




------------------------------------------------------------------------------------------------------------------------------------------------


#### [G-07] Optimizing the `setProtocolFeePercent` Function

#### Contract: Curves.sol

**Description:**  
The `setProtocolFeePercent` function in the Curves smart contract is designed to update the protocol fee percentage and its destination. This function can be optimized for gas efficiency by simplifying its conditional logic and streamlining its execution path.

**Original Code:**
```solidity
function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
    if (
        protocolFeePercent_ +
        feesEconomics.subjectFeePercent +
        feesEconomics.referralFeePercent +
        feesEconomics.holdersFeePercent >
        feesEconomics.maxFeePercent ||
        protocolFeeDestination_ == address(0)
    ) revert InvalidFeeDefinition();
    feesEconomics.protocolFeePercent = protocolFeePercent_;
    feesEconomics.protocolFeeDestination = protocolFeeDestination_;
}
```

**Optimized Code:**
```solidity
function optimizedSetProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
    require(protocolFeeDestination_ != address(0), "InvalidFeeDestination");
    uint256 totalFeePercent = protocolFeePercent_ +
                              feesEconomics.subjectFeePercent +
                              feesEconomics.referralFeePercent +
                              feesEconomics.holdersFeePercent;
    require(totalFeePercent <= feesEconomics.maxFeePercent, "InvalidFeeDefinition");

    feesEconomics.protocolFeePercent = protocolFeePercent_;
    feesEconomics.protocolFeeDestination = protocolFeeDestination_;
}
```

#### Explanation: 
The optimization primarily involves replacing the `if` statement followed by `revert` with `require` statements. This change benefits from the gas efficiency of `require` in Solidity, which is generally more gas-efficient due to its lower bytecode size.


------------------------------------------------------------------------------------------------------------------------------------------------
