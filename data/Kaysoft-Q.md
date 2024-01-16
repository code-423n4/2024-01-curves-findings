## [L-1] The `onlyTokenSubject` modifier can be removed to reduce unnecessary complexity

There are 5 instances of these.
-  https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L158
- https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L383
- https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L432
- https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L439
- https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L460

The main issue is that the `onlyTokenSubject` tries to validate that the supplied function parameter `curvesTokenSubject` is the msg.sender. 

There is no need to validate msg.sender since you always have it. 

Consider using the `msg.sender` directly in functions and removing the `curvesTokenSubject` parameter so that users don't need to pass their address as a parameter again since it is always available as msg.sender.

Here is the `curvesTokenSubject` modifier definition

```
File: Curves.sol
103: modifier onlyTokenSubject(address curvesTokenSubject) {//@audit unneccessary
104:        if (curvesTokenSubject != msg.sender) revert UnauthorizedCurvesTokenSubject();
105:        _;
106:    }
```

And here is one of the functions where it is used.

```
File: Curves.sol
155: function setReferralFeeDestination(
156:        address curvesTokenSubject,
157:        address referralFeeDestination_
158:     ) public onlyTokenSubject(curvesTokenSubject) {//@audit just use msg.sender
159:        referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
160:     }
```

Now the `setReferralFeeDestination(...)` above can be refactored to: 

```diff
function setReferralFeeDestination(
--        address curvesTokenSubject,
        address referralFeeDestination_
--    ) public onlyTokenSubject(curvesTokenSubject) {
++    ) public {
--      referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
++      referralFeeDestination[msg.sender] = referralFeeDestination_;
    }
```
Removing the `onlyTokenSubject` above simplifies the function and removes unnecessary complexity making it concise and more readable.

Recommendation: Remove `onlyTokenSubject` modifier as described in the diff below.

```diff
function setReferralFeeDestination(
--        address curvesTokenSubject,
          address referralFeeDestination_
--    ) public onlyTokenSubject(curvesTokenSubject) {
++    ) public {
--      referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
++      referralFeeDestination[msg.sender] = referralFeeDestination_;
    }
```

## [L-2] Prevent gas-griefing with low level `call` when the returned bytes data is not needed.

There are 3 instances of this
- https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L232
- https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L236
- https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L241


Using the `.call()` native function to send ether exposes the transaction to gas greifing attack with huge return data payload. 

The returned data will have to be copied to memory which exposes the contract to gas griefing attack. Even if the returned `bytes data` is not assigned below, it is still returned and copied to memory.

```
File: Curves.sol
236: (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
237:                if (!success2) revert CannotSendFunds();
//@audit Even though the returned bytes data is not assigned above, it is still returned and copied to memory making it vulnerable to gas griefing.
```

Recommendation: 

Short Term: Use low level assembly call to send ETH and only send ETher when `amount` is not zero in order to avoid sending zero ETH.

```
bool status;
assembly {
    status := call(gas(), receiver, amount, 0, 0, 0, 0)
}
```

Long Term: Consider using this https://github.com/nomad-xyz/ExcessivelySafeCall