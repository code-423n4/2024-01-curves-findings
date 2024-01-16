Analysis Report :-

**Index of table**
| Sl.no  | Particulars  |
|---|---|
| 1   | Overview  |
| 2  | Architecture view(Diagram)  |
| 3  | Approach taken in evaluating the codebase  |
| 4  | Centralization risks  |
| 5  | Mechanism review  |
| 6  | Recommendation   |
| 7  | Hours spent  |


## 1. Overview
Cruves is a protocol that aims to further decentralize Socialfi. It empowers users to got more involved in the protocol with it's new innovative features such as a fee distribution amongst holders of a subject - also known as "shares" from Friendtech. Curves takes inspiration from Friendtech's implementation of SocialFi while also building upon the core code of Friendtech. The referral & token holder fee feature is especially unique to SocialFi as this incentivises holding of shares rather than regular market speculation of them as well as recognizing referrals as an integral part of adoption thereby incentivizing such users. It's smart contracts goes beyond simply buying and selling tokens also known as "subjects". It explores an alternative usage method for such tokens by allowing them to be exported outside of the protocol in the form of ERC20s that can be used anywhere. This further makes curve tokens interoperable giving it a more usage path for users. Curve tokens are however only owned in single whole units. But nonetheless, these tokens can be seamlessly transferred out of the protocol and back in.


## 2.  Architecture view(Diagram)

