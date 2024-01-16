## checking the same condition twice in a function call without any changes in betweenn the two checks lead to higher gas consumption.
## Details 
[line L466](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L466) and [Line 314](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L314) are the same.
the `curve.withdraw()` made this double check as a result of calling `curve::_transfer()` which also made the same check, cause users to pay higher gas fee than expected, the same instance happen to users who interract with deposit as the check was done on [line 498](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L498) before calling `_transfer()` which also did the check.
with the test case below 
```
//snip-//
 function testgasreportOnwithdraw() external {
        vm.prank(owner);
        //tokensubject no fee for buying 1 token first
        curves.buyCurvesToken(owner, 1);

        uint pricewithFee = curves.getBuyPriceAfterFee(owner, 50);

        // user buycurveToken before all necessary fee parameters are set
        vm.prank(user1);
        curves.buyCurvesToken{value: pricewithFee}(owner, 50);

        vm.prank(user1);
        //transfer to address(0)
        curves.transferCurvesToken(owner, address(0), 10);
        assertEq(curves.curvesTokenBalance(owner, address(0)), 10);

        vm.prank(user1);
        curves.withdraw(owner, 20);
    }
```
forge snapshot of the current logic is:
`testCurve:testgasreportOnwithdraw() (gas: 1305767)`
removing the check on line 466, resulted in the below gas report difference
```
Diff in "testCurve::testgasreportOnwithdraw()": consumed "(gas: 1305483)" gas, expected "(gas: 1305767)" gas
 ```
percentage difference of 
` testgasreportOnwithdraw() (gas: -284 (-0.022%)) `.
## Recommendation
gas will be optimized if the team can remove the check on [line L466](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L466) and [line 498](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L498) leaving only the check on [Line 314](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L314) to do the work.
