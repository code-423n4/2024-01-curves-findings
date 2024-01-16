
##  Gas Summary

| Number | Issue | Instances|
|--------|-------|----------|
|[G-01]|  Efficient data management:  | 6 |
|[G-02]|  Avoid Unnecessary Use of Storage | 2 |
|[G-03]|  Create immutable variable to avoid an external call  | 2 |
|[G-04]|  IF’s/require() statements that check input arguments should be at the top of the function  | 3 |
|[G-05]|  Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient  | 1 |
|[G-06]|  Don’t cache call if used only once  | 2 |
|[G-07]|  Using delete statement can save gas  | 2 |
|[G-08]|  Default value initialization  | 8 |
|[G-09]|  Using storage instead of memory for structs/arrays saves gas   | 2 |
|[G-10]|  Save loop calls  | 1 |
|[G-11]|  Use hardcode address instead address(this)  | 6 |
|[G-12]|  When using storage instead of memory, we should cache any fields that need to be re-read in stack variables  | 1 |
|[G-13]|  Use uint256(1)/uint256(2) instead for true and false boolean states | 6 |
|[G-14]|  Using assembly to revert with an error message  | 34 |
|[G-15]|  Use assembly for loops  | 1 |
|[G-16]|  Shorten arrays with inline assembly  | 1 |
|[G-17]|  Use the existing Local variable/global variable when equal to a state variable to avoid reading from state  | 2 |
|[G-18]|  Don’t make variables public unless necessary  | 6 |
|[G-19]|  Don’t cache state varible if  that are only used once  | 3 |
|[G-20]|  Use inheritance:  | 6 |



 
## [G-01] Efficient data management:

Permanently save a storage variable in memory in a function. Strategically use memory within functions for temporary storage of variables.

Store the subjects state variabl in memory variable

```solidity
file: blob/main/contracts/Curves.sol

302  function transferAllCurvesTokens(address to) external {
        if (to == address(this)) revert ContractCannotReceiveTransfer();
        address[] storage subjects = ownedCurvesTokenSubjects[msg.sender];
        for (uint256 i = 0; i < subjects.length; i++) {
            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
            if (amount > 0) {
                _transfer(subjects[i], msg.sender, to, amount);
            }
        }
    }

```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L302-L311

Store the feesEconomics state variabl in memory variable

```solidity
file: blob/main/contracts/Curves.sol

117   function setMaxFeePercent(uint256 maxFeePercent_) external onlyManager {
        if (
            feesEconomics.protocolFeePercent +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            maxFeePercent_
        ) revert InvalidFeeDefinition();
        feesEconomics.maxFeePercent = maxFeePercent_;
    }

128  function setProtocolFeePercent(uint256 protocolFeePercent_, address protocolFeeDestination_) external onlyOwner {
        if (
            protocolFeePercent_ +
                feesEconomics.subjectFeePercent +
                feesEconomics.referralFeePercent +
                feesEconomics.holdersFeePercent >
            feesEconomics.maxFeePercent ||
            protocolFeeDestination_ == address(0)
        ) revert InvalidFeeDefinition();
        feesEconomics.protocolFeePercent = protocolFeePercent_;
        feesEconomics.protocolFeeDestination = protocolFeeDestination_;
    }

141  function setExternalFeePercent(
        uint256 subjectFeePercent_,
        uint256 referralFeePercent_,
        uint256 holdersFeePercent_
    ) external onlyManager {
        if (
            feesEconomics.protocolFeePercent + subjectFeePercent_ + referralFeePercent_ + holdersFeePercent_ >
            feesEconomics.maxFeePercent
        ) revert InvalidFeeDefinition();
        feesEconomics.subjectFeePercent = subjectFeePercent_;
        feesEconomics.referralFeePercent = referralFeePercent_;
        feesEconomics.holdersFeePercent = holdersFeePercent_;
    }

166   function getFees(
        uint256 price
    )
        public
        view
        returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
    {
        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
    }

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L117-L126

Store the feeRedistributor state variabl in memory variable

```solidity
file: blob/main/contracts/Curves.sol

246  if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
            }
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L246-L249

## [G-02] Avoid Unnecessary Use of Storage

Writing data to storage is one of the most expensive operations in Solidity. Reduce gas costs by avoiding unnecessary writes. For example, move constant values to memory:

