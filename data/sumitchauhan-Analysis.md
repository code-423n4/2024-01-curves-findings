High risk issues-
1. Uninitialized state variable:
   Curves.sol::101 The mapping ownedCurvesTokenSubjects is never initialized. It 
   is used in 2 functions i.e., transferAllCurvesTokens(address) and 
   _addOwnedCurvesTokenSubject(address,address).
   
2. Functions that send ether to arbitrary destinations:
   Curves.sol::218-261 The function 
   _transferFees(address,bool,uint256,uint256,uint256) sends eth to arbitrary 
   user

Medium risk issues-
1. Reentrancy:
   Curves.sol::338-362 Reentrancy in _deployERC20(address,string,string) 
   function.
   Curves.sol::490-502 Reentrancy in deposit(address,uint256) function.
   Curves.sol::465-488 Reentrancy in withdraw(address,uint256) function.
        
Low risk issues-
1. Avoid use of block.timestamp for comparisons:
   Curves.sol::211-216 The function buyCurvesToken(address,uint256) uses 
   timestamp for comparisons i.e., startTime != 0 && startTime >= block.timestamp
   Curves.sol::377-392 The function 
   buyCurvesTokenForPresale(address,uint256,uint256,bytes32,uint256) uses 
   timestamp for comparisons i.e., startTime <= block.timestamp
   Curves.sol::404-420 The function 
   buyCurvesTokenWhitelisted(address,uint256,bytes32[]) uses timestamp for 
   comparisons i.e., 
   presalesMeta[curvesTokenSubject].startTime == 0 || 
   presalesMeta[curvesTokenSubject].startTime <= block.timestamp 
   
2. Floating pragma version:
   Security.sol::2 => pragma solidity ^0.8.7;
   
3. Incorrect versions of solidity:
   Security.sol::2 => pragma solidity ^0.8.7;
   
Informational issues or gas optimizations-
1. Using storage instead of memory for structs/arrays saves gas:
   FeeSplitter.sol::53 => address[] memory tokens = getUserTokens(user);
   FeeSplitter.sol::54 => UserClaimData[] memory result = new UserClaimData[] 
   (tokens.length);

2. Constants in comparisons should appear on the left side:
   FeeSplitter.sol::66 => if (balance > 0) {
   FeeSplitter.sol::83 => if (claimable == 0) revert NoFeesToClaim();
   FeeSplitter.sol::91 => if (totalSupply_ == 0) revert NoTokenHolders();
   FeeSplitter.sol::99 => if (balanceOf(token, account) > 0) 
   userTokens[account].push(token);
   FeeSplitter.sol::109 => if (claimable > 0) {
   FeeSplitter.sol::115 => if (totalClaimable == 0) revert NoFeesToClaim();


### Time spent:
8 hours