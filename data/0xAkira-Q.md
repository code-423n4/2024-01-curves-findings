# 1. Title

Code style

## Summary

`FeeSplitter.sol` does not conform to Style Guide Solidity

## Vulnerability Details

`FeeSplitter.sol` does not respect the Order of Functions

## Impact

Ordering helps readers identify which functions they can call and to find the `constructor` and `fallback` definitions easier.

## Tools Used

[Solidity docs](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-functions)

## Recommendations

Functions should be grouped according to their visibility and ordered:

- constructor

- receive function (if exists)

- fallback function (if exists)

- external

- public

- internal

- private

Within a grouping, place the `view` and `pure` functions last.

# 2. Title

Floating pragma

## Description

`FeeSplitter` and `Security` contracts use floating pragma statement pragma solidity ^0.8.7;
This means that the contract can be compiled with any compiler version from 0.8.7 (inclusive) up to, but not including, version 0.9.0. This could potentially introduce unexpected behavior if the contract is compiled with a newer compiler version that includes breaking changes.

## Recommendations

It is generally recommended to lock the pragma statement to a specific compiler version to ensure that the contract behaves as expected. This can be done by removing the caret (^) from the pragma statement.

Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

## Tools

https://swcregistry.io/docs/SWC-103/
