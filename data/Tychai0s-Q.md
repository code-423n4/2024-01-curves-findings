## Consolidated QA/Non-Critical Issues Report for Solidity Smart Contracts

---

### Issue 1: Redundant Parameter in `mint` Function

#### Impact
The inclusion of the redundant `curvesTokenSubject` parameter in the `mint` function leads to inefficiencies and potential confusion.

#### Proof of Concept
The function restricts itself to only being called by the owner of the `curvesTokenSubject`, hence the `curvesTokenSubject` can be inferred from `msg.sender`.

#### Recommended Mitigation Steps
Remove the `curvesTokenSubject` parameter and use `msg.sender` to determine the subject.

#### Issue Type
Invalid Validation

---

### Issue 2: Unnecessary Payable Function in `buyCurvesTokenForPresale` 

#### Impact
The `buyCurvesTokenForPresale` function is marked as `payable` unnecessarily, leading to potential confusion, this function can only be called once while the curvesTokenSubject has not bought the first share, which is free (no need to send any value), therefore it's unnecessary to mark it as payable.

#### Proof of Concept
The function can only be called with a fixed amount of `1`, resulting in the `getPrice` function returning `0`, making Ether transfers irrelevant.

#### Recommended Mitigation Steps
Remove the `payable` keyword to accurately reflect the function's capabilities and limitations.

#### Issue Type
Invalid Validation

---

### Issue 3: Inefficient Loop Execution in `getUserTokensAndClaimable` 

#### Impact
Repetitive length checks of the `tokens` array in a loop lead to increased gas costs.

#### Proof of Concept
The `getUserTokensAndClaimable` function checks the length of the `tokens` array in each iteration, causing unnecessary computational overhead.

#### Recommended Mitigation Steps
Cache the length of the `tokens` array in a local variable before the loop to reduce memory reads and gas costs.

#### Issue Type
Loop

---

### Issue 4: Inefficient Storage Reading in `_addOwnedCurvesTokenSubject` 

#### Impact
Repeated storage reads within a loop in `_addOwnedCurvesTokenSubject` lead to unnecessary gas consumption.

#### Proof of Concept
The length of the `subjects` array is read from storage in each iteration of the loop, increasing the gas cost.

#### Recommended Mitigation Steps
Read the length of the `subjects` array into a local variable outside of the loop to optimize gas usage.

#### Issue Type
Loop

---
