
# [L-0] Emitted Value in Trade Event is Incorrect

## Severity: LOW

## Summary

The `Trade` event in the Curves contract incorrectly reports the `protocolEthAmount` for sell transactions. This is due to the fact that no fees are charged in the case of a sell event, yet the `protocolEthAmount` is still emitted with a value that does not accurately reflect the transaction.

## Vulnerability Details

In the Curves contract, the `Trade` event is emitted after a buy or sell operation is completed. This event includes various parameters, such as `protocolEthAmount`, which is intended to represent the amount of ETH allocated to the protocol as a fee. However, during a sell operation, the contract does not actually charge any fees to the protocol.

Despite this, the protocolEthAmount is still reported in the event, leading to a discrepancy between the actual transaction mechanics and the information emitted in the event.

This issue can potentially cause confusion or misinterpretation of transaction logs, as users or external systems monitoring these events might expect that a fee was charged to the protocol during sell operations, which is not the case.

## Tools Used

Manual code review

## Recommendations

To address this issue, the contract should accurately report the protocolEthAmount in the Trade event. This can be achieved by setting the protocolEthAmount to zero in the event emission for sell transactions, as no protocol fee is charged in these cases.

---

# [L-1] Set ERC20 Default Value for Other User Token

## Severity: LOW

## Summary

The `withdraw` function in the Curves contract does not validate if the `msg.sender` is the `curvesTokenSubject` before calling `_deployERC20`. This allows any user to set the ERC20 token details to the default value for other users' token subjects, potentially leading to unauthorized token creation and configuration.

## Vulnerability Details

The vulnerability lies in the `withdraw` function. It does not check whether the `msg.sender` is the owner or authorized entity related to the `curvesTokenSubject` before executing `_deployERC20`. As a result, any user can trigger the `withdraw` function with another user's `curvesTokenSubject` and inadvertently or maliciously deploy and set the ERC20 token details for that subject.

This is problematic because once an ERC20 token is set, the `setNameAndSymbol` function does not allow changing the token's name and symbol anymore `externalCurvesTokens[curvesTokenSubject].token != address(0). This can result in the unauthorized and irreversible assignment of token details.

Here is the proof of concept demonstrating the issue:

```javascript
    function test_set_default_for_other_user() public {
        address userSubject = makeAddr("userSubject");
        address malicious = makeAddr("malicious");

        startHoax(malicious, 100 ether);
        curves.withdraw(address(0), 0);
        curves.withdraw(userSubject, 0);
        vm.stopPrank();
    }
```

## Tools Used

Manual code review

## Impact

The impact of this vulnerability is that it allows any user to set up ERC20 tokens for other users' subjects without their consent. This could lead to confusion, misuse, and a loss of control over token configuration for the legitimate owners of these subjects.

## Recommendations

To mitigate this vulnerability, the contract should include a validation check in the withdraw function to ensure that the msg.sender is either the owner or an authorized entity for the specified curvesTokenSubject.

```diff
    function withdraw(address curvesTokenSubject, uint256 amount) public {
        if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
        if (externalToken == address(0)) {
            if (
                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name))
                    == keccak256(abi.encodePacked(""))
                    || keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol))
                        == keccak256(abi.encodePacked(""))
            ) {
                externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
                externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
            }
            if (msg.sender != curvesTokenSubject) revert UnauthorizedCurvesTokenSubject();
            _deployERC20(
                curvesTokenSubject,
                externalCurvesTokens[curvesTokenSubject].name,
                externalCurvesTokens[curvesTokenSubject].symbol
            );
            externalToken = externalCurvesTokens[curvesTokenSubject].token;
        }
        _transfer(curvesTokenSubject, msg.sender, address(this), amount);
        CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
    }
```

This validation will prevent unauthorized users from setting or ERC20 token details for subjects they do not own or control.

---

# [L-2] No Default Values Provided for Fee Percent, Allowing Fee-Free Minting

## Severity: LOW

## Summary

The `Curves` contract does not initialize the fee percentages in its constructor, potentially allowing users (especially snipers) to mint tokens without paying any fees.

## Vulnerability Details

In the current implementation of the `Curves` contract, the fee percentages (`protocolFeePercent`, `subjectFeePercent`, `holdersFeePercent`, `referralFeePercent`) are not initialized in the constructor.

This omission means that these values default to 0 until they are explicitly set by a manager or owner. As a result, users could potentially interact with the contract and execute token minting without incurring any fees, which might not be the intended behavior of the contract.

## Tools Used

Manual code review

## Impact

If early users or snipers mint tokens before the fee percentages are set, they would be able to avoid paying fees that are meant to benefit the protocol, its developers, or other participants.

It might create an unfair advantage for early users compared to later users who have to pay the full fees.

## Recommendations

To address this vulnerability, default values for the fee percentages should be set in the contract's constructor. This ensures that from the moment the contract is deployed, fees are applied to all transactions, eliminating the window where users can mint tokens without incurring any fees. The following implementation can be used:

```javascript
constructor(address curves
ERC20Factory_, address feeRedistributor_) Ownable() {
curvesERC20Factory = curvesERC20Factory_;
feeRedistributor = FeeSplitter(payable(feeRedistributor_));
// Set default fee percentages
feesEconomics.protocolFeePercent = 50000000000000000;
feesEconomics.subjectFeePercent = 50000000000000000;
feesEconomics.holdersFeePercent = 50000000000000000;
feesEconomics.referralFeePercent = 50000000000000000;
}
```

