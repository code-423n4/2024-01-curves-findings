### Low Risk Issues List

| Number | Issues Details | Context |
| --- | --- | --- |
| [L-1] | MerkleProof length is not validated  |  |
| [L-2] | Use of bytes.concat instead of abi.encodePacked |  |
| [L-3] | Two step ownership transfer |  |



## [L-1] MerkleProof length is not validated
The length of a Merkle proof is not properly checked or validated. 


### Links to affected code
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L422-L426


### Recommended Mitigation Steps
Implement Length Validation: Ensure that the length of the Merkle proof is properly validated before it is used in any verification process.

```diff
function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view {
+   require(proof.length > 0, "Merkleproof length 0");    
    // Verify merkle proof
    bytes32 leaf = keccak256(abi.encodePacked(caller));
    if (!MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf)) revert UnverifiedProof();
}
```


## [L-2] Use of bytes.concat instead of abi.encodePacked
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled.

### Links to affected code
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L424


### Recommendations
As the OpenZeppelin Github documentation provides
https://github.com/OpenZeppelin/merkle-tree/blob/master/README.md#validating-a-proof-in-solidity

```diff
function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view {
    // Verify merkle proof
-   bytes32 leaf = keccak256(abi.encodePacked(caller));
+   bytes32 leaf = keccak256(bytes.concat(keccak256(abi.encode(caller))));
    if (!MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf)) revert UnverifiedProof();
}
```


## [L-3] Two step ownership transfer
The function transferOwnership() transfer ownership to a new address. In case a wrong address is supplied
ownership is inaccessible. The solution does not support the two-step-ownership-transfer pattern.
The ownership transfer might be accidentally set to an inactive EOA
account. In the case of account hijacking, all functionalities get under
permanent control of the attacker.


### Affacted Code
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L27-L29
```
function transferOwnership(address owner_) public onlyOwner {
    owner = owner_;
}
```

### Recommendations
It is recommended to implement a two-step process where the owner nominates
an account and the nominated account needs to call an acceptOwnership()
function for the transfer of the ownership to fully succeed. This ensures
the nominated EOA account is a valid and active account.


