### Low Risk Issues List

| Number | Issues Details | Context |
| --- | --- | --- |
| [L-1] | MerkleProof length is not validated  |  |
| [L-2] | Use of bytes.concat instead of abi.encodePacked |  |
| [L-3] | Two step ownership transfer |  |
| [L-4] | Lost of dust in feeSplitter |  |



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



## [L-4] Lost of dust in feeSplitter
The feeSplitter contract is experiencing a loss of dust

### POC 
```solidity
function test_lostOfDust() public {
    vm.prank(alice);
    curves.buyCurvesToken(alice, 1);
    assertEq(address(feeSplitter).balance, 0);

    _buyToken(alice, bob);
    _buyToken(alice, charlie);

    uint256 bobsClaimableFees = feeSplitter.getClaimableFees(alice, bob);
    assertEq(bobsClaimableFees, 5729166666666);

    uint256 aliceClaimableFees = feeSplitter.getClaimableFees(alice, alice);
    assertEq(aliceClaimableFees, 5729166666666);

    uint256 charlieClaimableFees = feeSplitter.getClaimableFees(alice, charlie);
    assertEq(charlieClaimableFees, 4166666666666);

    uint256 feeSplitterBalance = address(feeSplitter).balance;
    assertEq(feeSplitterBalance, 15625000000000);

    vm.prank(alice);
    feeSplitter.claimFees(alice);

    vm.prank(bob);
    feeSplitter.claimFees(alice);

    vm.prank(charlie);
    feeSplitter.claimFees(alice);

    assertGt(15625000000000, bobsClaimableFees + charlieClaimableFees + aliceClaimableFees); // 15624999999998
    assertEq(address(feeSplitter).balance, 2); // lost of dust
}
```

### Recommended mitigation steps
Consider a mechanism to distribute leftover dust.
