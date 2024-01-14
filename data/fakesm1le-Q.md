# Single step ownership transfter might be leading to lost control over protocol

## Lines of code

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L27
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L7

## Impact
The impact is high as it might brick the protocol.
The likelihood is low as it requires a fault on the admin side.

## Proof of Concept

`transferOwnership` takes any address from the owner and sets it as new owner.
```
    function transferOwnership(address owner_) public onlyOwner {
        owner = owner_;
    }
```

There is no zero address check there, so the owner by mistake can set the new owner to zero address (or any address that is not in their control).
Such mistake effects in losing an access to the protocol forever as there is no other way to correct such mistake e.g. two step transfer of ownership.

Additionaly, the `CurvesERC20` contract uses `transferOwnership` from openzeppelin's `Ownable` library, while rest of the protocol uses custom transfer ownership implementation from `Security` contract - that should be unified to use one common implementation.

## Tools Used
Manual review

## Recommended Mitigation Steps
- use Ownable2Step instead of Ownable for safer protocol ownership transfer
- unify ownership transfer implementation across the protocol