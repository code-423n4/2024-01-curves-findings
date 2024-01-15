## Withdrawing a zero amount of tokens enables malicious users to take away other users the right to set a custom name and symbol for the external token

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L465-L488

The `withdraw` function is used to lock up the internal curves tokens to withdraw an external ERC-20 representation of it. If the owner of the curves token did not set a custom name and symbol for the external token, the default name and symbol are used.

This function does not check if the internal token was already initialized and instead checks if the user owns the tokens that the user wishes to withdraw. As the thought process was probably that the user could only withdraw if the internal token was already initialized because otherwise, the user would not have any tokens to withdraw. However, this does not prevent the user from withdrawing with a zero amount of tokens.

This allows malicious users to withdraw 0 tokens and deploy the external representation of the token before the token is even initialized. Therefore this trick can be used to remove the right of other users to set a custom name and symbol for the external token as it was already deployed with the default symbol and name.

Here we can see the withdraw function:

```solidity
function withdraw(address curvesTokenSubject, uint256 amount) public {
    if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

    address externalToken = externalCurvesTokens[curvesTokenSubject].token;
    if (externalToken == address(0)) {
        if (
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
            keccak256(abi.encodePacked("")) ||
            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
            keccak256(abi.encodePacked(""))
        ) {
            externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
            externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
        }
        _deployERC20(
            curvesTokenSubject,
            externalCurvesTokens[curvesTokenSubject].name,
            externalCurvesTokens[curvesTokenSubject].symbol
        );
        externalToken = externalCurvesTokens[curvesTokenSubject].token;
    }
    _transfer(curvesTokenSubject, msg.sender, address(this), amount);
    CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
}
```

## Asymetry inside `sellCurvesToken` function can lead to DoS

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L276-L279

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L327-L336

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L282-L293

The `buyCurvesToken` function calls the `_addOwnedCurvesTokenSubject` function which loops over an array of all the different tokens a user owns and adds a new one to it if not already owned.

Here we can see this flow:

```solidity
function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
    ...
    _buyCurvesToken(curvesTokenSubject, amount);
}

function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
    ...

    // If is the first token bought, add to the list of owned tokens
    if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
        _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
    }
}

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

The `sellCurvesToken` function on the other hand does not remove the token from the list if all tokens are sold. Therefore if a token was bought once it will stay inside this list forever and the list can only grow and not shrink. This increases the risk of DoS because of the block gas limit as the size of the array grows.

Here we can see that this symmetry is not given in the `sellCurvesToken` function:

```solidity
function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (supply <= amount) revert LastTokenCannotBeSold();
    if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

    uint256 price = getPrice(supply - amount, amount);

    curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
    curvesTokenSupply[curvesTokenSubject] = supply - amount;

    _transferFees(curvesTokenSubject, false, price, amount, supply);
}
```

## Min / max values should be implemented in the set fee functions to reduce centralization risks

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L117-L126

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L141-L153

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L166-L178

We can see in the `setMaxFeePercent` and the `setExternalFeePercent` function, that managers can set fees >= 100% of the buy price without any limits. Min/max checks for the given values should be added to reduce centralization risks.

The set fee functions:

```solidity
function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
    if (
        feesEconomics.protocolFeePercent +
            feesEconomics.subjectFeePercent +
            feesEconomics.referralFeePercent +
            feesEconomics.holdersFeePercent >
        maxFeePercent_
    ) revert InvalidFeeDefinition();
    feesEconomics.maxFeePercent = maxFeePercent_;
}

