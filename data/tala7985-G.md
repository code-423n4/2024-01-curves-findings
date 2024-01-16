## \[G-1\] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```
event TokenDeployed(address indexed curvesTokenSubject, address indexed erc20token, string name, string symbol);
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L93

```
event FeesClaimed(address indexed token, address indexed user, uint256 amount);
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L31

## \[G-2\] Use of Named Returns for Local Variables Saves Gas

You can have further advantages in term of gas cost by simply using named return values as temporary local variable. As an example, the following instance of code block can refactored as follows:

```
public
        view
        returns (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, uint256 totalFee)
    {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L169C7-L172C6

&nbsp;

## \[G‑3\] Save gas by preventing zero amount in mint() and burn()

```
_mint(to, amount);
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L13

```
_burn(from, amount);
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20.sol#L17

&nbsp;

##  \[G-4\] Use shift right/left instead of division/multiplication if possible 

A division/multiplication by any number `x` being a power of 2 can be calculated by shifting `log2(x)` to the right/left. While the `DIV` opcode uses 5 gas, the `SHR` opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

```
: ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
```

&nbsp;https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L184

```
uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L181

&nbsp;

