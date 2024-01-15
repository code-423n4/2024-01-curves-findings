Issue with Zero Price Handling in getFees Function

The getFees function does not appropriately handle cases where the input price is zero. Currently, it proceeds to calculate fees, which results in unnecessary computation and can lead to inefficiencies. In scenarios where the price is zero, logically, all calculated fees should also be zero.

Recommended Mitigation Steps
Implement an early return in the getFees function that checks if the price is zero. If it is, the function should immediately return zeros for all types of fees.

if (price == 0) {
    return (0, 0, 0, 0, 0);
}