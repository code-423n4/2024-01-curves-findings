### [L-1] The `Curves::getSellPrice` function is susceptible to returning a false value due to a possible underflow error

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

### [L-2] Math logic error + confusion in `Curves::deposit()` function will cause many unexpected reverts and disrupt protocol functionality

**Relevant Github Links**
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L491 
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L494C3-L494C3 

**Description:** The `Curves::deposit()` function has a check to make sure that the `amount` sent by `msg.sender` is an integer. It then uses this `amount` value to get the actual `tokenAmount` value to be used in other function calls. However, this calculation is not done correctly and will lead to a numerous amount of reverts unless very specific `amount` values are used. 

We currently have the following check in `Curves::deposit()`:

```javascript
if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
```

This check will always revert unless the user sends an `amount` that is amount * 1e18 because 1 ether = 1e18. For example, if `amount` is 65, we can multiply it by 1e18 to get 65e18 and the check will pass (65e18 % 1e18 = 0).
This fix will cause `tokenAmount` to be correct as well:
``` javascript
uint256 tokenAmount = amount / 1 ether;
```
Likewise, there is no check if `amount` is 0, and this will not cause a revert, which in turn leads the user to spend even more unnecessary gas as the rest of the function executes with more calls, ultimately leading to no real changes being made.

**Recommended Mitigation:** 
We can approach this in a couple of ways. The first would be to change the if-statement by multiplying `amount` by 1e18:
```javascript
if((amount * 1e18) % 1 ether != 0) revert NonIntegerDepositAmount();
```
This seems pointless since we just convert back to the original `amount` value. Instead, we can simply just check if `amount` is greater than 0 so that we don't waste gas by going through the rest of the function. We can also get rid of `tokenAmount` and just use `amount`.

Delete the 2 following lines:
```javascript
if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
```
```javascript
uint256 tokenAmount = amount / 1 ether;
```

Add the following to the beginning of `Curves::deposit()`

```javascript
if(amount <= 0) revert;
```