```solidity

// Expensive
string public constant NAME = "MyContract"; 

// Cheaper
string memory NAME = "MyContract";

```

Also be careful about storing large pieces of data on-chain. Use alternative patterns like IPFS for larger data.

Storing data costs 20,000 gas versus 3 gas for memory.
Minimize storage by using memory for fixed constants and temporary values.
Store large data like files off-chain (IPFS) and put hash on-chain.

```solidity
file: blob/main/contracts/Curves.sol

44  string public constant DEFAULT_NAME = "Curves";

45  string public constant DEFAULT_SYMBOL = "CURVES";

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L44

## [G-03] Create immutable variable to avoid an external call

Instead of performing an external call to get the root address each time _enableNode is invoked, we can perform this external call once in the constructor and store the root as an immutable variable. Doing this will save 1 external call each time _enableNode is invoked.

```solidity
file: blob/main/contracts/CurvesERC20Factory.sol

8    CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L8

```solidity
file: blob/main/contracts/FeeSplitter.sol

10  Curves public curves;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L10

## [G-04] IF’s/require() statements that check input arguments should be at the top of the function

in the _buyCurvesToken() function first you can check the globle msg.value variable to save gas if the the msg.value variable check is file then all the function will be revert if you check first msg.value if the check pass the desn't mutter if the check is fiale then can save gas 

```solidity
file: blob/main/contracts/Curves.sol

270   if (msg.value < price + totalFee) revert InsufficientPayment();

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L270

in the setWhitelist() function check first the function parameter

```solidity
file: blob/main/contracts/Curves.sol

390   if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
            presalesMeta[msg.sender].merkleRoot = merkleRoot;
            emit WhitelistUpdated(msg.sender, merkleRoot);
        }

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L390-L401

in the claimFees() function first check the if statment before the updateFeeCredit() function get update becouse if the if statment is fial and revert all change will be revert and the action spend more gase 

```solidity
file: blob/main/contracts/FeeSplitter.sol

83   if (claimable == 0) revert NoFeesToClaim();

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L83

## [G-05] Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient

SSTORE from 0 to 1 (or any non-zero value) costs 20000 gas. SSTORE from 1 to 2 (or any other non-zero value) costs 5000 gas.

