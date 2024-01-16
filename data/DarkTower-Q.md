## [L-01] Use similar ownership transfers to Ownable2Step
The `Security.transferOwnership()` function is designed to allow the current owner pass on ownership of the contract to another address. It does this in a one way step which poses a single point of failure risk for the protocol. When transferring ownership, it's better to use a similar technique to Ownable2Step. Ownable2Step further prevents risks posed by centralized privileges as there is a smaller likelihood of the owner being wrongfully changed.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L23-L25

```solidity
  function transferOwnership(address owner_) public onlyOwner {
    @> owner = owner_;
  }
```

## [L-02] Add zero value check for merkleroot setting
The function `setWhitelist()` of the `Curves.sol` contract allows setting a merkleroot for a subject token presale. However, if set to 0, the presale will fail during the merkle verification check which means no one can put in buy trades during the presale because if merkleroot == 0 nobody can buy.
We recommend adding validation check to ensure the bytes32 value passed is non-zero.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L394-L402

```solidity
  function setWhitelist(bytes32 merkleRoot) external {
  uint256 supply = curvesTokenSupply[msg.sender];
  if (supply > 1) revert CurveAlreadyExists();

  if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
      @> presalesMeta[msg.sender].merkleRoot = merkleRoot; // @audit can still set zero value
      emit WhitelistUpdated(msg.sender, merkleRoot);
  }
}
```

## [L-03] Add a function to retrieve ether
In the `Curves.sol` contract, the protocol should add a function to retrieve ether from the contract. For example, a user could lose access to their wallet or the protocol needs to upgrade fees contract. We recommend that the curves team add a function retreive ether from the `Curves.sol` contract as well as the `FeeSplitter.sol` contract.


```solidity
function adminRetrieveEther() public onlyOwner {
  // @audit do logic to retrieve ether from the contract
}
```

## [L-04] Have a limit for the size of strings
In the `setNameAndSymbol()` the protocol should limit the sizes of the `name` & `symbol` inputs for the `curvesTokenSubject`. A malicious user, could create random token subject with huge strings. However, general assumption is that nothing malicious can come from this, but setting a limit also reduces read/write cost ~ gas for the protocol in general.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L428-L437

```solidity
  function setNameAndSymbol(
      address curvesTokenSubject,
      string memory name,
      string memory symbol
  ) external onlyTokenSubject(curvesTokenSubject) {
      if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
      if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();
      @> externalCurvesTokens[curvesTokenSubject].name = name; // @audit no limit on size
      @> externalCurvesTokens[curvesTokenSubject].symbol = symbol; // @audit no limit on size
  }
```

## [NC-01] Use better revert messages
We understand that token buys can only start with the amount being one. But we think rather than bubbling a misleading revert message, the team can use better error messages across reverts on the contracts that helps the user pinpoint exactly what caused the revert of such transactions. For example, in the `_buyCurvesToken` if I try to buy more than 1 tokens for the first buy as a subject, it reverts with the error message `arithmetic underflow or overflow`. This is not useful for the end user can be handled to throw a custom error message during the revert.

```solidity
    [15459] CurvesTest::testUserBetterMessages()
    ├─ [0] VM::startPrank(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266])
    │   └─ ← ()
    ├─ [5058] Curves::buyCurvesToken(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], 2)
    │   └─ ← panic: arithmetic underflow or overflow (0x11)
    └─ ← panic: arithmetic underflow or overflow (0x11)

Test result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 21.35ms
```

## [NC-02] User can execute `buyCurvesTokenWithName` even if supply != 0
The protocol wants token subjects to only execute this function when supply == 0.
However, `tokenSubjects` can still run the combination `buyCurvesToken` + `setNameAndSymbol` + `mint` which does the exact same thing as `buyCurvesTokenWithName` even when supply != 0
We recommend removing this check as it stops nothing in this case.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L364-L375

```solidity
    function buyCurvesTokenWithName(
    address curvesTokenSubject,
    uint256 amount,
    string memory name,
    string memory symbol
) public payable {
    uint256 supply = curvesTokenSupply[curvesTokenSubject];
    @> if (supply != 0) revert CurveAlreadyExists();

    _buyCurvesToken(curvesTokenSubject, amount);
    _mint(curvesTokenSubject, name, symbol);
}
```