## Summary
|        | Issue | Instances | Gas Saved |
|--------|-------|-----------|-----------|
| [G-01]|     Variables inside  `struct`  should be optimised   |1|8490|
| [G-02] |  Use `calldata` instead of `memory`     |     1      |      524     |
|[G-03]|Use named return|15|75-195|
|[G-04]|Use `switch` from `assembly`  for direct number constants `if-else`|1|98|
|[G-05]| Use  `assembly`  to write  `address` storage values|1|45|
|[G-06]| Functions guaranteed to revert when called by normal users can be marked `payable`|2|42|
|[G-07| Ternary over  `if ... else`|3|39|

## Gas Optimizations
### [G-01] Variables inside  `struct`  should be optimised
Packing struct allow for multiple variables to be stored in the same slot, as the Solidity EVM works with 32-byte units. This results in save for all related operations.

*1 instances*
1 slot can be saved
 -   [DrawAccumulatorLib.sol#L25-L30](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L25-L30)
``` diff
struct Observation {
  // track the total amount available as of this Observation
  uint96 available;
  // track the total accumulated previously
- uint168 disbursed;
+ uint160 distursed;
}
```
*96+160=256 -> 32 bytes*
*Modifications obviously followed by the change line 92-93*
*The size of the variable is reduced but this does not add any problem given the orders of magnitude.*

With this change, these evolutions in gas average report can be observed:

    DrawAccumulatorLibWrapper: add: 88086 -> 79729 (-8357)
    DrawAccumulatorLibWrapper: getDisbursedBetween: 15114 -> 15029 (-85)
	DrawAccumulatorLibWrapper: getTotalRemaining: 2541 -> 2511 (-30)

### [G-02] Use `calldata` instead of `memory`
Using `calldata` instead of `memory` for function parameters can save gas if the argument is only read in the function

*1 instances*

-   [VaultFactory.sol#L57-L58](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L57-L58)

With this change, this evolution in gas average report can be observed:

    VaultFactory: deployVault: 3737309 -> 3736785 (-524)


### [G-03] Use named return
Using the named return whenever possible saves gas by avoiding a return and the double declaration of a variable.

*15 instances*
*Save up to **5-13 gas per call*** (depending on the complexity of the function)

-   [TieredLiquidityDistributor.sol#L471-L486](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L471-L486)
-   [TieredLiquidityDistributor.sol#L557-L582](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L557-L582)
-   [TierCalculationLib.sol#L39-L43](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L39-L43)
-   [TierCalculationLib.sol#L133-L146](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L133-L146)
-   [PrizePool.sol#L311-L330](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L311-L330)
-   [PrizePool.sol#L760-L776](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L760-L776)
-   [TwabController.sol#L590-L603](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L590-L603)
-   [Vault.sol#L407-L415](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L407-L415)
-   [Vault.sol#L440-L446](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L440-L446)
-   [Vault.sol#L458-L472](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L458-L472)
-  [Vault.sol#L509-L521](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L509-L521)
-  [Vault.sol#L524-L535](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L524-L535)
-  [Vault.sol#L607-L632](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L607-L632)
-  [Vault.sol#L982-L994](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L982-L994)
-  [Vault.sol#L1043-L1056](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1043-L1056)

For instance, the code block below may be refactored as follows:
-   [TierCalculationLib.sol#L39-L43](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L39-L43)
 ``` diff
- function prizeCount(uint8 _tier) internal pure returns (uint256) {
+ function prizeCount(uint8 _tier) internal pure returns (uint256 _numberOfPrizes) {
-    uint256 _numberOfPrizes = 4 ** _tier;
+    _numberOfPrizes = 4 ** _tier;

 -    return _numberOfPrizes;
  }
```
*The NatSpec format will obviously also have to be redesigned.*

### [G-04] Use `switch` from `assembly`  for direct number constants `if-else`
When there's a large series of `if else` assigning constant numbers, it's interesting to use the `assembly` switch. However, this may affect the readability of the code. Two alternatives are therefore proposed.

1 instance
-   [TieredLiquidityDistributor.sol#L730-L759](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L730-L759)


**Small optimisation (-23 gas each call)**

    function _estimatedPrizeCount(uint8 numTiers)  internal  pure  returns  (uint32)  {
	    uint32 estimatedPrizes;
	    
	    assembly {
		    switch numTiers
		    case 3  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_2_TIERS
		    }
		    case 4  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_3_TIERS
		    }
		    case 5  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_4_TIERS
		    }
		    case 6  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_5_TIERS
		    }
		    case 7  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_6_TIERS
		    }
		    case 8  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_7_TIERS
		    }
		    case 9  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_8_TIERS
		    }
		    case 10  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_9_TIERS
		    }
		    case 11  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_10_TIERS
		    }
		    case 12  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_11_TIERS
		    }
		    case 13  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_12_TIERS
		    }
		    case 14  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_13_TIERS
		    }
		    case 15  {
			    estimatedPrizes := ESTIMATED_PRIZES_PER_DRAW_FOR_14_TIERS
		    }
		    default {
			    estimatedPrizes := 0
		    }
	    }
	    return estimatedPrizes;
    }

**Great optimisation (-98 gas each call)**

    function _estimatedPrizeCount(uint8 numTiers)  internal  pure  returns  (uint32)  {
	    assembly {
		    let slot := mload(0x40)  // Get a pointer to some free memory.
		    mstore(0x40, add(slot,  0x20))  //Since the allocated memory will be return, the free memory pointer must be incremented
		    
		    switch numTiers
		    case 3  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_2_TIERS)
			    return(slot,  32)
		    }
		    case 4  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_3_TIERS)
			    return(slot,  32)
		    }
		    case 5  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_4_TIERS)
			    return(slot,  32)
		    }
		    case 6  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_5_TIERS)
			    return(slot,  32)
		    }
		    case 7  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_6_TIERS)
			    return(slot,  32)
		    }
		    case 8  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_7_TIERS)
			    return(slot,  32)
		    }
		    case 9  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_8_TIERS)
			    return(slot,  32)
		    }
		    case 10  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_9_TIERS)
			    return(slot,  32)
		    }
		    case 11  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_10_TIERS)
			    return(slot,  32)
		    }
		    case 12  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_11_TIERS)
			    return(slot,  32)
		    }
		    case 13  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_12_TIERS)
			    return(slot,  32)
		    }
		    case 14  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_13_TIERS)
			    return(slot,  32)
		    }
		    case 15  {
			    mstore(slot, ESTIMATED_PRIZES_PER_DRAW_FOR_14_TIERS)
			    return(slot,  32)
		    }
		    default {
			    mstore(slot,  0)
			    return(slot,  32)
		    }
		}
    }