By storing the original value once again, a refund is triggered (https://eips.ethereum.org/EIPS/eip-2200).

Since refunds are capped to a percentage of the total transaction’s gas, it is best to keep them low, to increase the likelihood of the full refund coming into effect.

Therefore, switching between 1, 2 instead of 0, 1 will be more gas efficient.

See: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/86bd4d73896afcb35a205456e361436701823c7a/contracts/security/ReentrancyGuard.sol#L29-L33

```solidity
file: blob/main/contracts/Security.sol

20   managers[msg.sender] = true;

```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L20


## [G-06] Don’t cache call if used only once

Caching here will simply increase gas cost, since it doesn’t affect readability, we should not cache since we only reference the cached call only once.

don't cache keccak256(abi.encodePacked(caller)) because the used only once inside the function 

```solidity
file: blob/main/contracts/Curves.sol

422  function verifyMerkle(address curvesTokenSubject, address caller, bytes32[] memory proof) public view {
        // Verify merkle proof
        bytes32 leaf = keccak256(abi.encodePacked(caller));
        if (!MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf)) revert UnverifiedProof();
    }

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L422-L426

don't cache presalesBuys[curvesTokenSubject][msg.sender]; because the used only once iside the function 

```solidity
file: blob/main/contracts/Curves.sol

404     function buyCurvesTokenWhitelisted(
        address curvesTokenSubject,
        uint256 amount,
        bytes32[] memory proof
    ) public payable {
        if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();

        presalesBuys[curvesTokenSubject][msg.sender] += amount;
        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

        verifyMerkle(curvesTokenSubject, msg.sender, proof);
        _buyCurvesToken(curvesTokenSubject, amount);
    }

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L404-L420 

## [G-07] Using delete statement can save gas

```solidity
file: blob/main/contracts/FeeSplitter.sol

84   tokensData[token].unclaimedFees[msg.sender] = 0;

110  tokensData[token].unclaimedFees[msg.sender] = 0;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L84


## [G-08] Default value initialization

 When writing a for loop, refrain from initializing variables to zero (uint256 index = 0;). Instead, use the uint256 index; As the default value of uint256 is zero. This practice lets you save some gas by avoiding initialization.

```solidity
file: blob/main/contracts/Curves.sol

47   uint256 private _curvesTokenCounter = 0;

305  for (uint256 i = 0; i < subjects.length; i++) {

330  for (uint256 i = 0; i < subjects.length; i++) {

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L47


```solidity 
file:  blob/main/contracts/FeeSplitter.sol

55   for (uint256 i = 0; i < tokens.length; i++) {

84   tokensData[token].unclaimedFees[msg.sender] = 0;

104  uint256 totalClaimable = 0;

105  for (uint256 i = 0; i < tokenList.length; i++) {

110  tokensData[token].unclaimedFees[msg.sender] = 0;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55

## [G-09] Using storage instead of memory for structs/arrays saves gas 

```solidity
file: blob/main/contracts/FeeSplitter.sol

53   address[] memory tokens = getUserTokens(user);

54   UserClaimData[] memory result = new UserClaimData[](tokens.length);

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L53

## [G-10] Save loop calls

Instead of calling subjects[i] 2 times in each loop for fetching data, it can be saved as a variable out side of the loop.


```solidity
file: main/contracts/Curves.sol

305  for (uint256 i = 0; i < subjects.length; i++) {
            uint256 amount = curvesTokenBalance[subjects[i]][msg.sender];
            if (amount > 0) {
                _transfer(subjects[i], msg.sender, to, amount);
            }
        }

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L305-L310


## [G-11] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References:

https://book.getfoundry.sh/reference/forge-std/compute-create-address https://twitter.com/transmissions11/status/1518507047943245824

```solidity
file: blob/main/contracts/Curves.sol

297   if (to == address(this)) revert ContractCannotReceiveTransfer();

303   if (to == address(this)) revert ContractCannotReceiveTransfer();

352   address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

468   _transfer(curvesTokenSubject, msg.sender, address(this), amount);

498   if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

501   _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L297

## [G-12] When using storage instead of memory, we should cache any fields that need to be re-read in stack variables

```solidity
file:  blob/main/contracts/FeeSplitter.sol

68    data.unclaimedFees[account] += owed / PRECISION;
69     data.userFeeOffset[account] = data.cumulativeFeePerToken;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L68-L69

## [G-13] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27


```solidity
file: blob/main/contracts/Security.sol

6   mapping(address => bool) public managers;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L6

```solidity
file: blob/main/contracts/Curves.sol

83   bool isBuy,

220  bool isBuy,

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L83

## [G-14] Using assembly to revert with an error message

Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Estimated Gas Savings - Here’s an example:

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}
/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```
From the example above, we can see that we get a gas saving of over 300 gas when reverting with the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

Code Snippets:

```solidity
file: main/contracts/FeeSplitter.sol

83   if (claimable == 0) revert NoFeesToClaim();

91   if (totalSupply_ == 0) revert NoTokenHolders();

115  if (totalClaimable == 0) revert NoFeesToClaim();

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L83

```solidity
file: blob/main/contracts/Curves.sol

104  if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();

124  ) revert InvalidFeeDefinition();

136  ) revert InvalidFeeDefinition();

149  ) revert InvalidFeeDefinition();

213  if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();

233  if (!success1) revert CannotSendFunds();

237  if (!success2) revert CannotSendFunds();

243  if (!success3) revert CannotSendFunds();

265  if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

270  if (msg.value < price + totalFee) revert InsufficientPayment();

284  if (supply <= amount) revert LastTokenCannotBeSold();

285  if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

297   if (to == address(this)) revert ContractCannotReceiveTransfer();

303   if (to == address(this)) revert ContractCannotReceiveTransfer();

314  if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

350  if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

371  if (supply != 0) revert CurveAlreadyExists();

384  if (startTime <= block.timestamp) revert InvalidPresaleStartTime();

386  if (supply != 0) revert CurveAlreadyExists();

396  if (supply > 1) revert CurveAlreadyExists();

412  ) revert PresaleUnavailable();

416  if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

425   if (!MerkleProof.verify(proof, presalesMeta[curvesTokenSubject].merkleRoot, leaf)) revert UnverifiedProof();

