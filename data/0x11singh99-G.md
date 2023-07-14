### Gas Optimizations List

| Number | Optimization Details                                                                                                     | Instances |
| :----: | :----------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | State variables should be cached in stack variables rather than re-reading them from storage variables                   |     3     |
| [G-02] | Use `delete` for state variables to reinitialize with their default value rather than assigning default value saves gas. |     3     |
| [G-03] | Write for loops in more gas efficient way                                                                                |     2     |
| [G-04] | Using ternary operator instead of if else saves gas                                                                      |     1     |
| [G-05] | Missing `zero-address` check in `constructor` | 3 |
| [G-06] | Use assembly for address(0) comparison | 1 |
| [G-07] | Function calls can be cached rather than re calling from same function with same value will save gas | 2 |
| [G-08] | <x> += <y> / <x> -= <y> costs more gas than <x> = <x> + <y> / <x> = <x> - <y> for state variables| 1 |
| [G-09] | Use unchecked{} whenever underflow/overflow not possible saves 130 gas each time | 2 |
| [G-10] | Do not calculate constant variables | 117 |
| [G-11] | Use nested if and, avoid multiple check combinations | 2 |

Total 11 issues

## [G‑01] State variables used in a function should be cached in stack variables rather than re-reading them from storage.

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read

Note: view functions also added since they are used in state changing functions

_3 instances - 1 file:_

### Instance#1: `drawManager` can be cached(saves 100 Gas)

```solidity
File : src/PrizePool.sol
287:   modifier onlyDrawManager() {
288:    if (msg.sender != drawManager) {
289:      revert CallerNotDrawManager(msg.sender, drawManager);
290:    }
291:    _;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L287C3-L291C7

### Instance#2: `lastClosedDrawId + 1` can be cached rather than re-reading and recalculating 3 times(saves 600+ Gas) for Gwarmaccess(100 each) and adding 1 with overflow check 3 times.

```solidity
File : src/PrizePool.sol

316:  DrawAccumulatorLib.add(
         vaultAccumulator[_prizeVault],
          _amount,
319:      lastClosedDrawId + 1,
         smoothing.intoSD59x18()
    );
    DrawAccumulatorLib.add(
      totalAccumulator,
      _amount,
325:      lastClosedDrawId + 1,
      smoothing.intoSD59x18()
    );
328:    emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L316C5-L328C76

### Instance#3: `_reserve` can be cached rather than reading 3 times.(Saves 200 gas )

```solidity
File : src/PrizePool.sol
336: if (_amount > _reserve) {
337:      revert InsufficientReserve(_amount, _reserve);
    }
339:    _reserve -= _amount;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L336C4-L339C25

## [G-02] Use `delete` for state variables to reinitialize with their default value rather than assigning default value saves gas.

This gas-saving behavior is due to how the delete operation is implemented in the Ethereum Virtual Machine (EVM).

When you use the delete keyword on a state variable, the EVM internally performs a low-level operation called SSTORE to update the variable's storage slot. The SSTORE operation clears the storage slot, effectively resetting the variable to its default value.

In contrast, if you manually assign a default value to a state variable, it involves writing a value to the storage slot. This operation is performed using the SSTORE opcode as well but with an extra cost compared to the delete operation. Writing the default value to the storage slot consumes more gas than simply clearing the storage slot with the delete operation.

_3 instances - 1 file:_

### Instance#1-3:

```solidity
File : src/PrizePool.sol

369:    claimCount = 0;
370:    canaryClaimCount = 0;
371:    largestTierClaimed = 0;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L369C5-L371C28

Recommended Changes:

```diff
- 369:    claimCount = 0;
- 370:    canaryClaimCount = 0;
- 371:    largestTierClaimed = 0;
+        delete claimCount;
+        delete canaryClaimCount;
+        delete largestTierClaimed;
```

## [G-03] Write for loops in more gas efficient way

Loop can be written in more gas efficient way, by

1. taking length in stack variable,
2. using unchecked{++i} with pre increment
3. not re-initializing to 0 since it is default

_2 instance - 1 file:_

### Instance#1: This for loop code can be modified to save gas

```solidity
File : src/Claimer.sol

68:  for (uint i = 0; i < winners.length; i++) {
69:      claimCount += prizeIndices[i].length;
70:    }
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68C3-L70C6

Recommended Changes: Convert above for loop to this for making gas efficient saves more than 130 Gas per iteration

```diff
- 68:     for (uint i = 0; i < winners.length; i++) {
  69:      claimCount += prizeIndices[i].length;
  70:    }

