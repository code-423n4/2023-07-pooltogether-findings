## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Cache state variables outside of loop to avoid reading storage on every iteration | 1 | - |
| [G-02] | Using private rather than public for constants, saves gas | 1 | - |
| [G-03] | Use constants instead of type(uintx).max | 11 | - |
| [G-04] | Use hardcode address instead address(this) | 19 | - |
| [G-05] | Don’t apply the same value to state variables | 5 | - |
| [G-06] | Avoid emitting storage values | 7 | - |
| [G-07] | Use assembly to write address storage values | 1 | - |
| [G-08] | Amounts should be checked for 0 before calling a transfer | 2 | - |
| [G-09] | Multiple accesses of a mapping/array should use a storage pointer | 1 | - |
| [G-10] | 2**<N> should be re-written as type(uint<N>).max | 7 | - |
| [G-11] | Using XOR (^) and OR (|) bitwise equivalents | 2 | - |
| [G-12] | IF’s/require() statements that check input arguments should be at the top of the function | 4 | - |
| [G-13] | Use assembly to perform efficient back-to-back calls | 2 | - |
| [G-14] | Using a positive conditional flow to save a NOT opcode | 14 | - |


## Gas Optimizations  

## [G-1] Cache state variables outside of loop to avoid reading storage on every iteration


Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration.

```solidity
file: src/abstract/TieredLiquidityDistributor.sol

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


## [G‑2] Using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

```solidity
file: src/TwabController.sol

24     address public constant SPONSORSHIP_ADDRESS = address(1);


```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L24


## [G-3] Use constants instead of type(uintx).max

Certainly! When working with maximum values in Solidity, it's more gas-efficient to use constants instead of the expression type(uintX).max, where X represents the number of bits in the unsigned integer type. This optimization can help reduce the gas costs associated with storing and manipulating large numbers.


```solidity
file: src/libraries/LinearVRGDALib.sol

56     targetTimeUnwrapped > 0 && decayConstantUnwrapped > type(int256).max / targetTimeUnwrapped

58     return type(uint256).max;

70     return type(uint256).max;

82     if (targetPriceInt > type(int256).max / expResult)

83     return type(uint256).max;

89     if (targetPriceInt > type(int256).max / extraPrecisionExpResult)

90     return type(uint256).max;

```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L56

```solidity
file: src/Vault.sol

282     asset_.safeApprove(address(yieldVault_), type(uint256).max);

376     return _isVaultCollateralized() ? type(uint96).max : 0;

384     return _isVaultCollateralized() ? type(uint96).max : 0;

677     _asset.safeApprove(address(liquidationPair_), type(uint256).max);

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L282


## [G-4] Use hardcode address instead address(this)

To save gas, you can hardcode the contract's address as a constant variable within the contract. By doing so, you eliminate the need for a function call to retrieve the contract's address, resulting in gas savings.


```solidity
file: src/Vault.sol

336       return _twabController.balanceOf(address(this), _account);

335      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

468      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

502      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

562      if (_tokenOut != address(this))

563      revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));

570      _prizePool.contributePrizeTokens(address(this), _amountIn);

578      uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));

581     _yieldVault.deposit(_vaultAssets, address(this));

801      return _yieldVault.maxWithdraw(address(this)) + super.totalAssets();

809      return _twabController.totalSupply(address(this));

830      if (_token != address(this)) revert LiquidationTokenOutNotVaultShare(_token, address(this));

932      uint256 _vaultAssets = _asset.balanceOf(address(this));

954      address(this),

959      _yieldVault.deposit(_assets, address(this));

986      _twabController.delegateOf(address(this), _receiver) != _twabController.SPONSORSHIP_ADDRESS()

1026     _yieldVault.withdraw(_assets, address(this), address(this));

1176      uint256 _withdrawableAssets = _yieldVault.maxWithdraw(address(this));

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L336

```solidity
file: src/VaultFactory.sol

83      emit NewFactoryVault(_vault, VaultFactory(address(this)));

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L83

## [G-5] Don’t apply the same value to state variables

If claimCount is already 0 , it’ll save 2100 gas to not set it to that value again

```solidity
file: src/PrizePool.sol

369          claimCount = 0;
             canaryClaimCount = 0;
             largestTierClaimed = 0;

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L369-L371


## [G-6] Avoid emitting storage values

Caching of a state variable replaces each Gwarmaccess (130 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```solidity
file: src/PrizePool.sol

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
file: src/TwabController.sol

