### [L-01] Curves.sol `_addOwnedCurvesTokenSubject()` runs the risk of out of gas error, if the user buys too many curve tokens

Every time a user buys a curve token from a new curve token subject, the address is added to the `ownedCurvesTokenSubjects` mapping. If the user buys many curve tokens from many curve token subjects, the length of the array of new token subjects will continue to grow larger, to a point where it may be too large that it causes an out of gas error.

For example, if the user buys a new curve token from 5000 people (which is pretty understandable since the protocol emphasizes on social networking), the function will have to loop 5000 times before pushing a new curves token subject.

```
function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
        address[] storage subjects = ownedCurvesTokenSubjects[owner_];
>       for (uint256 i = 0; i < subjects.length; i++) {
            if (subjects[i] == curvesTokenSubject) {
                return;
            }
        }
        subjects.push(curvesTokenSubject);
```

Not exactly sure what the exact purpose of `_addOwnedCurvesTokenSubject()` is, since the mapping is private and cannot be queried easily by the user.

```
mapping(address => address[]) private ownedCurvesTokenSubjects;
```

The original friend tech code also does not have this function.

A recommended solution is to skip the for loop, and have another function that pops the array length. Whenever theres a new curve token bought, add it to the array. When all of the new curve token of a particular curve token subject is sold, pop the array. This way, there is no need to check every single address in the array.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L328-L335

### [L-02] Curve tokens are added to the list of owned tokens but are not removed when sold

When the user buys a token, if it is the first token bought, `_addOwnedCurvesTokenSubject()` will be called to add the token address into the `ownedCurvesTokenSubjects` mapping.

```
  // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
```

The purpose of this addition is to check which curve token a user owns. If the user decides to sell all the token, the `ownedCurvesTokenSubjects` mapping will not delete the token address. For coherence, the protocol should remove the curve token address from the `ownedCurvesTokenSubjects` mapping if the user decides to sell all of a particular curve token.

```
  //@audit - Since there is a _addOwnedCurveTokens function, there should also be a _removeOwnedCurveTokens function, which does the inverse (popping the index of the particular token in the array).
  // Internal function to add a curvesTokenSubject to the list if not already present
    function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
        address[] storage subjects = ownedCurvesTokenSubjects[owner_];
        for (uint256 i = 0; i < subjects.length; i++) {
            if (subjects[i] == curvesTokenSubject) {
                return;
            }
        }
        subjects.push(curvesTokenSubject);
    }
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L327-L337

### [L-03] Excess msg.value is not refunded when buying curve tokens

When buying a curve token from a curve token subject through `_buyCurvesToken()`, the buyer must set a msg.value to buy the token. This value is not refunded and if the buyer sets a higher than normal amount, the excess msg.value will not be refunded. This value will be stuck in the contract since there is no way to extract excess msg.value.

```
  function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);
        //@audit - msg.value can be more than price + totalFee
>       if (msg.value < price + totalFee) revert InsufficientPayment();
```

It is mentioned in the medium article:

> If the buyer sends more Ether (msg.value)than the total cost of the transaction (i.e., price + protocolFee + subjectFee), the contract does not include a mechanism to refund the excess Ether back to the buyer.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263-L280

### [L-04] Buying/selling share tokens is vulnerable to sandwich attacks since no slippage and deadline is given

Anyone can buy a share from a particular share subject. Everytime a share is bought, the price of the share will increase. 

```
        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);
```

A malicious user A can conduct a sandwich attack on a vulnerable user B. User A notices that user B intends to buy just 1 share, and so he frontruns user B transaction and buys 10 shares first. After user B buys his share, user A will sell the 10 shares that he bought. 

Granted, it may not be useful for anyone to conduct a sandwich attack because the returns might not be profitable since the attacker has to pay fees as well and the incremental price of each supply is not that far off from each other. Also, I assume that the frontend will calculate the msg.value for the user first before the user transacts. 

```
        if (msg.value < price + totalFee) revert InsufficientPayment();
```

The more prominent issue is when a user decides to sell a share and is frontrunned by a whale selling his share. Since there is no msg.value or any minAmountOut input, the seller will get what is returned from the function.

For example, there are 1000 supply, and user A has 1 supply. He decides to sell his share, which is worth 5 ether. Before he sells, he is frontrunned by a whale with 500 shares. When it was user A's turn to sell his share, he was only given 2 ether, instead of 5 ether.

Consider adding a minAmountOut parameter / deadline when selling tokens so that the user can decide not to sell the token if the price is too low. 

The deadline also matters because if the user tries to sell a token but gives a low gas fee, the function will be in the mempool for a long while. By the time the function is executed, the price may be too low.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263-L270

### [L-05] The last person that owns a share cannot sell the share. However, he can tokenize the last share and sell it on external markets.

This is a common known issue in friend.tech, where the last share cannot be sold. Not sure about the reason why, but the economic damage caused will not amount to much since its the last (first) share of the subject. 

```
   function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply <= amount) revert LastTokenCannotBeSold();
