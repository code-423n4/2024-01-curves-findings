# Curves QA Report

| QA issues | Issues                                                                               | Instances | Severity    |
|-----------|--------------------------------------------------------------------------------------|-----------|-------------|
| [N-01](#n-01-no-zero-address-check)      | No zero address check                               | 4         | Non-Critical|
| [N-02](#n-02-onbalancechange-upon-each-call-push-the-token-invovled-in-the-transaction-to-the-usertokens-array-creating-duplicate)      | `onBalanceChange` upon each call push the token invovled in the transaction to the `userTokens` array creating duplicate    | 1         | Non-Critical|
| 
| [R-01](#r-01-use-openzeppelins-ownable-contract)      | Use OpenZeppelin's `Ownable` contract                      | 1         | Recommandation|


## N-01 No zero address check 


### Context 

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L35-L37
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L113-L115
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L155-L160
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L162-L164


### Description

The following setters do not check for the zero address:

- setERC20Factory
- setReferralFeeDestination
- setFeeRedistributor
- setCurves

### Recommendation

Check for the zero address in each setter and revert if it matches.

```diff
    function setFeeRedistributor(address feeRedistributor_) external onlyOwner {
+       require(feeRedistributor_ != address(0), "Curves: zero address");        
        feeRedistributor = FeeSplitter(payable(feeRedistributor_));
    }
```

```diff
    function setReferralFeeDestination(
        address curvesTokenSubject,
        address referralFeeDestination_
    ) public onlyTokenSubject(curvesTokenSubject) {
+       require(referralFeeDestination_ != address(0), "Curves: zero address");
        referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
    }
```

```diff
    function setERC20Factory(address factory_) external onlyOwner {
+       require(factory_ != address(0), "Curves: zero address");
        curvesERC20Factory = factory_;
    }
```

```diff
    function setCurves(Curves curves_) public {
+       require(address(curves_) != address(0), "Curves: zero address");
        curves = curves_;
    }
```

# N-02 `onBalanceChange` upon each call push the token invovled in the transaction to the `userTokens` array creating duplicate

## Impact

On each call to `onBalanceChange` the token involved in the transaction is pushed to the `userTokens` array. This can lead to duplicate in the array and thus to a higher gas cost when iterating over the array or event make the transaction revert if the array is too big.

## Recommended Mitigation Steps

can use something similar to only add when needed or use a mapping

```diff
@@ -96,7 +96,15 @@ contract FeeSplitter is Security {
     function onBalanceChange(address token, address account) public onlyManager {
         TokenData storage data = tokensData[token];
         data.userFeeOffset[account] = data.cumulativeFeePerToken;
-        if (balanceOf(token, account) > 0) userTokens[account].push(token);
+        address[] memory tokens = userTokens[account];
+        if (balanceOf(token, account) > 0) {
+            for (uint256 i = 0; i < tokens.length; i++) {
+                if (tokens[i] == token) {
+                    return;
+                }
+            }
+        userTokens[account].push(token);
+        }
```

## R-01 Use OpenZeppelin's `Ownable` contract 

### Context

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L1-L31

### Description 

Use OpenZeppelin's `Ownable ` instead of a custom implementation. This is more secured and battle tested.

### References

https://docs.openzeppelin.com/contracts/5.x/access-control