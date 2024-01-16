## Shadowed `Ownable.owner` by `owner` argument in `CurvesERC20::constructor`

## Links to affected code

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/CurvesERC20.sol#L7-L10

## Recommended Mitigation Steps
Rename argument `owner` in constructor to `owner_`