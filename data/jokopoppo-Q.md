| Number |  Issue   | Instances |
| ------ |:--------:| --------- |
| L-01   | Unable to Purchase Token When `block.timestamp` Equals `startTime` | 1         |
| L-02   | Price Calculation Should Use Addition Before Subtraction | 1         |



### [L-01] Unable to Purchase Token When `block.timestamp` Equals `startTime`

#### Line References
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L213

#### Description
The `buyCurvesToken()` function is used to purchase tokens from the `curvesTokenSubject` when the `block.timestamp` reaches the `startTime` state.

```solidity=211
function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
    uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

    _buyCurvesToken(curvesTokenSubject, amount);
}
```

However, the user is unable to do so due to the reversion at line 213, which employs the validation check `startTime >= block.timestamp`.

#### Recommended Mitigation Steps
Modify the validation check in line 213 as follow.

```solidity=211
function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
    uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
--    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
++    if (startTime != 0 && startTime > block.timestamp) revert SaleNotOpen();

    _buyCurvesToken(curvesTokenSubject, amount);
}
```


### [L-02] Price Calculation Should Use Addition Before Subtraction

#### Line References
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L184

#### Description
The `getPrice()` function is used to calculate the token price base on the buying amount and current supply of that `curvesTokenSubject`.

```solidity=180
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}
```

The initial state of the token requires the curvesTokenSubject to buy the initial tokens to set the initial price for the users.

However, the `curvesTokenSubject` is unable to buy more than 1 token initially due to the condition in line 182. If the amount is greater than 1, the price calculation will use the formula in line 184, which involves subtracting 1 from the `supply` (where `supply` is initially 0) before adding the `amount`, resulting in an underflow and a revert.

#### Recommended Mitigation Steps
Modify the formula in line 184 to use addition before subtraction as follows.

```diff
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
--        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
++        : ((supply + amount - 1 ) * (supply + amount) * (2 * (supply + amount - 1 ) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}
```