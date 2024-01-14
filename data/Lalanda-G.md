# Unnecessary if check

Two unnecessary if checks on Curves.sol [deposit() function](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L497). If the checks are not present, the function will still revert.

    function deposit(address curvesTokenSubject, uint256 amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
        uint256 tokenAmount = amount / 1 ether;

        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
       - if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance(); // @audit gas unnecessary check. Burn will fail 
       - if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance(); // @audit gas unnecessary check. _transfer() will fail 

        CurvesERC20(externalToken).burn(msg.sender, amount);
        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
    } 