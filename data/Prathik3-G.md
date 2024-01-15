## [G-01] - Add if-else block to check whether its a buy or sell instead to calculating both in the `Cureves::_transferFees()`.

In the `Cureves::_transferFees()` function every time `buyValue` and `sellValue` is calculated regardless of whether its buy or sell. Adding an if-else block would result into a gas efficient way to calculate the values.

Change [this](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218-L261) block of code to the following.

<details>

<summary>Gas Efficient Code</summary>

```javascript
function _transferFees(
           .
           .

            bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
            {
                //@audit gas changes made here
                if (isBuy) {
                    address firstDestination = feesEconomics.protocolFeeDestination;
                    uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
                    (bool success1, ) = firstDestination.call{value: buyValue}("");
                    if (!success1) revert CannotSendFunds();
                } else {
                    address firstDestination = msg.sender;
                    uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
                    (bool success1, ) = firstDestination.call{value: sellValue}("");
                    if (!success1) revert CannotSendFunds();
                }
            }
            .
            .
    }
```

</details>

<details>

<summary> Gas Usage</summary>

| Method Name               | Avg Gas Before | Avg Gas After |
|---------------------------|----------------|---------------|
| buyCurvesToken            | 144347         | 123338        |
| buyCurvesTokenForPresale  | 205341         | 204571        |
| buyCurvesTokenWhitelisted | 126028         | 125258        |
| buyCurvesTokenWithName    | 1774847        | 1774077       |
| sellCurvesToken           | 59894          | 59624         |


</details>

## [G-02] - `Curves::getPrice()` can be made internal .

Make the changes [here](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L166)

`Curves::getPrice()` function is used internally in many other functions such as `getBuyPrice`,`getSellPrice` etc but is marked as public . Making this function as internal would reduce gas consumption.


## [G-03] - Validate the values directly instead of storing them in a variable and the validating that variable.

There are a few functions in which variables are set to a specific value, only to validate its value once and never used again . Instead of storing them try to directly compare their values.
Some instances of this are as follows:

<details>

<summary> Instances and gas usage </summary>

[buyCurvesTokenForPresale](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L385-L386)

[buyCurvesTokenWithName](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L370-L371)

[setWhitelist](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L395-L396)

[verifyMerkle](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L424-L425)


| Method Name               | Avg Gas Before | Avg Gas After |
|---------------------------|----------------|---------------|
| buyCurvesTokenForPresale  | 205341         | 205328        |

</details>

