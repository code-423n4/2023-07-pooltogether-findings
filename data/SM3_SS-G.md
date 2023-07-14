
## Summary for GAS findings


|  NO  |   issue   |  instence  |
|------|-----------|------------|
|[G-01]|  Avoid emitting storage values  | 6 |
|[G-02]|  Multiple accesses of a mapping/array should use a storage pointer| 1 |
|[G-03]|  Use assembly to perform efficient back-to-back calls | 2 |
|[G-04]|  IF’s/require() statements that check input arguments should be at the top of the function | 4 |
|[G-05]|  2**<n> should be re-written as type(uint<n>).max | 7 |
|[G-06]|  Cache state variables outside of loop to avoid reading storage on every iteration | 1 |
|[G-07]|  Using private rather than public for constants, saves gas | 1 |
|[G-08]|  Don’t apply the same value to state variables | 1 |
|[G-09]|  Use hardcode address instead address(this) | 19 |
|[G-10]|  Amounts should be checked for 0 before calling a transfer  | 2 |
|[G-11]|  Using XOR (^) and OR  bitwise equivalents  | 2 |
|[G-12]|  Using a positive conditional flow to save a NOT opcode | 6 |
|[G-13]|  Use constants instead of type(uintx).max | 12 |
|[G-14]|  Use assembly to write address storage values | 1 |
|[G-15]|  Use do while loops instead of for loops| 4 |
|[G-16]|  Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)| 3 |


## [G-01] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (130 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.


#### Cache _lastClosedDrawStartedAt  and emit cached value instead of reading from storage

```solidity

file:  src/PrizePool.sol

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


#### Cache _account.details.balance, and _account.details.delegateBalance, variable result  and emit cached value instead of reading from storage


```solidity
file:

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

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L735-L742

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L777-L783

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L811-L817

#### Cache _lastRecordedExchangeRate and emit cached value instead of reading from storage

```solidity
file:   src/Vault.sol

1111     emit RecordedExchangeRate(_lastRecordedExchangeRate);


```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1111


## [G-02] Multiple accesses of a mapping/array should use a storage pointer

Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

#### Cache storage pointers for claimerRewards[msg.sender]
```solidity
file:    src/PrizePool.sol

483       function withdrawClaimRewards(address _to, uint256 _amount) external {
       uint256 _available = claimerRewards[msg.sender];

     if (_amount > _available) {
      revert InsufficientRewardsError(_amount, _available);
    }

    claimerRewards[msg.sender] -= _amount;
    _transfer(_to, _amount)

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L483-L491



## [G-03] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.



### use assembly for safeApprove function 
```solidity
file:    src/Vault.sol

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


### use assembly for getTierPrizeSize function 

```solidity
file:      src/Claimer.sol

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



## [G-04] IF’s/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.



```solidity
file:    src/libraries/TwabLib.sol

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

### use they if statment before uint32 currentTime = uint32(block.timestamp),uint16 oldestTwabIndex and this uint16 newestTwabIndex variables like this 


```solidity

 function _getPreviousOrAtObservation(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) private view returns (ObservationLib.Observation memory prevOrAtObservation) {

      // If there are no observations, return a zeroed observation
    if (_accountDetails.cardinality == 0) {
      return
        ObservationLib.Observation({ cumulativeBalance: 0, balance: 0, timestamp: PERIOD_OFFSET });
    }

      uint32 currentTime = uint32(block.timestamp);

    uint16 oldestTwabIndex;
    uint16 newestTwabIndex;

```


### use they if statment before UD60x18 reclaimedLiquidity; variable 

