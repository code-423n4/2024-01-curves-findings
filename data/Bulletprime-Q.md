|  | issues | instance |
|---|---|---|
| [L-01]| Access control bypass | 1 |
|[L-02] | Integer overflow & underflow | 1 |
|[L-03] | Insufficient Gas Limit | 1 |
|[L-04] | DOS Attack | 1 |


DETAILS

[L-01]  Access control bypass.
The contract, `curves.sol` uses access control mechanisms such as `onlyOwner`, `onlyManager`, and `onlyTokenSubject`. However, from the code it's not clear who exactly can assume these roles. and these roles are assigned to a single account and a small group of accounts, this could introduce centralization risk and potentially allow an attacker to gain elevated privileges, and should be an area of concern. An attacker could also potentially exploit this by compromising a device that holds the private keys. 
Mitigation: Clearly define who can assume these roles to ensure proper access control. Consider using a role-based access control (RBAC) model, which provides granular permissions to different roles.




[L-02]  Integer overflow & underflow 
DETAILS
The contract performs arithmetic operations without using safe math libraries. This could potentially lead to integer overflow or underflow vulnerabilities. An attacker could potentially exploit this to manipulate the contract's state.

```
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
   // ...
   feesEconomics.maxFeePercent = maxFeePercent_;
}
```
Mitigation: Use a library like `OpenZeppelin's SafeMath` library, which includes safe versions of arithmetic operations that throw an error on overflow or underflow.



[L-03] | Insuffiient Gas Limit
DETAILS
Insufficient Gas Limit: Some of the functions, particularly those involving sending Ether or calling other contracts, may require more gas than the default limit. If a user tries to call these functions with a low gas limit, the transaction will fail, potentially leading to frustration for the user. This is also an area of concern, especially for large transactions.
Mitigation: Provide guidance to users on setting appropriate gas limits for transactions. Also, consider optimizing your contract to reduce gas costs. For instance, minimize the use of storage and loops, and avoid unnecessary computations



|[L-04] | DOS Attack
DETAILS
The `setMaxFeePercent` function is an external function that allows only the manager to set the maximum fee percentage. This is a crucial function in the Curves protocol as it sets a limit on the amount of fees that can be charged. However, An attacker can repeatedly call this function with a large `maxFeePercent_` value, which could lead to a Denial of Service (DoS) attack.
A DoS attack is a type of cyber attack that aims to make a network resource unavailable to its intended users. In this case, if an attacker sets a very high `maxFeePercent_` value, it prevents the protocol from functioning correctly. As the protocol relies on the `maxFeePercent` value to calculate fees, setting a very high value could lead to incorrect fee calculations or even cause the protocol to stop functioning.

```
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
   // ...
   feesEconomics.maxFeePercent = maxFeePercent_;
}
```
To mitigate this potential DoS attack, it's important to ensure that the `maxFeePercent` value is set to a reasonable limit and that the function is not vulnerable to large or negative values. This could be done by adding additional checks to the function to ensure that the `maxFeePercent_` value is within a certain range.


