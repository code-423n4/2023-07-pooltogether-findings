# Codebase Optimization Report

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Table Of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Table Of Contents](#table-of-contents)
  - [Note on Gas estimates.](#note-on-gas-estimates)
  - [Tighly pack storage variables/optimize the order of variable declaration(Save 2.1K Gas)](#tighly-pack-storage-variablesoptimize-the-order-of-variable-declarationsave-21k-gas)
    - [We can pack `_liquidationPair` with `_yieldFeePercentage` by reducing size of `_yieldFeePercentage`  to uint64](#we-can-pack-_liquidationpair-with-_yieldfeepercentage-by-reducing-size-of-_yieldfeepercentage--to-uint64)
  - [Expensive operation inside a for loop](#expensive-operation-inside-a-for-loop)
  - [Use calldata instead of memory for function parameters](#use-calldata-instead-of-memory-for-function-parameters)
    - [Use calldata on `_name` and `_symbol`(Save 524 Gas on average)](#use-calldata-on-_name-and-_symbolsave-524-gas-on-average)
  - [Repeatedly doing same operation(addition) involving a state variable should be avoided](#repeatedly-doing-same-operationaddition-involving-a-state-variable-should-be-avoided)
  - [Use the existing memory value to do the subtraction](#use-the-existing-memory-value-to-do-the-subtraction)
  - [Emitting storage values instead of the memory one.](#emitting-storage-values-instead-of-the-memory-one)
  - [Optimizing check order for cost efficient function execution](#optimizing-check-order-for-cost-efficient-function-execution)
    - [Validate parameters before doing any sstores(More than 2200 gas could be saved in case of a revert)](#validate-parameters-before-doing-any-sstoresmore-than-2200-gas-could-be-saved-in-case-of-a-revert)
    - [Reorder the checks to have cheaper validations come first](#reorder-the-checks-to-have-cheaper-validations-come-first)
  - [Don't cache a state variable if it's used only once](#dont-cache-a-state-variable-if-its-used-only-once)
  - [Do not cache immutable values](#do-not-cache-immutable-values)
  - [Conclusion](#conclusion)


## Note on Gas estimates.

I've tried to give the exact amount of gas being saved from running the included tests. Whenever the function is within the test coverage, the average gas before and after will be included, and often a diff of the code will also accompany this.

Some functions are not covered by the test cases or are internal/private functions. In this case, the gas can be estimated by looking at the opcodes involved or simply benchmark any function that calls our function of interest.


## Tighly pack storage variables/optimize the order of variable declaration(Save 2.1K Gas)

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). 

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L215-L224

### We can pack `_liquidationPair` with `_yieldFeePercentage` by reducing size of `_yieldFeePercentage`  to uint64

```solidity
File: /src/Vault.sol
215:  LiquidationPair private _liquidationPair;

218:  uint256 private _assetUnit;

221:  uint256 private _lastRecordedExchangeRate;

224:  uint256 private _yieldFeePercentage;
```

When setting the value of `_yieldFeePercentage` we have some bounds that constrain the max value the assigned value can be

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1218-L1223
```solidity
1218:  function _setYieldFeePercentage(uint256 yieldFeePercentage_) internal {
1219:    if (yieldFeePercentage_ > FEE_PRECISION) {
1220:      revert YieldFeePercentageGTPrecision(yieldFeePercentage_, FEE_PRECISION);
1221:    }
1222:    _yieldFeePercentage = yieldFeePercentage_;
1223:  }
```

From the above setter, it is evident the value of `_yieldFeePercentage` cannot be greater than `FEE_PRECISION` lest we revert.
`FEE_PRECISION` is a constant defined as shown below
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L232-L233
```solidity
232:  /// @notice Fee precision denominated in 9 decimal places and used to calculate yield fee percentage.
233:  uint256 private constant FEE_PRECISION = 1e9;
```

This means the value assigned to `_yieldFeePercentage` cannot be greater than `1e9`. Using `uint64` we get a max of `18446744073709551615` which should be more than enough.

As such we can refactor and pack the variables as shown below
```diff
diff --git a/src/Vault.sol b/src/Vault.sol
index e92ecaf..4c21912 100644
--- a/src/Vault.sol
+++ b/src/Vault.sol
@@ -213,6 +213,9 @@ contract Vault is ERC4626, ERC20Permit, ILiquidationSource, Ownable {

   /// @notice Address of the LiquidationPair used to liquidate yield for prize token.
   LiquidationPair private _liquidationPair;
+
+  /// @notice Yield fee percentage represented in integer format with 9 decimal places (i.e. 10000000 = 0.01 = 1%).
+  uint64 private _yieldFeePercentage;

   /// @notice Underlying asset unit (i.e. 10 ** 18 for DAI).
   uint256 private _assetUnit;
@@ -220,9 +223,6 @@ contract Vault is ERC4626, ERC20Permit, ILiquidationSource, Ownable {
   /// @notice Most recent exchange rate recorded when burning or minting Vault shares.
   uint256 private _lastRecordedExchangeRate;

-  /// @notice Yield fee percentage represented in integer format with 9 decimal places (i.e. 10000000 = 0.01 = 1%).
-  uint256 private _yieldFeePercentage;
-
   /// @notice Address of the yield fee recipient that receives the fee amount when yield is captured.
   address private _yieldFeeRecipient;

```
Note: We would need to refactor all places this value is used to change the type to uint64


## Expensive operation inside a for loop

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L361-L369
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 1901    | 59662   | 67737 | 79917 |
| After  | 1901    | 59649   | 67723 | 79903 |
```solidity
File: /src/abstract/TieredLiquidityDistributor.sol
361:    for (uint8 i = start; i < end; i++) {
362:      _tiers[i] = Tier({
363:        drawId: closedDrawId,
364:        prizeTokenPerShare: prizeTokenPerShare,
365:        prizeSize: uint96(
366:          _computePrizeSize(i, _nextNumberOfTiers, _prizeTokenPerShare, newPrizeTokenPerShare)
367:        )
368:      });
369:    }
```

```diff
diff --git a/src/abstract/TieredLiquidityDistributor.sol b/src/abstract/TieredLiquidityDistributor.sol
index 8238103..ba0587b 100644
--- a/src/abstract/TieredLiquidityDistributor.sol
+++ b/src/abstract/TieredLiquidityDistributor.sol
@@ -358,10 +358,11 @@ contract TieredLiquidityDistributor {
       start = _nextNumberOfTiers - 1;
       end = _nextNumberOfTiers;
     }
+     UD34x4  prizeTokenPerShare_ = prizeTokenPerShare ;
     for (uint8 i = start; i < end; i++) {
       _tiers[i] = Tier({
         drawId: closedDrawId,
-        prizeTokenPerShare: prizeTokenPerShare,
+        prizeTokenPerShare: prizeTokenPerShare_,
         prizeSize: uint96(
           _computePrizeSize(i, _nextNumberOfTiers, _prizeTokenPerShare, newPrizeTokenPerShare)
         )
```

## Use calldata instead of memory for function parameters

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L55-L66

### Use calldata on `_name` and `_symbol`(Save 524 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 3737309    | 3737309   | 3737309 | 3737309 |
| After  | 3736785    | 3736785   | 3736785 | 3736785 |
```solidity
File: /src/VaultFactory.sol
55:  function deployVault(
56:    IERC20 _asset,
57:    string memory _name,
58:    string memory _symbol,
59:    TwabController _twabController,
60:    IERC4626 _yieldVault,
61:    PrizePool _prizePool,
62:    address _claimer,
63:    address _yieldFeeRecipient,
64:    uint256 _yieldFeePercentage,
65:    address _owner
66:  ) external returns (address) {
```

```diff
diff --git a/src/VaultFactory.sol b/src/VaultFactory.sol
index 559fd90..2b85cba 100644
--- a/src/VaultFactory.sol
+++ b/src/VaultFactory.sol
@@ -54,8 +54,8 @@ contract VaultFactory {
    */
   function deployVault(
     IERC20 _asset,
-    string memory _name,
-    string memory _symbol,
+    string calldata _name,
+    string calldata _symbol,
     TwabController _twabController,
     IERC4626 _yieldVault,
     PrizePool _prizePool,
```

## Repeatedly doing same operation(addition) involving a state variable should be avoided

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L316-L328
**Note this was not even found by the bot under caching state variables**
**Gas saved on average: 548**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17709    | 181028   | 190311 | 226423 |
| After  | 17709    | 180480   | 189753 | 225865 |
```solidity
File: /src/PrizePool.sol
316:    DrawAccumulatorLib.add(
317:      vaultAccumulator[_prizeVault],
318:      _amount,
319:      lastClosedDrawId + 1,
320:      smoothing.intoSD59x18()
321:    );
322:    DrawAccumulatorLib.add(
323:      totalAccumulator,
324:      _amount,
325:      lastClosedDrawId + 1,
326:      smoothing.intoSD59x18()
327:    );
328:    emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);
```
In the function above, the operation `lastClosedDrawId + 1` is being done repeatedly. As `lastClosedDrawId` is a state variable, the amount of gas becomes too much. We should do the operation and cache the result in a memory variable.
```diff
diff --git a/src/PrizePool.sol b/src/PrizePool.sol
index a42a27e..6a850ae 100644
--- a/src/PrizePool.sol
+++ b/src/PrizePool.sol
@@ -313,19 +313,20 @@ contract PrizePool is TieredLiquidityDistributor {
     if (_deltaBalance < _amount) {
       revert ContributionGTDeltaBalance(_amount, _deltaBalance);
     }
+    uint16 _lastClosedDrawIdPlusOne = lastClosedDrawId + 1;
     DrawAccumulatorLib.add(
       vaultAccumulator[_prizeVault],
       _amount,
-      lastClosedDrawId + 1,
+      _lastClosedDrawIdPlusOne,
       smoothing.intoSD59x18()
     );
     DrawAccumulatorLib.add(
       totalAccumulator,
       _amount,
-      lastClosedDrawId + 1,
+      _lastClosedDrawIdPlusOne,
       smoothing.intoSD59x18()
     );
-    emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);
+    emit ContributePrizeTokens(_prizeVault, _lastClosedDrawIdPlusOne, _amount);
     return _deltaBalance;
   }
```


## Use the existing memory value to do the subtraction

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L483-L493

**Average Gas Saved: 77**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2750    | 18704   | 23717 | 29646 |
| After  | 2750    | 18627   | 23615 | 29518 |
```solidity
File: /src/PrizePool.sol
483:  function withdrawClaimRewards(address _to, uint256 _amount) external {
484:    uint256 _available = claimerRewards[msg.sender];

486:    if (_amount > _available) {
487:      revert InsufficientRewardsError(_amount, _available);
488:    }

490:    claimerRewards[msg.sender] -= _amount;
491:    _transfer(_to, _amount);
492:    emit WithdrawClaimRewards(_to, _amount, _available);
493:  }
```

```diff
diff --git a/src/PrizePool.sol b/src/PrizePool.sol
index a42a27e..596fa1c 100644
--- a/src/PrizePool.sol
+++ b/src/PrizePool.sol
@@ -487,7 +487,7 @@ contract PrizePool is TieredLiquidityDistributor {
       revert InsufficientRewardsError(_amount, _available);
     }

-    claimerRewards[msg.sender] -= _amount;
+    claimerRewards[msg.sender] = _available - _amount;
     _transfer(_to, _amount);
     emit WithdrawClaimRewards(_to, _amount, _available);
   }
```

## Emitting storage values instead of the memory one.

Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L372-L385
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 1901    | 59662   | 67737 | 79917 |
| After  | 1901    | 59649   | 67723 | 79903 |
```solidity
File: /src/PrizePool.sol
372:    _lastClosedDrawStartedAt = openDrawStartedAt_;
373:    _lastClosedDrawAwardedAt = uint64(block.timestamp);

375:    emit DrawClosed(
376:      lastClosedDrawId,
377:      winningRandomNumber_,
378:      _numTiers,
379:      _nextNumberOfTiers,
380:      _reserve,
381:      prizeTokenPerShare,
382:      _lastClosedDrawStartedAt
383:    );
```

```diff
diff --git a/src/PrizePool.sol b/src/PrizePool.sol
index a42a27e..74d3a25 100644
--- a/src/PrizePool.sol
+++ b/src/PrizePool.sol
@@ -379,7 +379,7 @@ contract PrizePool is TieredLiquidityDistributor {
       _nextNumberOfTiers,
       _reserve,
       prizeTokenPerShare,
-      _lastClosedDrawStartedAt
+      openDrawStartedAt_
     );

     return lastClosedDrawId;
```


## Optimizing check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.


https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L235-L326

### Validate parameters before doing any sstores(More than 2200 gas could be saved in case of a revert)

```solidity
File: /src/abstract/TieredLiquidityDistributor.sol
235:  constructor(uint8 _numberOfTiers, uint8 _tierShares, uint8 _canaryShares, uint8 _reserveShares) {
236:    numberOfTiers = _numberOfTiers;
237:    tierShares = _tierShares;
238:    canaryShares = _canaryShares;
239:    reserveShares = _reserveShares;

320:    if (_numberOfTiers < MINIMUM_NUMBER_OF_TIERS) {
321:      revert NumberOfTiersLessThanMinimum(_numberOfTiers);
322:    }
323:    if (_numberOfTiers > MAXIMUM_NUMBER_OF_TIERS) {
324:      revert NumberOfTiersGreaterThanMaximum(_numberOfTiers);
325:    }
326:  }
```
We are assigning some Storage variables a function parameter on top of doing so other minor operations. We then check if the parameters passed to the constructor meet the required specification ie
```solidity
    if (_numberOfTiers < MINIMUM_NUMBER_OF_TIERS) {
      revert NumberOfTiersLessThanMinimum(_numberOfTiers);
    }
    if (_numberOfTiers > MAXIMUM_NUMBER_OF_TIERS) {
      revert NumberOfTiersGreaterThanMaximum(_numberOfTiers);
    }
```
If the above checks fail, we end up reverting the whole transaction meaning the gas spent writing to storage would go to waste

```diff
diff --git a/src/abstract/TieredLiquidityDistributor.sol b/src/abstract/TieredLiquidityDistributor.sol
index 8238103..34c2050 100644
--- a/src/abstract/TieredLiquidityDistributor.sol
+++ b/src/abstract/TieredLiquidityDistributor.sol
@@ -233,6 +233,12 @@ contract TieredLiquidityDistributor {
    * @param _reserveShares The number of shares to allocate to the reserve.
    */
   constructor(uint8 _numberOfTiers, uint8 _tierShares, uint8 _canaryShares, uint8 _reserveShares) {
+    if (_numberOfTiers < MINIMUM_NUMBER_OF_TIERS) {
+      revert NumberOfTiersLessThanMinimum(_numberOfTiers);
+    }
+    if (_numberOfTiers > MAXIMUM_NUMBER_OF_TIERS) {
+      revert NumberOfTiersGreaterThanMaximum(_numberOfTiers);
+    }
     numberOfTiers = _numberOfTiers;
     tierShares = _tierShares;
     canaryShares = _canaryShares;
@@ -317,12 +323,6 @@ contract TieredLiquidityDistributor {
       _tierShares
     );

-    if (_numberOfTiers < MINIMUM_NUMBER_OF_TIERS) {
-      revert NumberOfTiersLessThanMinimum(_numberOfTiers);
-    }
-    if (_numberOfTiers > MAXIMUM_NUMBER_OF_TIERS) {
-      revert NumberOfTiersGreaterThanMaximum(_numberOfTiers);
-    }
   }

```


https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L550-L568

### Reorder the checks to have cheaper validations come first

```solidity
File: /src/Vault.sol
550:  function liquidate(
551:    address _account,
552:    address _tokenIn,
553:    uint256 _amountIn,
554:    address _tokenOut,
555:    uint256 _amountOut
556:  ) public virtual override returns (bool) {
557:    _requireVaultCollateralized();
558:    if (msg.sender != address(_liquidationPair))
559:      revert LiquidationCallerNotLP(msg.sender, address(_liquidationPair));
560:    if (_tokenIn != address(_prizePool.prizeToken()))
561:      revert LiquidationTokenInNotPrizeToken(_tokenIn, address(_prizePool.prizeToken()));
562:    if (_tokenOut != address(this))
563:      revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));
564:    if (_amountOut == 0) revert LiquidationAmountOutZero();

566:    uint256 _liquidableYield = _liquidatableBalanceOf(_tokenOut);
567:    if (_amountOut > _liquidableYield)
568:      revert LiquidationAmountOutGTYield(_amountOut, _liquidableYield);
```

The first few checks are either reading from storage or making some external calls. This operations are quite expensive in terms of gas. We also have some checks that validate some function parameters. Reading function parameters is very cheap. As it is, in case we revert on the function parameter check, the gas consumed doing external calls or state reads would be wasted. We should reorder the checks to reduce gas consumed in case of a revert. Cheaper checks should be done first as shown below
```diff
diff --git a/src/Vault.sol b/src/Vault.sol
index e92ecaf..7652d7a 100644
--- a/src/Vault.sol
+++ b/src/Vault.sol
@@ -554,19 +554,23 @@ contract Vault is ERC4626, ERC20Permit, ILiquidationSource, Ownable {
     address _tokenOut,
     uint256 _amountOut
   ) public virtual override returns (bool) {
-    _requireVaultCollateralized();
-    if (msg.sender != address(_liquidationPair))
-      revert LiquidationCallerNotLP(msg.sender, address(_liquidationPair));
-    if (_tokenIn != address(_prizePool.prizeToken()))
-      revert LiquidationTokenInNotPrizeToken(_tokenIn, address(_prizePool.prizeToken()));
+    if (_amountOut == 0) revert LiquidationAmountOutZero();
+
     if (_tokenOut != address(this))
       revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));
-    if (_amountOut == 0) revert LiquidationAmountOutZero();

     uint256 _liquidableYield = _liquidatableBalanceOf(_tokenOut);
     if (_amountOut > _liquidableYield)
       revert LiquidationAmountOutGTYield(_amountOut, _liquidableYield);

+    _requireVaultCollateralized();
+
+    if (msg.sender != address(_liquidationPair))
+      revert LiquidationCallerNotLP(msg.sender, address(_liquidationPair));
+
+    if (_tokenIn != address(_prizePool.prizeToken()))
+      revert LiquidationTokenInNotPrizeToken(_tokenIn, address(_prizePool.prizeToken()));
+
     _prizePool.contributePrizeTokens(address(this), _amountIn);

     if (_yieldFeePercentage != 0) {
```


## Don't cache a state variable if it's used only once

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L851-L877
**Gas benchmarks based on function `isWinner` that calls this internal function**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 1046    | 26982   | 20177 | 55866 |
| After  | 1046    | 27014   | 20232 | 55921 |
```solidity
File: /src/PrizePool.sol
851:    uint8 _numberOfTiers = numberOfTiers;
852:    uint32 tierPrizeCount = _getTierPrizeCount(_tier, _numberOfTiers);

854:    if (_prizeIndex >= tierPrizeCount) {
855:      revert InvalidPrizeIndex(_prizeIndex, tierPrizeCount, _tier);
856:    }

858:    uint256 userSpecificRandomNumber = TierCalculationLib.calculatePseudoRandomNumber(
859:      _user,
860:      _tier,
861:      _prizeIndex,
862:      _winningRandomNumber
863:    );
864:    (uint256 _userTwab, uint256 _vaultTwabTotalSupply) = _getVaultUserBalanceAndTotalSupplyTwab(
865:      _vault,
866:      _user,
867:      _drawDuration
868:    );

870:    return
871:      TierCalculationLib.isWinner(
872:        userSpecificRandomNumber,
873:        uint128(_userTwab),
874:        uint128(_vaultTwabTotalSupply),
875:        _vaultPortion,
876:        _tierOdds
877:      );
```

```diff
diff --git a/src/PrizePool.sol b/src/PrizePool.sol
index a42a27e..9407b1c 100644
--- a/src/PrizePool.sol
+++ b/src/PrizePool.sol
@@ -848,8 +848,7 @@ contract PrizePool is TieredLiquidityDistributor {
     SD59x18 _tierOdds,
     uint16 _drawDuration
   ) internal view returns (bool) {
-    uint8 _numberOfTiers = numberOfTiers;
-    uint32 tierPrizeCount = _getTierPrizeCount(_tier, _numberOfTiers);
+    uint32 tierPrizeCount = _getTierPrizeCount(_tier, numberOfTiers);

     if (_prizeIndex >= tierPrizeCount) {
       revert InvalidPrizeIndex(_prizeIndex, tierPrizeCount, _tier);
```

## Do not cache immutable values

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L781-L810
`Benchmark based on function nextNumberOfTiers which calls our internal function`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 765    | 765   | 765 | 765 |
| After  | 757    | 757   | 757 | 757 |
```solidity
File: /src/PrizePool.sol
781:  function _computeNextNumberOfTiers(uint8 _numTiers) internal view returns (uint8) {
782:    UD2x18 _claimExpansionThreshold = claimExpansionThreshold;

793:    // check to see if we need to expand the number of tiers
794:    if (
795:      _nextNumberOfTiers >= _numTiers &&
796:      canaryClaimCount >=
797:      fromUD60x18(
798:intoUD60x18(_claimExpansionThreshold).mul(_canaryPrizeCountFractional(_numTiers).floor())
799:      ) &&
800:      claimCount >=
801:      fromUD60x18(
    802:intoUD60x18(_claimExpansionThreshold).mul(toUD60x18(_estimatedPrizeCount(_numTiers)))
803:      )
```

```diff
diff --git a/src/PrizePool.sol b/src/PrizePool.sol
index a42a27e..accf0a8 100644
--- a/src/PrizePool.sol
+++ b/src/PrizePool.sol
@@ -779,7 +779,6 @@ contract PrizePool is TieredLiquidityDistributor {
   /// @param _numTiers The current number of tiers
   /// @return The number of tiers for the next draw
   function _computeNextNumberOfTiers(uint8 _numTiers) internal view returns (uint8) {
-    UD2x18 _claimExpansionThreshold = claimExpansionThreshold;

     uint8 _nextNumberOfTiers = largestTierClaimed + 2; // canary tier, then length
     _nextNumberOfTiers = _nextNumberOfTiers > MINIMUM_NUMBER_OF_TIERS
@@ -795,11 +794,11 @@ contract PrizePool is TieredLiquidityDistributor {
       _nextNumberOfTiers >= _numTiers &&
       canaryClaimCount >=
       fromUD60x18(
-        intoUD60x18(_claimExpansionThreshold).mul(_canaryPrizeCountFractional(_numTiers).floor())
+        intoUD60x18(claimExpansionThreshold).mul(_canaryPrizeCountFractional(_numTiers).floor())
       ) &&
       claimCount >=
       fromUD60x18(
-        intoUD60x18(_claimExpansionThreshold).mul(toUD60x18(_estimatedPrizeCount(_numTiers)))
+        intoUD60x18(claimExpansionThreshold).mul(toUD60x18(_estimatedPrizeCount(_numTiers)))
       )
     ) {
       // increase the number of tiers to include a new tier
```

## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
