## [G-1] Should use calldata instead of memory to save gas 

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L428

```diff
  function setNameAndSymbol(
        address curvesTokenSubject,
- string memory name,
+ string calldata name
- string memory symbol
+ string calldata symbol
    ) external onlyTokenSubject(curvesTokenSubject) {
        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
        externalCurvesTokens[curvesTokenSubject].name = name;
        externalCurvesTokens[curvesTokenSubject].symbol = symbol;
    }
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L456

```diff
 function _mint(
        address curvesTokenSubject,
-       string memory name,
+       string calldata name,
-       string memory symbol,
+       string calldata symbol
    ) internal onlyTokenSubject(curvesTokenSubject) {
        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
        _deployERC20(curvesTokenSubject, name, symbol);
    }
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L364
```
  function buyCurvesTokenWithName(
        address curvesTokenSubject,
        uint256 amount,
-       string memory name,
+       string calldata name,
-       string memory symbol
+       string calldata symbol
    ) public payable {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply != 0) revert CurveAlreadyExists();

        _buyCurvesToken(curvesTokenSubject, amount);
        _mint(curvesTokenSubject, name, symbol);
    }
```
