## 1. `Curves.sol/getPrice()` will fail if the user tries to buy more than one token in one txn when supply is zero or sell more than one token to making the supply to be zero
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L180-L187 
This will make the seller or buyer to make more than one txn during those conditions.
Consider having this implementation(breaking the getPrice function into two functions)
```Solidity
    function _getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        // Same implementation as of earlier version of getPrice
        
        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
        uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
        return (summation * 1 ether) / 16000;
    }

    function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        if(supply == 0 && amount > 1) {
            return _getPrice(0,1) + _getPrice(1,amount - 1);
        }
        else {
            return _getPrice(supply, amount)
        }
    }

```


## 2. The Buyer or Seller will loose their claimable fee portion in `FeeSplitter.sol` if they have not claimed earlier
[FeeSplitter.sol/onBalanceChange()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L96) 
[Curves.sol/_transferFees()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L247) 

`FeeSplitter.sol/onBalanceChange()`(due to which the fees) is called after every call to `Curves.sol/_transferFees()`. Which is called after every buy and sell function call.
This function don't update `TokenData/unclaimedFees` and only updates the `TokenData/userFeeOffset` due to which user now can not claim the fees that was accrued before Buying or Selling.

Consider this implementation 
```Solidity
    function onBalanceChange(address token, address account) public onlyManager {
        updateFeeCredit(token, msg.sender);  // This will accrue the fees into unclaimableFees
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }
```


## 3. There is no function/logic implemented to take out or reuse the left over funds in `Curves.sol` and `FeeSplitter.sol` 
There are left over funds in both the contracts due to  <br>
- Round Down value left in [FeeSplitter.sol/addFees()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L93)
- Extra value(from msg.value) that is left to buy the curves token in [Curves.sol/_buyCurvesToken()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L270)

## 4. Just after the construction time, the fees are not set as the fee values are not taken in the constructor itself.
This will expose a possibility to frontrun the fee setting txn in [Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L117-L153) and buy the initial tokens without any fee.
Consider setting the `FeesEconomics` in construction time. May set it zero, this will reduce the possibility of frontrun.