433  if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();

434   if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

491  if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

496   if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();

497   if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();

498  if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

505  if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L104


```solidity
file: blob/main/contracts/Curves.sol

330  for (uint256 i = 0; i < subjects.length; i++) {
            if (subjects[i] == curvesTokenSubject) {
                return;
            }
        }

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L330-L334


## [G-15] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

```solidity
file: blob/main/contracts/FeeSplitter.sol

55  for (uint256 i = 0; i < tokens.length; i++) {
            address token = tokens[i];
            uint256 claimable = getClaimableFees(token, user);
            result[i] = UserClaimData(claimable, token);
        }

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L55-L59

## [G-16] Shorten arrays with inline assembly

Issue Description - When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating storage.

Proposed Optimization - Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new array.

Estimated Gas Savings - Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending on original length.

```solidity
function shorten(uint[] storage array, uint newLen) internal {
  assembly {
    sstore(array_slot, newLen)
  }
}
// Rather than:
function shorten(uint[] storage array, uint newLen) internal {
  uint[] memory newArray = new uint[](newLen);
  for(uint i = 0; i < newLen; i++) {
    newArray[i] = array[i];
  }
  delete array;
  array = newArray;
}
```
Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas savings.

Code Snippet:

```solidity
file: blob/main/contracts/FeeSplitter.sol

54  UserClaimData[] memory result = new UserClaimData[](tokens.length);
 
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L54

## [G-17] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

### Local variable supply should be used instead of reading curvesTokenSupply[curvesTokenSubject]

```solidity
file: blob/main/contracts/Curves.sol

263  function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

        uint256 price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);

        if (msg.value < price + totalFee) revert InsufficientPayment();

        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;
        curvesTokenSupply[curvesTokenSubject] = supply + amount;
        _transferFees(curvesTokenSubject, true, price, amount, supply);

        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
    }

282   function sellCurvesToken(address curvesTokenSubject, uint256 amount) public {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply <= amount) revert LastTokenCannotBeSold();
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] < amount) revert InsufficientBalance();

        uint256 price = getPrice(supply - amount, amount);

        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
        curvesTokenSupply[curvesTokenSubject] = supply - amount;

        _transferFees(curvesTokenSubject, false, price, amount, supply);
    }


```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L263-L280


## [G-18] Don’t make variables public unless necessary

Issue Description - Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

Proposed Optimization - Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private or internal visibility.

Estimated Gas Savings - The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up over many transactions targeting a contract with public state variables that don’t need to be public.

Code Snippets:


```solidity
file: blob/main/contracts/Security.sol

5  address public owner;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L5

```solidity
file: blob/main/contracts/Curves.sol

42   address public curvesERC20Factory;

43   FeeSplitter public feeRedistributor;

47   uint256 private _curvesTokenCounter = 0;

77   FeesEconomics public feesEconomics;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L42


```solidity
file: blob/main/contracts/FeeSplitter.sol

10    Curves public curves;

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L10

## [G-19] Don’t cache state varible if  that are only used once

```solidity
file: blob/main/contracts/Curves.sol

370   uint256 supply = curvesTokenSupply[curvesTokenSubject];

385   uint256 supply = curvesTokenSupply[curvesTokenSubject];

395   uint256 supply = curvesTokenSupply[msg.sender];

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L370

## [G-20] Use inheritance:

In Solidity, using inheritance is often simpler and more gas-efficient than composition. When extending contracts through inheritance, child contracts can efficiently pack their variables alongside those of the parent contract.



```solidity
file:

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Parent contract
contract Animal {
    string public species;

    constructor(string memory _species) {
        species = _species;
    }

    function makeSound() public virtual returns (string memory);
}

// Child contract inheriting from Animal
contract Dog is Animal {
    string public name;

    constructor(string memory _name) Animal("Dog") {
        name = _name;
    }

    // Overriding the makeSound function from the parent
    function makeSound() public override returns (string memory) {
        return "Woof!";
    }
}

// Another child contract inheriting from Animal
contract Cat is Animal {
    string public color;

    constructor(string memory _color) Animal("Cat") {
        color = _color;
    }

    // Overriding the makeSound function from the parent
    function makeSound() public override returns (string memory) {
        return "Meow!";
    }
}
```