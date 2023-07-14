## [G-1 ]  The result of a function call should be cached rather than re-calling the function
```
  function computeTotalFees(uint8 _tier, uint _claimCount) external view returns (uint256) {
    return
      _computeFeePerClaim(
        _computeMaxFee(_tier, prizePool.numberOfTiers()),
        _claimCount,
        prizePool.claimCount()
      ) * _claimCount;
  }
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L89C1-L96C4

```
 function computeTotalFees(
    uint8 _tier,
    uint _claimCount,
    uint _claimedCount
  ) external view returns (uint256) {
    return
      _computeFeePerClaim(
        _computeMaxFee(_tier, prizePool.numberOfTiers()),
        _claimCount,
        _claimedCount
      ) * _claimCount;
  }
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L103C2-L114C4

155:  `return fee / _claimCount;`
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L155

## [G-2] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
	//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

```
 if (
      observationDrawIdBeforeOrAtStart > 0 &&
      firstObservationDrawIdOccurringAtOrAfterStart > 0 &&
      observationDrawIdBeforeOrAtStart != lastObservationDrawIdOccurringAtOrBeforeEnd
    ) {
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L269C4-L273C8

## [G-03] If statements that use && can be refactored into nested if statements
```
 if (
      firstObservationDrawIdOccurringAtOrAfterStart > 0 &&
      firstObservationDrawIdOccurringAtOrAfterStart < lastObservationDrawIdOccurringAtOrBeforeEnd
    ) {
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L286C4-L289C8

```
if (targetAtOrAfter && _targetLastClosedDrawId <= afterOrAtDrawId) {
        break;
      }
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L454C7-L456C8

```
if (_availableYield != 0 && _yieldFeePercentage != 0) {
      return _availableYieldFeeBalance(_availableYield);
    }
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L325C5-L327C6

## [G-04] Use calldata instead of memory for function arguments that do not get mutated
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L367

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L55C1-L66C33

## [G-5] `variable == false ` instead of   `!variable`.
==a bit cheapier when you replace:==
459: `if (!targetAtOrAfter) {`
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L459C7-L460C1
## [G-6] Rearranging order of state variable declarations to pack them will save storage slots and gas

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

## [G-07] The result of a function call should be cached rather than re-calling the function
```
    return
      _observation.cumulativeBalance +
      uint128(_observation.balance) *
      (_timestamp.checkedSub(_observation.timestamp, _timestamp));
  }
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L404C1-L408C4

## [G-08] State Variable should be cached instead of useing in `emit `.
```
 emit NewVault(
      asset_,
      name_,
      symbol_,
      twabController_,
      yieldVault_,
      prizePool_,
      claimer_,
      yieldFeeRecipient_,
      yieldFeePercentage_,
      owner_
    );
  }
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L284C4-L296C4