## Low Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [L-01] | Potential Gas Griefing Due to Non-Handling of Return Data in External Calls | 1 |
| [L-02] | Risk of Permanently Locked Ether | 1 |
| [L-03] | Input Validation Missing in Setter Functions | 7 |
| [L-04] | Missing Contract-Existence Checks Before Low-Level Calls | 3 |
| [L-05] | Unbounded Gas Consumption on External Calls | 2 |
| [L-06] | Consider Using `Ownable2Step` rather than `Ownable` | 2 |


## NonCritical Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [N-01] | Style guide: Non-`external`/`public` variable names should begin with an underscore | 4 |
| [N-02] | Style guide: Contract does not follow the Solidity style guide's suggested layout ordering | 4 |
| [N-03] | Large numeric literals should use underscores for readability | 1 |
| [N-04] | Long functions should be refactored into multiple, smaller, functions | 1 |
| [N-05] | `public` functions not called by the contract should be declared `external` instead | 18 |
| [N-06] | Prefer Casting to `bytes` or `bytes32` Over `abi.encodePacked()` for Single Arguments | 9 |
| [N-07] | Style guide: Lines are too long | 2 |
| [N-08] | Avoid the use of sensitive terms | 4 |
| [N-09] | Unused import | 1 |
| [N-10] | Explicit Visibility Recommended in Variable/Function Definitions | 1 |


## Low Findings Details


### [L-01] Potential Gas Griefing Due to Non-Handling of Return Data in External Calls

Due to the EVM architecture, return data (bool success,) has to be stored.
However, when 'out' and 'outsize' values are given (0,0), this storage disappears.
This can lead to potential gas griefing/theft issues, especially when dealing with external contracts.
```solidity
assembly {
    success: = call(gas(), dest, amount, 0, 0)
}
require(success, "transfer failed");

```
Consider using a safe call pattern above to avoid these issues.
The following instances show the unsafe external call patterns found in the code.

`curvesTokenSubject` is from `referralFeeDestination` mapping can be any address.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

