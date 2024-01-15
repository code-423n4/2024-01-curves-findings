1) Issue with Zero Price Handling in getFees Function

The getFees function does not appropriately handle cases where the input price is zero. Currently, it proceeds to calculate fees, which results in unnecessary computation and can lead to inefficiencies. In scenarios where the price is zero, logically, all calculated fees should also be zero.

Recommended Mitigation Steps
Implement an early return in the getFees function that checks if the price is zero. If it is, the function should immediately return zeros for all types of fees.

if (price == 0) {
    return (0, 0, 0, 0, 0);
}

Code: https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L166-L178

====

2) The _buyCurvesToken function exhibits inefficiency in handling fee calculations. The function calls getFees to calculate the total fee and then calls _transferFees, which internally calls getFees again for the same purpose. This redundant calculation of fees results in unnecessary gas expenditure, leading to increased transaction costs for users.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L268
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L274
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L225

To optimize the function and reduce gas costs, consider modifying the _buyCurvesToken function to fetch the fee details once and pass them as arguments to the _transferFees function. This change would eliminate the need for a second call to getFees within _transferFees, thereby reducing the overall gas cost of the operation.

===

3) Mark math operations as unchecked

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L321-L322