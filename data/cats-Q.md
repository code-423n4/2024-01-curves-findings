# [L-01] All excess `msg.value` accumulated in `Curves.sol` will be stuck forever if not refunded

# Vulnerability Details
When a user buys a Curves token they are required to send in `msg.value >= price + totalFee`

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263-L280

```javascript
    function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);

@>      if (msg.value < price + totalFee) revert InsufficientPayment();

        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply); // @audit if msg.value sent is > amount + fees, leftover amount is not returned

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
    }
```

But if a user sends in more than the required amount, the excess value is not refunded. All excess funds are forever stuck in the contract since there are no functions to withdraw any native eth.

# Mitigation
Add logic to refund users.

```diff
    function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);

        if (msg.value < price + totalFee) revert InsufficientPayment();
+       if (msg.value > price + totalFee) {
+           uint256 refund = msg.value - price + totalFee
+           (bool success, ) = msg.sender.call{value: refund}("");
+               if (!success2) revert CannotSendFunds(); 
+       }

        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
    }
```

# [L-02] Add `curves` implementation to `FeeSplitter.sol` constructor

# Vulnerability Details
Currently the Fee Splitter contract is deployed without the Curves implementation in it's constructor. Several key functions in the contract need to reference back to the Curves Contract to work properly. This is an issue because if someone is trying to buy/sell a Curves Subject Token in the `Curves.sol` contract, at the end `FeeSplitter.sol#addFees()` is invoked which references back to Curves for some data and if the splitter is deployed with a non set (address0) Curves then the buy/sells will revert.