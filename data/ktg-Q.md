[L-01] `ownedCurvesTokenSubjects` array should remove the curve token subject if the user's balance becomes zero
Currently, `ownedCurvesTokenSubjects` array does not shrink if users's balance becomes zero. For example, function `_transfer` only add token subject to `to` but don't remove for `from` (if they transfer all tokens):
```solidity
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

Or in function `sellCurvesToken`, it doesn't check if users balance is zero and remove the token from the array:
```solidity
function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply <= amount) revert LastTokenCannotBeSold();
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

        uint256 price = getPrice(supply - amount, amount);

        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
        curvesTokenSupply[curvesTokenSubject] = supply - amount;

        _transferFees(curvesTokenSubject, false, price, amount, supply);
    }
```
This will make the array `ownedCurvesTokenSubjects` grows in size and cost large amount of gas when users call `transferAllCurvesTokens`


[L-02] If feeRedistributor == address(0) then function `_transferFee` should revert
This is the part in `_transferFee` to handle holder fee:
```solidity
if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
            }
```
As you can see, if `holdersFeePercent > 0` and `address(feeRedistributor) == address(0)`, the fee will get stuck in Curves contract and lost forever (since the contract has no function to transfer ETH out).
 