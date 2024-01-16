## L - 1 : Every Balance Change Push The Same Token to The Array

"onBalanceChange()" in FeeSplitter.sol is called whenever buyCurvesTokens() or sellCurvesToken() is called in Curves.sol which is every time some buy or sell action occurs (for msg.sender)
This function adds token address to userTokens arrays without checking if it exist or not:
```solidity
    function onBalanceChange(address token, address account) public {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }
```
Hence, every time user buy or sell from the same token, the same token address will be pushed to this array constantly making it bloat. This has 2 effects:
1. Functions "getUserTokens()" and "getUserTokensAndClaimables()" will return duplicate values:
```solidity
    function getUserTokens(address user) public view returns (address[] memory) {
        return userTokens[user];
    }

    function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
        address[] memory tokens = getUserTokens(user);
        UserClaimData[] memory result = new UserClaimData[](tokens.length);
        for (uint256 i = 0; i < tokens.length; i++) {
            address token = tokens[i];
            uint256 claimable = getClaimableFees(token, user);
            result[i] = UserClaimData(claimable, token);
        }
        return result;
    }
```
Hence these two functions will not return what is expected.
2. Protocol suggests using these functions before batchClaiming():
```solidity
    //@dev: this may fail if the the list is long. Get first the list with getUserTokens to estimate and prepare the batch
    function batchClaiming(address[] calldata tokenList) external {
```
As we can see with this exact reason the list will be very long if not assessed before calling batchClaiming.

### Recommendation
Instead of direct push, create a function that checks if the token is available in userTokens[], if not, only then push the token inside array.

## L - 2 : First Share Buy Should Be "1" Because of Math Underflow

Because of FriendTech's curve implementation, the first token buy by "subject" should be equal to 1. Any other tries to buy tokens will underflow and fail. Instead of expecting users to call the right value it is easy to solve this issue (not entirely but mostly).

### Recommendation
Since "buyCurvesTokenWithName()" and "buyCurvesTokenForPreSale()" functions are functions that will only be called once by "subject" in the first token buy. Instead of letting user to specify "amount" parameter, fix the amount to "1":
```diff
    function buyCurvesTokenForPresale(
        address curvesTokenSubject,
-       uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
    ) public payable onlyTokenSubject(curvesTokenSubject) {
        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply != 0) revert CurveAlreadyExists();
        presalesMeta[curvesTokenSubject].startTime = startTime;
        presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);

-        _buyCurvesToken(curvesTokenSubject, amount);
+        _buyCurvesToken(curvesTokenSubject, 1);
    }
...
    function buyCurvesTokenWithName(
        address curvesTokenSubject,
-       uint256 amount,
        string memory name,
        string memory symbol
    ) public payable {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply != 0) revert CurveAlreadyExists();

-       _buyCurvesToken(curvesTokenSubject, amount);
+       _buyCurvesToken(curvesTokenSubject, 1);
        _mint(curvesTokenSubject, name, symbol);
    }
```

## L - 3: Send Excess Ethers Back
buyCurvesToken() function and its derivatives are payable and receives ether in order to buy tokens. But in the internal function which complete the buy process, excess funds are not sent back and they are stuck in contract forever:
```solidity
    function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);

        if (msg.value < price + totalFee) revert InsufficientPayment();

        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
    }
```
### Recommendation 
If (msg.value > price + totalFee), add excess funds to the newly created mapping userExcessEther, and create a function for user to withdraw this excess ether. It is best to make this a two step process, otherwise low level call's can create further problems.

## L - 4: Unnecessary receive() function in FeeSplitter

FeeSplitter has a receive() function but there is no way to use ether that is received via ether transfer to this contract. Also there is no way to withdraw funds that are sended to contract. Either remove the receive() function or create a way to use them (if you plan to receive donations).