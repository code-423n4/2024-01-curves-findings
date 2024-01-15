For function `getPrice` in contract [Curves.sol](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L180-L187) may fail in some cases, but it is a `pure` function, that is only use this function to read data rather than set data, so its impact is very limited.

Code:
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

Test case:
```javascript
describe("ERC20 test", () => {
    it.only("Can not get price when supply is zero", async function () {
      let priceResult = await testContract.getPrice(0,11);
    });
}
```

Result:
```javascript
1) Curves ERC20 test
       ERC20 test
         Should get price when supply is zero:
     Error: call revert exception; VM Exception while processing transaction: reverted with panic code 17 [ See: https://links.ethers.org/v5-errors-CALL_EXCEPTION ] (method="getPrice(uint256,uint256)", data="0x4e487b710000000000000000000000000000000000000000000000000000000000000011", errorArgs=[{"type":"BigNumber","hex":"0x11"}], errorName="Panic", errorSignature="Panic(uint256)", reason=null, code=CALL_EXCEPTION, version=abi/5.7.0)
      at Logger.makeError (node_modules/@ethersproject/logger/src.ts/index.ts:269:28)
      at Logger.throwError (node_modules/@ethersproject/logger/src.ts/index.ts:281:20)
      at Interface.decodeFunctionResult (node_modules/@ethersproject/abi/src.ts/interface.ts:427:23)
      at Contract.<anonymous> (node_modules/@ethersproject/contracts/src.ts/index.ts:400:44)
      at step (node_modules/@ethersproject/contracts/lib/index.js:48:23)
      at Object.next (node_modules/@ethersproject/contracts/lib/index.js:29:53)
      at fulfilled (node_modules/@ethersproject/contracts/lib/index.js:20:58)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)
      at runNextTicks (node:internal/process/task_queues:64:3)
      at listOnTimeout (node:internal/timers:538:9)
```