236: (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
```
[232](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L232) | [236](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L236)
</details>


### [L-02] Risk of Permanently Locked Ether

When Ether is mistakenly sent to a contract without a means of retrieval, it becomes irrevocably locked.
Incidents of accidental Ether transfers have been observed even in high-profile projects, potentially leading to significant financial setbacks.
To enhance contract resilience, it's recommended to incorporate an "Ether recovery" function to serve as a protective measure against unintended Ether lockups.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/FeeSplitter.sol

119: receive() external payable {}
```
[119](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L119)
</details>


### [L-03] Input Validation Missing in Setter Functions

Setter functions are utilized to update the state variables of a contract.
It is critical to ensure these functions have adequate input sanitization to prevent unwanted alterations or malicious attacks.
Without input validation, there's a potential risk of enabling vulnerabilities like overflow/underflow, unauthorized access, or insertion of invalid data.
Consider incorporating appropriate validation mechanisms, such as checking the range or type of inputs, to enhance the security of your contract.

<details>
<summary><i>7 issue instances in 3 files:</i></summary>

```solidity
File: contracts/Curves.sol

/// @audit `setReferralFeeDestination` function does not validate `curvesTokenSubject` input
154: function setReferralFeeDestination(
        address curvesTokenSubject,
        address referralFeeDestination_
    ) public onlyTokenSubject(curvesTokenSubject) {
/// @audit `setReferralFeeDestination` function does not validate `referralFeeDestination_` input
154: function setReferralFeeDestination(
        address curvesTokenSubject,
        address referralFeeDestination_
    ) public onlyTokenSubject(curvesTokenSubject) {
/// @audit `setERC20Factory` function does not validate `factory_` input
161: function setERC20Factory(address factory_) external onlyOwner {
/// @audit `setNameAndSymbol` function does not validate `name` input
427: function setNameAndSymbol(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) external onlyTokenSubject(curvesTokenSubject) {
```
[154](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L154) | [154](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L154) | [161](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L161) | [427](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L427)

```solidity
File: contracts/FeeSplitter.sol

/// @audit `setCurves` function does not validate `curves_` input
34: function setCurves(Curves curves_) public {
```
[34](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L34)

```solidity
File: contracts/Security.sol

/// @audit `setManager` function does not validate `manager_` input
22: function setManager(address manager_, bool value) public onlyOwner {
/// @audit `setManager` function does not validate `value` input
22: function setManager(address manager_, bool value) public onlyOwner {
```
[22](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L22) | [22](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L22)
</details>


### [L-04] Missing Contract-Existence Checks Before Low-Level Calls

When making low-level calls, it's crucial to ensure the existence of the contract at the specified address. 
If the contract doesn't exist at the given address, low-level calls will still return success, potentially causing errors in the code execution.
Therefore, alongside zero-address checks, adding an additional check to verify that <address>.code.length > 0 before making low-level calls would be recommended.

<details>
<summary><i>3 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

232: (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
236: (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
241: ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
```
[232](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L232) | [236](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L236) | [241](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L241)
</details>


### [L-05] Unbounded Gas Consumption on External Calls

External calls in your code don't specify a gas limit, which can lead to scenarios where the recipient consumes all transaction's gas causing it to revert. 
Consider using `addr.call{gas: <amount>}("")` to set a gas limit and prevent potential reversion due to gas consumption.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

232: (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
236: (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
```
[232](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L232) | [236](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L236)
</details>


### [L-06] Consider Using `Ownable2Step` rather than `Ownable`

To enhance the security and prevent inadvertent ownership transfers, it's advisable to use `Ownable2Step` or `Ownable2StepUpgradeable`.
Contracts necessitate an active confirmation from the recipient before the ownership transfer is finalized.
This mechanism serves as a safeguard against scenarios where, for instance, a typo in the address could lead to unintentional ownership changes.

By implementing a two-step confirmation process, contracts can better ensure the accurate and intentional transfer of ownership.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: contracts/CurvesERC20.sol

4: import "@openzeppelin/contracts/access/Ownable.sol";
7: contract CurvesERC20 is ERC20, Ownable {
```
[4](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L4) | [7](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L7)
</details>


## NonCritical Findings Details

### [N-01] Style guide: Non-`external`/`public` variable names should begin with an underscore

The naming convention for non-public (private and internal) variables in Solidity recommends the use of a leading underscore.

Since `constants` can be public, to avoid confusion, they should also be prefixed with an underscore.

This practice clearly differentiates between public/external and non-public variables, enhancing code clarity and reducing the likelihood of misinterpretation or errors.
Following this convention improves code readability and maintainability.

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Curves.sol

101: mapping(address => address[]) private ownedCurvesTokenSubjects;
```
[101](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L101)

```solidity
File: contracts/FeeSplitter.sol

11: uint256 constant PRECISION = 1e18;
28: mapping(address => TokenData) internal tokensData;
29: mapping(address => address[]) internal userTokens;
```
[11](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L11) | [28](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L28) | [29](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L29)
</details>

### [N-02] Style guide: Contract does not follow the Solidity style guide's suggested layout ordering

Adhering to a recommended order in Solidity contracts enhances code readability and maintenance.
[More information in Documentation](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout)
It's recommended to use the following order:
1. Type declarations
2. State variables
3. Events
4. Errors
5. Modifiers
6. Functions

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Curves.sol

/// @audit `type` declared after `state variable`
48: struct ExternalTokenMeta {
        string name;
        string symbol;
        address token;
    }
/// @audit `type` declared after `state variable`
67: struct FeesEconomics {
        address protocolFeeDestination;
        uint256 protocolFeePercent;
        uint256 subjectFeePercent;
        uint256 referralFeePercent;
        uint256 holdersFeePercent;
        uint256 maxFeePercent;
    }
/// @audit `state variable` declared after `event`
96: mapping(address => mapping(address => uint256)) public curvesTokenBalance;
```
[48](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L48) | [67](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L67) | [96](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L96)

```solidity
File: contracts/FeeSplitter.sol

/// @audit `type` declared after `error`
16: struct TokenData {
        uint256 cumulativeFeePerToken;
        mapping(address => uint256) userFeeOffset;
        mapping(address => uint256) unclaimedFees;
    }
```
[16](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L16)
</details>

### [N-03] Large numeric literals should use underscores for readability

Large numeric literals are often hard to read and prone to mistakes.
To improve readability and reduce the likelihood of errors, it is recommended to separate the digits of large numeric literals using underscores.
For example, instead of '1000000', use '1_000_000'.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

186: return (summation * 1 ether) / 16000;
```
[186](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L186)
</details>

### [N-04] Long functions should be refactored into multiple, smaller, functions

Functions that span many lines can be hard to understand and maintain.
It is often beneficial to refactor long functions into multiple smaller functions.
This improves readability, makes testing easier, and can even lead to gas optimization if it eliminates the need for variables that would otherwise have to be stored in memory.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

 /// @audit 35 lines (excluding comments)
217: function _transferFees(
        address curvesTokenSubject,
        bool isBuy,
        uint256 price,
        uint256 amount,
        uint256 supply
    ) internal {
```
[217](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L217)
</details>

### [N-05] `public` functions not called by the contract should be declared `external` instead

Contracts are allowed to override their parents' functions and change the visibility from `external` to `public`.
If a `public` function is not called internally within the contract, it should be declared as `external` to save gas.

<details>
<summary><i>18 issue instances in 5 files:</i></summary>

```solidity
File: contracts/Curves.sol

154: function setReferralFeeDestination(
        address curvesTokenSubject,
        address referralFeeDestination_
    ) public onlyTokenSubject(curvesTokenSubject) {
196: function getBuyPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
203: function getSellPriceAfterFee(address curvesTokenSubject, uint256 amount) public view returns (uint256) {
210: function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
363: function buyCurvesTokenWithName(
        address curvesTokenSubject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) public payable {
376: function buyCurvesTokenForPresale(
        address curvesTokenSubject,
        uint256 amount,
        uint256 startTime,
        bytes32 merkleRoot,
        uint256 maxBuy
    ) public payable onlyTokenSubject(curvesTokenSubject) {
403: function buyCurvesTokenWhitelisted(
        address curvesTokenSubject,
        uint256 amount,
        bytes32[] memory proof
    ) public payable {
464: function withdraw(address curvesTokenSubject, uint256 amount) public {
503: function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public {
```
[154](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L154) | [196](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L196) | [203](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L203) | [210](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L210) | [363](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L363) | [376](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L376) | [403](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L403) | [464](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L464) | [503](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L503)

```solidity
File: contracts/CurvesERC20.sol

11: function mint(address to, uint256 amount) public onlyOwner {
15: function burn(address from, uint256 amount) public onlyOwner {
```
[11](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L11) | [15](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L15)

```solidity
File: contracts/CurvesERC20Factory.sol

7: function deploy(string memory name, string memory symbol, address owner) public returns (address) {
```
[7](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L7)

```solidity
File: contracts/FeeSplitter.sol

34: function setCurves(Curves curves_) public {
51: function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
88: function addFees(address token) public payable onlyManager {
95: function onBalanceChange(address token, address account) public onlyManager {
```
[34](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L34) | [51](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L51) | [88](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L88) | [95](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L95)

```solidity
File: contracts/Security.sol

22: function setManager(address manager_, bool value) public onlyOwner {
26: function transferOwnership(address owner_) public onlyOwner {
```
[22](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L22) | [26](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L26)
</details>

### [N-06] Prefer Casting to `bytes` or `bytes32` Over `abi.encodePacked()` for Single Arguments

When using `abi.encodePacked()` on a single argument, it is often clearer to use a cast to `bytes` or `bytes32`.
This improves the semantic clarity of the code, making it easier for reviewers to understand the developer's intentions.

<details>
<summary><i>9 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

424: bytes32 leaf = keccak256(abi.encodePacked(caller));
441: keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
442: keccak256(abi.encodePacked("")) ||
443: keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
444: keccak256(abi.encodePacked(""))
471: keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
472: keccak256(abi.encodePacked("")) ||
473: keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
474: keccak256(abi.encodePacked(""))
```
[424](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L424) | [441](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L441) | [442](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L442) | [443](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L443) | [444](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L444) | [471](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L471) | [472](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L472) | [473](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L473) | [474](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L474)
</details>

### [N-07] Style guide: Lines are too long

It is generally recommended that lines in the source code should not exceed 80-120 characters.
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: contracts/FeeSplitter.sol

44: //@dev: this is the amount of tokens that are not locked in the contract. The locked tokens are in the ERC20 contract
102: //@dev: this may fail if the the list is long. Get first the list with getUserTokens to estimate and prepare the batch
```
[44](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L44) | [102](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L102)
</details>

### [N-08] Avoid the use of sensitive terms

Inclusive language plays a critical role in fostering an environment where everyone belongs.
Please use alternative terms as suggested below:
master -> source
slave -> replica
blacklist -> blocklist
whitelist -> allowlist

<details>
<summary><i>4 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Curves.sol

92: event WhitelistUpdated(address indexed presale, bytes32 indexed root);
394: function setWhitelist(bytes32 merkleRoot) external {
400: emit WhitelistUpdated(msg.sender, merkleRoot);
404: function buyCurvesTokenWhitelisted(
```
[92](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L92) | [394](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L394) | [400](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L400) | [404](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L404)
</details>

### [N-09] Unused import

The contract contains import statements for libraries or other contracts that are not utilized within the code.
Excessive or unused imports can clutter the codebase, leading to inefficiency and potential confusion.
Consider removing any imports that are not essential to the contract's functionality.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/FeeSplitter.sol

/// @audit - IERC20 imported but not used
7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```
[7](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L7)
</details>

### [N-10] Explicit Visibility Recommended in Variable/Function Definitions

In Solidity, variable/function visibility is crucial for controlling access and protecting against unwanted modifications. 
While Solidity functions default to `internal` visibility, it is best practice to explicitly state the visibility for better code readability and avoiding confusion.

The missing visibility could lead to a false sense of security in contract design and potential vulnerabilities.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/FeeSplitter.sol

11: uint256 constant PRECISION = 1e18;
```
[11](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L11)
</details>


Formal verification offers a mathematical proof confirming that your code operates as intended and is devoid of edge cases 
that may lead to unintended behavior. By leveraging this rigorous audit technique, you not only enhance the robustness 
of your code but also strengthen the trust of stakeholders in the safety of your contract.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: app/2024-01-curves

1: All files
```
[1](https://github.com/code-423n4/2024-01-curves/blob/main/app/2024-01-curves#L1)
</details>