Project name: Curves
Project description: The first interoperable SocialFi protocol. Buy, sell and trade SocialFi tokens across a universe of dapps.

Findings are discuss below:
Security concern:
In curves.sol file:
Only owner and only manager modifiers are referenced, but the corresponding modifier definitions are not provided in the code.

Fallback function: The contract has a fallback function that doesn't have a payable modifier. If you intend to receive Ether through the fallback function, then  consider adding the payable modifier.

In security measure the contract doesn't have a reentrancy guard, making it susceptible to reentrancy attacks. using the "ReentrancyGuard" contract from the OpenZeppelin library helps to protect functions that involve Ether transfers.

variable: 
Immutable variables, such as DEFAULT_NAME and DEFAULT_SYMBOL, should be declared with the immutable keyword for clarity and potential gas optimizations.

Gas Cost:
functions that involve looping over arrays or making multiple external calls should have potential gas limitations.


Thank You
Sanjay Sharma
Email: Compliance.gst.123@gmail.com

### Time spent:
7 hours