![Excalidraw FlowChart](https://rexjoseph.github.io/images/curves-diagram.png)

Please Visit the link to the full diagram view :-

https://rexjoseph.github.io/images/curves-diagram.png


## 3. Approach taken in evaluating the codebase 
We employed manual review in looking at each in-scope contracts of the codebase - exploring core functions.

Though time consuming, it is interesting how much you can find in a minimal codebase.

The main contracts are :-
1.Curves.sol
2.CurvesERC20.sol
3.CurvesERC20Factory.sol
4.FeeSplitter.sol
5.Security.sol

**1.Curves.sol**
This is the core contract of the protocol. It's houses the entry & exit points for users in the system. With 432 lines of code, it does a ton more than you would expect compared to Friendtech. As a subject, this is the contract you go to in order to begin trading activity for your token, as a user, this is also the same contract you interact with to purchase a subject token or shares. Think about it as a powerhouse where you and your friend (subject) do business. We will explore some crucial functions and variables that exist in this contract.

State variables are :-
`address public curvesERC20Factory` stores the ERC20 token factory adress.
`feeRedistributor` stores the FeeSplitter whose methods we interact with such as the `addFees` method
`uint256 private _curvesTokenCounter` stores & keeps track of curve token subject supply.
`externalCurvesToSubject` mapping of external curves to the subjects.
`symbolToSubject` mapping of curves token sybmols to the specific subjects.
`presalesBuys` mapping of addresses of a user to the amount of tokens they bought of a subject's presale (if the token sale model utilized by the subject is a presale one)
`feesEconomics` defines and stores percentages of fees all 4 fee receivers get.
`curvesTokenSupply` mapping of how much supply exists for a curve subject.
`curvesTokenBalance` mapping of how much balance a holder has for a specific curve subject
`ownedCurvesTokenSubjects` give it an address and it returns an array of the curve token subjects owned for that address
`event Trade` fired when a buy/sell trade occurs
`event Transfer` fired when users transfer curve subjects between themselves.


Some core functions are:-
`setFeeRedistributor()`
This function should only be called by the `onlyOwner` role. It set's the `feeRedistributor` implementation contract which is crucial for distribution of holder fees.

`setMaxFeePercent()`
Should only be called by the `onlyOwner` role. Pass it a valid percent argument e.g 1e16 ~ 10% and it caps the total fees that go to the protocol, subject, referral, and holders at 10% of a trade's revenue/amount.

`setReferralFeeDestination()`
Should only be called by the curve token subject in question to set which address the referral fee goes to. 

`getBuyPrice()`
This function leverages one more function which is the `getPrice()` function to calculate how much a curve token is worth at that time of calling the function. It returns the linear price of the curve token which is then used in buying and selling of curve tokens.

`getSellPrice()`
Same functionality as the `getBuyPrice()` function but instead used to compute sell prices rather than buys. Both functions, `getBuyPrice()` & `getSellPrice()` uses the total supply of a specific curve token subject to calculate it's returned price.

`getBuyPriceAfterFee()`
This function gets the buying price of a curve token subject while also factoring in the fees.

`getSellPriceAfterFee()`
Same functionality as `getBuyPriceAfterFee()` but used for sells rather than buys.

`buyCurvesToken()`
Crucial function of the protocol, this is the entry point at which your balances of a curve token subject increases. It takes two arguments; `address curvesTokenSubject` & `uint256 amount`. 
This function then calls `_buyCurvesToken` after a pre-check which is making sure if the token presale start time for the subject has indeed been reached. `_buyCurvesToken` further does a pre-check to make sure only the subject can buy the first token. Then it computes the price for the specified amount to buy, ensures the ether sent for the buy is indeed enough to cover the price + fee (although it does nothing to refund excess ether in the case you end up sending more. So those ends up locked in the contract). Then it logs and stores your token balance for the curve subject after which it trasfers fees to the various destinations utilizing the `_transferFees` function. If it's the first time you purchase the curve subject, it calls the `_addOwnedCurvesTokenSubject` to add to your list of owned curves subject which should be 0 at that point until now added. It is important to note that this function sets the first destination for fee distribution as the protocol in the case of buys and set's it to the msg.sender (caller) in the case of sells.


`sellCurvesToken()`
Another crucial function of the protocol, similar logic to `buyCurvesToken` but different in the sense that you don't send ether because you are selling and so expect to recieve ether. You also get to be set as the first destination rather than that being the protocol, curves. The function reduces your token balance of the specific curve token subject by the specified sale amount. Then proceeds to transfer fees to all parties by calling the `_transferFees`. At the end of the day, this function call brings you ether.

`transferCurvesToken()`
Another crucial function is this one which allows you to transfer your curve tokens between friends, allies or just regular addresses. It transfers the specified amount of a specified curve subject to the receiving `to` address decrementing your balance and increasing theirs. Another function similar to this but for transferring all your curve token subjects in one go is the `transferAllCurvesTokens` which is particularly handy for reducing gas fees across transfers by sending everything in one go.

`buyCurvesTokenWithName()`
Similar functionality to the `buyCurvesToken()` function but allows you to buy the token and mints the ERC20 as well whereas `buyCurvesToken()` also buys without minting the ERC20. It is worth nothing that only the subject in question can call this function in order for it to pass and is called only once because the supply must be 0 during the call.

`buyCurvesTokenForPresale()`
Important function to set a presale metadata for a curve token while also allowing the subject in question to kick-off the buys for such token. It takes several inputs such as the address curvesTokenSubject, uint256 amount, uint256 startTime (in seconds), bytes32 merkleRoot (for verification & auth of buys in presale), uint256 maxBuy (maximum amount to be bought during the presale) which are then used to set the `presaleMeta` for the `curveTokenSubject`

`setWhitelist()`
Allows the curve token subject to set a new `merkleRoot` for the presale if they've not before or look to change it.

`buyCurvesTokenWhitelisted()`
This is the function that allows users to participate in a whitelist or presale of a subject token. It does some checks such as ensuring that the `startTime` for the presale has indeed been reached, makes sure a user's buy won't breach the `maxBuy` set for presale for that token then proceeds to verify the merkle supplied by the user is valid at which point it then allows the token purchase of the curve token having verified that the merkle root is indeed valid. 

`setNameAndSymbol()`
A convenient method for the token subject to set a name and symbol for their curve token at a later time.

`deposit()`
Handles the logic for migrating users back into the protocol in the case they transferred out their token before. It burns the curve token subject from the caller and logs their balance in the internal accounting system to reflect their transferred-in amount.

`withdraw()`
Similar function to the `deposit()` but slightly different as it allows the migration of a curve token subject outside of the protocol at which point it mints the ERC20 equivalent to the caller.


**2. CurvesERC20.sol**
This smart contract houses the ERC20 token contract for the curve token. This is a regular ERC20 implementation and uses Openzeppelin's implementation. It also uses the Ownable library to transfer ownership to the `CurvesERC20Factory.sol` contract which handles deployments of this `CurvesERC20` contract and protects sensitive functions such as `mint` && `burn` enforcing only the owner can perform these crucial actions.

**3. CurvesERC20Factory.sol**
Simple contract that handles the deployments of the CurvesERC20 token. This contract is also the owner of any deployed CurvesERC20.


**4. FeeSplitter.sol**
This contract inherits the `Security` contract which houses roles and access controls. It has some variables and functions that are crucial to the Curves contract. It is the fee allocation contract that users can interact with to claim fees, the curves contract interacts with to log and send fees delegated to holders.

Some variables are the `curves` variable that stores the curves contract and the `PRECISION` variable set to 1 ether or 1e18.

Core functionalities are :-
`balanceOf()`
This function returns an account's balance of a curve subject token. It's a public view function which can be called standalone and is also used across several functions such as `getClaimableFees`, `updateFeeCredit` and `onBalanceChange`

`totalSupply()`
This function returns the total amount of a curve subject token locked in the ERC20 contract.

`getUserTokens()`
This function returns the total amount of a curve subject token that a user holds.

`getUserTokensAndClaimable()`
Returns an array of the claimable fees for each curve subject token that a user holds.

`getClaimableFees()`
Pass it an address of a curve token subject and an address of a user/holder and the function returns the unclaimed fees for the user/holder for that specific curve subject token.

`claimFees()`
A crucial function that has a flaw when combined with `onBalanceChange()` in the sense that any fee unclaimed before the former becomes reset once `onBalanceChange`` is called. This function `claimFees` however allows a user to claim fees that have been accrued to them for a subject token.

`addFees()`
This function exposes an endpoint/method for the curves contract to transfer holder fees into the `FeeSplitter` contract. Without this function, the `FeeSplitter` contract would have no eth balance to be split amongst holders. It can only be called by a whitelisted manager though and such manager can be the curves contract ideally.

`batchClaiming()`
Similar logic for `claimFees()` but allows bundling the action across multiple subjects in one transaction. Really handy function that tries to mitigate heavy gas costs.

**5.Security.sol**
This contract plays a vital role in the curves protocol by declaring and implementing the access controls required to interact such as the `owner` & `managers` state variables.

It has two crucial modifiers such as `onlyOwner` and `onlyManager`. These functions have criticial issues that will be fixed as a result of this audit in the sense that both are useless because of a slight implementation error which flaws them of their use case. For example, only a whitelisted manager address should be able interact with `onlyManager` protected methods. And the `onlyOwner` modifier enforces that only the owner can indeed transfer ownership as well as set a manager address.

Core function are :-
`setManager()`
This function allows the owner of the contract to set/enlist a manager.

`transferOwnership()`
This function allows the owner to transfer ownership of the contract to another address. However, it lacks a 2-step transfer method which can end up being a single point of failure as a result of this lack.

Upon deployment of this contract, the owner is set to the deployer address. The deployer/msg.sender is also set as a manager of this contract.


## 4. Centralization risks
The functions listed below have `onlyOwner()`, `onlyManager()`, `onlySubject()` modifiers that can cause issues if such addresses are compromised by various factors.
[`setFeeRedistributor()`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L113)
[`setMaxFeePercent()`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L117)
[`setProtocolFeePercent()`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L128)
[`setExternalFeePercent()`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L141)
[`setERC20Factory()`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L162)
[`setReferralFeeDestination()`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L155)


## 5. Mechanism review 
After the review of this codebase, we concluded that the Curves protocol reimagines the Friendstech protocol, while adding a couple new features non-existent in the Friendstech protocol such as the ability to mint your subject token in it's ERC20 format, allowing the fees to also go to holders and referrals which boost adoption of the Curves protocol as well as keeps users engaged with the holder fee percentage.

This innovative platform will serve as a socialfi platform for people of all kind such as artists, regular friends, reputable people in the crypto space etc., to gather a following of holders and builders. We made a few comparison to the Friendstech protocol below:

**Comparision between Curves and Friendstech**
| Feature  | Curves  | Friendstech  |
|---|---|---|
| Goal  | A more decentralized Socialfi protocol empowering all parties involved in the community  | A Socialfi protocol empowering subjects and the protocol  |
| ERC20 Tokens  | Subject tokens/shares in ERC20 form to be used across ecosystems  | Just subject tokens with no ERC20 equivalents  |
| Holder Fee Allocation  | Pays out a percentage of fees to holders of a subject token  | No fee incentive for holding shares  |
| Referral Fee Allocation  | Pays out a percentage to referrals to boost adoption of the protocol  | No such incentive exists and only focuses on protocol and subject fee distribution  |
| Benefits for Holders  | Keep holding the token, enjoy the fee distribution  | No such incentive and regularly sees bots engineering the trading activity  |
| Accessibility  | To everyone who wishes to participates  | Initially only availabale to a few  |
| Token Acquisition  | Allows presales for subjects to enlist how much should be available to pre-sale participants  | No such similar model employed  |

## 6. Recommendation

After the completion of this audit we have some thoughts and recommendations for the Curves protocol team to implement.

1. Pricing model.
The price of curve subject tokens increases incredibly fast. The first token is always free for the subject token owner then the price rapidly increases afterwards starting from 0.0000625 for the lucky second buy. To facilitate 200 tokens buys of the curve subject the cost will be 200 * 200 / 16000 * 1e18 = 2.5 ether - that's a crazy amount for only just the 200th token. The 1000th token will cost 62.5 ether. With this model, only a few tokens will be sold meaning few people can afford to hold these tokens.
 The pricing model can be seen below which facilitates this:

```solidity
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}
```

Then this does the calculation taking account of the curve token supply for the specific subject:
```solidity
function getBuyPrice(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
        return getPrice(curvesTokenSupply[curvesTokenSubject], amount);
    }
