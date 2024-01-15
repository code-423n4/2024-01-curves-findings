| Number | Issues  | Instances |
| ------ | ------- | --------- |
| [G-01] |  Waste of Gas to Execute Zero Amount | 1 |

### [G-01] Waste of Gas to Execute Zero Amount

#### Line References

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L215

#### Description
The `buyCurvesTokenWithName()` and `buyCurvesTokenForPresale()` functions are used to buy the initial amount of token by the `curvesTokenSubject()`.

```solidity=364
function buyCurvesTokenWithName(
    address curvesTokenSubject,
    uint256 amount,
    string memory name,
    string memory symbol
) public payable {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply != 0) revert CurveAlreadyExists();

    _buyCurvesToken(curvesTokenSubject, amount);
    _mint(curvesTokenSubject, name, symbol);
}
```

However, the `amount` parameter that defined by the `curvesTokenSubject()` can be `0` and it will be the waste of gas to execute the `_buyCurvesToken()` function.

#### Recommended Mitigation Steps
Adding the validation check for the `amount` parameter before execute the `_buyCurvesToken()` function, for example.

```diff
function buyCurvesTokenWithName(
    address curvesTokenSubject,
    uint256 amount,
    string memory name,
    string memory symbol
) public payable {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply != 0) revert CurveAlreadyExists();
++  if(amount > 0){
        _buyCurvesToken(curvesTokenSubject, amount);
++  }
    _mint(curvesTokenSubject, name, symbol);
}
```