# [L-01] Unbounded loop

### Description:

New items are `pushed` into the following arrays but there is no option to `pop` them out. Currently, the array can grow indefinitely. 
E.g. there’s no maximum limit and there’s no functionality to remove array values.
If the array grows too large, calling relevant functions might run out of gas and revert. Calling these functions could result in a `DOS` condition.

### Instances:
https://github.com/code-423n4/2024-01-curves/blob/main/contracts%2FCurves.sol#L335