+      uint256 _size= winners.length;
+      for (uint256 i; i < _size ; ) {
         claimCount += prizeIndices[i].length;
+            unchecked {
+                ++i;
+            }
        }

```

### Instance#2: This for loop code can be modified to save gas

```solidity
File : src/Claimer.sol

144:  for (uint i = 0; i < _claimCount; i++) {
145:      fee += _computeFeeForNextClaim(
146:        minimumFee,
147:        decayConstant,
148:        perTimeUnit,
149:        elapsed,
150:        _claimedCount + i,
151:        _maxFee
152:      );
153:    }

```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144C5-L153C6

Recommended Changes: Convert this for loop like above instance#1 recommendation for making gas efficient saves more than 130 Gas per iteration.

## [G-04] Using ternary operator instead of if else saves gas

When using the if-else construct, Solidity generates bytecode that includes a JUMP operation. The JUMP operation requires the EVM to change the program counter to a different location in the bytecode, resulting in additional gas costs. On the other hand, the ternary operator doesn't involve a JUMP operation, as it generates bytecode that utilizes conditional stack manipulation.
The exact amount of gas saved by using the ternary operator instead of the if-else construct in Solidity can vary depending on the specific scenario, the complexity of the surrounding code, and the conditions involved

_1 instance - 1 file:_

### Instance#1 :

```solidity
File: src/Claimer.sol

171:  if (_tier != _canaryTier) {
      // canary tier
173:      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1));
    } else {
175:      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
    }
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L171C4-L176C6

Recommended Changes:

```diff
-
- 171:  if (_tier != _canaryTier) {
-      // canary tier
- 173:      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1));
-    } else {
- 175:      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));

+    return
+      (_tier != _canaryTier)
+        ? _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1))
+        : _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
```

## [G-05] Missing `zero-address` check in `constructor`

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also waste gas as it requires the redeployment of the contract.

_3 instances - 1 file:_

### Instance#1 : Params can be checked for address(0) before assigning value into `prizeToken`,`twabController` and `drawManager`.

```solidity
File : src/PrizePool.sol
258:     constructor(
259:    ConstructorParams memory params
  )
    TieredLiquidityDistributor(
      params.numberOfTiers,
      params.tierShares,
      params.canaryShares,
      params.reserveShares
    )
  {
    if (unwrap(params.smoothing) >= unwrap(UNIT)) {
      revert SmoothingGTEOne(unwrap(params.smoothing));
    }
271:    prizeToken = params.prizeToken;
272:    twabController = params.twabController;
273:    smoothing = params.smoothing;
274:    claimExpansionThreshold = params.claimExpansionThreshold;
275:    drawPeriodSeconds = params.drawPeriodSeconds;
276:    _lastClosedDrawStartedAt = params.firstDrawStartsAt;

278:    drawManager = params.drawManager;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L258C2-L278C38

## [G-06] Use assembly for address(0) comparison

### saves 65 gas each time

_1 instance - 1 file:_

### Instance#1 :

```solidity
File : src/PrizePool.sol
300:    if (drawManager != address(0)) {
301:      revert DrawManagerAlreadySet();
302:       }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L300C5-L301C38

Recommended Changes:

```diff
+       bool isAddressZero;
+        assembly {
+            isAddressZero := eq(address(drawManager), 0)
+        }
-   if (drawManager != address(0)) {
+   if(!isAddressZero){
             revert DrawManagerAlreadySet();
      }
```

## [G-07] Function calls can be cached rather than re calling from same function with same value will save gas

_2 instances - 1 file:_

### Instance#1 : `_openDrawEndsAt()` can be cached.

```solidity
File: src/PrizePool.sol
353:      if (block.timestamp < _openDrawEndsAt()) {
354:      revert DrawNotFinished(_openDrawEndsAt(), uint64(block.timestamp));

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L353C4-L354C74

## [G-08] <x> += <y> / <x> -= <y> costs more gas than <x> = <x> + <y> / <x> = <x> - <y> for state variables

The Solidity compiler does not optimize the <x> += <y> / <x> -= <y> operation for state variables. This means that every time the state variable is updated, the entire value is copied to memory, the operation is performed, and then the value is copied back to storage. This is expensive and can be avoided by using <x> = <x> + <y> / <x> = <x> - <y> instead.

_1 instances - 1 file:_

### Instance#1:

```diff
File: src/PrizePool.sol
- 455:    claimerRewards[_feeRecipient] += _fee;
+ 455:    claimerRewards[_feeRecipient] =claimerRewards[_feeRecipient] +_fee;

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L455C4-L455C45

