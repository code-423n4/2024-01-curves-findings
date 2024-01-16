# QA Report

## [NC-01] Missing access modifier

The `deploy` function of the `CurvesERC20Factory` contract is missing an access modifier. Anyone can call the function and deploy a new `CurvesERC20` contract. It should be possible for only the `Curves` contract to have access to the deploy function of the factory contract.