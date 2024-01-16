[curves.sol](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L213)
In the function buyCurvesToken() sales start exactly at the startTime but buying is not allowed at block.timestamp == startTime and is actually starting at block.timestamp > startTime which should not be the case.

modify condition as follows:
""" if (startTime != 0 && startTime > block.timestamp) revert SaleNotOpen(); """