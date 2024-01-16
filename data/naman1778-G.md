## [G-01] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

There is 1 instance of this issue in 1 file:

```
File: contracts/Curves.sol	

428: function setNameAndSymbol(
429:     address curvesTokenSubject,
430:     string memory name,
431:     string memory symbol
432: ) external onlyTokenSubject(curvesTokenSubject) {
```

    diff --git a/contracts/Curves.sol b/contracts/Curves.sol
    index 817c79b..d569aa3 100644
    --- a/contracts/Curves.sol
    +++ b/contracts/Curves.sol
    @@ -427,8 +427,8 @@ contract Curves is CurvesErrors, Security {

         function setNameAndSymbol(
             address curvesTokenSubject,
    -        string memory name,
    -        string memory symbol
    +        string calldata name,
    +        string calldata symbol
         ) external onlyTokenSubject(curvesTokenSubject) {
             if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
             if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

    | Contract0 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 100747                                    | 535             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | not_optimized                             | 790             | 790 | 790    | 790 | 1       |
    
    
    | Contract1 contract                        |                 |     |        |     |         |
    |-------------------------------------------|-----------------|-----|--------|-----|---------|
    | Deployment Cost                           | Deployment Size |     |        |     |         |
    | 66917                                     | 366             |     |        |     |         |
    | Function Name                             | min             | avg | median | max | # calls |
    | optimized                                 | 556             | 556 | 556    | 556 | 1       |

## [G-02] Use assembly to write *address* storage values

When writing value for variables whose type is address, make use of assembly code instead of solidity code.

There are 4 instances of this issue in 2 files:

```
File: contracts/Curves.sol	

109: curvesERC20Factory = curvesERC20Factory_;

163: curvesERC20Factory = factory_;
```
    diff --git a/contracts/Curves.sol b/contracts/Curves.sol
    index 817c79b..5462344 100644
    --- a/contracts/Curves.sol
    +++ b/contracts/Curves.sol
    @@ -106,7 +106,9 @@ contract Curves is CurvesErrors, Security {
         }

         constructor(address curvesERC20Factory_, address feeRedistributor_) Security() {
    -        curvesERC20Factory = curvesERC20Factory_;
    +        assembly{
    +                sstore(curvesERC20Factory,curvesERC20Factory_)
    +        }
             feeRedistributor = FeeSplitter(payable(feeRedistributor_));
         }

    @@ -160,7 +162,9 @@ contract Curves is CurvesErrors, Security {
         }

         function setERC20Factory(address factory_) external onlyOwner {
    -        curvesERC20Factory = factory_;
    +        assembly{
    +                sstore(curvesERC20Factory,factory_)
    +        }
         }

         function getFees(

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol

```
File: contracts/Security.sol	

19: owner = msg.sender;

28: owner = owner_;
```

    diff --git a/contracts/Security.sol b/contracts/Security.sol
    index b1dfee5..b319a3a 100644
    --- a/contracts/Security.sol
    +++ b/contracts/Security.sol
    @@ -16,7 +16,9 @@ contract Security {
         }

         constructor() {
    -        owner = msg.sender;
    +        assembly{
    +                sstore(owner,msg.sender)
    +        }
             managers[msg.sender] = true;
         }

    @@ -25,6 +27,8 @@ contract Security {
         }

         function transferOwnership(address owner_) public onlyOwner {
    -        owner = owner_;
    +        assembly{
    +                sstore(owner,owner_)
    +        }
         }
     }

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }

    contract Contract0 {

        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }

    }

    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |

| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |

## [G-03] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There are 2 instances of this issue in 1 file:

```
File: contracts/Curves.sol	

441: keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
442: keccak256(abi.encodePacked("")) ||
443: keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
444: keccak256(abi.encodePacked(""))

471: keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
472: keccak256(abi.encodePacked("")) ||
473: keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
474: keccak256(abi.encodePacked(""))

```
    diff --git a/contracts/Curves.sol b/contracts/Curves.sol
    index 817c79b..b2dd879 100644
    --- a/contracts/Curves.sol
    +++ b/contracts/Curves.sol
    @@ -438,10 +438,10 @@ contract Curves is CurvesErrors, Security {

         function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
             if (
    -            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
    -            keccak256(abi.encodePacked("")) ||
    -            keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
    -            keccak256(abi.encodePacked(""))
    +            ((keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ^
    +            keccak256(abi.encodePacked(""))) &
    +            (keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ^
    +            keccak256(abi.encodePacked("")))) == 0
             ) {
                 externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
                 externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
    @@ -468,10 +468,10 @@ contract Curves is CurvesErrors, Security {
             address externalToken = externalCurvesTokens[curvesTokenSubject].token;
             if (externalToken == address(0)) {
                 if (
    -                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
    -                keccak256(abi.encodePacked("")) ||
    -                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
    -                keccak256(abi.encodePacked(""))
    +                ((keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ^
    +                keccak256(abi.encodePacked(""))) &
    +                (keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ^
    +                keccak256(abi.encodePacked("")))) == 0
                 ) {
                     externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
                     externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-04] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

There are 4 instances of this issue in 1 file:

```
File: contracts/Curves.sol	

370: uint256 supply = curvesTokenSupply[curvesTokenSubject];

385: uint256 supply = curvesTokenSupply[curvesTokenSubject];

395: uint256 supply = curvesTokenSupply[msg.sender];

415: uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
```
    diff --git a/contracts/Curves.sol b/contracts/Curves.sol
    index 817c79b..e6ba89f 100644
    --- a/contracts/Curves.sol
    +++ b/contracts/Curves.sol
    @@ -367,8 +367,7 @@ contract Curves is CurvesErrors, Security {
             string memory name,
             string memory symbol
         ) public payable {
    -        uint256 supply = curvesTokenSupply[curvesTokenSubject];
    -        if (supply != 0) revert CurveAlreadyExists();
    +        if (curvesTokenSupply[curvesTokenSubject] != 0) revert CurveAlreadyExists();

             _buyCurvesToken(curvesTokenSubject, amount);
             _mint(curvesTokenSubject, name, symbol);
    @@ -382,8 +381,7 @@ contract Curves is CurvesErrors, Security {
             uint256 maxBuy
         ) public payable onlyTokenSubject(curvesTokenSubject) {
             if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
    -        uint256 supply = curvesTokenSupply[curvesTokenSubject];
    -        if (supply != 0) revert CurveAlreadyExists();
    +        if (curvesTokenSupply[curvesTokenSubject] != 0) revert CurveAlreadyExists();
             presalesMeta[curvesTokenSubject].startTime = startTime;
             presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
             presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
    @@ -392,8 +390,7 @@ contract Curves is CurvesErrors, Security {
         }

         function setWhitelist(bytes32 merkleRoot) external {
    -        uint256 supply = curvesTokenSupply[msg.sender];
    -        if (supply > 1) revert CurveAlreadyExists();
    +        if (curvesTokenSupply[msg.sender] > 1) revert CurveAlreadyExists();

             if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
                 presalesMeta[msg.sender].merkleRoot = merkleRoot;
    @@ -412,8 +409,7 @@ contract Curves is CurvesErrors, Security {
             ) revert PresaleUnavailable();

             presalesBuys[curvesTokenSubject][msg.sender] += amount;
    -        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
    -        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();
    +        if (presalesBuys[curvesTokenSubject][msg.sender] > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

             verifyMerkle(curvesTokenSubject, msg.sender, proof);
             _buyCurvesToken(curvesTokenSubject, amount);

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    
    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }
    
    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |

| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |
