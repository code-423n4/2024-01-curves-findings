### [L-1] The `Curves::getSellPrice` function is susceptible to returning a false value due to a possible underflow error.

**Relevant Github Links**
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L193
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L180 

**Description:** The `Curves::getSellPrice()` is a view function that returns the price of a specific amount of tokens the user may want to sell and does the calculations in `Curves::getPrice()`. However, it's possible that the `msg.sender` calling `Curves::getSellPrice` passes in a value in `amount` that is greater than the `curvesTokenSupply[curvesTokenSubject]` mapping, resulting in an underflow error in the calculations. 

```javascript
    function getSellPrice(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
        return getPrice(curvesTokenSupply[curvesTokenSubject] - amount, amount); // <-- possible underflow error
    }
```
 
**Recommended Mitigation:** Consider adding a check to make sure that `amount` is less than or equal to `curvesTokenSupply[curvesTokenSubject]`.

```javascript
require(amount <= curvesTokenSupply[curvesTokenSubject], "Amount exceeds token supply");
```