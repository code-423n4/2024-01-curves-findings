1)verifyMerkle function can be optimized to reduce gas usage:


function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view {
    bytes32 leaf = keccak256(abi.encodePacked(caller));
    bytes32 computedHash = leaf;

    for (uint256 i = 0; i < proof.length; i++) {
        bytes32 proofElement = proof[i];

        if (computedHash < proofElement) {
            computedHash = keccak256(abi.encodePacked(computedHash, proofElement))
        } else {
            computedHash = keccak256(abi.encodePacked(proofElement, computedHash));
        }
    }

    if (computedHash != presalesMeta[curvesTokenSubject].merkleRoot) {
        revert UnverifiedProof();
    }
}

Reduced Gas Consumption: This optimized version minimizes gas usage by optimizing the hashing process within the loop.
Iterating through Proof Elements: It iterates through the proof elements and computes the hash accordingly, reducing redundant calculations.

2) Function batchClaiming, ensure that loops are not overly long for optimization.


function batchClaiming(address[] calldata tokenList) external {
    uint256 totalClaimable = 0;
    for (uint256 i = 0; i < tokenList.length; i++) {
        address token = tokenList[i];
        uint256 claimable = getClaimableFees(token, msg.sender);
        if (claimable > 0) {
            totalClaimable += claimable;
            tokensData[token].unclaimedFees[msg.sender] = 0;
            emit FeesClaimed(token, msg.sender, claimable);
        }
    }
    if (totalClaimable == 0) revert NoFeesToClaim();
    payable(msg.sender).transfer(totalClaimable);
}

In this update:

Reduced Storage Writes: The revised batchClaiming function reduces storage writes by updating unclaimedFees only when fees are claimed, reducing gas consumption.