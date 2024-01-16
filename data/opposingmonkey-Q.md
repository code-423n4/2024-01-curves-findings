## Overflow in "buyCurvesToken", "buyCurvesTokenWithName", "buyCurvesTokensForPresale" when the first buyer (curvesTokenSubject) tries to buy more than 1 share 

In that case, `supply < amount` in the "getPrice()" function and the transaction reverts with panic code 0x11. This prevent someone from buying more than 1 share in the first place. One should make sure to revert with a proper error, or fix the "getPrice()" function appropriately so that the initial owner can buy more than 1 share. 

The issue can be easily reproduced with the following test cases to be introduced in `test/curves.ts`: 

```
    it("overflow buyCurvesToken", async () => {
      await testContract.buyCurvesToken(owner.address,10);
    });

    it("overflow buyCurvesTokenWithName", async () => {
      await testContract.buyCurvesTokenWithName(owner.address,10, "name", "symbol");
    });

    it("overflow buyCurvesTokensForPresale", async () => {
      const block = await ethers.provider.getBlock(); 
      await testContract.buyCurvesTokenForPresale(owner.address,10, block.timestamp + 1000, ethers.constants.HashZero, 20);
    });

```