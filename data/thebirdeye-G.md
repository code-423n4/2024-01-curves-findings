1. Contracts/curves.sol line 232,236,240
".Call(bool success)" is more gas optimized than "(bool success).
2. Contracts/curves.sol line 352,297
Instead of using address (this),it is more efficient to pre calculate and use hard coded address.
3. Contracts/curves.sol line 108,contracts/feesplitter.sol line 33,contract/security.sol line 18, contracts/curvesERC20.sol
Use "payable" with constructor to save gas.
4. Contracts/curves.sol line 298,308,313,324,486
Use "call" instead of transfer function. When using "call" function,we can specify the gas we are going to be sending and Ether.