With this last change, this evolution in gas average report can be observed:

    TieredLiquidityDistributorWrapper: estimatedPrizeCount: 740 -> 642 (-98)

### [G-05] Use  `assembly`  to write  `address` storage values
This change has little effect on code reading.

*1 instance*
*Save up to **45 gas per call***

-   [PrizePool.sol#L303](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L303)

 ``` diff
- drawManager = _drawManager;
+ assembly {sstore(drawManager.slot, _drawManager)}
```
	
### [G-06] Functions guaranteed to revert when called by normal users can be marked `payable`
Marking a function as payable can lower gas costs for legitimate callers by avoiding opcodes such as `CALLVALUE`(2), `DUP1`(3), `ISZERO`(3), `PUSH2`(3), `JUMPI`(10), `PUSH1`(3), `DUP1`(3), `REVERT`(0), `JUMPDEST`(1), `POP`(2).

*2 instances*
*Save up to **21 gas per call***

 - [PrizePool.sol#L335](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L335)
 -   [PrizePool.sol#L348](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L348)
### [G-07] Ternary over  `if ... else`
Replacing an `if-else` statement with the ternary operator can save gas.

For instance, the code block below may be refactored as follows:
-   [Claimer.sol#L171-L176](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L171-L176)
``` diff
- if (_tier != _canaryTier) {
- return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1));
- } else {
- return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
+ return  (_tier != _canaryTier)
+ ? _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier -  1))
+ : _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
```
*3 instances*
*Save up to **13 gas per call***
-   [Claimer.sol#L171-L176](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L171-L176) (*return*)
-   [TwabLib.sol#L498-L507](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L498-L507) (*return*)
-   [Vault.sol#L1052-L1056](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1052-L1056) (*variable set*)