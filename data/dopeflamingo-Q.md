`getPrice()` function can revert during runtime due to underflow.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L180C1-L187C6

if `supply` is equal to 0 and the amount the user wishes to buy is greater than 1, the `supply - 1` will revert due to underflow. This means the first buyer of a `curvesToken` for a specific `curvesTokenSubject` can't buy more than 1. 

```
 uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
```





