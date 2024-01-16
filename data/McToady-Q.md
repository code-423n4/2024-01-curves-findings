## Purchases are vulnerable to being sandwich attacked or griefed
	
**Description:** 
When someone calls `Curves::buyCurvesToken` they have no choice of acceptable slippage for their purchase. Could cause two issues.
1. If the user sends only the correct amount of ether with their `buyCurvesToken` transaction, if any purchase of the same token completes before the users transaction the users transaction will revert with a `InsufficientPayment` error as the price will have changed. If this is popular token that is being traded frequently this happening could be quite probably and ruin UX and cost them gas in reverted transactions.
2. If the user chooses to send a little more ether with their purchase then they A. won't get refunded the additional ether if the price doesn't change (or goes down) and B. would allow someone to sandwich attack their transaction, buying and selling tokens before/after the users tx is complete.

**Impact:** 
At the very least this approach is poor UX and will cause a lot of gas wasted on reverted transactions if there is a lot of volume on a token someone is trying to buy.
At the worst it would allow MEV bots to sandwich attack any users that send more than minimum requires amount for a purchase. 

**Proof of Concept:**
The following foundry test shows how a user (alice) can make a profit by just purchasing a token before another user (bob) and then selling it immediately afterwards.

```

	function testSandwichAttack() public {
		// create bob token
		createToken(bob);

		// have bob buy some initial tokens
		buyToken(bob, bob, 5);

		// have alice sandwich chads purchase
		uint256 aliceStartBalance = alice.balance;
		buyToken(alice, bob, 1);
		buyToken(chad, bob, 1);
		sellToken(alice, bob, 1);	
		uint256 aliceEndBalance = alice.balance;

		console2.log("Alice Start:", aliceStartBalance);
		console2.log("Alice End  :", aliceEndBalance);
	}
    

	function buyToken(address buyer, address subject, uint256 amount) public {
		vm.startPrank(buyer, buyer);
		uint256 price = curves.getBuyPrice(subject, amount);
		curves.buyCurvesToken{value: price}(subject, amount);
		vm.stopPrank();
	}

	function sellToken(address seller, address subject, uint256 amount) public {
		vm.startPrank(seller, seller);
		curves.sellCurvesToken(subject, amount);
		vm.stopPrank();
	}

```

**Recommended Mitigation:** 
It should be considered to use ERC20 tokens instead of Ether as it would be eaiser to set an allowance and then allow users to set a maximum price they're willing to pay. This way they will only revert if the price moves above this maximum price, but if the price remains the same (or goes lower) they only pay the current price.

As well as this it should be thought about whether the protocols bonding curve can be adjusted so a user cannot profit so much from sandwiching a single token purchase.

## Requiring that token symbols are unique could result in a gas race/frontrunning of popular symbols 

**Description:** 
The functions `Curves::_deployERC20` and `Curves::setNameAndSymbol` ensure that a symbol is not already registered to a subject before allowing the user to register them. This also causes a possible issue where a griefer could frontrun someone attempting to deploy a token with their own transaction deploying that token with the same ticker causing the initial transaction to revert with the error `InvalidERC20Metadata`. 

**Impact:** 
Griefering being able to camp symbols and even frontrun users attempting to register tokens with specific symbols will lead to worse UX & potentially lead to legitimate users wasting gas on transactions that will revert.

**Proof of Concept:**
Two ways this can be griefed.

Registering 'popular' tickers
1. The protocol deploys and goes live.
2. Greifing user uses multiple addresses to deploy tokens and permanently stop other uses using the same symbol ("ETH", "BTC", "PEPE", "COBIE" etc etc)

Frontrunning users to make their transactions revert
1. Legitimate user Bob tries to launch token with the symbol "BOB"
2. Griefer Alice sees Bob's transaction in the mempool and submits the same transaction with more gas to take the "BOB" symbol from him.
3. Bob's transaction reverts, has to submit a new transaction and find a new ticker.

**Recommended Mitigation:** 
Consider if it's necessary for every symbol to be unique. While in theory it would be easier for users if they can search for a users token by their unique symbol, in reality it might actually become more difficult if the tickers you assume would be those of prominent users have been swept up by griefing users.

## Several functions within the Curve contract have unclear naming

Examples:
`Curves::mint` - Unclear who can call this and what is being minted.
`Curves::buyCurvesTokenForPresale` - This function can only be called by a subject to buy their own first token. 
`Curves::buyCurvesTokenWithName` - This function can also only be called by a subject to buy their own first token.

## Function Curves::verifyMerkle is marked a public view but doesn't return a value.

For clarity public view function should return a value for user clarity. In this instance it would be better for the function to return a boolean of the proofs verification rather than reverting if `MerkleProof::verify` returns false.
Mitigation:
```

    function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view returns(bool) {
        bytes32 leaf = keccak256(abi.encodePacked(caller));
        return MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf));
    }
    
```

## Multiple unnecessary function parameters
Following functions:
- `Curves::setReferralFeeDestination`
- `Curves::buyCurvesTokenForPresale`
- `Curves::setNameAndSymbol`
- `Curves::mint`

All 4 of these functions take a parameter `address curvesTokenSubject` but user the modifier `onlyTokenSubject` which requires that `curvesTokenSubect` is msg.sender. So the parameter is unecessary as it must always be msg.sender or the transaction will revert.

On top of this `Curves::mint` calls `Curves::_mint` which also uses the `onlyTokenSubject` modifier to check this again.

The function `Curves::buyCurvesTokenWithName` also has the parameter `address curvesTokenSubject` and later calls `Curves::_mint` making the `curvesTokenSubject` parameter in this function unnecessary as well.

