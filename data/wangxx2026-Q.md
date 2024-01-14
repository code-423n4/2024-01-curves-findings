# Quality Assurance Report

| ID     | Issues                                                                                                               |
|--------|----------------------------------------------------------------------------------------------------------------------|
| [L-01] | CannotSendFunds used multiple times in the same method, if an error is reported, it will not be clear where the problem lies                                               |

## [L-01] CannotSendFunds, used multiple times in the same method, if an error is reported, it will not be clear where the problem lies
File: Curves.sol
```solidity
            bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
            {
                address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
                uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
                uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
                (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
                if (!success1) revert CannotSendFunds(); // <= FOUND
            }
            {
                (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
                if (!success2) revert CannotSendFunds(); // <= FOUND
            }
            {
                (bool success3, ) = referralDefined
                    ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                    : (true, bytes(""));
                if (!success3) revert CannotSendFunds(); // <= FOUND
            }
```