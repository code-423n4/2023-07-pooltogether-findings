### Gas Optimizations List

| Number | Optimization Details                                                                                       | Instances |
| :----: | :--------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Use constants instead of type(uintx).max or type(uintx).min or type(intx).max or type(intx).min            |     6     |
| [G-02] | If’s/require() statements that check input arguments should be at the top of the function                  |     1     |
| [G-03] | Use unchecked{} whenever underflow/overflow not possible                                                   |     4     |
| [G-04] | Non efficient zero initialization                                                                          |     2     |
| [G-05] | Using ternary operator instead of if else saves gas                                                        |     5     |
| [G-06] | Make 3 event parameters indexed when possible                                                              |     8     |
| [G-07] | x + y is more efficient than using += for state variables (likewise for -=)                                |     3     |
| [G-08] | Use storage instead of memory for structs/arrays                                                           |     3     |
| [G-09] | Storage variables should be cached in stack(if possible) variables rather than re-reading them from memory |     7     |
| [G-10] | The result of a function call should be cached rather than re-calling the function                         |     1     |

Total 10 issues

## [G-01] Use constants instead of type(uintx).max or type(uintx).min or type(intx).max or type(intx).min

_Total 6 instances - 1 file:_

### Instance#1:

```solidity
File: /claimer/src/libraries/LinearVRGDALib.sol
56:   targetTimeUnwrapped > 0 && decayConstantUnwrapped > type(int256).max / targetTimeUnwrapped
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L56

### Instance#2:

```solidity
File: /claimer/src/libraries/LinearVRGDALib.sol
62:    targetTimeUnwrapped < 0 && decayConstantUnwrapped > type(int256).min / targetTimeUnwrapped
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L62C8-L62C99

### Instance#3:

```solidity
File: /claimer/src/libraries/LinearVRGDALib.sol
82:     if (targetPriceInt > type(int256).max / expResult) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L82

### Instance#4:

```solidity
File: /claimer/src/libraries/LinearVRGDALib.sol
83:     return type(uint256).max;
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L83

### Instance#5:

```solidity
File: /claimer/src/libraries/LinearVRGDALib.sol
89:    if (targetPriceInt > type(int256).max / extraPrecisionExpResult) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L89

### Instance#6:

```solidity
File: /claimer/src/libraries/LinearVRGDALib.sol
90:    return type(uint256).max;
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L90C11-L90C11

## [G-02]IF’s/require() statements that check input arguments should be at the top of the function

_1 instance - 1 file:_

### Instance#1:

```solidity
File: /prize-pool/src/PrizePool.sol
278:drawManager = params.drawManager;
279:    if (params.drawManager != address(0)) {
280:      emit DrawManagerSet(params.drawManager);
281:    }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L278C5-L281C6

## [G-03] Use unchecked{} whenever underflow/overflow not possible

_4 instances - 2 files:_

### Instance#1: updation of variable i can be marked as unchecked to save gas

```solidity
File: /claimer/src/Claimer.sol
68:  for (uint i = 0; i < winners.length; i++) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68

### Instance#2: updation of variable i can be marked as unchecked to save gas

```solidity
File: /claimer/src/Claimer.sol
144: for (uint i = 0; i < _claimCount; i++) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144C1-L144C1

### Instance#3: updation of variable i can be marked as unchecked to save gas

```solidity
File: prize-pool/src/abstract/TieredLiquidityDistributor.sol
361:  for (uint8 i = start; i < end; i++) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144C1-L144C1

### Instance#4: updation of variable i can be marked as unchecked to save gas

```solidity
File: prize-pool/src/abstract/TieredLiquidityDistributor.sol
630:   for (uint8 i = start; i < end; i++) {
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L630

## [G-04] Non efficient zero initialization

_2 instances - 1 file:_

### Instance#1: Initialisation of i as zero is redundant

```solidity
File: /claimer/src/Claimer.sol
68: for (uint i = 0; i < winners.length; i++) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68

### Instance#2: Initialisation of i as zero is redundant

```solidity
File: /claimer/src/Claimer.sol
144: for (uint i = 0; i < _claimCount; i++) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144C1-L144C1

## [G-05] Using ternary operator instead of if-else saves gas

_5 instances - 3 file:_

### Instance#1:

```solidity
File: /claimer/src/Claimer.sol
171: if (_tier != _canaryTier) {
172:      // canary tier
173:      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1));
174:    } else {
175:      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
176:   }
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L171C4-L176C6

### Instance#2:

```solidity
File: /prize-pool/src/abstract/TieredLiquidityDistributor.sol
353: if (_nextNumberOfTiers > numTiers) {
354:     start = numTiers - 1;
355:      end = _nextNumberOfTiers;
356:    } else {
357:      // just reset the canary tier
358:      start = _nextNumberOfTiers - 1;
359:      end = _nextNumberOfTiers;
360:    }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L353C5-L360C6

### Instance#3:

```solidity
File: /prize-pool/src/abstract/TieredLiquidityDistributor.sol
565:if (_isCanaryTier(_tier, _numberOfTiers)) {
566:        prizeSize = _computePrizeSize(
567:          _tierPrizeTokenPerShare,
568:          _prizeTokenPerShare,
569:          _canaryPrizeCountFractional(_numberOfTiers),
570:          canaryShares
571:        );
572:      } else {
573:        prizeSize = _computePrizeSize(
574:          _tierPrizeTokenPerShare,
575:          _prizeTokenPerShare,
576:          toUD60x18(TierCalculationLib.prizeCount(_tier)),
577:          tierShares
578:        );
579:      }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L565C7-L579C8

