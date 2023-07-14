| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#1-state-variables-only-set-in-the-constructor-should-be-declared-immutable) |
| 2 | [state variables can be packed into fewer storage slots](#2-state-variables-can-be-packed-into-fewer-storage-slots) |
| 3 | [Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate](#3-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate) |
| 4 | [state variables should be cached in stack variables rather than re-reading them from storage](#4-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-found-in-c4-audit) |
| 5 | [can make the variable outside the loop to save gas](#5-can-make-the-variable-outside-the-loop-to-save-gas) |
| 6 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#6-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 7 | [require() or revert() statements that check input arguments should be at the top of the function](#7-require-or-revert-statements-that-are-cheaper-should-be-at-the-top-of-the-function) |
| 8 | [ Ternary over if ... else](#8-ternary-over-if--else) |
| 9 | [Functions guaranteed to revert when called by normal users can be marked `payable`](#9-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) |
| 10 | [Optimize names to save gas](#10-optimize-names-to-save-gas) |
| 11 | [Use hardcoded address instead address(this)](#11-use-hardcoded-address-instead-addressthis) |


## 1. State variables only set in the constructor should be declared immutable.

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas). While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

- [Vault.sol#L279](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L279)


## 2. state variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

`drawManager`#L222 which is 20 bytes can be put together with `claimCount`#L240 and `canaryClaimCount`#L243 and `largestTierClaimed`#L246 which will be less than 32 bytes so they will only use one slot instead of current 2. also its used in the same function with all 3 once which will reduce gas cost as well
- [PrizePool.sol#L222](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L222)


## 3. Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

all 3 mappings can be a mapping to a struct and also in that struct mapping of #L36 and #L42 also can be a mapping to another struct 
- [TwabController.sol#L36-L42](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L36-L42)


## 4. state variables should be cached in stack variables rather than re-reading them from storage (ones not found in c4 audit)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`prizeTokenPerShare` should be cached before #L337 because its read from storage in #L364 as well and the second use is inside a for loop as well so caching this state var will save lots of gas
- [TieredLiquidityDistributor.sol#L364](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L364)

`prizeTokenPerShare` can be cached before #L529 so we can use the cached version there and in #L540
- [TieredLiquidityDistributor.sol#L540](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L540)

if we change the #L538 from using -= to using just subtraction we can save gas if we had `_reserve` cached. so we can cache `_reserve` before #L535 and save gas
- [TieredLiquidityDistributor.sol#L538](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L538)

same logic as last one for `_reserve` but here even if the first condition reverts we save 100 gas. cache before the if statement #L336
- [PrizePool.sol#L336](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L336)

same as above for `claimerRewards[msg.sender]`
- [PrizePool.sol#L490](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L490)

same logic as above for `_yieldFeeTotalSupply` 
- [Vault.sol#L398](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L398)

`lastClosedDrawId` should be cached before #611 and use the cached version for that line and #L618 to save 100 gas
- [PrizePool.sol#L618](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L618)

if the condition be true we will have a extra SLOAD for `_liquidationPair` so we can cache it before and risk losing only 3 gas but save 100
- [Vault.sol#L559](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L559)

if the condition be true we will have a extra SLOAD for `_claimer` so we can cache it before and risk losing only 3 gas but save 100
- [Vault.sol#L614](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L614)

cache answer to `_currentExchangeRate()` and use it in the #L1111 emit
- [Vault.sol#L1111](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1111)


## 5. can make the variable outside the loop to save gas

make the variable outside the loop and only give the value to variable inside

- [TieredLiquidityDistributor.sol#L631](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L631)
- [TieredLiquidityDistributor.sol#L632](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L632)
- [TieredLiquidityDistributor.sol#L633](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L633)

- [Vault.sol#L619](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L619)
- [Vault.sol#L620](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L620)


## 6. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [Claimer.sol#L68](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68)
- [Claimer.sol#L144](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144)

- [TierCalculationLib.sol#L137](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L137)
- [TierCalculationLib.sol#L138](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L138)

- [Vault.sol#L618](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L618)
- [Vault.sol#L620](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L620)


## 7. require() or revert() statements that are cheaper should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

the first if statement for revert is the most expensive to run so we can do it last
- [Vault.sol#L558-L564](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L558-L564)


## 8. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [Claimer.sol#L171-L175](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L171-L175)

- [DrawAccumulatorLib.sol#L459-L463](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L)

- [Vault.sol#L1052-L1055](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1052-L1055)

- [ObservationLib.sol#L93-L97](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol#L93-L97)


## 9. Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

- [PrizePool.sol#L335](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L335)
- [PrizePool.sol#L348](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L348)


## 10. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signitures 


## 11. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [PrizePool.sol#L312](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L312)
- [PrizePool.sol#L500](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L500)

- [Vault.sol#L336](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L336)
- [Vault.sol#L435](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L435)
- [Vault.sol#L468](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L468)
- [Vault.sol#L502](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L502)
- [Vault.sol#L562](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L562)
- [Vault.sol#L563](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L563)
- [Vault.sol#L570](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L570)
- [Vault.sol#L578](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L578)
- [Vault.sol#L581](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L581)
- [Vault.sol#L801](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L801)
- [Vault.sol#L809](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L809)
- [Vault.sol#L830](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L830)
- [Vault.sol#L932](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L932)
- [Vault.sol#L954](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L954)
- [Vault.sol#L959](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L959)
- [Vault.sol#L986](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L986)
- [Vault.sol#L1026](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1026)
- [Vault.sol#L1176](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1176)

- [VaultFactory.sol#L83](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L83)
