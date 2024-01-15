## Curves Overview

The Curves protocol, an extension of friend.tech, enhances functionality with features like token export to ERC20 for interoperability, a referral fee system for collaborative incentives, and a presale feature for controlled token launches. The protocol also introduces a token holder fee distribution model to encourage long-term investment. These innovations contribute to a more robust and inclusive financial ecosystem within Curves.

## Findings Summary

| Risk   | Title                                          |
| ------ | -----------------------------------------------|
| [L-01] | Buying will be paused for 1 second             |
| [L-02] | Unbounded loop in _addOwnedCurvesTokenSubject  |

## [L-01] Buying will be paused for 1 second 

## Impact

All tokens that use the presale will be DoSed for 1 second. Although the duration may seem short, every second is crucial in a trading protocol. Some users employ scripts and bots to secure early purchases during public sales, which might not be feasible in this scenario.
## Description

Presale is one of the special cases of BuyToken. If presale is chosen, a merkle root and a start period must be indicated. The presale begins at the same moment the buy token is called and ends when the open sale starts. The open sale begins at the indicated start period.

Whitelisted users utilizing buyCurvesTokenWhitelisted() encounter this check for a successful purchase:

```solidity
    if (presalesMeta[curvesTokenSubject].startTime == 0 || presalesMeta[curvesTokenSubject].startTime <= block.timestamp )
        revert PresaleUnavailable();
```

The transaction will revert if startTime <= block.timestamp, indicating that the public sale has started. However, for public users initiating purchases:

```solidity
if (startTime != 0 && startTime >= block.timestamp) revert SaleNotOpen();
```

The call will throw an error, stating that the public sale is not yet open due to an incorrect operator.

#### Here is a coded PoC to demonstrate the issue:

```solidity
    function testBuyPresaleInvalidOperator() public {
        bytes32 leaf1 = keccak256(abi.encodePacked(bob));
        bytes32 leaf2 = keccak256(abi.encodePacked(jim));
        bytes32 leaf3 = keccak256(abi.encodePacked(carl));
        bytes32 leaf4 = keccak256(abi.encodePacked(joey));

        bytes32 hash1 = keccak256(abi.encodePacked(leaf1, leaf2));
        bytes32 hash2 = keccak256(abi.encodePacked(leaf3, leaf4));

        bytes32 merkleRoot = keccak256(abi.encodePacked(hash1, hash2));

        bytes32[] memory proof = new bytes32[](2);
        proof[0] = leaf3;
        proof[1] = hash1;

        vm.prank(alice);
        curves.buyCurvesTokenForPresale({
            curvesTokenSubject: alice,
            amount: 1,
            startTime: (block.timestamp + 1 days),
            merkleRoot: merkleRoot,
            maxBuy: 100
        });

        uint256 amount = curves.getBuyPriceAfterFee(alice, 5);
        vm.deal(joey, amount);

        vm.prank(joey);
        curves.buyCurvesTokenWhitelisted{value: amount}({
            curvesTokenSubject: alice,
            amount: 5,
            proof: proof
        });

        amount = curves.getBuyPriceAfterFee(alice, 10);

        vm.deal(jim, amount);

        vm.expectRevert(CurvesErrors.SaleNotOpen.selector);
        vm.prank(jim);
        curves.buyCurvesToken{value: amount}(alice, 10);

        vm.warp(block.timestamp + 1 days);

        vm.expectRevert(CurvesErrors.SaleNotOpen.selector);
        vm.prank(jim);
        curves.buyCurvesToken{value: amount}(alice, 10);

        vm.deal(joey, amount);

        vm.expectRevert(CurvesErrors.PresaleUnavailable.selector);
        vm.prank(joey);
        curves.buyCurvesTokenWhitelisted{value: amount}({
            curvesTokenSubject: alice,
            amount: 10,
            proof: proof
        });
    }
```

## Recommended Mitigation Steps

Consider updating the check in `buyCurvesToken()` to:

```solidity
if (startTime != 0 && startTime > block.timestamp) revert SaleNotOpen();
```

## [L-**] {Title}

## Impact

## Description

Detailed description of this finding.

## Recommended Mitigation Steps

## [L-02] Unbounded loop in _addOwnedCurvesTokenSubject

## Impact

Holder's may be unable to withdraw/deposit their tokens due to an unbounded loop in _addOwnedCurvesTokenSubject.

## Description

There are no bounds on the number of tokens subjects that can pushed to ownedCurvesTokenSubjects array:

```solidity
    function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
        address[] storage subjects = ownedCurvesTokenSubjects[owner_];
        for (uint256 i = 0; i < subjects.length; i++) {
            if (subjects[i] == curvesTokenSubject) {
                return;
            }
        }
        subjects.push(curvesTokenSubject);
    }
```

## Recommended Mitigation Steps

Consider defining and documenting a safe maximum value for the upper bound on every loop, to guarantee the correct functionality of the contracts.
