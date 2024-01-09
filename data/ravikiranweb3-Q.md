#### FeeSplitter::claimFees() uses transfer() function
The function uses low level function transfer() which is not recommended due to gas limit of 2300 gas. As the Ethereum Blockchain evolves, there could be a case where 2300 gas may stop working.

Hence it is recommended to use low level function "call()" instead.
```  
  (bool success, ) = payable(msg.sender).call{value: claimable}(""); 
```

#### FeeSplitter::batchClaiming() uses transfer() function
Same as above in batchClaiming as well.