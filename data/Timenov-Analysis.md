## Curves Protocol's Analysis Report 

### Intoduction
This report is a summary of the analysis of Curves Codebase.
- This report is not a copy or extension of the documentation provided by Curves.
- This report provides a high-level overview of the codebase, security problems and suggestions to improve the codebase.
- The aim of this report is to provide value to the developers working on the Curves Codebase.

### Approach used to audit the codebase
Day 1-3:
- Read all of the [audit page](https://code4rena.com/audits/2024-01-curves).
- Read the [Friend Tech Smart contracts](https://basescan.org/address/0xcf205808ed36593aa40a44f10c7f7c2f67d4a4d4#code), [Friend Tech analysis](https://ada-d.medium.com/understanding-friend-tech-through-smart-contracts-edac5d98cd49) and [breakdown](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf).
- Read the codebase multiple times.
- Added audit tags in suspicious places.

Day 4-5:
- Created diagrams.
- Started thinking of edge cases.
- Filtered the findings based on severity.

Day 6:
- Submitted findings.

### Architecture
The protocol structure is simple, easy to understand and maintain. The main contract is `Curves.sol` and uses all other contracts.

### What the protocol did well
- Small and simple contracts.
- Good separation of concerns. 

### What can be improved
- External addresses can block contract interaction through `_transferFees`.
- Lack of documentation and NatSpec. Yes, this is a `friend.tech` fork, but still there should be a good explanation of the codebase.
- Tests do `NOT` cover 100% as stated in the contest info, miss key functions and edge cases.

### Centralisation Risks
- The `owner` and `manager` can control all the fees.
- `feeRedistributor` can brick `_transferFees()`

### Codebase Diagrams
[Gist](https://gist.github.com/Pavel2202/b3283dda0a867c61a7e7f691514fb7c2) with the codebase diagram.

`Curves.sol` -> base contract, allowing users to trade tokens.
`FeeSplitter.sol` -> accounts the fees of the token holders and provides claim functionality.
`Security.sol` -> provides function modifiers for the other contracts.
`CurvesERC20.sol` -> Curves token with mint and burn functionality, callable only by the owner.
`CurvesERC20Factory.sol` -> factory contract for deploying Curves tokens.

### Chains Supported
Form Network

### Systematic Risks
- Bad implementation of modifiers in `Security.sol`.
- `buy` and `sell` functionality in `Curves.sol` depend on `_transferFees` function, which itself sends `ETH` to 3 external addresses and calls 2 functions in `FeeSplitter.sol`.

### Resources Used
- Documentation provided by the Curves team

### Time Spent
40 hours

### Time spent:
40 hours