692      emit ObservationRecorded(
        _vault,
        _user,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

735         emit ObservationRecorded(
        _vault,
        _user,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

777      emit TotalSupplyObservationRecorded(
        _vault,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

811      emit TotalSupplyObservationRecorded(
        _vault,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L692-L699


```solidity
file: src/Vault.sol

1111     emit RecordedExchangeRate(_lastRecordedExchangeRate);

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1111




## [G-7] Use assembly to write address storage values

Using assembly in Solidity allows you to directly access and manipulate the low-level operations of the Ethereum Virtual Machine (EVM). By leveraging assembly, you can optimize gas usage in certain scenarios, such as when reading or writing values to storage slots of an address.

```solidity
file: src/PrizePool.sol

299      function setDrawManager(address _drawManager) external {
         if (drawManager != address(0)) {
           revert DrawManagerAlreadySet();
         }
         drawManager = _drawManager;

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L299-L303


## [G-8] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

```solidity
file: src/PrizePool.sol

832      prizeToken.safeTransfer(_to, _amount);

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L832

```solidity
file: src/Vault.sol

1126      emit Transfer(address(0), _receiver, _shares);

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1126

## [G-9] Multiple accesses of a mapping/array should use a storage pointer

```solidity
file: src/PrizePool.sol

483       function withdrawClaimRewards(address _to, uint256 _amount) external {
         uint256 _available = claimerRewards[msg.sender];

         if (_amount > _available) {
         revert InsufficientRewardsError(_amount, _available);
    }

       claimerRewards[msg.sender] -= _amount;
       (_to, _amount)
 
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L483-L491


## [G-10] 2**<N> should be re-written as type(uint<N>).max

Earlier versions of solidity can use uint<n>(-1) instead. Expressions not including the - 1 can often be re-written to accomodate the change (e.g. by using a > rather than a >=, which will also save some gas)

```solidity
file: src/libraries/TierCalculationLib.sol

40     uint256 _numberOfPrizes = 4 ** _tier;

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L40

```solidity
file: src/libraries/OverflowSafeComparatorLib.sol

19        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
20        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

35        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
36        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

52        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
53        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L19

## [G-11] Using XOR (^) and OR (|) bitwise equivalents

Estimated savings: 73 gas
Max savings according to yarn profile: 282 gas

On Remix, given only uint256 types, the following are logical equivalents, but don’t cost the same amount of gas:

(a != b || c != d || e != f) costs 571
((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas)

```solidity
file: src/TwabController.sol

545     (_to == address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) ||   

575     (_from == address(0) && _toDelegate != SPONSORSHIP_ADDRESS) ||
 
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L545

## [G-12] IF’s/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.



```solidity
file: src/libraries/TwabLib.sol

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
file: src/abstract/TieredLiquidityDistributor.sol

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
file: src/libraries/TwabLib.sol

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
file: src/Vault.sol

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


## [G-13] Use assembly to perform efficient back-to-back calls


```solidity
file: src/Vault.sol

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
file: src/Claimer.sol

169        function _computeMaxFee(uint8 _tier, uint8 _numTiers) internal view returns (uint256) {
              uint8 _canaryTier = _numTiers - 1;
           if (_tier != _canaryTier) {
             // canary tier
             return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1));
           } else {
             return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
           }


```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L169-L176


## [G-14] Using a positive conditional flow to save a NOT opcode

Estimated savings: 3 gas
Max savings according to yarn profile: 150 gas

The following function either revert or returns some value. To save some gas (NOT opcode costing 3 gas), switch to a positive statement:


```solidity
file: src/PrizePool.sol

428         if (
      !_isWinner(msg.sender, _winner, _tier, _prizeIndex, _vaultPortion, _tierOdds, _drawDuration)


```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L428

```solidity
file: src/libraries/DrawAccumulatorLib.sol

459      if (!targetAtOrAfter)

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L459

```solidity
file: src/TwabController.sol

530   if (!_isFromDelegate && _fromDelegate != SPONSORSHIP_ADDRESS)

561   if (!_isToDelegate && _toDelegate != SPONSORSHIP_ADDRESS)

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L530

```solidity
file: src/libraries/ObservationLib.sol

93      if (!targetAfterOrAt)

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol#L93

```solidity
file: src/Vault.sol

1200     if (!_isVaultCollateralized()) revert VaultUnderCollateralized();

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1200
