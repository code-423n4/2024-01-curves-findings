## [N-01] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

There are 2 instances of this issue in 1 file:

    File: contracts/Curves.sol	

    66: mapping(address => mapping(address => uint256)) public presalesBuys;

    96: mapping(address => mapping(address => uint256)) public curvesTokenBalance;

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol