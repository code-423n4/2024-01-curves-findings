
  TEST SUITE  

```
Run through the install process listed in the install.md
```
```
add hardhat support for truffle [LINK] https://hardhat.org/hardhat-runner/docs/other-guides/truffle-testing
```
```
create a new audit.js file 
```
```
const {ethers, artifacts} = require("hardhat");
const { web3 } = require("hardhat");
const curves =  artifacts.require("Curves")
const factory = artifacts.require("CurvesERC20Factory")
const feeController = artifacts.require("FeeSplitter")
contract(("CURVES"), (accounts) => {
    let curvesContract,curvesFactory,FeeSplitter
    let sellValueBefore15 ,buyValueBefore15,sellValueBefore30 ,buyValueBefore30,sellValueBetween,buyValueBetweenSellAndBuy,sellValueAfter
    let balanceOfCurvesContract = 0
       beforeEach(async () => {
            curvesFactory = await factory.at("0xaC9fCBA56E42d5960f813B9D0387F3D3bC003338")
            FeeSplitter = await  feeController.at("0x38A70c040CA5F5439ad52d0e821063b0EC0B52b6")
            curvesContract = await curves.at("0x54B8d8E2455946f2A5B8982283f2359812e815ce")
         });
})
```



### [H-1] The `Security::onlyOwner` and `Security::onlyManager` modifiers do not ensure proper access control meaning anyone can call functions that are only meant to the accessable to the owner and the manager

**Description:** The `Security::onlyOwner` and `Security::onlyManager` modifiers do not enforce the condition with a require statement or a revert if the sender is not authorized 

**Impact:** Anyone can call functions only meant to be accessable to the owner or manager

**Proof of Concept:**
```
 it("SHOULD HAVE ITS OWNER SET", async ( ) => {
    console.log((await curvesContract.owner()).toString() == accounts[0]);
})
```
 ``` 
       it("SHOULD WORK ON RIGHT ACCESS CONTROL", async () => {
    await curvesContract.setManager(accounts[1],true,{from:accounts[0]})
})
```
```
it("SHOULD REVERT ON FALSE ACCESS CONTROL",async () =>{
 await curvesContract.setManager(accounts[2],true,{from:accounts[2]})
})
```

The first test returns true whcih confirms that the owner is the address at accounts[0]
The second test passes which confirms that the owner can set the manager 
However the third test also passes allowing the address at accounts[2] to set itsef as a manager 

**Recommended Mitigation:**

``` 
 modifier onlyOwner() {
        msg.sender == owner;
        _;
  }
```
should be replaced with 
```
  modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }
```

 

### [H-2] The FeeSplitter::setCurves function has no access control allowing anyone to change the curves contract 

**Description:** The core functionality of the feeSplitter invloves reading data from the curves contract, if bad data is sent from a malicious contract, an attacker can withdraw all the fees from the contract 

**Impact:** All funds in the contract could be stolen

**Proof of Concept:** An attacker could set the curves to a malicious one and set it so that it returns a higher balance when the FeeSplitter::balanceOf is called, The FeeSplitter contract can also be forced to read a false totalSupply


**Recommended Mitigation:** The FeeSplitter::setCurves function should implement proper access control



### [H-3] There is no hardcoded MaxFeePercent in the Curves contract 

**Description:** When A user executes a buy order using Curves::buyCurvesToken, the call simply reverts if the value sent is not enough to cob=ver price and fees, however is a sell order is executed using Curves::sellCurvesToken, the curves::getSellPriceAfterFee simply removes the fee from the price and sends the rest to the user, hence the fee could end up being the total value of the price ensuring no ether is then sent back to the user

**Impact:** All of users funds could be used to cover fees 

**Proof of Concept:** 
```
 it("POSES A CENTRALIZATION RISK", async () => {
       curvesContract.setMaxFeePercent(BigInt(10e18),{from:accounts[1]})
 })
```
The above test passes and any sell that occurs after this would have to pay this abnormally high fee

**Recommended Mitigation:** The protocol should implement a hardcoded constant MaxFeePercentage


### [H-4] The Curves::getPriceAfterFee functions uses logic that encourages negative mev

**Description:** An attacker could front run a sell reducing the price of the subject tokens and the total amount the user was expecting to get 

**Impact:** User could be griefed of ether 

