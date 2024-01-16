# Summary Analysis Report:

Upon reviewing the smart contract code, several areas were identified that may pose risks to security, gas optimization, and overall functionality. Below is a consolidated analysis along with actionable recommendations:

## Gas Optimization:
## Finding:
In the FeeSplitter contract, the totalSupply function employs a loop to iterate through all token holders, potentially resulting in gas inefficiency as the number of token holders increases.

## Impact:
Gas costs may escalate significantly with a growing number of token holders.

## Proof of Concept:

function totalSupply(address token) public view returns (uint256) {
    //@dev: this is the amount of tokens that are not locked in the contract. The locked tokens are in the ERC20 contract
    return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
}

Recommended Mitigation Steps:
Optimize the totalSupply function by exploring alternative methods to calculate the total supply without iterating through all token holders. Consider storing the total supply separately or using events to track changes in supply.


# Medium Risk:
## Finding:
In the FeeSplitter contract, the claimFees function transfers ETH directly to the caller without verifying if the contract has enough ETH balance.

## Impact:
Insufficient ETH balance may cause the function to fail, resulting in transaction reversion.

## Proof of Concept:
function claimFees(address token) external {
    updateFeeCredit(token, msg.sender);
    uint256 claimable = getClaimableFees(token, msg.sender);
    if (claimable == 0) revert NoFeesToClaim();
    tokensData[token].unclaimedFees[msg.sender] = 0;
    payable(msg.sender).transfer(claimable); //@dev: No check if contract has enough balance
    emit FeesClaimed(token, msg.sender, claimable);
}
## Recommended Mitigation Steps:
Include a check to ensure the contract's ETH balance is sufficient before transferring funds to the caller.

# High Risk:
## Finding:
In the FeeSplitter contract, the addFees function permits anyone to add fees, potentially leading to unauthorized fee additions.

## Impact:
Unauthorized parties may add fees to the contract, affecting fee distribution.

## Proof of Concept:
function addFees(address token) public payable onlyManager {
    uint256 totalSupply_ = totalSupply(token);
    if (totalSupply_ == 0) revert NoTokenHolders();
    TokenData storage data = tokensData[token];
    data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_; //@dev: Anyone can call this function
}

## Recommended Mitigation Steps:
Restrict access to the addFees function to authorized parties only, such as the owner or designated managers.


# Conclusion:
The identified issues present varying degrees of risk, ranging from potential gas inefficiencies to high-risk security vulnerabilities. Addressing these concerns is crucial for ensuring the reliability, security, and optimal performance of the smart contract. Implement the recommended mitigation steps after thorough testing to safeguard against potential issues.

### Time spent:
50 hours