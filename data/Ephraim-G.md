# Curve.sol - Always return true for the success3 local variable in the _transferFees function

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

# Double Check in withdraw function Hike the Gas Cost of calling Functions

There is a double check happening in the withdraw function that verifies that the intended amount that a user wants to withdraw is less than the balance in curvesTokenBalance[curvesTokenSubject][msg.sender]. When a user calls the withdraw function, this first check happens at the withdraw level and the check also happened in the _transfer.

```
      function withdraw(address curvesTokenSubject, uint256 amount) public {
        if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();
       address externalToken = externalCurvesTokens[curvesTokenSubject].token;
        if (externalToken == address(0)) {
        ...
```

On a thorough observation, this check is irrelevant in the withdraw function since the essence of the check did not happen in the function until it runs into the _transfer function.

```
       function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

        // If transferring from oneself, skip adding to the list
        if (from != to) {
      ...
```

Remediation - Remove this check from the withdraw function since this is present in the _transfer.