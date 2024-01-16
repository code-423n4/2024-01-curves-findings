# G-01.  OR in if-condition can be rewritten to two single if conditions #

Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

This is a simple forge test, which demonstrates gas usage for both versions:
```
     function testIfWithOR(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) public returns (uint) { 
        
        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) return 1;
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) return 1;
    }
   function testIfWithoutOr(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) public returns (uint) { 
        
        if (py_init >= MAX_PRICE_VALUE) return 1;
        if (py_final >= MAX_PRICE_VALUE) return 1;
        if (px_init <= MIN_PRICE_VALUE) return 1;
        if (px_final <= MIN_PRICE_VALUE) return 1;
    }
```

```
Gas:testIfWithOR(int128,int128,int128,int128,uint256):(uint256) (runs: 256, μ: 914, ~: 926)
Gas:testIfWithoutOr(int128,int128,int128,int128,uint256):(uint256) (runs: 256, μ: 844, ~: 855)
```
Instances :
```
File : contracts/Curves.sol

265 :         if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

409 - 413 :         if (
            presalesMeta[curvesTokenSubject].startTime == 0 ||
            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
        ) revert PresaleUnavailable();
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L265

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L409C6-L413C1


# G-02. Using a positive conditional flow to save a NOT opcode #

Estimated savings: 3 gas per instance.

Reference :  https://code4rena.com/reports/2023-07-basin#g-13-using-a-positive-conditional-flow-to-save-a-not-opcode

Instances :
```
File : contracts/Curves.sol

233 :                 if (!success1) revert CannotSendFunds();

237 :                 if (!success2) revert CannotSendFunds();

243 :                 if (!success3) revert CannotSendFunds();
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L233

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L237

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L243

# G-03.Use hardcode address instead address(this) #

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

Reference : https://code4rena.com/reports/2023-05-base#10-use-hardcode-address-instead-addressthis

Instances :
```
File : contracts/Curves.sol

297 :         if (to == address(this)) revert ContractCannotReceiveTransfer();

303 :         if (to == address(this)) revert ContractCannotReceiveTransfer();

352 :         address tokenContract = CurvesERC20Factory(curvesERC20Factory).deploy(name, symbol, address(this));

486 :         _transfer(curvesTokenSubject, msg.sender, address(this), amount);

498 :         if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

501 :         _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
```
https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L297

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L303

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L352

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L486

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L498

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L501