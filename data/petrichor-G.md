|      | Issue   | Instance |
|------|---------|----------|  
|[G-01]|Amounts should be checked for 0 before calling a transfer|2|
|[G-02]| Access mappings directly rather than using accessor functions|4|
|[G-03]|Using a positive conditional flow to save a NOT opcode|5|
|[G-04]|2**<n> should be re-written as type(uint<n>).max|7|
|[G-05]|Internal/Private functions only called once can be inlined to save gas |16|
|[G-06]| Use constants instead of type(uintx).max|11|
|[G-07]| IF’s/require() statements that check input arguments should be at the top of the function|4|
|[G-08]|Use assembly to write address storage values|1|
|[G-09]| Use hardcode address instead address(this)|18|
|[G-10]|abi.encode() is less efficient than abi.encodePacked()|1|
|[G-11]|Using XOR (^) and OR (|) bitwise equivalents|2|
|[G-12]| Don’t apply the same value to state variables|1|
|[G-13]|Using private rather than public for constants, saves gas|1|
|[G-14]|Using calldata instead of memory for read-only arguments in external functions saves gas|3|
|[G-15]|Avoid emitting storage values|6|
|[G-16]|Use assembly to perform efficient back-to-back calls|2|
|[G-17]|Multiple accesses of a mapping/array should use a storage pointer|1|
|[G-18]|Cache state variables outside of loop to avoid reading storage on every iteration|1|
|[G-19]|Duplicated require()/revert() checks should be refactored to a modifier or function.|1|

## [G-01] Amounts should be checked for 0 before calling a transfer

it is generally a good practice to check the amount for 0 before calling a transfer function in order to save gas. This is because if the amount is 0, there is no need to execute the transfer function, which can save gas costs.

```solidity

832      prizeToken.safeTransfer(_to, _amount);

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L832

```solidity

1126      emit Transfer(address(0), _receiver, _shares);

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1126




## [G-02] Access mappings directly rather than using accessor functions

Accessing mappings directly instead of using accessor functions can potentially save gas costs in some cases. This is because accessing a mapping directly involves a single storage read operation, while using an accessor function may involve multiple storage reads and writes.

