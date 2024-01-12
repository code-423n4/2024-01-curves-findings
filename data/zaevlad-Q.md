### **[L-01] Events should be emited for core functions**

There are some core functions such as setFeeRedistributor(), setMaxFeePercent(), setProtocolFeePercent(), setExternalFeePercent() that can influence on the protocol functioning. 

A good practice is to add events to any changes for most important variables in smart contracts.

### **[L-02] Strict token amounts while deploying a new CurvesERC20**

If we take a look on the price calculation func:

```
   function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
        //uint256 sum2 = supply == 0 && amount == 1 ? 0 : (supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1) / 6;  
        uint256 sum2 = supply == 0 && amount == 1 ? 0 : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
        uint256 summation = sum2 - sum1;
        return (summation * 1 ether) / 16000;
    }
```

we can see strick condition `uint256 sum2 = supply == 0 && amount == 1`. If user will plan to buy more than 1 it will always revert. 

Not sure if it is planned by protocol, so I put it in a QA section.