---

# [L-3] buyCurvesTokenWithName bypass mint check and allow empty token creation

## Severity: LOW

## Summary

The `buyCurvesTokenWithName` and `setNameAndSymbol` functions does not properly validate the provided token name and symbol, allowing the creation of tokens with empty strings as their name and symbol.

## Vulnerability Details

The `buyCurvesTokenWithName` function is intended to allow users to buy Curves tokens and simultaneously set the name and symbol of the associated ERC20 token. However, the function lacks validation checks to ensure that the provided name and symbol are not empty as it is calling the `_mint` method instead of calling the `mint` one which has the valid check

```javascript
    function buyCurvesTokenWithName(
        address curvesTokenSubject,
        uint256 amount,
        string memory name,
        string memory symbol
    ) public payable {
        uint256 supply = curvesTokenSupply[curvesTokenSubject];
        if (supply != 0) revert CurveAlreadyExists();

        _buyCurvesToken(curvesTokenSubject, amount);
@>      _mint(curvesTokenSubject, name, symbol);
    }
```

As a result, it is possible to create a token with an empty name and symbol, which could lead to operational challenges and confusion among users and contract interactions.

## Tools Used

Manual code review

## Impact

The creation of tokens with empty names and symbols can cause confusion and potentially disrupt the normal operation of the contract, especially in interfaces and external systems that rely on these identifiers for token differentiation. It may also lead to user errors, where tokens could be incorrectly identified or handled due to the lack of proper naming.

## Recommendations

To mitigate this issue, the `buyCurvesTokenWithName` function should call the `mint` method instead

---

# [L-4] User can send 0 amount to buy and sell

## Severity: LOW

## Summary

The smart contract does not prevent transactions with a zero token amount in the `buyCurvesToken` and `sellCurvesToken` functions, potentially leading to unnecessary gas expenditure without any real transaction effect.

## Vulnerability Details

In the `buyCurvesToken` and `sellCurvesToken` functions, there is no check to ensure that the `amount` parameter is greater than 0. This oversight allows users to call these functions with an amount of 0, which would result in the execution of these functions without any actual buying or selling of tokens. While this does not directly impact the contract's funds or token balances, it can lead to wasted gas for the users.

## Tools Used

Manual code review

## Impact

The primary impact is on the user experience and resource wastage.

This does not pose a risk to contract funds or token balances but can be an annoyance and a point of inefficiency.

## Recommendations

Implement a simple validation check in both buyCurvesToken and sellCurvesToken functions to ensure that the amount parameter is greater than 0. This can be done by adding a condition such as require(amount > 0, "Amount must be greater than zero"); at the start of each function. This will prevent the functions from executing when the amount is zero and save users from unnecessary gas expenses.

---


# [L-5] Incorrect Error Reported When Calling setWhitelist

## Severity: LOW

## Summary

When invoking the `setWhitelist` function, the contract erroneously uses the `CurveAlreadyExists` error if the supply of the token subject is greater than one. This behavior is misleading as it conflates the existence of a curve with the setting of a whitelist.

## Vulnerability Details

The `setWhitelist` function is designed to update the whitelist merkle root for a given token subject. However, it incorrectly uses the `CurveAlreadyExists` error to indicate that the function cannot proceed if the token supply is greater than one. This is misleading because the error message suggests that the function's execution is hindered by the existence of the curve, whereas the actual intent seems to be to restrict whitelist updates to early stages of the token's lifecycle.

The code snippet demonstrates this issue:

```javascript
    function setWhitelist(bytes32 merkleRoot) external {
        uint256 supply = curvesTokenSupply[msg.sender];
@>      if (supply > 1) revert CurveAlreadyExists();

        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
            presalesMeta[msg.sender].merkleRoot = merkleRoot;
            emit WhitelistUpdated(msg.sender, merkleRoot);
        }
}
```

The condition `if (supply > 1)` implies that the whitelist can be set only when the supply is one or zero, which may not be the intended logic. The error `CurveAlreadyExists` does not accurately describe the condition being checked, leading to potential confusion.

## Tools Used

- Manual code review

## Impact

The impact of this issue is primarily on the clarity and accuracy of error reporting in the contract. It can lead to confusion for users trying to understand why their attempt to set a whitelist failed. 

## Recommendations

1. **Clarify the Intent:** Determine the intended logic for setting the whitelist. If the whitelist should only be settable when the supply is zero, adjust the condition accordingly.

2. **Update the Error Message:** Use an error message that accurately reflects the condition being checked. For example, if the intent is to prevent whitelist updates after a certain stage in the token's lifecycle, a more descriptive error such as `WhitelistUpdateNotAllowed` could be used.

3. **Code Modification:** The conditional statement should be modified to reflect the correct business logic. For instance:
```diff
- if (supply > 1) revert CurveAlreadyExists();
+ if (supply > 0) revert WhitelistUpdateNotAllowed();
```

This change ensures that the contract's behavior and the emitted error message are in alignment with the intended restrictions on setting the whitelist.


---