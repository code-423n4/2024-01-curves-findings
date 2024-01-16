## [1] Last Token can be withdrawn
Low/Medium impact. Even though it is prevented that the last token of a specific curve token can be sold in `Curves.sellCurvesToken` it is possible to withdraw the last curve token and get it as an ERC20 token. This ERC20 token could be potentially be sold to an unknowing user which would not be able to deposit and sell it in the curves contract.

### Proof of Concecpt
Paste the following inside tests/curves-erc20.ts and run yarn hardhat test tests/curves-erc20

```
   it.only("Last ERC20 Token can be withdrawn", async () => {
      testContract.connect(owner).buyCurvesToken(owner.address, 1);
      testContract.connect(owner).withdraw(owner.address, 1);
      const keyTokenAddress = (await testContract.externalCurvesTokens(owner.address)).token;
      const keyToken = CurvesERC20__factory.connect(keyTokenAddress, owner);
      expect(await keyToken.balanceOf(owner.address)).to.be.equal(ethers.utils.parseEther("1"));
    });
```

### Recommended Mitigation Steps
Implement check in `Curves.withdraw` that prevents that the last token will be withdrawn.


## [2] Use of Less than or equal to prevents start of buy token whitelisted at the right time.

Low impact. To determine if a whitelisted curve token can be bought `buyCurvesTokenWhitelisted` checks incorrectly if `startTime` is already reached.

### Proof of Concept

As seen in following code snippet startTime <= block.timestamp:
```
function buyCurvesTokenWhitelisted(address curvesTokenSubject, uint256 amount, bytes32[] memory proof)
        public
        payable
    {
        if (
            presalesMeta[curvesTokenSubject].startTime == 0
                || presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();

        presalesBuys[curvesTokenSubject][msg.sender] += amount;
        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
        //q what if maxBuy is not set?
        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

        verifyMerkle(curvesTokenSubject, msg.sender, proof);
        _buyCurvesToken(curvesTokenSubject, amount);
    }
```

This leads to a later start of the presale than intended.

### Recommended Mitigation Steps

```diff
- presalesMeta[curvesTokenSubject].startTime <= block.timestamp
+ presalesMeta[curvesTokenSubject].startTime < block.timestamp
```



