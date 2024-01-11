# GAS OPTIMIZATIONS

##

## [G-1] curvesERC20Factory and _curvesTokenCounter state variables can be packed with same slot

The _curvesTokenCounter is a state variable employed to keep track of the number of tokens minted. It is incremented by one each time a new token is minted. Utilizing a uint96 data type for _curvesTokenCounter is highly efficient and more than sufficient for tracking token counts in this context. The maximum representable value by a uint96 is 79,228,162,514,264,337,593,543,950,335. Given this upper limit, the _curvesTokenCounter state variable would only overflow after an astronomical count of 79,228,162,514,264,337,593,543,950,335 new tokens have been minted, a scenario that is practically unfeasible. Therefore, using a uint96 for _curvesTokenCounter and the associated downcasting will not impact the protocol's functionality or its security.

```diff
FILE: 2024-01-curves/contracts/Curves.sol

41: address public curvesERC20Factory;
+ 46:    uint96 private _curvesTokenCounter = 0;
42:    FeeSplitter public feeRedistributor;
43:    string public constant DEFAULT_NAME = "Curves";
44:    string public constant DEFAULT_SYMBOL = "CURVES";
45:    // Counter for CURVES tokens minted
- 46:    uint256 private _curvesTokenCounter = 0;

```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L42-L47

##

## [G-2] Refactor ``setMaxFeePercent`` function for better gas efficiency

Optimization of the setMaxFeePercent function focuses on reducing gas consumption. The if statement checks if the sum of various fees exceeds maxFeePercent_.

### Actual Code

```solidity
FILE: 2024-01-curves/contracts/Curves.sol

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

### Optimized Code

```solidity

function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
    FeesEconomics memory cachedFeesEconomics = feesEconomics;
    uint256 totalFeePercent = cachedFeesEconomics.protocolFeePercent +
                              cachedFeesEconomics.subjectFeePercent +
                              cachedFeesEconomics.referralFeePercent +
                              cachedFeesEconomics.holdersFeePercent;

    if (totalFeePercent > maxFeePercent_) revert InvalidFeeDefinition();

    feesEconomics.maxFeePercent = maxFeePercent_;
}

```

### Explanation

- ``Caching State Variable``: The feesEconomics state variable is stored in memory at the start. Memory operations are cheaper than repeatedly reading from storage.

- ``Local Total Fee Calculation``: The total of the fees is calculated once and stored in totalFeePercent. This avoids recalculating the sum in the if condition, reducing the computational load and gas usage.

##
 
## [G-3] ``setProtocolFeePercent()`` can be more gas optimized 

### Original Code

```solidity
FILE: 2024-01-curves/contracts/Curves.sol

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

### Optimized Code

```solidity

function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
    FeesEconomics memory cachedFeesEconomics = feesEconomics;

    uint256 totalFeePercent = protocolFeePercent_ +
                              cachedFeesEconomics.subjectFeePercent +
                              cachedFeesEconomics.referralFeePercent +
                              cachedFeesEconomics.holdersFeePercent;
    
    if (totalFeePercent > cachedFeesEconomics.maxFeePercent) 
        revert InvalidFeeDefinition();

    if (protocolFeeDestination_ == address(0))
        revert InvalidFeeDefinition();

    feesEconomics.protocolFeePercent = protocolFeePercent_;
    feesEconomics.protocolFeeDestination = protocolFeeDestination_;
}

```
Caching State Variable: feesEconomics is cached in memory to reduce gas costs associated with multiple storage reads.

##

## [G-4] Use entire ``feesEconomics`` struct is updated at once in ``setExternalFeePercent()`` for better gas efficiency


### Actual Code

```solidity
FILE: 2024-01-curves/contracts/Curves.sol

function setExternalFeePercent(
        uint256 subjectFeePercent_,
        uint256 referralFeePercent_,
        uint256 holdersFeePercent_
    ) external onlyManager {
        if (
            feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ >
            feesEconomics.maxFeePercent
        ) revert InvalidFeeDefinition();
        feesEconomics.subjectFeePercent = subjectFeePercent_;
        feesEconomics.referralFeePercent = referralFeePercent_;
        feesEconomics.holdersFeePercent = holdersFeePercent_;
    }

```

