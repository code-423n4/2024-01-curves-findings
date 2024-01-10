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