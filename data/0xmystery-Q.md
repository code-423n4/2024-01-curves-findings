## [L-01] Deposit Function and ERC20 Token Management
`Curves.deposit()` enforces a condition that only whole numbers of Ether can be deposited, potentially complicating the user experience in ERC20 token management. This restriction, aimed at simplifying internal accounting, might not align well with common cryptocurrency practices where fractional token ownership is standard. Users who trade or transact with these tokens may face challenges when redepositing fractional amounts back into the contract. This would mean `amount % 1 ether` would mandate up to 0.9999... ETH worth of external token non-depositable for a user. Balancing contract simplicity with user convenience and market norms is crucial for optimal functionality and user satisfaction.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L490-L491

```solidity
    function deposit(address curvesTokenSubject, uint256 amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
```
## [L-02] Enhancing Security and Reliability in Low-level Ether Transfers
In the context of `Curves._transferFees()`, the use of low-level `call()` for transferring Ether presents risks, particularly when dealing with contracts lacking `receive()` or those intentionally designed to fail transactions via gas griefing. This would lead to DoS on calls originating from [_buyCurvesToken()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L274) and [sellCurvesToken()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L282) primarily. To mitigate these risks, implementing a gas limit on `call()` could prevent gas griefing, while transitioning to Wrapped Ether (WETH) offers a more robust and ERC20-compliant transfer method, avoiding issues with contracts not equipped to handle direct Ether transfers. 

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L232-L244

```solidity
                (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
                if (!success1) revert CannotSendFunds();
            }
            {
                (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
                if (!success2) revert CannotSendFunds();
            }
            {
                (bool success3, ) = referralDefined
                    ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                    : (true, bytes(""));
                if (!success3) revert CannotSendFunds();
            }
```
## [L-03] Optimizing Fee Calculation to Minimize Rounding Errors
In `Curves.getFees()`, the current methodology calculates each fee component (`protocolFee`, `subjectFee`, `referralFee`, `holdersFee`) separately from the transaction price, leading to potential rounding errors due to Solidity's integer arithmetic. An alternative approach, where the `totalFee` is calculated first as a cumulative percentage of the price, and then `protocolFee` is derived by subtracting other fees from `totalFee`, could reduce these rounding discrepancies. The key is to align the method with the contract's fee structure while ensuring precision, especially important in financial transactions where even minimal rounding errors can accumulate significantly over time.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L173-L177

```solidity
        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
```
## [L-04] Implementing a Recovery Function for Accumulated Ether in Smart Contracts
Incorporating a function to retrieve accumulated Ether due to rounding errors, [excess ETH](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L270) when buying curves, and unexpected deposits (such as Ether sent by self-destruct from other contracts or accidentally in the presence of [receive()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L119)) can significantly enhance the functionality and trustworthiness of a smart contract. This function addresses the potential issue of small, unclaimed Ether amounts getting trapped in the contract over time. However, its implementation must prioritize strict access control to prevent misuse, ensure transparency for maintaining user trust. Additionally, this function should be designed to only withdraw Ether that is not part of the contract's normal operational balance, thereby safeguarding the interests of all stakeholders involved with the contract.

## [L-05] Essential Recovery Function for Exported ERC20 Tokens in the Curves Protocol
Implementing a recovery function for exported ERC20 tokens in the Curves smart contract is critical for maintaining the integrity of its ecosystem, where the ERC20 token supply must precisely mirror the value locked within the protocol, as highlighted by the protocol in the readme section: 

"For any token associated with Curves, it's imperative that the total ERC20 supply remains exactly equivalent to the value locked within the Curves protocol. This one-to-one correspondence ensures consistency and integrity between the ERC20 tokens in circulation and the underlying assets within the Curves ecosystem."

This function addresses the risk of accidental token deposits, ensuring the protocol's balance and consistency. It necessitates strict access control, comprehensive testing, and transparency to maintain user trust and prevent unauthorized access. This feature not only acts as a safeguard against user errors but also reinforces the reliability and stability of the Curves protocol, ensuring that the total ERC20 supply accurately reflects the underlying assets, thus upholding the ecosystem's integrity and trustworthiness.

Here's a suggested fix that will have a side benefit of retrieving any other non-exported ERC20 tokens if need be:

