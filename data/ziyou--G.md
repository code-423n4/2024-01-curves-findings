| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | The function transferCurvesToken lacks a check for an input value of 0 for amount | 1 |

### [GAS-1] The function transferCurvesToken lacks a check for an input value of 0 for amount

If the value of the user input parameter amount is 0, gas is wasted.

*Instances (1)*:
```solidity
File: contracts/Curves.sol


296    function transferCurvesToken(address curvesTokenSubject, address to, uint256 amount) external {
        if (to == address(this)) revert ContractCannotReceiveTransfer();
        _transfer(curvesTokenSubject, msg.sender, to, amount);
    }

313    function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
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