## [G-09] Use unchecked{} whenever underflow/overflow not possible saves 130 gas each time

Because of if condition check before the operation(+/-), it can be decided that underflow/overflow not possible.

_2 instances - 1 file:_

### Instance#1 : On line 459 `tierLiquidity.prizeSize - _fee` can't underflow due to line 417 if () condition so they can be unchecked. If that condition is false then only line 459 will run otherwise it will revert from line 418.

```diff
File: src/PrizePool.sol

417: if (_fee > tierLiquidity.prizeSize) {
418:      revert FeeTooLarge(_fee, tierLiquidity.prizeSize);
419:   }

      ...
      //@audit gas add unchecked{} here
- 459: uint256 amount = tierLiquidity.prizeSize - _fee;
+ 459: unchecked{
+        uint256 amount = tierLiquidity.prizeSize - _fee;}

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L417-L459

### Instance#2 : line 490 can be unchecked{} due to line 484 and 486. Use cached variable `_available` instead of re reading `claimerRewards[msg.sender]` again saves another 100 Gas.

```solidity
File: src/PrizePool.sol

483:   function withdrawClaimRewards(address _to, uint256 _amount) external {
    uint256 _available = claimerRewards[msg.sender];

    if (_amount > _available) {
      revert InsufficientRewardsError(_amount, _available);
    }

490:    claimerRewards[msg.sender] -= _amount;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L483C1-L490C43

Recommended Changes :

```diff
 483: function withdrawClaimRewards(address _to, uint256 _amount) external {
    uint256 _available = claimerRewards[msg.sender];

 486:   if (_amount > _available) {
      revert InsufficientRewardsError(_amount, _available);
    }

- 490:    claimerRewards[msg.sender] -= _amount;
+ 490:    unchecked { claimerRewards[msg.sender] = _available - _amount };
```

## [G-10] Do not calculate constant variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas each time of use.

_Total 117 instances - 1 files:_

### Instance#1-117 : Assign direct simple constant value after calculating off chain

```solidity
File : src/abstract/TieredLiquidityDistributor.sol
84: SD59x18 internal constant TIER_ODDS_0_3 = SD59x18.wrap(2739726027397260);
85:  SD59x18 internal constant TIER_ODDS_1_3 = SD59x18.wrap(52342392259021369);
        ...

200:  SD59x18 internal constant TIER_ODDS_14_15 = SD59x18.wrap(1000000000000000000);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L84-L200

## [G-11] Use nested if and, avoid multiple check combinations

Using nested if is cheaper gas wise than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

_2 instances - 1 files:_

### Instance#1-2:

```solidity
File : src/PrizePool.sol
794: if (
795:      _nextNumberOfTiers >= _numTiers &&
      canaryClaimCount >=
      fromUD60x18(
        intoUD60x18(_claimExpansionThreshold).mul(_canaryPrizeCountFractional(_numTiers).floor())
      ) &&
      claimCount >=
      fromUD60x18(
802:        intoUD60x18(_claimExpansionThreshold).mul(toUD60x18(_estimatedPrizeCount(_numTiers)))
803:     )
         ) {
      // increase the number of tiers to include a new tier
      _nextNumberOfTiers = _numTiers + 1;
    }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L794C5-L803C8

Recommended Changes:

```diff
- 794: if (
- 795:      _nextNumberOfTiers >= _numTiers &&
-      canaryClaimCount >=
-      fromUD60x18(
-        intoUD60x18(_claimExpansionThreshold).mul(_canaryPrizeCountFractional(_numTiers).floor())
-      ) &&
-      claimCount >=
-      fromUD60x18(
- 802:        intoUD60x18(_claimExpansionThreshold).mul(toUD60x18(_estimatedPrizeCount(_numTiers)))
- 803:     )
-         ) {
-      // increase the number of tiers to include a new tier
-      _nextNumberOfTiers = _numTiers + 1;
-    }

+    if (_nextNumberOfTiers >= _numTiers) {
+            if (
+                canaryClaimCount >=
+                fromUD60x18(
+                    intoUD60x18(_claimExpansionThreshold).mul(
+                        _canaryPrizeCountFractional(_numTiers).floor()
+                    )
+                )
+            ) {
+                if (
+                    claimCount >=
+                    fromUD60x18(
+                        intoUD60x18(_claimExpansionThreshold).mul(
+                            toUD60x18(_estimatedPrizeCount(_numTiers))
+                        )
+                    )
+                ) {
+                    // increase the number of tiers to include a new tier
+                    _nextNumberOfTiers = _numTiers + 1;
+                }
+            }
+        }
```
