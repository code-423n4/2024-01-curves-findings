Curve.sol - Always return true for the success3 local variable in the _transferFees function

In the _transferFees function, after the referralDefined boolean is determined to know if a referral destination address was set, the success3 is afterwards dependent on this. While the first part of the tenary operator sends ether to the address of the referrral address, if referralDefined is true - success value would be true on successful ether transfer with an empty calldata. Otherwise, when referralDefined is still false, this still returns (true, "").

```
         {
                (bool success3, ) = referralDefined
                    ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                    : (true, bytes(""));
                if (!success3) revert CannotSendFunds();
          }
```

The constant return of true and empty data even when there is no referral address is set, will always cost an additional 78 gas for a function call. This 78 gas can be reduced if this part of the function is redesigned this way.

```
          if(referralDefined) {
                (bool success3, ) =  referralFeeDestination[curvesTokenSubject].call{value: referralFee}("");
                 if (!success3) revert CannotSendFunds();
            }
```

With the snippet above, this is stictly dependent on when referral is set, when not set, it skips the ether transfer block and save gas.