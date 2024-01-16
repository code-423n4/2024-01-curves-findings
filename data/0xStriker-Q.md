# Summary



# Low Risk Issues    
no | Issue |Instances||
|-|:-|:-:|:-:|
| [L-01] |  the buyCurvesTokenWithName() function should implement the onlyTokenSubject() modifier | 1 |
| [L-02] |  the getSellPrice() function should check if the curvesTokenSupply is less than or equal to amount | 1 | 
| [L-03] |  the function _transfer() should just emit the event if the (from,to) addresses are the same | 1 |
| [L-04] |  the setWhitelist() function should also check if a token supply is zero | 1 |
| [L-05] |  the verifyMerkle() check should be done at the beginning of the function | 1 |

# Non-critical
no | Issue |Instances||
|-|:-|:-:|:-:|
| [N-01] |  a word has been forgotten in the comment | 1 |

## [L-01] the buyCurvesTokenWithName() function should implement the onlyTokenSubject() modifier
the buyCurvesTokenWithName() function checks the curvesTokenSubject's supply to be zero after the check it proceeds to call the _buyCurvesToken() function to buy the inteded amount, the _buyCurvesToken() function checks for the if either the supply to be more than zero or the msg.sender to be the curveTokenSubject to proceed executing, now the supply is already zero if the curveTokenSubject is not msg.sender it will revert.
It is recommended to add the onlyTokenSubject() modifier in the buyCurvesTokenWithName() function to avoid any one else from calling the function but the curvsTokenSubjec, and avoiding extra calculation and transaction costs.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L364 


## [L-02] the getSellPrice() function should check if the amount  is less than or equal to curvesTokenSupply
the getSellPrice() function should revert if the amount is more than or equal to the supply of the curvesTokenSubject's token supply and don't return any price.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L193



## [L-03] the function _transfer() should just emit the event if the (from,to) addresses are the same
in the _transfer() function there is a check if the the (from) address is not equal to the (to) address it calls the _addOwnedCurvesTokenSubject() function adding it to the (ownedCurvesTokenSubjects) mapping and then continues to subtract from the (from) address and adds to the (to) address, but when the (from) address is equal to the (to) address the function doesn't need to add or subtract anything since the (from) and the (to) addresses are the same and would be sufficiant to just emit the event Transfer avoiding any extra transaction costs.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L317


## [L-04] the setWhitelist() function should also check if a token supply is zero
the function setWhitelist() checks if a curvesTokenSupply is more than one which means the curve token already exists and will revert, but the function should also check if the curvesTokenSupply is equal to zero which will mean that the curve token hasn't been created yet and should revert.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L396

## [L-05] the verifyMerkle() check should be done at the beginning of the function
the function buyCurvesTokenWhitelisted() allows those users that are whitelisted to buy tokens ahead of startTime, the caller passes proof that indeed he is whitelisted, but the buyCurvesTokenWhitelisted() function checks for this proof at the end of the function where if the caller of the function is not whitelisted it will cost him the extra fees, this check needs to be done at the beginning of the function before updating anything.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L418



## [N-01] a word has been forgotten in the comment
in the comment " If is the first token bought, add to the list of owned tokens" the word " this " is not written which is grammatically incorrect, add the word " this " in comment.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L276