```

The last person can sidestep the issue by tokenizing the last share and selling it on external market.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L284

### [L-06] Curves.setwhitelist will not work

`setwhitelist()` lets the token subject set the whitelist of the presale. The function makes sure that the token must be 0 before the whitelist can be changed.

```
    function setWhitelist(bytes32 merkleRoot) external {
        uint256 supply = curvesTokenSupply[msg.sender];
>       if (supply > 1) revert CurveAlreadyExists();
```

This implies that `setWhiteList` will be called before `buyCurvesTokenForPresale()` , because `buyCurvesTokenForPresale()` will set up the presale and let the curve token subject buy the first token. Notice that the merkle root is already set in `buyCurvesTokenForPresale()`.

```
  function buyCurvesTokenForPresale(
        address curvesTokenSubject,
        uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
    ) public payable onlyTokenSubject(curvesTokenSubject) {
        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply != 0) revert CurveAlreadyExists();
        presalesMeta[curvesTokenSubject].startTime = startTime;
        //@audit: merkleRoot is already set here
>       presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
        //@audit: supply will not be zero after this line
>       _buyCurvesToken(curvesTokenSubject, amount);
    }
```

After `buyCurvesTokenForPresale()`, the supply will not be zero, so set whitelist will not work anymore. Also, even if the token subject calls `setWhiteList()` first, the merkleroot will be overwritten in `buyCurvesTokenForPresale()`.

It is probably fine to remove the supply check in setWhitelist, because it is the token subject's own whitelist and thus he should be able to change the merkleroot at any time for the presale.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394-L401

### [L-07] The token subject himself should be exempted from the maxBuy limit

For the presale, the token subject has the first opportunity to buy his own shares when he is setting up the presale using `buyCurvesTokenForPresale()`. Afterwards, users will call `buyCurvesTokenWhitelisted()` to buy from the presale.

If the token subject intends to buy more tokens, he can also call `buyCurvesTokenWhitelisted()`, provided he set his own address in the merkle tree.

However, the token subject himself is limited to the `maxBuy` limit when buying through `buyCurvesTokenWhitelisted()`.

He is not limited to the `maxBuy` limit when first buying through `buyCurvesTokenForPresale()`.

To keep it consistent, either restrict the token subject for both functions to the `maxBuy` limit, or allow the token subject to bypass his own `maxBuy` limit in `buyCurvesTokenWhitelisted()`.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L416

### [L-08] There's no upper limit when setting the fees percentage

When setting the fees percentage, there's no upper limit. `maxFeesPercent` also has no upper limit, so the percentage can be more than a 100%. 

```
  function setExternalFeePercent(
        uint256 subjectFeePercent_,
        uint256 referralFeePercent_,
        uint256 holdersFeePercent_
    ) external onlyManager {
        if (
            feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ >
            feesEconomics.maxFeePercent
        ) revert InvalidFeeDefinition();
>       feesEconomics.subjectFeePercent = subjectFeePercent_;
>       feesEconomics.referralFeePercent = referralFeePercent_;
>       feesEconomics.holdersFeePercent = holdersFeePercent_;
    }

```

Set a maximum fee percentage to all the percent to reduce centralization risk, eg <1e17 (must be below 10%) and max fee must be <2e17 (must be below 20%)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L141-L153


### [L-09] Curves.setNameAndSymbol will be overwritten every time


Similar to the setWhitelist issue, `setNameAndSymbol()` can only be called when `externalCurvesTokens[curvesTokenSubject].token == address(0)`.

```
  function setNameAndSymbol(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) external onlyTokenSubject(curvesTokenSubject) {
        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
```

Before the token is deployed, if `setNameAndSymbol()` is called, the name and symbol of the externalCurvesTokens[curvesTokenSubject] will be changed. 

When the token is deployed, the name and symbol will be changed. The token variable will also be set to the tokenContract.

```
    function _deployERC20(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal returns (address) {
        // If the token's symbol is CURVES, append a counter value
        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
            _curvesTokenCounter += 1;
            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
        }

        if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

        address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

>       externalCurvesTokens[curvesTokenSubject].token = tokenContract;
>       externalCurvesTokens[curvesTokenSubject].name = name;
>       externalCurvesTokens[curvesTokenSubject].symbol = symbol;
```

This means that when the token is deployed, `setNameAndSymbol()` cannot be called anymore. 

After `setNameAndSymbol()` is called, when the token is deployed, the name and symbol will be overwritten.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L428-L437

### [L-10] Tokenizing a curve token will mess up the accounting of data.cumulativeFeePerToken in FeeSplitter.sol

The totalSupply of a token does not count those token that are tokenized.

```
    function totalSupply(address token) public view returns (uint256) {
        //@dev: this is the amount of tokens that are not locked in the contract. The locked tokens are in the ERC20 contract
        return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
    }
```

This `totalSupply()` funciton is used in `addFees()`, which calculates the cumulativeFeePerToken.

```
 function addFees(address token) public payable onlyManager {
        uint256 totalSupply_ = totalSupply(token);
        if (totalSupply_ == 0) revert NoTokenHolders();
        TokenData storage data = tokensData[token];
>       data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_;
    }
```

Let's say 1000 token is bought, and 100 of them becomes tokenized, the totalSupply will then become 900 instead of 1000.

Taking an extreme, if 999 tokens become tokenized the total supply left will be one. The `cumulativeFeePerToken` will increase at a much quicker rate since the denominator is lower.

When the 999 token become untokenized, `getClaimableFees()` will count the owed fees wrongly because the userFeeOffset is too low and the `cumulativeFeePerToken` is too high.

```
  function getClaimableFees(address token, address account) public view returns (uint256) {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
>       uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
        return (owed / PRECISION) + data.unclaimedFees[account];
    }
```

Consider taking all the total supply instead of differentiating the tokenized and untokenized share.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L43-L46