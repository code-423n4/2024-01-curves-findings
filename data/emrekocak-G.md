# Avoid Making any state reads if we can revert early
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L490-L498

Avoid reading from storage if we might revert early. 

```diff
    function deposit(address curvesTokenSubject, uint256 amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

+		uint256 tokenAmount = amount / 1 ether;
+		if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();
        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
-       uint256 tokenAmount = amount / 1 ether;

        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
-       if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();
```
