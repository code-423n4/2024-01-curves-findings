# A curve token exists when the supply is greater than zero rather than one

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L396

When a user creates any amount of the curve token, they should refrain from updating the whitelist as the curve token already exists. Consequently, the minimal supply value can be 1. The validation in the `setWhitelist` function should not permit updating the whitelist for a curve token with only one token supply. The revised validation is as follows:
```solidity
if (supply >= 1) revert CurveAlreadyExists();
```

# Avoid floating pragma in the project for the consistency of compiler version
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L2
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L2

According to SWC-103 suggestion, contracts should be deployed with the same compiler version. As a result, locking the pragma ensures the consistency of the contract compiler version and the version will not introduce bugs or unintended behavior.

References: 
https://swcregistry.io/docs/SWC-103/
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

# Weak validation for transferOwnership

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L27

The Curves contract can update ownership through the transferOwnership function inherited from the Security function. However, there is a lack of address validation for the new owner. Given that several privileged functions use the onlyOwner modifier, transferring ownership to an uncontrolled or incorrect account can lead to severe consequences. It is advisable to consider implementing an acceptOwnership function for the new contract account owner or incorporate additional validation measures.