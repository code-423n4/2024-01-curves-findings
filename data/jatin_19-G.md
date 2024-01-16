# Finding:
In the FeeSplitter contract, the totalSupply function uses a loop to iterate through all token holders, which can be gas-inefficient if there are a large number of token holders. This could lead to high gas costs.

# Impact:
Gas costs may increase significantly as the number of token holders grows.

# Proof of Concept:

function totalSupply(address token) public view returns (uint256) 
{
    //@dev: this is the amount of tokens that are not locked in the contract. 
    //The locked tokens are in the ERC20 contract
    return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
}

# Recommended Mitigation Steps:
Consider optimizing the totalSupply function by finding alternative ways to calculate the total supply without iterating through all token holders. This could include storing the total supply separately or using events to track changes in supply.