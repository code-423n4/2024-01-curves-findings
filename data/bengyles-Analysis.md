# Analysis report

## tools used

- [VSCodium](https://vscodium.com)
- [Slither](https://github.com/crytic/slither) with [Slitherin](https://github.com/pessimistic-io/slitherin)
- [Aderyn](https://github.com/Cyfrin/aderyn)
- [Solidity metrics](https://github.com/Consensys/solidity-metrics)

## Steps taken

- Read all the documentation to get an idea of how the protocol works
- Run slither with slitherin, aderyn and solidity metrics reports to get an idea of potential problem areas
- Run tests
- Read solidity code line by line and mark potential issues
- Read test files to see if there are edge cases that were missed
- Create additional tests / POC's to see if my findings are true and create reports for them

## general remarks

In general the code looks like it probably hasn't been reviewed yet (which is probably the case). I would advice to let a few other solidity developers look at the code before doing an audit as it would probably remove a lot of issues already just to have some more eyes on it. There are also a lot of free tools that can be used to enhance the code quality and reduce the number of bugs.

## proposed improvements

- Many issues were found by bots in the bot race, which means using statis analyser tools like the ones listed above could be very beneficial in finding issues in the code. They won't find all of them but it's a good start.
- Make sure that all input data is verified as much as possible using `require()` statements, even if a function is protected by access control it doesn't mean that the sender cannot make mistakes or variables are not initialized properly.
- Always assume that accounts could be smart contracts. Use `call` for ETH transfers, use of `transfer` is outdated and will fail when the user uses a smart contract wallet.
- always limit the size of arrays when looping through objects, especially when using external calls. This would reduce the chances of a transaction running out of gas or your contracts being attacked in order to break the protocol (DOS)
- Don't re-invent the weel, it is recommended to use already audited libraries as much as possible, example here would be the access control library by openZeppelin. This would have prevented issues in `Security.sol` where a wrong check in the modifiers leaves the contracts open to everyone.
- Tests don't cover all edge cases which is why a lot of issues were missed. Writing more and better tests is a proven way to improve the solidity code. In additing making use of the hardhat-coverage package would go a long way. Fuzz testing is also recommended as it would present a large array of possibilities in your tests.


### Time spent:
16 hours