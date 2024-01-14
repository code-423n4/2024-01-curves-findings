## [L-01] `maxFeePercent` should be a constant, rather than admin-settable

In many protocols, the purpose of a `MAX_FEE` variable is that:
- To reduce the possibility of catastrophic fee setting by a centralization risk: e.g. a malicious admin or admin mistake.
- To give a hint on the fee denomination i.e. "is 100% fee defined by 10000 or 1e18".

The current `maxFeePercent` does not achieve any of that. It can be freely set by an admin, and it costs extra gas for setting such fee by taking up storage. 

We recommend making such value `constant` to reduce centralization risk.

## [L-02] Token subject's initial purchase must be exactly one token, otherwise the tx will revert

It is intended that the token subject must make the initial purchase of one token, to initialize their total supply.

However, the token subject is not able to purchase more than one token, but rather must purchase *exactly one* token on their first purchase. 

This is due to an underflow bug in `getPrice()`, which is used by `_buyCurvesToken()`:

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L180-L187

```solidity
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
    uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1 // @audit underflow when supply = 0 but amount > 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    uint256 summation = sum2 - sum1;
    return (summation * 1 ether) / 16000;
}
```

We urge the sponsor to double check on whether this is intended.

We also attach a PoC. Paste this entire file into `\test` directory, then run tests normally. The test will fail with an underflow:

```solidity
import { expect, use } from "chai";
import { solidity } from "ethereum-waffle";
use(solidity);
import { SignerWithAddress } from "@nomiclabs/hardhat-ethers/signers";
//@ts-ignore
import { ethers } from "hardhat";

import { type Curves } from "../contracts/types";
import { buyToken } from "../tools/test.helpers";
import { deployCurveContracts } from "./test.helpers";

describe("C4 contest PoC", () => {
  let testContract: Curves, owner: SignerWithAddress, friend: SignerWithAddress, addrs: SignerWithAddress[];

  beforeEach(async () => {
    [owner, friend, ...addrs] = await ethers.getSigners();
    testContract = await deployCurveContracts();
  });

  describe("PoC", () => {
    it("L-03", async () => {
      await testContract.buyCurvesToken(owner.address, 2);
    });
  });
});
```

## [L-03] Anyone can steal token symbols that has been set but not deployed (or deny any tx involving `_deployERC20()`)

Affected code: https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L350

The function `_deployERC20()` deploys a new ERC20 token for a specified token subject. 

However, tokens with duplicate symbols are not allowed. This allows any adversary to steal a token symbol for themselves, or deny some of the contract functionalities by simply front-running.

This may pose a more serious impact if a user has set their own token name and symbol, but has not deployed it. Then anyone else can deploy a token first with the same symbol, and the user who has set first will lose the symbol.

PoC scenario:
1. Alice sets her own token name and symbol using `setNameAndSymbol()`.
2. Bob deploys his token, using the same name and symbol as Alice.
3. Alice now loses the token symbol. However she can still set other symbols instead.

Coded PoC: There is actually already a test case showing that duplicate symbols are not allowed. For that reason we think this requires no extra coded PoC.
- https://github.com/code-423n4/2024-01-curves/blob/main/test/curves-erc20.ts#L190-L197

There are two main impacts to this issue:
- As opposed to friend.tech where each address is only defined by itself, there is a certain value to the symbol itself, which may be incentives to perform this kind of attack.
- Users migrating tokens through `transferAllCurvesTokens()` will lose their own token symbol.

To mitigate the latter issue, the sponsor may want to allow migrating the ERC20 itself to a new blank address.

## [L-04] Anyone can brick the function `mint()` by self-minting a token named "CURVES1"

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L344-L350

In the `_deployERC20()`, if the token's symbol is `CURVES`, then a counter is increased, and the symbol (as well as the name) is appended with the number (e.g. CURVES1, CURVES2, CURVES3). This default name and symbol is used in the `mint()` function.

An adversary (or any curious user) can completely DoS the `mint()` by front-running the counter (not the regular blockchain front-running). Attack scenario:
1. `Curves` contract is deployed
2. Alice mints a token using `buyCurvesTokenWithName()`, the token she mints has the name `CURVES1`.
3. Now no one can call `mint()` or mint with the default name anymore.

It is worth noting anyone can deny the counter at any point, not just at value 1.

We recognize this as a low severity issue because, while one function of the contract can be bricked, it does not affect overall contract functionality, and any other names can still be minted using other functions.

Coded PoC:

```javascript
import { expect, use } from "chai";
import { solidity } from "ethereum-waffle";
use(solidity);
import { SignerWithAddress } from "@nomiclabs/hardhat-ethers/signers";
//@ts-ignore
import { ethers } from "hardhat";

import { type Curves } from "../contracts/types";
import { buyToken } from "../tools/test.helpers";
import { deployCurveContracts } from "./test.helpers";
import { BigNumber } from "ethers";

describe("C4 contest PoC", () => {
  let testContract: Curves, owner: SignerWithAddress, friend: SignerWithAddress, addrs: SignerWithAddress[];

  beforeEach(async () => {
    [owner, friend, ...addrs] = await ethers.getSigners();
    testContract = await deployCurveContracts();
  });

  describe("PoC", () => {
    it("CURVES symbol DoS", async () => {
      const alice = addrs[0];
      const bob = addrs[1];
      const charlie = addrs[2];

      await testContract.connect(alice).buyCurvesTokenWithName(alice.address, 1, "Curves 1", "CURVES1");
      console.log("Successful buy token with symbol CURVES1");

      await expect(testContract.mint(owner.address)).to.be.revertedWith(`InvalidERC20Metadata`);
      console.log("Default minting reverted");
    });
  });
});
```

## [N-01] Function logic locations are not following convention

Normally internal functions are the ones holding the core logic, while external functions only provide the top-level logic (e.g. input validation, loops, calls to internal functions).

This contract set has inconsistency in how external and internal functions are used. For example, `_buyCurvesToken()` is an internal function with a lot of logic, buy any token purchase eventually points to this function. However, `sellCurvesToken()` is an external function, containing all the logic for selling.

## [N-02] Function locations are not following convention

The general practice in modern solidity codebases are that, all codes of the same type should be grouped together in one location. "Same types" may include:
- Storage variables.
- Event definitions.
- Custom errors.
- Admin functions.
- Internal functions.
- External functions.
- External view/pure functions.

While most of the code attributes are grouped together, among functions, there is a mix of locations between internal and external functions. This makes it harder to review the codebase as a whole, should there be a need to.