### Instance#4:

```solidity
File: /prize-pool/src/abstract/TieredLiquidityDistributor.sol
622: if (_nextNumberOfTiers < _numberOfTiers) {
623:      start = _nextNumberOfTiers - 1;
624:      end = _numberOfTiers;
625:    } else {
626:      // just reset the canary tier
627:      start = _numberOfTiers - 1;
628:      end = _numberOfTiers;
629:    }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L622C5-L629C6

### Instance#5:

```solidity
File: /prize-pool/src/PrizePool.sol
440: if (_isCanaryTier(_tier, numberOfTiers)) {
441:      canaryClaimCount++;
442:    } else {
443:      claimCount++;
444:    }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L440C5-L444C6

## [G-06] Make 3 event parameters indexed when possible

_8 instances - 2 file:_

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

### Instance#1:

```solidity
File: /prize-pool/src/abstract/TieredLiquidityDistributor.sol
41: event ReserveConsumed(uint256 amount);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L41

### Instance#2:

```solidity
File: /prize-pool/src/PrizePool.sol
 138:event ClaimedPrize(
 139:   address indexed vault,
 140:   address indexed winner,
 141:   address indexed recipient,
 142:   uint16 drawId,
 143:   uint8 tier,
 144:   uint32 prizeIndex,
 145:   uint152 payout,
 146:   uint96 fee,
 147:   address feeRecipient
 148: );
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L138C2-L148C5

### Instance#3:

```solidity
File: /prize-pool/src/PrizePool.sol
158: event DrawClosed(
159:    uint16 indexed drawId,
160:    uint256 winningRandomNumber,
161:    uint8 numTiers,
162:    uint8 nextNumTiers,
163:    uint104 reserve,
164:    UD34x4 prizeTokensPerShare,
165:    uint64 drawStartedAt
166:  );
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L158C3-L166C5

### Instance#4:

```solidity
File: /prize-pool/src/PrizePool.sol
171:event WithdrawReserve(address indexed to, uint256 amount);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L171

### Instance#5:

```solidity
File: /prize-pool/src/PrizePool.sol
176:event IncreaseReserve(address user, uint256 amount);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L176C3-L176C55

### Instance#6:

```solidity
File: /prize-pool/src/PrizePool.sol
182: event ContributePrizeTokens(address indexed vault, uint16 indexed drawId, uint256 amount);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L182C3-L182C93

### Instance#7:

```solidity
File: /prize-pool/src/PrizePool.sol
188:   event WithdrawClaimRewards(address indexed to, uint256 amount, uint256 available);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L188C1-L188C85

### Instance#8:

```solidity
File: /prize-pool/src/PrizePool.sol
193:   event IncreaseClaimRewards(address indexed to, uint256 amount);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L193C3-L193C66

## [G-07] x + y is more efficient than using += for state variables (likewise for -=)

_3 instances - 1 file:_

### Instance#1:

```solidity
File: /prize-pool/src/PrizePool.sol
339:  _reserve -= _amount;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L339

### Instance#2:

```solidity
File: /prize-pool/src/PrizePool.sol
490:  claimerRewards[msg.sender] -= _amount;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L490

### Instance#3:

```solidity
File: /prize-pool/src/PrizePool.sol
770:     _nextExpectedEndTime +=
771:drawPeriodSeconds *
772:        (uint64((block.timestamp - _nextExpectedEndTime) / drawPeriodSeconds));
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L770C6-L772C80

## [G-08] Use storage instead of memory for structs/arrays

_3 instances - 2 file:_

### Instance#1:

```solidity
File: /prize-pool/src/abstract/TieredLiquidityDistributor.sol
472: Tier memory tier = _tiers[_tier];
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L472C5-L472C38

### Instance#2:

```solidity
File: /prize-pool/src/abstract/TieredLiquidityDistributor.sol
631: Tier memory tierLiquidity = _tiers[i];
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L631C7-L631C45

### Instance#3:

```solidity
File: /prize-pool/src/PrizePool.sol
415: Tier memory tierLiquidity = _getTier(_tier, numberOfTiers);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L415C5-L415C64

## [G-09] Storage variables should be cached in stack(if possible) variables rather than re-reading them from memory

_7 instances - 2 file:_

### Instance#1: `lastClosedDrawId` can be cached to stack variable

Findings are labeled with ' <= FOUND