### Optimized Code

```solidity

function setExternalFeePercent(
    uint256 subjectFeePercent_,
    uint256 referralFeePercent_,
    uint256 holdersFeePercent_
) external onlyManager {
    FeesEconomics memory cachedFeesEconomics = feesEconomics;

    if (cachedFeesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ > cachedFeesEconomics.maxFeePercent)
        revert InvalidFeeDefinition();

    feesEconomics = FeesEconomics({
        protocolFeePercent: cachedFeesEconomics.protocolFeePercent,
        maxFeePercent: cachedFeesEconomics.maxFeePercent,
        subjectFeePercent: subjectFeePercent_,
        referralFeePercent: referralFeePercent_,
        holdersFeePercent: holdersFeePercent_
    });
}

```

### Explanation

- ``Caching State Variable``: The feesEconomics state variable is cached in memory to reduce repeated storage reads.

- ``Consolidated Fee Assignment``: Instead of assigning each fee component separately, the entire feesEconomics struct is updated at once. This approach can be more gas-efficient due to the way Solidity handles struct storage. However, this approach is beneficial if most or all elements of the struct are being updated.

##

## [G-5] Reduce repetitive calculations in getFees() function

### Actual Code

```solidity
FILE: 2024-01-curves/contracts/Curves.sol

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
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L166-L178

### Optimized Code

```solidity

function getFees(uint256 price)
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
    FeesEconomics memory cachedFeesEconomics = feesEconomics;
    uint256 pricePerEther = price / 1 ether;

    protocolFee = pricePerEther * cachedFeesEconomics.protocolFeePercent;
    subjectFee = pricePerEther * cachedFeesEconomics.subjectFeePercent;
    referralFee = pricePerEther * cachedFeesEconomics.referralFeePercent;
    holdersFee = pricePerEther * cachedFeesEconomics.holdersFeePercent;
    
    return (
        protocolFee, 
        subjectFee, 
        referralFee, 
        holdersFee, 
        protocolFee + subjectFee + referralFee + holdersFee
    );
}

```

### Explanation:
- ``Caching State Variable``: feesEconomics is stored in memory to reduce gas consumption from multiple state variable reads.
- ``Price Per Ether Calculation``: By calculating price / 1 ether once, the division operation (which is relatively expensive in terms of gas) is minimized.
- ``Inline Total Fee Calculation``: The total fee is computed directly in the return statement, removing the need for a separate local variable for totalFee. This change is more about code efficiency and readability, as the impact on gas cost is minimal in this case.

##

## [G-6] Refine the ``getPrice()`` function better Conditional Checks and Mathematical Operations

### Actual Code

```solidity
FILE: 2024-01-curves/contracts/Curves.sol

function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
        return (summation * 1 ether) / 16000;
    }

```

### Optimized Code

```solidity

function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    if (supply == 0) {
        return amount == 1 ? 0 : ((amount * (amount + 1) * (2 * amount + 1)) / 6) * 1 ether / 16000;
    }

    uint256 sum1 = ((supply - 1) * supply * (2 * supply - 1)) / 6;
    uint256 sum2 = ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    return ((sum2 - sum1) * 1 ether) / 16000;
}

```

### Explanation:

- ``Consolidated Conditional Checks``: The condition supply == 0 && amount == 1 is simplified. If supply is zero, the function returns early, simplifying the logic and potentially saving gas if amount is not 1.
- ``Optimized Mathematical Operations``: Mathematical expressions are kept as-is, but the structure is rearranged to reduce redundant checks and operations. This change makes the function slightly more efficient and easier to read. The division of integers can be tricky due to floor division, but the given operations are unlikely to benefit significantly from rearrangement in this aspect.

##

## [G-7] 