```solidity
file:

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

### Recommanded code 

```solidity

    function _getTierLiquidityToReclaim(
    uint8 _numberOfTiers,
    uint8 _nextNumberOfTiers,
    UD60x18 _prizeTokenPerShare
  ) internal view returns (uint256) {
   
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
     UD60x18 reclaimedLiquidity;
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
### use they if statment of they beganing of they function  

```solidity
file:     src/libraries/TwabLib.sol

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


### Recommanded code

```solidity

  function _getNextOrNewestObservation(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) private view returns (ObservationLib.Observation memory nextOrNewestObservation) {

      // If there are no observations, return a zeroed observation
    if (_accountDetails.cardinality == 0) {
      return
        ObservationLib.Observation({ cumulativeBalance: 0, balance: 0, timestamp: PERIOD_OFFSET });
    }
        uint32 currentTime = uint32(block.timestamp);

       uint16 oldestTwabIndex;
```

### use they if statments of they beganing of they function  

```solidity
file:

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


## [G-05] 2**<n> should be re-written as type(uint<n>).max
Earlier versions of solidity can use uint<n>(-1) instead. Expressions not including the - 1 can often be re-written to accomodate the change (e.g. by using a > rather than a >=, which will also save some gas)

```solidity
file:   src/libraries/TierCalculationLib.sol

40     uint256 _numberOfPrizes = 4 ** _tier;

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L40

```solidity
file:    src/libraries/OverflowSafeComparatorLib.sol

19        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
20        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

35        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
36        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

52        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
53        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L19

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L20

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L35

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L36

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L52

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L53




## [G-06] Cache state variables outside of loop to avoid reading storage on every iteration


Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration.


there is a loop that iterates from start to end. Inside the loop, a new Tier struct is created, and the prizeTokenPerShare value is assigned. By caching the prizeTokenPerShare value outside the loop, you can avoid reading it from storage on each iteration, resulting in gas savings.

Without caching, on each iteration of the loop, the contract would need to perform a storage read operation to fetch the prizeTokenPerShare value from storage. This read operation incurs gas costs. However, by caching the value before entering the loop, you eliminate the need for these repeated storage reads, reducing the overall gas consumption.


```solidity

file:    src/abstract/TieredLiquidityDistributor.sol

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

To implement this optimization, you can assign the value of prizeTokenPerShare to a local variable before entering the loop, like this:

```solidity

uint256 prizeTokenPerShare = _tiers[i].prizeTokenPerShare;

for (uint8 i = start; i < end; i++) {
  _tiers[i] = Tier({
    drawId: closedDrawId,
    prizeTokenPerShare: prizeTokenPerShare,
    prizeSize: uint96(
      _computePrizeSize(i, _nextNumberOfTiers, _prizeTokenPerShare, newPrizeTokenPerShare)
    )
  });
}


```



## [G‑07] Using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

```solidity
file:  src/TwabController.sol

24     address public constant SPONSORSHIP_ADDRESS = address(1);


```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L24




## [G-08] Don’t apply the same value to state variables



If claimCount is already 0 , it’ll save 2100 gas to not set it to that value again

```solidity
file:      src/PrizePool.sol

369          claimCount = 0;
             canaryClaimCount = 0;
             largestTierClaimed = 0;

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L369-L371


## [G-09] Use hardcode address instead address(this)

To save gas, you can hardcode the contract's address as a constant variable within the contract. By doing so, you eliminate the need for a function call to retrieve the contract's address, resulting in gas savings

Here's an example to illustrate the concept:

```solidity

contract MyContract {
    address public constant CONTRACT_ADDRESS = 0x1234567890ABCDEF;  // Hardcoded contract address

    function doSomething() public {
        // Use CONTRACT_ADDRESS directly instead of address(this)
        // ...
    }
}

```


```solidity
file:     src/Vault.sol

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
file:   src/VaultFactory.sol

83      emit NewFactoryVault(_vault, VaultFactory(address(this)));

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L83






## [G-10] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

```solidity
file:    src/PrizePool.sol

832      prizeToken.safeTransfer(_to, _amount);

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L832

```solidity
file:     src/Vault.sol

1126      emit Transfer(address(0), _receiver, _shares);

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1126



## [G-11] Using XOR (^) and OR (|) bitwise equivalents

Estimated savings: 73 gas
Max savings according to yarn profile: 282 gas

On Remix, given only uint256 types, the following are logical equivalents, but don’t cost the same amount of gas:

(a != b || c != d || e != f) costs 571
((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas)

```solidity

file:   src/TwabController.sol

545     (_to == address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) ||   

575     (_from == address(0) && _toDelegate != SPONSORSHIP_ADDRESS) ||
 
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L545

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#575


## [G-12] Using a positive conditional flow to save a NOT opcode

Estimated savings: 3 gas
Max savings according to yarn profile: 150 gas

The following function either revert or returns some value. To save some gas (NOT opcode costing 3 gas), switch to a positive statement:


```solidity

file:    src/PrizePool.sol

428         if (
      !_isWinner(msg.sender, _winner, _tier, _prizeIndex, _vaultPortion, _tierOdds, _drawDuration)


```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L428

```solidity
file:     src/libraries/DrawAccumulatorLib.sol

459      if (!targetAtOrAfter)

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L459

```solidity
file:  src/TwabController.sol

530   if (!_isFromDelegate && _fromDelegate != SPONSORSHIP_ADDRESS)

561   if (!_isToDelegate && _toDelegate != SPONSORSHIP_ADDRESS)


```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L530

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L561

```solidity
file:   src/libraries/ObservationLib.sol

93      if (!targetAfterOrAt)

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol#L93

```solidity
file:    src/Vault.sol

1200     if (!_isVaultCollateralized()) revert VaultUnderCollateralized();

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1200


## [G-13] Use constants instead of type(uintx).max

Certainly! When working with maximum values in Solidity, it's more gas-efficient to use constants instead of the expression type(uintX).max, where X represents the number of bits in the unsigned integer type. This optimization can help reduce the gas costs associated with storing and manipulating large numbers.

Here's an example that demonstrates the use of constants instead of type(uintX).max:

```solidity
pragma solidity ^0.8.18;

contract GasOptimizationExample {
    uint256 constant MAX_VALUE = type(uint256).max;
    
    function process(uint256 value) public {
        require(value < MAX_VALUE, "Value exceeds maximum limit");
        
        // Perform some operations with the value
        // ...
    }
}
```

In the above code, we define a constant MAX_VALUE and assign it the value type(uint256).max. This constant represents the maximum value that can be stored in a uint256 variable.


```solidity
file:  src/libraries/LinearVRGDALib.sol

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
file:   src/Vault.sol

282     asset_.safeApprove(address(yieldVault_), type(uint256).max);

376     return _isVaultCollateralized() ? type(uint96).max : 0;

384     return _isVaultCollateralized() ? type(uint96).max : 0;

677     _asset.safeApprove(address(liquidationPair_), type(uint256).max);

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L282

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L376

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L384

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L677


## [G-14] Use assembly to write address storage values

Using assembly in Solidity allows you to directly access and manipulate the low-level operations of the Ethereum Virtual Machine (EVM). By leveraging assembly, you can optimize gas usage in certain scenarios, such as when reading or writing values to storage slots of an address.

Here's an example that demonstrates how to use assembly to write address storage values and save gas:

```solidity

pragma solidity ^0.8.0;

contract GasOptimizationExample {
    mapping(address => uint256) private balances;
    
    function updateBalance(address account, uint256 newBalance) public {
        // Update the balance using assembly
        assembly {
            // Load the storage slot of the balances mapping for the given account
            let storageSlot := balances.slot
            
            // Write the new balance to the storage slot
            sstore(storageSlot, newBalance)
        }
    }
}

```
In the above code, we have a simple GasOptimizationExample contract with a balances mapping that stores balances for different addresses. The updateBalance function allows us to update the balance of a specific account.


```solidity
file:    src/PrizePool.sol

299      function setDrawManager(address _drawManager) external {
         if (drawManager != address(0)) {
           revert DrawManagerAlreadySet();
         }
         drawManager = _drawManager;
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L299-L303


## [G-15] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.


|before| 345590 |
|after | 344100 |
```solidity
file:    src/Claimer.sol

68        for (uint i = 0; i < winners.length; i++) {
          claimCount += prizeIndices[i].length;
    }

```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68-L70
 
### recommanded code 

```solidity

uint i = 0;
do {
  claimCount += prizeIndices[i].length;
  i++;
} while (i < winners.length);


```

```solidity

file:  src/abstract/TieredLiquidityDistributor.sol

361    for (uint8 i = start; i < end; i++) {
      _tiers[i] = Tier({
        drawId: closedDrawId,
        prizeTokenPerShare: prizeTokenPerShare,
        prizeSize: uint96(
          _computePrizeSize(i, _nextNumberOfTiers, _prizeTokenPerShare, newPrizeTokenPerShare)
        )
      });
    }

630        for (uint8 i = start; i < end; i++) {
      Tier memory tierLiquidity = _tiers[i];
      uint8 shares = _computeShares(i, _numberOfTiers);
      UD60x18 liq = _getTierRemainingLiquidity(
        shares,
        fromUD34x4toUD60x18(tierLiquidity.prizeTokenPerShare),
        _prizeTokenPerShare
      );
      reclaimedLiquidity = reclaimedLiquidity.add(liq);
    }

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L361-L369

```solidity
file:    src/Vault.sol

618        for (uint w = 0; w < _winners.length; w++) {
      uint prizeIndicesLength = _prizeIndices[w].length;
      for (uint p = 0; p < prizeIndicesLength; p++) {
        totalPrizes += _claimPrize(
          _winners[w],
          _tier,
          _prizeIndices[w][p],
          _feePerClaim,
          _feeRecipient
        );
      }
    }


```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L618-L629




## [G-16] Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)

By changing the pattern of assigning value to the structure, gas savings of ~130 per instance are achieved. In addition, this use will provide significant savings in distribution costs.

there is example use like this 


```solidity

         parameter = Parameter({optionFactory: optionsFactory, token0: token0, token1: token1});
         parameter.optionFactory = optionFactory;
         parameter.token0 = token0;
         parameter.token1 = token1;

```

```solidity

file:   src/abstract/TieredLiquidityDistributor.sol

362     _tiers[i] = Tier({
        drawId: closedDrawId,
        prizeTokenPerShare: prizeTokenPerShare,
        prizeSize: uint96(
          _computePrizeSize(i, _nextNumberOfTiers, _prizeTokenPerShare, newPrizeTokenPerShare)
        )
      });


```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L362-L368

```solidity
file:     src/libraries/TwabLib.sol

119     observation = ObservationLib.Observation({
        balance: uint96(accountDetails.delegateBalance),
        cumulativeBalance: _extrapolateFromBalance(newestObservation, currentTime),
        timestamp: currentTime
      });

203     observation = ObservationLib.Observation({
        balance: uint96(accountDetails.delegateBalance),
        cumulativeBalance: _extrapolateFromBalance(newestObservation, currentTime),
        timestamp: currentTime
      });

  


```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#119-L123

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#203-L207



