## Impact
CurveToken:_transfer has no validation to avoid transfer of tokens to the zero address. the transfer of funds to the zero address will cause curvetoken to be lost forever.
## instances 1
- https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L322
## Proof of Concept
https://gist.github.com/Falilah/3e1778390d82939e2ab9089a13929dc8
```
 testCurve::testTransferToAddressZero()
    ├─ [0] VM::prank(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266])
    │   └─ ← ()
    ├─ [114765] Curves::buyCurvesToken(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], 1)
    │   ├─ [0] 0x0000000000000000000000000000000000000000::fallback()
    │   │   └─ ← ()
    │   ├─ [0] owner::fallback()
    │   │   └─ ← ()
    │   ├─ emit Trade(trader: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], subject: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], isBuy: true, tokenAmount: 1, ethAmount: 0, protocolEthAmount: 0, subjectEthAmount: 0, supply: 1)
    │   └─ ← ()
    ├─ [3851] Curves::getBuyPriceAfterFee(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], 50) [staticcall]
    │   └─ ← 2682812500000000000 [2.682e18]
    ├─ [0] VM::prank(user1: [0x29E3b139f4393aDda86303fcdAa35F60Bb7092bF])
    │   └─ ← ()
    ├─ [77645] Curves::buyCurvesToken{value: 2682812500000000000}(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], 50)
    │   ├─ [0] 0x0000000000000000000000000000000000000000::fallback()
    │   │   └─ ← ()
    │   ├─ [0] owner::fallback()
    │   │   └─ ← ()
    │   ├─ emit Trade(trader: user1: [0x29E3b139f4393aDda86303fcdAa35F60Bb7092bF], subject: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], isBuy: true, tokenAmount: 50, ethAmount: 2682812500000000000 [2.682e18], protocolEthAmount: 0, subjectEthAmount: 0, supply: 51)
    │   └─ ← ()
    ├─ [0] VM::prank(user1: [0x29E3b139f4393aDda86303fcdAa35F60Bb7092bF])
    │   └─ ← ()
    ├─ [70743] Curves::transferCurvesToken(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], 0x0000000000000000000000000000000000000000, 10)
    │   ├─ emit Transfer(curvesTokenSubject: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], from: user1: [0x29E3b139f4393aDda86303fcdAa35F60Bb7092bF], to: 0x0000000000000000000000000000000000000000, value: 10)
    │   └─ ← ()
    ├─ [787] Curves::curvesTokenBalance(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], 0x0000000000000000000000000000000000000000) [staticcall]
    │   └─ ← 10
    └─ ← ()
```



## Tools Used
manual review and foundry

## Recommended Mitigation Steps
It is advisable for the team to add a check to be sure fun is not been transferred to the zero address to avoid loss of funds