function setExternalFeePercent(
    uint256 subjectFeePercent_,
    uint256 referralFeePercent_,
    uint256 holdersFeePercent_
) external onlyManager {
    if (
        feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ >
        feesEconomics.maxFeePercent
    ) revert InvalidFeeDefinition();
    feesEconomics.subjectFeePercent = subjectFeePercent_;
    feesEconomics.referralFeePercent = referralFeePercent_;
    feesEconomics.holdersFeePercent = holdersFeePercent_;
}
```

A minimum check would also be important as otherwise setting a too low fee could round the fee to zero in the `getFees` function:

```solidity
function getFees(
    uint256 price
)
    public
    view
    returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
{
    protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
    subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
    referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
    holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
    totalFee = protocolFee + subjectFee + referralFee + holdersFee;
}
```

## Send the funds acquired from selling tokens to the seller at last and not at first to avoid reentrancy

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L229-L233

When a user sells tokens the funds which the user receives and the different fees which other addresses receive are calculated and transfered in the `_transferFees` function. The seller is the first which receive the ether and therefore can reenter this or any other contract. As states inside the `FeeSplitter` contract are updated after that, this could potentially lead to read-only reentrancy attacks in third-party protocols.

Here we can see the `_transferFees` function and that the seller is the `firstDestination` which allows reentering before the `feeRedistributor` is called which updates states inside the `FeeSplitter` contract:

```solidity
function _transferFees(
    address curvesTokenSubject,
    bool isBuy,
    uint256 price,
    uint256 amount,
    uint256 supply
) internal {
    (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
    {
        bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
        {
            address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
            uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
            uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
            (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
            if (!success1) revert CannotSendFunds();
        }
        {
            (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
            if (!success2) revert CannotSendFunds();
        }
        {
            (bool success3, ) = referralDefined
                ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                : (true, bytes(""));
            if (!success3) revert CannotSendFunds();
        }

        if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
            feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
            feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
        }
    }
    emit Trade(
        msg.sender,
        curvesTokenSubject,
        isBuy,
        amount,
        price,
        protocolFee,
        subjectFee,
        isBuy ? supply + amount : supply - amount
    );
}
```

## Merkle root should not be changeable after the presale started

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394-L402

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L404-L420

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L265

Subject token owners can set up a presale phase where only whitelisted accounts can buy tokens for a given amount of time. A Merkle root is used for that and it is possible to change the Merkle root after the presale started as a supply > 1 check instead of a supply > 0 check is used.

Here we can see that subject token owners can update the Merkle root if the supply of the token equals 1:

```solidity
function setWhitelist(bytes32 merkleRoot) external {
    uint256 supply = curvesTokenSupply[msg.sender];
    if (supply > 1) revert CurveAlreadyExists();

    if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
        presalesMeta[msg.sender].merkleRoot = merkleRoot;
        emit WhitelistUpdated(msg.sender, merkleRoot);
    }
}
```

Here we can see that users can already buy tokens if the supply equals 1:

```solidity
function buyCurvesTokenWhitelisted(
    address curvesTokenSubject,
    uint256 amount,
    bytes32[] memory proof
) public payable {
    if (
        presalesMeta[curvesTokenSubject].startTime == 0 ||
        presalesMeta[curvesTokenSubject].startTime <= block.timestamp
    ) revert PresaleUnavailable();

    presalesBuys[curvesTokenSubject][msg.sender] += amount;
    uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
    if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

    verifyMerkle(curvesTokenSubject, msg.sender, proof);
    _buyCurvesToken(curvesTokenSubject, amount);
}

function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

    ...
}
```

This can lead to race conditions where the owner wants to remove a user from the whitelist, while this user wants to buy tokens and one of the calls will fail depending on which one pays a higher gas fee. A supply > 0 check should be used instead.

## Presale should end before the public sale starts

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L409-L412

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L212-L213

Subject token owners can set up a presale phase where only whitelisted accounts can buy tokens for a given amount of time.

Here we can see that at starteTime == block.timestamp the presale is still available while the public sale is already open. As one function uses `<=` while the other one uses `>=`:

```solidity
function buyCurvesTokenWhitelisted(
    address curvesTokenSubject,
    uint256 amount,
    bytes32[] memory proof
) public payable {
    if (
        presalesMeta[curvesTokenSubject].startTime == 0 ||
        presalesMeta[curvesTokenSubject].startTime <= block.timestamp
    ) revert PresaleUnavailable();

    ...
}

function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
    uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
    if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

    ...
}
```

This does not raise a real security risk at the moment, but such off-by-one errors should be avoided by removing the `=` from one of these checks, as it could raise a security risk in future updates.

## The `transferOwnership` function should implement a 2-step process

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Security.sol#L27-L29

The `transferOwnership` function inside the `Security` contract allows to transfer the ownership of the contract to a new address:

```solidity
function transferOwnership(address owner_) public onlyOwner {
    owner = owner_;
}
```

As we can see it does that in a single step. This raises a security risk if the owner passes the wrong address accidentally. The chance is very low, but the impact of it very high as the admin privileges of the contracts are lost forever. Therefore a 2-step process should be implemented. See the `Ownable2Step` function from OpenZeppelin to get used to the pattern.

## Swapping between internal balances with full integers and external ERC-20 tokens with 18 decimals can lead to losses

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L491

The internal tokens that can be bought and sold are represented as full integers as in friends.tech. Curves allow to withdraw these tokens in an ERC-20 representation with 18 decimals and deposit them back to receive the internal version of it again.

The deposit function only allows to deposit a full integer amount of tokens:

```solidity
function deposit(address curvesTokenSubject, uint256 amount) public {
    if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

    ...
}
```

The chance that at least dust amounts are getting lost when the tokens are withdrawn and used in third-party protocols, transferred around, ... is high.

If so the last user who wants to deposit the ERC-20 tokens back into the contract to sell it does not have a full integer amount of tokens left and therefore is not able to deposit them back into the contract. This leads to a loss for this user as the tokens can not be sold anymore and the ether paid for it is stuck inside the contract.

It could also happen that a malicious user will keep 1 wei out of circulation intentionally to grief the last seller as this one will never be able to sell the token as 1 wei can nowhere be bought and therefore the last seller loses ether, or has to buy the 1 wei from the malicious user extremely overpriced.
