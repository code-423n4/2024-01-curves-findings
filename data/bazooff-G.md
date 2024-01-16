## Gas optimizations

## [G-01] Change public function visibility to external

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity

file: Curves.sol

155  function setReferralFeeDestination(
        address curvesTokenSubject,
        address referralFeeDestination_
    ) public onlyTokenSubject(curvesTokenSubject)

197   function getBuyPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256)

204     function getSellPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256)

211   function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable

282   function sellCurvesToken(address curvesTokenSubject, uint256 amount) public

364   function buyCurvesTokenWithName(
        address curvesTokenSubject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) public payable

377 function buyCurvesTokenForPresale(
        address curvesTokenSubject,
        uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
    ) public payable

404 function buyCurvesTokenWhitelisted(
        address curvesTokenSubject,
        uint256 amount,
        bytes32[] memory proof
    ) public payable

465 function withdraw(address curvesTokenSubject, uint256 amount) public

504 function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L155
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L197
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L204
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L211
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L282
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L364
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L377
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L404
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L465
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L504

## [G-02] unnecessary modifier onlyTokenSubject

```solidity
modifier onlyTokenSubject(address curvesTokenSubject) {
        if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();
        _;
    }
```

Since this modifier checks that address in the argument is the same as the msg.sender address, no need to use it in the following functions. Instead use msg.sender directly in the functions without address `curvesTokenSubject` as a function argument.

The following functions:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L155
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L377
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L428

In functions where `curvesTokenSubject` used more than once just cache it `address curvesTokenSubject = msg.sender;`