**Proof of Concept:**
```
it("IS VULNERABLE TO SELL BEING FRONT RUN", async () => {
   const expectedEther = (await curvesContract.getSellPriceAfterFee(accounts[3],15)).toString() 
   console.log("expectedEther",expectedEther);
   const costOfBuying30TokensBeforeAttack = (await curvesContract.getBuyPriceAfterFee(accounts[3],30)).toString() 
   console.log("costOfBuying30TokensBeforeAttack",costOfBuying30TokensBeforeAttack);
   await curvesContract.sellCurvesToken(accounts[3],15,{from:accounts[8]})
   const actualEtherGotten = (await curvesContract.getSellPriceAfterFee(accounts[3],15)).toString() 
   console.log("actualEtherGotten",actualEtherGotten);
   await curvesContract.sellCurvesToken(accounts[3],15,{from:accounts[6]})
   const costOfBuying30TokensAfterAttack = (await curvesContract.getBuyPriceAfterFee(accounts[3],30)).toString() 
   console.log("costOfBuying30TokensAfterAttack",costOfBuying30TokensAfterAttack);
   console.log("AMOUNT LOST BY VICTIM",expectedEther - actualEtherGotten);
if (Number(costOfBuying30TokensAfterAttack) < Number(costOfBuying30TokensBeforeAttack)) {
   console.log("TOTAL AMOUNT GAINED BY FRONT RUNNING",Number(costOfBuying30TokensAfterAttack)  - Number(costOfBuying30TokensBeforeAttack))
   await curvesContract.buyCurvesToken(accounts[3],30,{from:accounts[8],value:costOfBuying30TokensAfterAttack})
}
})
```
In this example the attacker frontruns the sell griefing the victim of ether and then uses the lowered price to buy more tokens 

**Recommended Mitigation:** The protocol should consider using a different logic for determining price, the curret logic is susceptible to different kinds of mev




### [M-1] Curves::_transferFees uses push payment which could lead to a denial of service

**Description:** The Curves::_transferFees transfer the fees directly to the recipient using call, if any of the recipients revert on ether transfer, it could lead to a dos 

**Impact:** All ether in the contract could be locked

**Proof of Concept:** The protocolFee Destination could be set to a contract without a recieve or fallback function, such contract will not be able to recieve ether as such all attempts to sell will be void and the ether in the contract will be locked 

**Recommended Mitigation:** The contract should keep track of all participats fees and allow them to withdraw the fees at their leisure


### [M-2] Security::transferOwnerShip uses a one step transfer

**Description:** The Security::transferOwnerShip function does not implement two step transfer, this could be a problem if the account or contract it is transferring ownership to does not have the logic to perform the duties of the owner 

**Impact:** The protocol could be left with an ineffective owner

**Proof of Concept:** The owner could transfer ownership to a contract that doesnt have logic to perform the duties of the owner 

**Recommended Mitigation:** The transfer ownership logic should be two step


### [L-1] Curves::BuyCurvesToken uses hardcoded amounts 

**Description:** A Malicious player could grief a user by selling just before their buy is executed, the user will stil send as much ether as it would have cost them to buy before the sell happened, but because of the hardcoded amount, will not get as much value for their tokens 

**Impact:** Users could be griefed 

**Proof of Concept:**
```
it("IS VULNERABLE TO USER GETTING LESS VALUE THAN AMOUNT SENT",async () => {
    balanceOfCurvesContract  = await web3.eth.getBalance(curvesContract.address)
    console.log(balanceOfCurvesContract);
    const sellAmount =  Number((await curvesContract.getSellPriceAfterFee(accounts[4],10)).toString())
    const buyAmount  = Number((await curvesContract.getBuyPriceAfterFee(accounts[4],15)).toString())
    await curvesContract.sellCurvesToken(accounts[4],10,{from:accounts[6]})
    const buyAmountNow  = Number((await curvesContract.getBuyPriceAfterFee(accounts[4],15)).toString())
    await curvesContract.buyCurvesToken(accounts[4],15,{from:accounts[7],value:buyAmount})
    console.log("AMOUNT OF EXTRA ETHER SENT",buyAmount - buyAmountNow);
 })

```

**Recommended Mitigation:** The protocol should consider implementing the buyCurvesToken function to mint as much tokens as value sent to price inside the function once the fee has been removed, so even if the buy is front run with a sell, the user will get as much amount as their value is worth once the fee has been removed 


### [I-1] The Curves::ownedCurvesTokenSubjects is not checked for update after a sell or transfer

**Description:** The Curves::ownedCurvesTokenSubjects is not updated even if the user sells or transfers all their tokens 

**Impact:** If a party wants to allocate some rewards to owners of a certain token, the party could be decieved into thinking a user has these tokens despite already transferring all of them.

**Recommended Mitigation:** The Curves::ownedCurvesTokenSubjects should be c=hecked for update after sell and transfer


### Time spent:
20 hours