```solidity

316      DrawAccumulatorLib.add(
      vaultAccumulator[_prizeVault],
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L316-L317

```solidity
541     DrawAccumulatorLib.getDisbursedBetween(
        vaultAccumulator[_vault],
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L541-L542

```solidity
963     DrawAccumulatorLib.getDisbursedBetween(
              vaultAccumulator[_vault],

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L963-L964

```solidity

660     uint96(userObservations[_vault][_from].details.balance)

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L660


## [G-03] Using a positive conditional flow to save a NOT opcode

Accessing mappings directly rather than using accessor functions and using positive conditional flow to save a NOT opcode are both techniques that can be used to optimize smart contracts for gas efficiency.

```solidity


428         if (
      !_isWinner(msg.sender, _winner, _tier, _prizeIndex, _vaultPortion, _tierOdds, _drawDuration)


```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L428

```solidity

459      if (!targetAtOrAfter)

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L459

```solidity

530   if (!_isFromDelegate && _fromDelegate != SPONSORSHIP_ADDRESS)
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L530

```solidity
561   if (!_isToDelegate && _toDelegate != SPONSORSHIP_ADDRESS)

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L561

```solidity

93      if (!targetAfterOrAt)

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol#L93

```solidity

1200     if (!_isVaultCollateralized()) revert VaultUnderCollateralized();

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1200


## [G-04] 2**<n> should be re-written as type(uint<n>).max

In Solidity, the expression 2**<n> is commonly used to represent the maximum value that can be stored in a variable of size n bits. However, this expression can be expensive in terms of gas cost, particularly for large values of n, because it requires a lot of computational steps to calculate the result.
To avoid this gas cost, the expression 2**<n> can be replaced with the built-in constant type(uint<n>).max. This constant represents the maximum value that can be stored in a variable of size n bits, and it is calculated by setting all bits in the variable to 1.

```solidity
40     uint256 _numberOfPrizes = 4 ** _tier;
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L40

```solidity
19        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L19

```solidity
20        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L20

```solidity
35        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L35

```solidity
36        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L36

```solidity

52        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L52

```solidity
53        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L53

## [G-05] Internal/Private functions only called once can be inlined to save gas 

Inlining internal or private functions that are only called once can be an effective way to optimize gas costs in Solidity contracts.

When a function is inlined, its code is inserted directly into the calling function at compile time, rather than being called as a separate function at runtime. This can eliminate the gas cost associated with the function call, as well as any overhead associated with setting up the function's stack frame.

```solidity

24     function getPerTimeUnit(
    uint256 _count,
    uint256 _durationSeconds
  ) internal pure returns (SD59x18)
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L24-L28


```solidity

39       function getVRGDAPrice(
    uint256 _targetPrice,
    uint256 _timeSinceStart,
    uint256 _sold,
    SD59x18 _perTimeUnit,
    SD59x18 _decayConstant
  ) internal pure returns (uint256)

```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L39-L45

```solidity

331    function _nextDraw(uint8 _nextNumberOfTiers, uint96 _prizeTokenLiquidity) internal
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L331

```solidity
518      function _consumeLiquidity(
       Tier memory _tierStruct,
       uint8 _tier,
       uint104 _liquidity
  )    internal returns (Tier memory) 

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L518-L522


```solidity

120       function getTotalRemaining(
          Accumulator storage accumulator,
          uint16 _startDrawId,
         SD59x18 _alpha
  )      internal view returns (uint256)

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L120-L124


```solidity
166        function getDisbursedBetween(
           Accumulator storage _accumulator,
           uint16 _startDrawId,
           uint16 _endDrawId,
           SD59x18 _alpha
      )   internal view returns (uint256)

```


https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L166-L171

```solidity

32     function estimatePrizeFrequencyInDraws(SD59x18 _tierOdds) internal pure returns (uint256)
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L32

```solidity
52       function canaryPrizeCount(
          uint8 _numberOfTiers,
          uint8 _canaryShares,
          uint8 _reserveShares,
          uint8 _tierShares
         ) internal pure returns (UD60x18)
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L52-L57

```solidity
73        function isWinner(
          uint256 _userSpecificRandomNumber,
          uint128 _userTwab,
          uint128 _vaultTwabTotalSupply,
          SD59x18 _vaultContributionFraction,
          SD59x18 _tierOdds
          )internal pure returns (bool) 
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L73-L79

```solidity
104        function calculatePseudoRandomNumber(
           address _user,
           uint8 _tier,
           uint32 _prizeIndex,
           uint256 _winningRandomNumber
           )internal pure returns (uint256)
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L104-L109

```solidity
133        function estimatedClaimCount(
           uint8 _numberOfTiers,
           uint16 _grandPrizePeriod
           ) internal pure returns (uint32)           

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L133-L136

```solidity

31     function lte(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (bool)
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L31

```solidity
47     function checkedSub(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (uint32)

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L47

```solidity

75          function increaseBalances(
            uint32 PERIOD_LENGTH,
            uint32 PERIOD_OFFSET,
            Account storage _account,
            uint96 _amount,
            uint96 _delegateAmount
            ) 
          internal
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L75-L82

```solidity
144        function decreaseBalances(
          uint32 PERIOD_LENGTH,
          uint32 PERIOD_OFFSET,
          Account storage _account,
          uint96 _amount,
          uint96 _delegateAmount,
          string memory _revertMessage
       )
        internal
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L144-L152


```solidity
263      function getBalanceAt(
         uint32 PERIOD_OFFSET,
         ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
         AccountDetails memory _accountDetails,
         uint32 _targetTime
        )    internal view returns (uint256)        

```


https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L263-L268


## [G-06] Use constants instead of type(uintx).max

Using constants instead of type(uintX).max can be an effective way to optimize gas costs in Solidity contracts.
type(uintX).max is a built-in constant in Solidity that represents the maximum value that can be stored in a variable of size X bits. However, this constant can be expensive in terms of gas cost, particularly for large values of X, because it requires a lot of computational steps to calculate the result.

```solidity

56     targetTimeUnwrapped > 0 && decayConstantUnwrapped > type(int256).max / targetTimeUnwrapped
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L56


```solidity
58     return type(uint256).max;
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L58

```solidity
70     return type(uint256).max;
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L70

```solidity
82     if (targetPriceInt > type(int256).max / expResult)
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L82

```solidity
83     return type(uint256).max;
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L83

```solidity
89     if (targetPriceInt > type(int256).max / extraPrecisionExpResult)
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L89

```solidity
90     return type(uint256).max;

```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L90

```solidity

282     asset_.safeApprove(address(yieldVault_), type(uint256).max);
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L282

```solidity
376     return _isVaultCollateralized() ? type(uint96).max : 0;
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L376


```solidity
384     return _isVaultCollateralized() ? type(uint96).max : 0;
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L384

```solidity
677     _asset.safeApprove(address(liquidationPair_), type(uint256).max);

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L677


## [G-07] IF’s/require() statements that check input arguments should be at the top of the function

Placing if statements and require() statements that check input arguments at the top of a function can be an effective way to optimize gas costs in Solidity contracts.
When an if statement or require() statement fails, the function immediately exits and all subsequent code is not executed. Therefore, placing these statements at the top of a function can reduce the amount of gas consumed by the function, as any unnecessary computation that would have been performed after the failed check is avoided.

```solidity

473        function _getPreviousOrAtObservation(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) private view returns (ObservationLib.Observation memory prevOrAtObservation) {
    uint32 currentTime = uint32(block.timestamp);

    uint16 oldestTwabIndex;
    uint16 newestTwabIndex;

    // If there are no observations, return a zeroed observation
    if (_accountDetails.cardinality == 0) {
      return
        ObservationLib.Observation({ cumulativeBalance: 0, balance: 0, timestamp: PERIOD_OFFSET });
    }


```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L473-L488






```solidity

612        function _getTierLiquidityToReclaim(
    uint8 _numberOfTiers,
    uint8 _nextNumberOfTiers,
    UD60x18 _prizeTokenPerShare
  ) internal view returns (uint256) {
    UD60x18 reclaimedLiquidity;
    // need to redistribute to the canary tier and any new tiers (if expanding)
    uint8 start;
    uint8 end;
    // if we are expanding, need to reset the canary tier and all of the new tiers
    if (_nextNumberOfTiers < _numberOfTiers) {
      start = _nextNumberOfTiers - 1;
      end = _numberOfTiers;
    } else {
      // just reset the canary tier
      start = _numberOfTiers - 1;
      end = _numberOfTiers;
    }
      for (uint8 i = start; i < end; i++) {
      Tier memory tierLiquidity = _tiers[i];
      uint8 shares = _computeShares(i, _numberOfTiers);
      UD60x18 liq = _getTierRemainingLiquidity(
        shares,
        fromUD34x4toUD60x18(tierLiquidity.prizeTokenPerShare),
        _prizeTokenPerShare
      );
      reclaimedLiquidity = reclaimedLiquidity.add(liq);
    }
    return fromUD60x18(reclaimedLiquidity);
  }

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L612-L641
```solidity

561        function _getNextOrNewestObservation(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) private view returns (ObservationLib.Observation memory nextOrNewestObservation) {
    uint32 currentTime = uint32(block.timestamp);

    uint16 oldestTwabIndex;

    // If there are no observations, return a zeroed observation
    if (_accountDetails.cardinality == 0) {
      return
        ObservationLib.Observation({ cumulativeBalance: 0, balance: 0, timestamp: PERIOD_OFFSET });
    }

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L561-L575



```solidity

550       function liquidate(
    address _account,
    address _tokenIn,
    uint256 _amountIn,
    address _tokenOut,
    uint256 _amountOut
  ) public virtual override returns (bool) {
    _requireVaultCollateralized();
    if (msg.sender != address(_liquidationPair))
      revert LiquidationCallerNotLP(msg.sender, address(_liquidationPair));
    if (_tokenIn != address(_prizePool.prizeToken()))
      revert LiquidationTokenInNotPrizeToken(_tokenIn, address(_prizePool.prizeToken()));
    if (_tokenOut != address(this))
      revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));
    if (_amountOut == 0) revert LiquidationAmountOutZero();

    uint256 _liquidableYield = _liquidatableBalanceOf(_tokenOut);
    if (_amountOut > _liquidableYield)
      revert LiquidationAmountOutGTYield(_amountOut, _liquidableYield);

    _prizePool.contributePrizeTokens(address(this), _amountIn);

    if (_yieldFeePercentage != 0) {
      _increaseYieldFeeBalance(
        (_amountOut * FEE_PRECISION) / (FEE_PRECISION - _yieldFeePercentage) - _amountOut
      );
    }

    uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));

    if (_vaultAssets != 0 && _amountOut >= _vaultAssets) {
      _yieldVault.deposit(_vaultAssets, address(this));
    }

    _mint(_account, _amountOut);

    return true;
  }

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L550-L587

