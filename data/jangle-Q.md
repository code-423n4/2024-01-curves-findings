### QA

1. The `onlyTokenSubject(curvesTokenSubject)` modifier in the `Curves` contract is meaningless and can be deleted by change the `curvesTokenSubject` parameter to the `msg.sender` to save gas.

2. In the `Security` contract, the `transferOwnership` function should add validation for the new owner before the ownership is transferred.

3. Add key events or indexed keyword in events for tracking the important parameters changes easily, for example:
    3.1 emit an event for the `transferOwnership` or `setManager` function in the `Security` contract to track the owner change.
    3.2 add indexed keywords for the `Trade` event in the `Curves` contract.