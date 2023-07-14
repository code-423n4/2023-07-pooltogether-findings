`Dear Pool Together team, as we have gone through each contract within the scope, we have noticed very good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing.We believe that at least 90% of the recommendations in the following report should be applied for better readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future standasrs.`

# QA Report for Pool Together contest

## [01] In the function PrizePool.setDrawManager(), anyone can frontrun it and become the drawManager
Reading the documentation of the Prize Pool contract, the following is specified: `The Prize Pool allows a 'draw manager' contract to complete the Draw and withdraw tokens from the reserve.` In the code, on [line 296](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L296), it is specified that the PrizePool.setDrawManager() function `Allows a caller to set the DrawManager if not already set.` This function is not protected in cases where a malicious attacker wants to front-run and take control of the draw manager permissions.

### PROOF OF CONCEPT
[PoolTogether docs link :](https://dev.pooltogether.com/protocol/next/design/prize-pool#incentivized-draws)
```text
The Prize Pool allows a "draw manager" contract to complete the Draw and withdraw tokens from the reserve. 
```

### Mittigation 
We recommend adding access control to the function or directly setting the drawManager in the constructor and removing this function.

## [02] The UD34x4.sol library is missing visibility on variables and functions 
In multiple instances in this library, there are many variables and functions that do not have an explicit visibility modifier. In the interest of clarity, consider including it.

### Mittigation
We recommend adding visibility and following the standards that Solidity recommends for [variables](https://docs.soliditylang.org/en/v0.8.17/contracts.html#state-variable-visibility) and [functions](https://docs.soliditylang.org/en/v0.8.17/contracts.html#function-visibility).

## [03] Use a more recent version of solidity 
It is crucial to update the project to a newer version of Solidity as the current version being used, `0.8.17`, does not include critical updates in `PRBMath V4`. Upgrade to a more recent version of Solidity, such as version `0.8.19`.

## [04] The TieredLiquidityDistributor.sol library presents function overloading
Having multiple functions with the same name in a smart contract can be dangerous or not a good practice for several reasons:

- Confusion: If there are several functions with the same name, it can be confusing for developers and users who are interacting with the smart contract. This can lead to errors and misunderstandings in the use of the contract.

- Security vulnerabilities: If multiple functions are defined with the same name, attackers can attempt to exploit this vulnerability to access or modify data or functionalities of the smart contract.

- Network overload: If there are multiple functions with the same name, there may be an impact on the efficiency and speed of the contract, as the network may be confused in trying to determine which function should be executed.

See more about this on [Solidity Docs.](https://docs.soliditylang.org/en/v0.8.19/contracts.html#function-overloading)

### PROOF OF CONCEPT
```solidity
prize-pool/src/abstract/TieredLiquidityDistributor.sol
385:  function _computeNewDistributions(
406:  function _computeNewDistributions(
```
Also in the Claimer.sol contract happens:
```solidity
claimer/src/Claimer.sol
89:  function computeTotalFees(
103:  function computeTotalFees(
```
## [05] In the function PrizePool.reserveForOpenDraw(), there may be a potential issue with the correct computation of the amount of prize tokens that will be added to the reserve.
The amount of prize tokens that will be added to the reserve may not be computed correctly since the function is casting values that it shouldn't and where it should, it's not happening. When we asked the sponsor about our concerns regarding the inputs via DM, they acknowledged that we were right.

### PROOF OF CONCEPT
```solidity
606: function reserveForOpenDraw() external view returns (uint256) {
        uint8 _numTiers = numberOfTiers;
        uint8 _nextNumberOfTiers = _numTiers;

        if (lastClosedDrawId != 0) {
          _nextNumberOfTiers = _computeNextNumberOfTiers(_numTiers);
        }

        (, uint104 newReserve, ) = _computeNewDistributions(
          _numTiers,
          _nextNumberOfTiers,
          // @audit-info ask the devs why they cast to uint96 when the parameter is 256
          uint96(_contributionsForDraw(lastClosedDrawId + 1))
        );
        // @audit-info Ask why it doesn't cast from 104 to 256
        return newReserve;
      }
---------------------------------------------------------------------------------------------------------------------
  
  // inherected function to compute 
  
  function _computeNewDistributions(
    uint8 _numberOfTiers,
    uint8 _nextNumberOfTiers,
    uint256 _prizeTokenLiquidity
  ) internal view returns (uint16 closedDrawId, uint104 newReserve, UD60x18 newPrizeTokenPerShare) {
    return
      _computeNewDistributions(
        _numberOfTiers,
        _nextNumberOfTiers,
        fromUD34x4toUD60x18(prizeTokenPerShare),
        _prizeTokenLiquidity
      );
  }
```
Brendan on DM:
```text
Brendan 🌊🏆 — 12/07/2023 10:32
There are two _computeNewDistributions implementations in TieredLiquidityDistributor
The cast to uint96 doesn't make sense to me 
The return casting prob isn't needed either
```

### Recommendation
Improve the documentation of this function and clarify if the implemented changes align with the team's intentions. This will ensure that future auditors or even during code maintenance, there is a clear understanding of the purpose behind the chosen implementation.

## [06] In the contract TieredLiquidityDistributor._computeNewDistributions Use uint256 instead uint
Some developers prefer to use uint256 because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs. insecure and more error-prone.

### PROOF OF CONCEPT
```solidity
// @audit Use uint256 instead of uint.
function _computeNewDistributions(
   ..........................................

    uint reclaimed = _getTierLiquidityToReclaim( 
      _numberOfTiers,
      _nextNumberOfTiers,
      _currentPrizeTokenPerShare
    );
    uint computedLiquidity = fromUD60x18(deltaPrizeTokensPerShare.mul(toUD60x18(totalShares)));
    uint remainder = (_prizeTokenLiquidity - computedLiquidity);

    .......................................
  }
```
### Recommendation
It's best to use uint256. It brings about readability and consistency in your code, and it allows you to adhere to best practices in smart contracts.

## [07] The Vault.setClaimer() function does not verify an important input
The setClaimer function should have a check that verifies the claimer_ address is not equal to address(0).

### PROOF OF CONCEPT
```solidity
function setClaimer(address claimer_) external onlyOwner returns (address) {
    // @audit check the claimer_ address != address(0)
    address _previousClaimer = _claimer;
    _setClaimer(claimer_);

    emit ClaimerSet(_previousClaimer, claimer_);
    return claimer_;
  }

--------------------------------------------------

// inherected function
function _setClaimer(address claimer_) internal {
    _claimer = claimer_;
}
```
This lack of checks is also present in other functions, such as the Vault.setYieldFeeRecipient() function. Another function that should also have this check is Vault._transfer() for the inputs _from and _to, as specified in the function documentation in [lines 1150-1151](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1150-L1151)


## [08] In the TieredLiquidityDistributor Variable names should be in uppercase.
We recommend maintaining consistency in the code and naming immutable variables in uppercase. However, in the `TieredLiquidityDistributor` contract, some variables follow this convention while others do not. It is recommended to be consistent with your own practices.

### PROOF OF CONCEPT
```solidity
209: uint8 public immutable tierShares;
212: uint8 public immutable canaryShares;
215: uint8 public immutable reserveShares;
```
### Recommendation
It is important to adhere to good Solidity coding practices to maintain code clarity and project maintainability in both the short and long term. Consistency in following these practices ensures that the code remains readable and understandable. This promotes better collaboration among developers and reduces the chances of introducing errors or confusion in the codebase.

## [09] Use a single file for all system-wide constants
In all LPS implementations, there are many "constants" used in the system. It is recommended to store all constants in a single file (for example, Constants.sol) and use inheritance to access these values.

This helps improve readability and facilitates future maintenance. It also helps with any issues, as some of these hard-coded contracts are administrative contracts.

Constants.sol is used and imported in contracts that require access to these values.
This is just a suggestion, as the project currently has more than 11 files that store constants.

We have created an example of how the Constants file should look like:
- [Constants Template](https://gist.github.com/catellaTech/23f88d1cdcb90c8db1fbc01e3c65babc)


## [10] We suggest to use named parameters for mapping type
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.

```solidity
mapping(address account => uint256 balance); 
```

## [11] We suggest using the OpenZeppelin SafeCast library
We have noticed that the contracts in the scope implements many type conversions. We recommend using the OpenZeppelin SafeCast library to make the project more robust and take advantage of the gas optimizations and best practices provided by OpenZeppelin.


