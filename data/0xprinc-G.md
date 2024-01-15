## 1. The modifier `Curves.sol/onlyTokenSubject()` is not needed
`onlyTokenSubject()` only let the msg.sender whose address is stored in calldata to call that function. <br>
This takes gas to for extra CALLDATACOPY, REVERT etc. <br>
Instead remove that modifier and replace `curvesTokenSubject` with `msg.sender` in all the functions in which the modifier has appeared.<br>
This will also save the CALLDATA space and also codesize that is increased by including the modifier.

Just like this implementation of `Curves.sol/_mint()` function
```Solidity
    function _mint(
        string memory name,
        string memory symbol
    ) internal {
        if (externalCurvesTokens[msg.sender].token != address(0)) revert ERC20TokenAlreadyMinted();
        _deployERC20(msg.sender, name, symbol);
    }
```

Functions that include the modifier and require change are:<br>
- setReferralFeeDestination
- buyCurvesTokenForPresale
- setNameAndSymbol
- mint
- _mint