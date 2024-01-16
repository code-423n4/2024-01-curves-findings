## Using Specified Amount and Gas Efficient Setter Functions in the `Curves` Contract

## Summary

This code review highlights two problems for improvement: 
1)`Curves: withdraw` and `Curves: deposit`: `If amount > curvesTokenBalance[curvesTokenSubject][msg.sender]`, it is recommended to use the specified amount of `curvesTokenBalance[curvesTokenSubject][msg.sender]` instead of the parameter of `amount` to call `_transfer`, `mint` and `burn` functions. 
2)Optimize Setter Function In `Curves` contract: To enhance change tracking and memory efficiency by directly recording old and new values of critical parameters within setter function, eliminating the need for redundant caching.



## Detail

1.Enhancing Token Withdraw and Deposit Logic in the `Curves` Contract

In the `withdraw` and `deposit` functions of the `Curves` contract, it is suggested to modify the logic as follows:

`If amount > curvesTokenBalance[curvesTokenSubject][msg.sender]`, it is recommended to use the specified amount of `curvesTokenBalance[curvesTokenSubject][msg.sender]` instead of the parameter of `amount` to call `_transfer`, `mint` and `burn` functions. 

Careful testing and documentation updates are essential to ensure the effectiveness and clarity of these enhancements. The improvements contribute to a more user-friendly experience for token withdrawals, emphasizing simplicity and reliability.

[Curves::withdraw, deposit](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L465C1-L502C6)
```solidity
-   function withdraw(address curvesTokenSubject, uint256 amount) public {
+   function withdraw(address curvesTokenSubject, uint256 _amount) public {
+       uint256 amount = _amount;

-       if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();
+       if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]){
+           uint256 amount = curvesTokenBalance[curvesTokenSubject][msg.sender];
+       };

        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
        if (externalToken == address(0)) {
            if (
                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
                keccak256(abi.encodePacked("")) ||
                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
                keccak256(abi.encodePacked(""))
            ) {
                externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
                externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
            }
            _deployERC20(
                curvesTokenSubject,
                externalCurvesTokens[curvesTokenSubject].name,
                externalCurvesTokens[curvesTokenSubject].symbol
            );
            externalToken = externalCurvesTokens[curvesTokenSubject].token;
        }
        _transfer(curvesTokenSubject, msg.sender, address(this), amount);
        CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
    }

-   function deposit(address curvesTokenSubject, uint256 amount) public {
+   function deposit(address curvesTokenSubject, uint256 _amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
-       uint256 tokenAmount = _amount / 1 ether;
+       uint256 amount = _amount;
+       uint256 tokenAmount = _amount / 1 ether;

        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
-       if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
-       if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

+       if (amount > CurvesERC20(externalToken).balanceOf(msg.sender) || if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)])){
+           uint256 amount = CurvesERC20(externalToken).balanceOf(msg.sender);
+           uint256 tokenAmount = curvesTokenBalance[curvesTokenSubject][address(this)];
+       };

        CurvesERC20(externalToken).burn(msg.sender, amount);
        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
}
```

2.Improve Logic in Setter Functions in the `Curves` contract

Record both old and new values: Directly within setter functions, it is recommended to record both its previous and updated values of critical parameters for setter function in the `Curves` contract. This enables accurate tracking of modifications, informed analysis of patterns, and the potential to revert to prior states if needed.

Eliminate unnecessary caching: If the old value solely serves immediate actions (e.g., event emission), access it directly within the setter. This streamlines code, potentially reduces memory overhead, and enhances readability.

[Curves::setFeeRedistributor](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L113C1-L115C6)
```solidity
function setFeeRedistributor(address feeRedistributor_) external onlyOwner {
+       emit SetFeeRedistributor(feeRedistributor, feeRedistributor_);
        feeRedistributor = FeeSplitter(payable(feeRedistributor_));
    }
```

All other instances entailed:

[Curves::setMaxFeePercent](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117C1-L126C6)
```solidity
    function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
        if (
            feesEconomics.protocolFeePercent +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            maxFeePercent_
        ) revert InvalidFeeDefinition();
+       emit SetMaxFeePercent(feesEconomics.maxFeePercent, maxFeePercent_);
        feesEconomics.maxFeePercent = maxFeePercent_;
}
```

[Curves::setProtocolFeePercent](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L128C1-L139C6)
```solidity
    function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
        if (
            protocolFeePercent_ +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            feesEconomics.maxFeePercent ||
            protocolFeeDestination_ == address(0)
        ) revert InvalidFeeDefinition();
+       emit SetProtocolFeePercent(feesEconomics.protocolFeePercent, protocolFeePercent_);
+       emit SetProtocolFeePercent(feesEconomics.protocolFeeDestination, protocolFeeDestination_);
        feesEconomics.protocolFeePercent = protocolFeePercent_;
        feesEconomics.protocolFeeDestination = protocolFeeDestination_;
}
```

[Curves::setExternalFeePercent](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L141C1-L153C6)
```solidity
    function setExternalFeePercent(
        uint256 subjectFeePercent_,
        uint256 referralFeePercent_,
        uint256 holdersFeePercent_
    ) external onlyManager {
        if (
            feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ >
            feesEconomics.maxFeePercent
        ) revert InvalidFeeDefinition();
+       emit SetSubjectFeePercent(feesEconomics.subjectFeePercent, subjectFeePercent_;);
+       emit SetReferralFeePercent(feesEconomics.referralFeePercent, referralFeePercent_);
+       emit SetHoldersFeePercent(feesEconomics.holdersFeePercent, holdersFeePercent_);
        feesEconomics.subjectFeePercent = subjectFeePercent_;
        feesEconomics.referralFeePercent = referralFeePercent_;
        feesEconomics.holdersFeePercent = holdersFeePercent_;
}
```

[Curves::setReferralFeeDestination](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L155C1-L160C6)
```solidity
    function setReferralFeeDestination(
        address curvesTokenSubject,
        address referralFeeDestination_
) public onlyTokenSubject(curvesTokenSubject) {
+       emit SetReferralFeeDestination(referralFeeDestination[curvesTokenSubject], referralFeeDestination_);
        referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
}
```

[Curves::setERC20Factory](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L162C1-L164C6)
```solidity
function setERC20Factory(address factory_) external onlyOwner {
+       emit ERC20Factory(curvesERC20Factory, factory_);
        curvesERC20Factory = factory_;
}
```


## Tools Used

Manual Review

## Recommendations
1)`Curves: withdraw` and `Curves: deposit`: `If amount > curvesTokenBalance[curvesTokenSubject][msg.sender]`, it is recommended to use the specified amount of `curvesTokenBalance[curvesTokenSubject][msg.sender]` instead of the parameter of `amount` to call `_transfer`, `mint` and `burn` functions. 
2)Prioritize Change Tracking and Memory Efficiency: Capture both old and new values directly within setter functions for critical parameters to establish a detailed change history. When the old value is only required for immediate actions within the function, directly access it instead of caching, which streamlines code, potentially reduces memory overhead, and improves code readability.