**Mitigation**
If a user can only call these functions for their own instance of the Curves token then `curvesTokenSubject` must always be msg.sender in a function call therefore it's safe to remove the parameter and replace uses of it with msg.sender as well as the modifier `onlyTokenSubject`. This will clean up the code & ssave users gas by removing unnecessary logic. 

The function `Curves::setWhitelist` already follows this pattern of naturally assuming msg.sender is the `curvesTokenSubject` in question.

## Users receive holder fees for their own purchase

How it is set up currently, when a user buys a token they will receive fees for the tokens purchased in that transaction. This essentially works out as a discount on purchase, but one that requires them to then claim the excess fees manually.
**PoC**
The below shows that a user buying 5 tokens will be eligible to claim fees for that transaction.

```

	function testUserGetsFeesOnPurchase() public {
		// make sure the FeeSplitter has the Curves instance set
		fees.setCurves(curves);
		// initialise some fees
		curves.setMaxFeePercent(1e14);
		curves.setExternalFeePercent(2e13,2e13,2e13);

		// create token as alice
		createToken(alice, "ALICE");
		
		// buy tokens as bob
		buyToken(bob, alice, 5);
		
		uint256 aliceFeesEarned = fees.getClaimableFees(alice, alice);
		console2.log("aliceFeesEarned", aliceFeesEarned);
		uint256 bobFeesEarned = fees.getClaimableFees(alice, bob);
		console2.log("bobFeesEarned", bobFeesEarned);

		// confirm bob got fees for his purchase
		assert(aliceFeesEarned * 5 == bobFeesEarned);
	}

```

**Mitigation**
If this is not intended it's better to calculate the fees prior to updating supply and balance of the new purchasing user.

## CurvesERC20Factory::deploy function lacks any access control could cause user error

The function `CurvesERC20Factory::deploy` has public visibility and no access controls meaning anyone can deploy their own `CurvesERC20` contract. This may make users mistakenly believe that deployed token would be linked to the Curves protocol itself when this is not the case.

**Mitigation**
Add some access control to this function that only allows the function to be called by `Curves::_deployERC20`. 

## Redundant condition in Curves::buyCurvesToken

See the following line from `Curves::buyCurvesToken`:
```

        uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
        if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

```
If `startTime == 0` then `startTime >= block.timestamp` will always be true so the `startTime != 0` condition is redundant and can be removed.

## Curves::buyCurvesTokenForPresale should emit a Curves::WhitelistUpdated event

For events to be trusted all changes of state linked to the event should emit the event in question. `Curves::setWhitelist` already emits this event, so it's unclear why it's not emit by `Curves::buyCurvesTokenForPresale` which also allows the setting of a subject presale Merkle root.

## Project lacks zero address/bytes checks to validate user inputs

Checking address or bytes inputs by trusted/untrusted accounts are not zero is a useful sanity check to prevent user error. This is particularly important in cases where it would be impossible to revert the error such as mistakenly entering nothing as the address parameter when calling `Security::transferOwnership`


## `Curves::_buyCurvesToken` doesn't follow the checks, effects, interactions pattern

In the function `Curves::_buyCurvesToken` external calls to untrusted addresses are made in `Curves::_transferFees` before later calling `Curves::_addOwnedCurves`. Although the usage appears to be safe in this instance it's best practice to avoid this wherever possible.


## The FeeSplitter contract unnecessarily instantiates `Security` in it's constructor

`constructor() Security() {}`
As security requires no arguments it is not necessary to include it in `FeeSplitter`'s constructor.


## The FeeSplitter contract has a payable receive function unecessarily

FeeSplitter having a receive function makes it more likely a user could mistakenly send eth to this contract which would be impossible to retreive. Unless there is a clear reason why a receive function is necessary it's best to remove itto avoid chances of user error.


## Shadowing of an inherited variable name in the CurvesERC20 constructor

See the code below:
```

   constructor(string memory name_, string memory symbol_, address owner) ERC20(name_, symbol_) {
        transferOwnership(owner);
    }

```

The variable name owner is used in the `Ownable` contract inherited by `CurvesERC20` although this doesn't cause any issues is decreases the code readability therefore it is best practices to avoid this and give the parameter a unique name.


## None of the parameters in the Curves::Trade event are indexed

In the event `Curves::Trade` no fields are marked as indexed making it difficult for trade infomation to be collected off chain.
Consider marking fields `address trader` and `address subject` as `address indexed trader` and `address indexed subject` respectively. Marking the fields indexed will allow for easier offchain tracking of trades made by a specific trader or the trades of a specific subjects token.

## The deployed `Curves` instance should be set as `manager` of `FeeSplitter` during deployment

If the contract goes live without the `Curves` contract being set as a `manager` of the `FeeSplitter` contract any attempt to buy or sell tokens will fail with the error `NotManager`.
To avoid the possibility of this the protocol should ensure a thorough deployment script is prepared to ensure all contracts are initialized correctly. This should be highlighted because the current issue with `Security::onlyManager` means that this isn't yet an issue, but will become one once that contract is fixed.
As `Curves` should be the only address to call `FeeSplitter::onBalanceChange` and `FeeSplitter::addFees` it would be preferable to have an `onlyCurves` modifier rather than `onlyManager` such as this:
```

	modifier onlyCurves() {
		require(msg.sender == address(curves), "Not Curves Address");
		_;
	}

	function onBalanceChange(address token, address account) external onlyCurves {}
	function addFees(addres token) external payable onlyCurves {}

```

This would make the code more readable as it's clear that these functions should only be called by the associated `Curves` contract. It would also remove the ability for rogue accounts with the `manager` role to call these functions maliciously.