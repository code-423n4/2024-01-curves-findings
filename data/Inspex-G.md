### [G-01] Waste of Gas to Execute Zero Amount

#### Line References

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L215

#### Description
The `buyCurvesToken()` and `buyCurvesTokenWhitelisted()` functions are used to buy a specific amount of token from the `curvesTokenSubject()`.

```solidity=211
function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
    uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

    _buyCurvesToken(curvesTokenSubject, amount);
}
```

However, the `amount` parameter that defined by user can be `0` and it will be the waste of gas to execute.

#### Recommended Mitigation Steps
Adding the validation check for the `amount` parameter, for example.

```diff
function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
    uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
++    if (amount == 0) revert ZeroNotAllowed();
    _buyCurvesToken(curvesTokenSubject, amount);
}
```