```solidity
File: /prize-pool/src/abstract/TieredLiquidityDistributor.sol
311:function contributePrizeTokens(address _prizeVault, uint256 _amount) external returns (uint256) {
312:    uint256 _deltaBalance = prizeToken.balanceOf(address(this)) - _accountedBalance();
313:    if (_deltaBalance < _amount) {
314:      revert ContributionGTDeltaBalance(_amount, _deltaBalance);
315:    }
316:    DrawAccumulatorLib.add(
317:      vaultAccumulator[_prizeVault],
318:      _amount,
319:      lastClosedDrawId + 1,// <=FOUND
320:      smoothing.intoSD59x18()
321:    );
322:    DrawAccumulatorLib.add(
323:      totalAccumulator,
324:      _amount,
325:      lastClosedDrawId + 1,
326:      smoothing.intoSD59x18()
327:    );
328:    emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);// <=FOUND
329:    return _deltaBalance;
330: }

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L311C3-L330C3

### Instance#2: `_reserve` can be cached to stack variable

Findings are labeled with ' <= FOUND

```solidity
File: /prize-pool/src/PrizePool.sol
335:function withdrawReserve(address _to, uint104 _amount) external onlyDrawManager {
336:    if (_amount > _reserve) {//<= FOUND
337:      revert InsufficientReserve(_amount, _reserve);//<= FOUND
338:    }
339:    _reserve -= _amount;//<= FOUND
340:    _transfer(_to, _amount);
341:    emit WithdrawReserve(_to, _amount);
342:  }

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L311C3-L330C3

### Instance#3: `lastClosedDrawId` can be cached to stack variable

```solidity
File: /prize-pool/src/PrizePool.sol
 618:uint96(_contributionsForDraw(lastClosedDrawId + 1))

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L618C6-L618C58

### Instance#4: `lastClosedDrawId` can be cached to stack variable

```solidity
File: /prize-pool/src/PrizePool.sol
 626:return _contributionsForDraw(lastClosedDrawId);

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L626C5-L626C52

### Instance#5: `drawPeriodSeconds` can be cached to stack variable

Findings are labeled with ' <= FOUND

```solidity
File: /prize-pool/src/PrizePool.sol
685:  endTimestamp = _lastClosedDrawStartedAt + drawPeriodSeconds;//<= FOUND
686:    SD59x18 tierOdds = _tierOdds(_tier, _numberOfTiers);
687:    uint256 durationInSeconds = TierCalculationLib.estimatePrizeFrequencyInDraws(tierOdds) * drawPeriodSeconds;//<= FOUND

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L685C4-L687C112

## Instance#6: `drawPeriodSeconds` can be cached to stack variable

Findings are labeled with ' <= FOUND

```solidity
File: /prize-pool/src/PrizePool.sol
749:  function _openDrawStartedAt() internal view returns (uint64) {
750:    return _openDrawEndsAt() - drawPeriodSeconds;//<= FOUND
751:  }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L749C3-L752C1

## Instance#7: `_lastClosedDrawStartedAt` ,`lastClosedDrawId`,`drawPeriodSeconds`,`numMissedDraws`can be cached to stack variable

Findings are labeled with ' <= FOUND

```solidity
File: /prize-pool/src/PrizePool.sol
760:  function _openDrawEndsAt() internal view returns (uint64) {
761:    // If this is the first draw, we treat _lastClosedDrawStartedAt as the start of this draw
762:    uint64 _nextExpectedEndTime = _lastClosedDrawStartedAt +//<= FOUND
763:      (lastClosedDrawId == 0 ? 1 : 2) *//<= FOUND
764:      drawPeriodSeconds;//<= FOUND
765:
766:    if (block.timestamp > _nextExpectedEndTime) {
767:      // Use integer division to get the number of draw periods passed between the expected end time and now
768:      // Offset the end time by the total duration of the missed draws
769:      // drawPeriodSeconds * numMissedDraws//<= FOUND
770:      _nextExpectedEndTime +=
771:        drawPeriodSeconds *//<= FOUND
772:        (uint64((block.timestamp - _nextExpectedEndTime) / drawPeriodSeconds));//<= FOUND
773:    }
774:
775:    return _nextExpectedEndTime;
776:  }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L760C3-L776C4

## [G-10] The result of a function call should be cached rather than re-calling the function

_1 instance - 1 file:_

### Instance#1: `smoothing.intoSD59x18()` can be cached to stack variable

Findings are labeled with ' <= FOUND

```solidity
File: /prize-pool/src/abstract/TieredLiquidityDistributor.sol
311:function contributePrizeTokens(address _prizeVault, uint256 _amount) external returns (uint256) {
312:    uint256 _deltaBalance = prizeToken.balanceOf(address(this)) - _accountedBalance();
313:    if (_deltaBalance < _amount) {
314:      revert ContributionGTDeltaBalance(_amount, _deltaBalance);
315:    }
316:    DrawAccumulatorLib.add(
317:      vaultAccumulator[_prizeVault],
318:      _amount,
319:      lastClosedDrawId + 1,
320:      smoothing.intoSD59x18()// <=FOUND
321:    );
322:    DrawAccumulatorLib.add(
323:      totalAccumulator,
324:      _amount,
325:      lastClosedDrawId + 1,
326:      smoothing.intoSD59x18()// <=FOUND
327:    );
328:    emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);
329:    return _deltaBalance;
330: }

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L311C3-L330C3