```

We understand this is by design but the pricing will be expensive to entice a ton of users to keep the supply of a token steadily increasing.

2. The FeeSplitter contract is vulnerable:
  The FeeSplitter contract has some issues that will deter users from using the protocol. One thing is users would want to have an easier time using the protocol but having to fees each time is not good. If they don't to make a claim before the offset is reset for them, they lose the associated fees. Users want to feel safe that they can come back in 2 months and be able to claim accumulated fees not needing to spend gas here and there on claiming fees every single time some trading activity happens on the platform. Gas for this will easily deter users from doing such actions. Our recommendation is to save the user fee accrued before resetting the offset so a user can hold a subject token, forget it, come back in 30 days and be able to claim accumulated fees without losing a substantial amount.


3. There is a ridiculous amount of external calls that happen after every token trade on the platform specifically in the `_transferFees()` function. Any failure of a single external call reverts the whole transaction. This is no good as such failure could continue for a long time even leading to a DOS of trading for a subject token. We recommend caching/saving fees and implementing a pull technique for all fee receivers rather than a push technique that the protocol currently utilizes.


## 7. Hours spent
48 hours(High & Medium findings, Gas findings, QA findings and Analysis report)

**Note :-**
Consider the following points to judge 

Highlights of this report
1. Representing the smart contract workflow through a diagram.
2. Extensive analysis of the contract by function-to-function call trace analysis.
3. Comparison table against another competitive protocol.
4. Best recommendation that can be safely implemented. 

### Time spent:
48 hours

### Time spent:
48 hours