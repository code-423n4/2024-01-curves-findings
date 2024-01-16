# Quality Assurance Report

| ID     | QA issue                                                                          |
|--------|-----------------------------------------------------------------------------------|
| [L-01](#l-01-missing-access-control-on-deploy-function-of-curveserc20factory-contract) | Missing access control on deploy() function of CurvesERC20Factory contract        |
| [L-02](#l-02-incorrect-denominator-is-used-in-getfees-function-calculation) | Incorrect denominator is used in getFees() function calculation                   |
| [L-03](#l-03-buycurvestoken-reverts-when-starttime--blocktimestamp) | buyCurvesToken() reverts when startTime == block.timestamp                        |
| [L-04](#l-04-extra-eth-can-be-locked-in-the-curves-contract-due-to-slippage) | Extra ETH can be locked in the Curves contract due to slippage                    |
| [L-05](#l-05-token-is-not-removed-from-users-list-of-tokens-when-all-shares-are-sold) | Token is not removed from user's list of tokens when all shares are sold          |
| [L-06](#l-06-zero-amount-transfers-allow-for-address-poisoning-attacks) | Zero amount transfers allow for address poisoning attacks                         |
| [L-07](#l-07-offchain-trackingmonitoring-can-be-spammed-by-0-amount-depositswithdrawals) | Offchain tracking/monitoring can be spammed by 0 amount deposits/withdrawals      |
| [L-08](#l-08-function-sellexternalcurvestokens-does-not-work-for-last-seller) | Function sellExternalCurvesTokens() does not work for last seller                 |
| [L-09](#l-09-function-onbalancechange-pushes-same-token-to-mapping-usertokens-multiple-times) | Function onBalanceChange() pushes same token to mapping userTokens multiple times |
| [N-01](#n-01-no-need-to-initialize-_curvestokencounter-to-0) | No need to initialize _curvesTokenCounter to 0                                    |
| [R-01](#r-01-consider-improving-function-names-for-readability) | Consider improving function names for readability                                 |

## [L-01] Missing access control on deploy() function of CurvesERC20Factory contract

In the [deploy()](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/CurvesERC20Factory.sol#L6) function below, we can see that it is public without any modifier to restrict access to only the Curves.sol contract (from where deployments occur). Leaving this function public would allow duplication of ERC20 token names and symbols with same or different owners. This could open up doors for phishing attacks, where an attacker can trick users into believing the malicious token is legitimate.
```solidity
File: CurvesERC20Factory.sol
09:     function deploy(string memory name, string memory symbol, address owner) public returns (address) {
10:         CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);
11:         return address(tokenContract);
12:     }
```

## [L-02] Incorrect denominator is used in getFees() function calculation

In the getFees() function below, we see that 1 ether is used as the denominator when calculating each of the fees. This use of 1 ether is incorrect since each of the fee percents are scaled according to the maxFeePercent. For example, if maxFeePercent = 1e18, 20% would be represented as 2e17. If maxFeePercent = 10000, 20% would be represented as 2000 (this is the bps model where 1% = 100 bps). 

Unfortunately, since we use 1e18 in the denominator, maxFeePercent is restricted to only use 1e18 as its value. I'm not sure if the team realized this issue but it would be a huge problem if maxFeePercent is set to a value other than 1e18. This would cause more or less fees being charged since each of the percents would be divided by 1e18 instead of the correct order of magnitude maxFeePercent resides in.

**Solution: Use maxFeePercent in the denominator to make sure the fees will always be calculated correctly if maxFeePercent and the respective percents change.**
```solidity
File: Curves.sol
168:     function getFees(
169:         uint256 price
170:     )
171:         public
172:         view
173:         returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
174:     {
175:         protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
176:         subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
177:         referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
178:         holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
179:         totalFee = protocolFee + subjectFee + referralFee + holdersFee;
180:     }
```

## [L-03] buyCurvesToken() reverts when startTime == block.timestamp

The second condition startTime >= block.timestamp denies user/caller from purchasing curves token right at block.timestamp. Since the open sale is supposed to start at startTime, ensure that calling buyCurvesToken() works even when block.timestamp is equal to it.

**Solution: Use > instead of >=**
```solidity
File: Curves.sol
214:     function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
215:         uint256 startTime = presalesMeta[curvesTokenSubject].startTime;
217:         if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
218: 
219:         _buyCurvesToken(curvesTokenSubject, amount);
220:     }
```

## [L-04] Extra ETH can be locked in the Curves contract due to slippage

Extra ETH is permanently locked in the Curves contract and not refunded to users. This kind of situation is bound to occur since users will face MEV and slippage when buying/selling shares. This will cause the extra ETH to be sent to the contract since a strict equality check is not used.

**Solution: Use == instead of <.**
```solidity
File: Curves.sol
273:         if (msg.value < price + totalFee) revert InsufficientPayment(); 
```

## [L-05] Token is not removed from user's list of tokens when all shares are sold

The function sellCurvesToken() does not remove the token from user's list of tokens in mapping `ownedCurvesTokenSubjects` if user sells all tokens i.e. has balance = 0. This would cause issues when the frontend pulls the list of tokens a user holds.
```solidity
File: Curves.sol
294:         curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount; 
295:         curvesTokenSupply[curvesTokenSubject] = supply - amount;
296: 
297:         _transferFees(curvesTokenSubject, false, price, amount, supply);
```

## [L-06] Zero amount transfers allow for address poisoning attacks

In the current code, zero value transfers are allowed. This could be lead to address poisoning attacks, which might cause users to send their tokens to the wrong address (due to frontend displaying recent address transfers). 

**Solution: Disallow zero value transfers**
```solidity
File: Curves.sol
320:     function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
321:         if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();
322: 
323:         // If transferring from oneself, skip adding to the list
324:         if (from != to) {
325:             _addOwnedCurvesTokenSubject(to, curvesTokenSubject); 
326:         }
327: 
328:         curvesTokenBalance[curvesTokenSubject][from] = curvesTokenBalance[curvesTokenSubject][from] - amount;
329:         curvesTokenBalance[curvesTokenSubject][to] = curvesTokenBalance[curvesTokenSubject][to] + amount;
330: 
331:         emit Transfer(curvesTokenSubject, from, to, amount);
332:     }
```

## [L-07] Offchain tracking/monitoring can be spammed by 0 amount deposits/withdrawals

Currently, zero value deposits and withdrawals are possible through the deposit() and withdraw() function. Due to this, the offchain tracking system implemented by the team can be spammed with these useless transfers.

**Solution: Consider disallowing 0 value deposits/withdrawals.**
```solidity
File: Curves.sol
473: 
474:     function withdraw(address curvesTokenSubject, uint256 amount) public {
475:         if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance(); 
476: 
477:         address externalToken = externalCurvesTokens[curvesTokenSubject].token;
478:         if (externalToken == address(0)) {
479:             if (
480:                 keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
481:                 keccak256(abi.encodePacked("")) ||
482:                 keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
483:                 keccak256(abi.encodePacked(""))
484:             ) {
485:                 externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
486:                 externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
487:             }
488:             _deployERC20(
489:                 curvesTokenSubject,
490:                 externalCurvesTokens[curvesTokenSubject].name,
491:                 externalCurvesTokens[curvesTokenSubject].symbol
492:             );
493:             externalToken = externalCurvesTokens[curvesTokenSubject].token;
494:         }
495:         _transfer(curvesTokenSubject, msg.sender, address(this), amount);
496:         CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
497:     }
498: 
499:    
500:     function deposit(address curvesTokenSubject, uint256 amount) public {
501:         if (amount % 1 ether != 0) revert NonIntegerDepositAmount(); 
502: 
503:         address externalToken = externalCurvesTokens[curvesTokenSubject].token;
504:         uint256 tokenAmount = amount / 1 ether;
505: 
506:         if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
507:         if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
508:         if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();
509: 
510:         CurvesERC20(externalToken).burn(msg.sender, amount);
511:         _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
512:     }
```

## [L-08] Function sellExternalCurvesTokens() does not work for last seller

The last seller will never be able to reintegrate his ERC20 token through the sellExternalCurvesTokens() function because after depositing amount through deposit() (on Line 516), the call to sellCurvesToken() (on Line 517) would revert since we pass in the whole amount instead of (amount - 1). Due to this, this function would revert with lastTokenCannotBeSold() error and the normal functioning of the function is impacted.
```solidity
File: Curves.sol
513:     function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public {
514:         if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject(); 
515: 
516:         deposit(curvesTokenSubject, amount);
517:         sellCurvesToken(curvesTokenSubject, amount / 1 ether); 
518:     }
```

## [L-09] Function onBalanceChange() pushes same token to mapping userTokens multiple times

In the function onBalanceChange(), if the balance of a user for a specific token subject is greater than 0, it will push the token to the userTokens mapping. The issue is that if a user buys more tokens on top of existing ones, the function will still push the same token to userTokens once again. This will keep on happening as the user keeps on buying.

The functions affected are [getUserTokens()](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L48) and [getUserTokensAndClaimable()](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L52). This can especially affect the [getUserTokensAndClaimable()](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L52) function since it runs a for loop which could run out gas and revert if the array gets too long. Due to this, the user tokens and claimable fees will not be obtainable by the frontend.
```solidity
File: FeeSplitter.sol
099:     function onBalanceChange(address token, address account) public onlyManager {
100:         TokenData storage data = tokensData[token]; 
101:         data.userFeeOffset[account] = data.cumulativeFeePerToken; 
102:         if (balanceOf(token, account) > 0) userTokens[account].push(token); 
103:     }
```

## [N-01] No need to initialize _curvesTokenCounter to 0

Default value of uint256 _curvesTokenCounter is 0, therefore assigning it 0 explicitly is not required.
```solidity
File: Curves.sol
47:     uint256 private _curvesTokenCounter = 0; 
```

## [R-01] Consider improving function names for readability

Currently alot of functions in the Curves contract have names that do not suit their intended behaviour. For example, the mint() function actually deploys a CurvesERC20 token and does not mint any tokens. Naming it deployERC20() would improve the readability of the code and make it much easier to follow.