```solidity
import {IERC20} from '@openzeppelin/contracts/token/ERC20/IERC20.sol';
import {SafeERC20} from '@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol';

using SafeERC20 for IERC20;

    function recoverERC20(address token_, uint256 amount, address recipient) external onlyOwner {
        if (amount > 0) {
            IERC20(token_).safeTransfer(recipient, amount);
            emit ERC20Recovered(token_, amount);
        }
    }
```
## [L-06] Implementing a Modifier for `updateFeeCredit` function
With numerous vulnerabilities needing `updateFeeCredit()` to be called pre or post critical functions I have reported separately, I suggest the protocol introduce two separate modifiers in Security.sol as follows:

This will concern [Curves._buyCurvesToken()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L263), [Curves.sellCurvesToken()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L282), [Curves.withdraw()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L465), and [FeeSplitteronBalanceChange()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L96):  

```solidity
    modifier preUpdateFeeCredit(address token, address account) {
        feeRedistributor.updateFeeCredit(token, account);
        _;
    } 
```
This will concern [Curves.deposit()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L490):

```solidity
    modifier postUpdateFeeCredit(address token, address account) {
        _;
        feeRedistributor.updateFeeCredit(token, account);
    } 
```
## [L-07] Addressing Redundancies in `FeeSplitter.getUserTokensAndClaimable()`
In the `onBalanceChange` function, it could lead to redundant entries in the `userTokens` array. This flaw would cause misleading outputs in the [getUserTokensAndClaimable](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L52-L61) function, potentially confusing users about their claimable fees. Consider implementing a check to prevent duplicate entries of tokens in a user's array. This modification ensures each token is listed only once per user, thereby enhancing the contract's accuracy and user experience. However, it's important to note the potential gas inefficiencies with large arrays, suggesting that an alternative approach using mappings might be more scalable for systems with numerous tokens.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L96

```diff
    function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
-        if (balanceOf(token, account) > 0) userTokens[account].push(token);

+        // Check if the token is already in the user's token array to prevent duplicates
+        bool isTokenAlreadyAdded = false;
+        for (uint i = 0; i < userTokens[account].length; i++) {
+            if (userTokens[account][i] == token) {
+                isTokenAlreadyAdded = true;
+                break;
+            }
+        }

+        // Add the token to the user's token array if it's not already there
+        if (!isTokenAlreadyAdded && balanceOf(token, account) > 0) {
+            userTokens[account].push(token);
+        }
    }
```
## [L-08] ETH Liquidity Challenges in Curves.sol
In the realm of decentralized finance, the design of smart contracts plays a pivotal role in maintaining market stability and user trust. A crucial aspect of this design is ensuring adequate liquidity, particularly in scenarios where there's a surge in selling activity. Contracts that solely rely on token purchases as their ETH source:

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L211

```solidity
    function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L369

```solidity
    ) public payable {

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L383

```solidity
    ) public payable onlyTokenSubject(curvesTokenSubject) {

```
https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L408

```solidity
    ) public payable {
```
may face challenges in fulfilling [sell orders](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L232-L233) if the demand to sell surpasses the available ETH, leading to potential liquidity crises.

Apparently, the quadratic bonding formula might not guarantee a zero sum game, i.e. the gain of users buying low and selling high exactly matches the loss of users buying high and selling low.

This situation underscores the importance of incorporating diverse mechanisms such as external ETH funding sources, reserve pools, or dynamic pricing models to safeguard against liquidity shortages. Such proactive measures in smart contract design not only enhance the robustness of the financial ecosystem but also bolster user confidence, ensuring a smoother and more reliable trading experience.

## [NC-01] Optimizing the `_buyCurvesToken` Function for Efficiency and Clarity
Introducing an if block in `Curves._buyCurvesToken()` to check if supply is non-zero before executing `getPrice()` and `getFees()` could significantly enhance transaction efficiency and align with the contract's design of offering a free initial token purchase. 

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L267-L268

```diff
+    uint256 price;
+    if (supply != 0) {
-        uint256 price = getPrice(supply, amount);
+        price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);
+    }
```
## [NC-02] Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.7",
settings: {
 optimizer: {
   enabled: true,
   runs: 1000,
 },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.