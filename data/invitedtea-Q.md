### 1. Concern: Lack of Input Validation
**Severity**: Low

**Description**:  
There's no validation of the `curves_` address in `setCurves`, which can be set to a zero address or an invalid contract.

**Line Number**: In `setCurves`.
*GitHub* : [35](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L35-L37)

```solidity
    function setCurves(Curves curves_) public {
        curves = curves_;
    }
```

**Recommended Fix**:  
Add a requirement to check that `curves_` is not the zero address.
```solidity
function setCurves(Curves curves_) public {
    require(address(curves_) != address(0), "curves_ cannot be the zero address");
    curves = curves_;
}
```


### 2. Input Validation
**Severity**: Low

**Description**:  
The `deploy` function does not validate the input parameters (e.g., ensuring `name` and `symbol` are not empty strings, `owner` is not the zero address).

**Line Number**: In the `deploy` function.
*GitHub* : [7](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L7-L10)

**Recommended Fix**:  
Add input validation checks to ensure that the provided parameters are valid. For example, you could require that `name` and `symbol` are not empty and `owner` is not the zero address.

```solidity
contract CurvesERC20Factory {
    function deploy(string memory name, string memory symbol, address owner) public returns (address) {
        require(bytes(name).length > 0, "Name cannot be empty");
        require(bytes(symbol).length > 0, "Symbol cannot be empty");
        require(owner != address(0), "Owner cannot be the zero address");

        CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);
        return address(tokenContract);
    }
}
```


### 3. Event Emission for Contract Deployment
**Severity**: Low

**Description**:  
The contract does not emit an event when a new `CurvesERC20` contract is deployed. Emitting an event could be useful for off-chain tracking and verification purposes.

**Line Number**: In the `deploy` function. [6](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/CurvesERC20Factory.sol#L6)

**Recommended Fix**:  
Define and emit an event after each successful deployment. The event could include details like the address of the new contract, its name, symbol, and the address of the owner.

```solidity
contract CurvesERC20Factory {
    // Define the event
    event CurvesERC20Deployed(address indexed contractAddress, string name, string symbol, address indexed owner);

    function deploy(string memory name, string memory symbol, address owner) public returns (address) {
        require(bytes(name).length > 0, "Name cannot be empty");
        require(bytes(symbol).length > 0, "Symbol cannot be empty");
        require(owner != address(0), "Owner cannot be the zero address");

        CurvesERC20 tokenContract = new CurvesERC20(name, symbol, owner);

        // Emit the event after successful deployment
        emit CurvesERC20Deployed(address(tokenContract), name, symbol, owner);

        return address(tokenContract);
    }
}
```