## [G-08] Use assembly to write address storage values

Using assembly to write address storage values can be an effective way to optimize gas costs in Solidity contracts.

```solidity

299      function setDrawManager(address _drawManager) external {
         if (drawManager != address(0)) {
           revert DrawManagerAlreadySet();
         }
         drawManager = _drawManager;
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L299-L303

## [G-09] Use hardcode address instead address(this)

Using hardcoded addresses instead of address(this) can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, address(this) is used to refer to the address of the current contract. However, calling address(this) requires the EVM to execute a CALL opcode, which can be expensive in terms of gas cost.
By hardcoding the contract's address in the contract code, the gas cost of executing a CALL opcode can be avoided. This is because the EVM can simply use the hardcoded address instead of executing a CALL opcode to retrieve the contract's address.


```solidity
335      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L335

```solidity
468      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L468

```solidity
502      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L502

```solidity
562      if (_tokenOut != address(this))
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L562

```solidity
563      revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L563

```solidity
570      _prizePool.contributePrizeTokens(address(this), _amountIn);
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L570

```solidity
578      uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L578

```solidity
581     _yieldVault.deposit(_vaultAssets, address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L581

```solidity
801      return _yieldVault.maxWithdraw(address(this)) + super.totalAssets();
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L801

```solidity
809      return _twabController.totalSupply(address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L809


```solidity
830      if (_token != address(this)) revert LiquidationTokenOutNotVaultShare(_token, address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L830


```solidity
932      uint256 _vaultAssets = _asset.balanceOf(address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L932


```solidity
954      address(this),
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L954

```solidity
959      _yieldVault.deposit(_assets, address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L959

```solidity
986      _twabController.delegateOf(address(this), _receiver) != _twabController.SPONSORSHIP_ADDRESS()
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L986

```solidity
1026     _yieldVault.withdraw(_assets, address(this), address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1026


```solidity
1176      uint256 _withdrawableAssets = _yieldVault.maxWithdraw(address(this));

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1176



```solidity

83      emit NewFactoryVault(_vault, VaultFactory(address(this)));

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L83

## [G-10] abi.encode() is less efficient than abi.encodePacked()


```solidity

110    return uint256(keccak256(abi.encode(_user, _tier, _prizeIndex, _winningRandomNumber)));

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L110

## [G-11] Using XOR (^) and OR (|) bitwise equivalents

Using XOR (^) and OR (|) bitwise equivalents can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, the XOR and OR bitwise operators can be expensive in terms of gas cost, particularly for large values. This is because these operators require a lot of computational steps to perform the bitwise operation.

```solidity

545     (_to == address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) ||   
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L545

```solidity
575     (_from == address(0) && _toDelegate != SPONSORSHIP_ADDRESS) ||
 
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#575



## [G-12] Don’t apply the same value to state variables



Not applying the same value to state variables can be an effective way to optimize gas costs in Solidity contracts.

```solidity

369          claimCount = 0;
             canaryClaimCount = 0;
             largestTierClaimed = 0;

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L369-L371





## [G‑13] Using private rather than public for constants, saves gas


```solidity

24     address public constant SPONSORSHIP_ADDRESS = address(1);


```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L24



## [G‑14] Using calldata instead of memory for read-only arguments in external functions saves gas

Using calldata instead of memory for read-only arguments in external functions can be an effective way to optimize gas costs in Solidity contracts.
When an external function is called, its arguments are initially stored in calldata, which is a read-only data location. In contrast, memory is a data location where the function can read and write data.

```solidity


653     function setHooks(VaultHooks memory hooks) external

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L653

```solidity


57        string memory _name,
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L57

```solidity
58       string memory _symbol,
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#58






## [G-15] Avoid emitting storage values


Avoiding emitting storage values can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, emitting events with storage values can be expensive in terms of gas cost, particularly if the storage values are large or complex data types. This is because emitting an event with storage values requires the EVM to perform a copy operation to memory, which can be costly in terms of gas.

```solidity

375   emit DrawClosed(
      lastClosedDrawId,
      winningRandomNumber_,
      _numTiers,
      _nextNumberOfTiers,
      _reserve,
      prizeTokenPerShare,
      _lastClosedDrawStartedAt
    );

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L375-L383


```solidity

692      emit ObservationRecorded(
        _vault,
        _user,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L692-L699

```solidity
735         emit ObservationRecorded(
        _vault,
        _user,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L735-L742

```solidity
777      emit TotalSupplyObservationRecorded(
        _vault,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L777-L783

```solidity
811      emit TotalSupplyObservationRecorded(
        _vault,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L811-L817


```solidity

1111     emit RecordedExchangeRate(_lastRecordedExchangeRate);


```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1111


## [G-16] Use assembly to perform efficient back-to-back calls

Using assembly to perform efficient back-to-back calls can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, when a contract calls another contract, it typically uses the CALL opcode. However, using CALL can be expensive in terms of gas cost, particularly if the contract frequently makes back-to-back calls to the same contract.

```solidity

665       function setLiquidationPair(
    LiquidationPair liquidationPair_
  ) external onlyOwner returns (address) {
    if (address(liquidationPair_) == address(0)) revert LPZeroAddress();

    IERC20 _asset = IERC20(asset());
    address _previousLiquidationPair = address(_liquidationPair);

    if (_previousLiquidationPair != address(0)) {
      _asset.safeApprove(_previousLiquidationPair, 0);
    }

    _asset.safeApprove(address(liquidationPair_), type(uint256).max);

    _liquidationPair = liquidationPair_;


```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L665-L679



```solidity

169        function _computeMaxFee(uint8 _tier, uint8 _numTiers) internal view returns (uint256) {
       uint8 _canaryTier = _numTiers - 1;
    if (_tier != _canaryTier) {
      // canary tier
      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1));
    } else {
      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
    }


```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L189-L176



## [G-17] Multiple accesses of a mapping/array should use a storage pointer

Using a storage pointer when making multiple accesses to a mapping or array can be an effective way to optimize gas costs in Solidity contracts.

```solidity

483       function withdrawClaimRewards(address _to, uint256 _amount) external {
       uint256 _available = claimerRewards[msg.sender];

     if (_amount > _available) {
      revert InsufficientRewardsError(_amount, _available);
    }

    claimerRewards[msg.sender] -= _amount;
    _transfer(_to, _amount)

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L483-L491



## [G-18] Cache state variables outside of loop to avoid reading storage on every iteration


Caching state variables outside of a loop can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, reading state variables from storage can be expensive in terms of gas cost, particularly if the state variable is large or complex. This is because reading from storage requires loading data from the storage to memory, which can be slow and costly.

```solidity

361        for (uint8 i = start; i < end; i++) {
      _tiers[i] = Tier({
        drawId: closedDrawId,
        prizeTokenPerShare: prizeTokenPerShare,
        prizeSize: uint96(
          _computePrizeSize(i, _nextNumberOfTiers, _prizeTokenPerShare, newPrizeTokenPerShare)
        )
      });

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L361-L368


## [G-19] Duplicated require()/revert() checks should be refactored to a modifier or function.

Refactoring duplicated require() or revert() checks to a modifier or function can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, require() and revert() are used to validate input and state conditions and to revert the transaction if the conditions are not met. However, repeating the same require() or revert() check multiple times in a contract can be expensive in terms of gas cost, particularly if the check is complex or involves accessing storage.

```solidity

320         if (_numberOfTiers < MINIMUM_NUMBER_OF_TIERS) {
               revert NumberOfTiersLessThanMinimum(_numberOfTiers);
             }
             if (_numberOfTiers > MAXIMUM_NUMBER_OF_TIERS) {
               revert NumberOfTiersGreaterThanMaximum(_numberOfTiers);
             }
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L320-L325

