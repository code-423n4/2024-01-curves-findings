# [1] Lack of `address(0)` check in `setFeeRedistributor()` and `setERC20Factory()`

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L113)
```
    function setFeeRedistributor(address feeRedistributor_) external onlyOwner {
        feeRedistributor = FeeSplitter(payable(feeRedistributor_));
    }
```

Function does not verify, if parameter passed by the user: `feeRedistributor_` - is not `address(0)`.
Add additional `require` check, which will make sure, that above variable is not `address(0)`:

```
require(feeRedistributor_ != address(0), "Cannot be address(0)");
```

Additional instances, where `address(0)`-check are missed:

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L162)
```
 function setERC20Factory(address factory_) external onlyOwner {
        curvesERC20Factory = factory_;
    }
```

[File: Curvesl.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L159)
```
 function setReferralFeeDestination(
        address curvesTokenSubject,
        address referralFeeDestination_
    ) public onlyTokenSubject(curvesTokenSubject) {
        referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
    }
```


# [2] Setters should prevent re-setting of the same value

The bot-report missed some instances, which are reported here.
According to the bot-report:

```
[N-25] Setters should prevent re-setting of the same value
There are 5 instance(s) of this issue:

function setFeeRedistributor
function setERC20Factory
function setManager
function setMaxFeePercent
function setExternalFeePercent
```

The instances missed by the bot-report are:

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L128)
```
function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
```


[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L155C23-L155C23)
```
function setReferralFeeDestination(
```



# [3] Use comments for advanced calculations

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L180)
```
    function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
        return (summation * 1 ether) / 16000;
    }
```

Function `getPrice()` performs advanced calculations, but misses comment explaining what (and why) is being calculated.
Consider commenting each line of code, which will let everyone know the reasoning under the complex math.


# [4] Use descriptive constants instead of magic numbers

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L186)
```
return (summation * 1 ether) / 16000;
```

The code-base does not explain, why the end result is being divided by `16000`. It's better idea to use descriptive constant variable (the name of the constant should explain why we are dividing here), instead of `16000` number.


# [5] `sellValue` calculations in `_transferFees()` can be simplified

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L225)
```
 (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
        {
            bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
            {
                address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
                uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
                uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
                (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
                if (!success1) revert CannotSendFunds();
            }
```

When we look at `getFees()` implementation, we can confirm, that it returns also `totalFee`:

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L166)
```
 function getFees(
        uint256 price
    )
        public
        view
        returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
        [...]
        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
```

Function `_transferFees()` calculates `price` by subtracting  `protocolFee` and `subjectFee` and `referralFee` and `holderFee`.
Please notice, that `price - protocolFee - subjectFee - referralFee - holderFee` is the same as `price - (protocolFee + subjectFee + referralFee + holderFee)`, and we know that `(protocolFee + subjectFee + referralFee + holderFee)` is returned as `totalFee` by `getFees()` function.

This means, that `_transferFees()` code can be changed from:

```
(uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
[...]
uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
```

to:

```
(uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, uint256 totalFee) = getFees(price);
[...]
uint256 sellValue = price - totalFee;
```


# [6] Lack of `address(0)` check in `_transfer()` allows to transfer tokens to `address(0)`

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L313)
```
    function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

        // If transferring from oneself, skip adding to the list
        if (from != to) {
            _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
        }

        curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
        curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;

        emit Transfer(curvesTokenSubject, from, to, amount);
    }
```

Function `_transfer()` does not verify if parameter `to` is `address(0)`. This basically means, that it's possible to transfer tokens directly to `address(0)`. Since function `_transfer()` is internal, we've examined other functions which call `_transfer()`:


`transferCurvesToken(address curvesTokenSubject, address to, uint256 amount)` -> calls ` _transfer(subjects[i], msg.sender, to, amount);`
`transferAllCurvesTokens(address to)` -> calls `_transfer(subjects[i], msg.sender, to, amount)`
`withdraw(address curvesTokenSubject, uint256 amount)` -> calls `_transfer(curvesTokenSubject, msg.sender, address(this), amount)`
`deposit(address curvesTokenSubject, uint256 amount)` -> calls `_transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount)`.

In functions `withdraw()` and `deposit()`, parameter `to` is set to `msg.sender` and `address(this)`, however, in functions `transferCurvesToken()` and `transferAllCurvesTokens()` - parameter `to` is passed directly by the user.

None of `transferCurvesToken()` and `transferAllCurvesTokens()` functions perform additional `address(0)` check, which implies, that it's possible to call those functions and transfer tokens to `address(0)`.


# [7] Incorrect white-space formatting in `setMaxFeePercent()` and `setProtocolFeePercent()`

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L118)
```
        if (
            feesEconomics.protocolFeePercent +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            maxFeePercent_
        ) revert InvalidFeeDefinition();
```

There are redundant white spaces in the code-syntax, in lines 120-122. The code-format should be fixed:

```
        if (
            feesEconomics.protocolFeePercent +
            feesEconomics.subjectFeePercent +
            feesEconomics.referralFeePercent +
            feesEconomics.holdersFeePercent >
            maxFeePercent_
        ) revert InvalidFeeDefinition();
``` 


The same issue occurs in `setProtocolFeePercent()`.


# [8] Incorrect grammar in comments

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L46)
```
// Counter for CURVES tokens minted
```

should be changed to:

```
// Counter for minted CURVES tokens
```

[File: Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L276)
```
 // If is the first token bought, add to the list of owned tokens
```

should be changed to:

```
 // If it is the first bought token, add it to the list of owned tokens
```

# [9] `@dev` comment should be in a NatSpec, above the function

[File: FeeSplitter.sol](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L43)
```
    function totalSupply(address token) public view returns (uint256) {
        //@dev: this is the amount of tokens that are not locked in the contract. The locked tokens are in the ERC20 contract
        return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
    }
```

`@dev` comments should be above function, in the NatSpec. Any additional comments explaining the code base should not contain `@dev` tag.
Recommendation: use `@dev` above function:

```
// @dev: this is the amount of tokens that are not locked in the contract. The locked tokens are in the ERC20 contract
 function totalSupply(address token) public view returns (uint256) {
        return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
    }
``` 
