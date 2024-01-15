
# Title: Ether can be Locked in FeeSplitter.sol

## Impact
Funds sent to feeSplitter through the receive function can be permanently locked inside the contract.

## Proof of Concept
The contract includes a receive function, but does not include a method to withdraw funds. Additionally, there is no logic to track when the receive function is executed, so any funds added in this manner will not be reflected in the contract state variables.

## Tools Used
Manual Review 

## Recommended Mitigation Steps
Add a function to allow the contract owner or manager to withdraw funds in the contract, or remove the receive function and rely on the addFees function to add ether to the contract. 





# Title: No Address(0) Check for transferOwnership in Security.sol

## Impact
Loss of ownership for contracts inheriting Security.sol including Curves.sol and FeeSplitter.sol.  

## Proof of Concept
It is best practice to include a default variable check in most functions that accept user input due to user error. While this is a debated topic, functions of critical importance such as transfering the ownership of the contract should most definitely include such protections. Should the contract be designed to eventually "burn" the owner role, a contract with no functionality can be deployed and the ownership transfered to this new contract to achieve the same effect. 

## Tools Used
Manual Review 

## Recommended Mitigation Steps
Add a check to prevent the owner calling transferOwnership with empty calldata using an address(0) check.



