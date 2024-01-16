[L-1] Not sending the protocol fee when selling tokens.

When selling tokens, we do not send anything to the `protocolFeeDestination`, although the protocol fee is generated in both buying and selling scenarios.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L229-L232
```
function _transferFees() {
    address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
    uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
    uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
    (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
}
```
This can potentially result in the protocol fee and referral fee being stuck in the `Curves` contract when selling tokens.

[L-2] The subject token can initially purchase only `1` token.

The `getPrice` function is invoked when buying or selling tokens.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L184
```
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum2 = supply == 0 && amount == 1
            ? 0
            : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
}
```
When the subject token attempts to purchase initial tokens, the supply is `0` and the amount can be larger than `1`. 
However, in this case, the `getPrice` function will revert due to underflow.
To acquire more tokens, the user should first purchase `1` token and then proceed to purchase the remaining tokens.

[L-3] There is no check for whether the transfer amount is `0` in the `transferCurvesToken` function.

The `transferCurvesToken` function does not include a check for the transfer amount.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L296-L299
```
function transferCurvesToken(address curvesTokenSubject, address to, uint256 amount) external {
    if (to == address(this)) revert ContractCannotReceiveTransfer();
    _transfer(curvesTokenSubject, msg.sender, to, amount);
}
```

As a result, curves tokens are added to a user even when the user does not possess any of those tokens.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L318
```
function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
    if (from != to) {
        _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
    }
}
```
This situation may lead to a loss of funds, especially considering the gas involved in the `transferAllCurvesTokens` function.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L302-L311
```
function transferAllCurvesTokens(address to) external {
    if (to == address(this)) revert ContractCannotReceiveTransfer();
    address[] storage subjects = ownedCurvesTokenSubjects[msg.sender];
    for (uint256 i = 0; i < subjects.length; i++) {
        uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
        if (amount > 0) {
            _transfer(subjects[i], msg.sender, to, amount);
        }
    }
}
```

[L-4] It is not possible to set `presalesMeta` when the subject only possesses its corresponding curves token.

Any subject can set `presalesMeta` before others hold its corresponding curves token using the `setWhitelist` function.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L396
```
function setWhitelist(bytes32 merkleRoot) external {
    uint256 supply = curvesTokenSupply[msg.sender];
    if (supply > 1) revert CurveAlreadyExists();
}
```
However, having a supply greater than `1` doesn't necessarily imply that other users also hold curves tokens. 
The subject may hold more than `1` token in this scenario.

[L-5 ] We do not take rounding into account in the `addFees` function.

When adding fees, a small portion of fees is missing due to rounding.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L93
```
function addFees(address token) public payable onlyManager {
    data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_;
}
```
Consider storing the remaining fee and adding functionality to withdraw excess fees in the `FeeSplitter`.

[L-6] In certain cases, the fees for holders can become stuck in the `Curves` token.

When `holdersFeePercent` is greater than `0` and `feeRedistributor` is not set, the holder's fees can become stuck in the `Curves` token.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L246-L249
```
function _transferFees() {
    if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
        feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
        feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
    }
}
```
Add functionality to withdraw excess fees in the `Curves`.

[L-7] The `userTokens` in `FeeSplitter` may contain duplicated tokens.

In the `onBalanceChange` function, tokens are inserted when the balance is greater than `0`, potentially leading to duplicated tokens. 
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L99
```
function onBalanceChange(address token, address account) public onlyManager {
    if (balanceOf(token, account) > 0) userTokens[account].push(token);
}
```
This can impact certain view functions, including `getUserTokensAndClaimable`.

If the balance transitions from `0` to a value larger than `0`, we do not insert that token. 
This will also impact the `getUserTokensAndClaimable` function.

[L-8] Consider sending the fee and updating the user's balance after when buying or selling tokens.

Currently, we update the user's balance first and then send fees when buying and selling tokens. 
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L272-L274
```
function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
    curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
    curvesTokenSupply[curvesTokenSubject] = supply + amount;
    _transferFees(curvesTokenSubject, true, price, amount, supply);
}
```
Consequently, the newly changed balance is affected by this fee.
We could change this order.

[l-9] Do not refund excess funds when purchasing tokens.

We only verified if the user has sent sufficient funds when purchasing tokens.
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L270
```
function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
    if (msg.value < price + totalFee) revert InsufficientPayment();
}
```
The user intends to buy tokens and obtain the price before making the purchase.
They can send the money by calling the `buyCurvesToken` function. 
However, before this transaction, others may sell the same tokens, leading to a reduction in the buy price. 
As there is no logic to refund excess funds, the extra funds may become stuck in `Curves`.