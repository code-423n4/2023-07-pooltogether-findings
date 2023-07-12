# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 56 at 2023-07-pooltogether/claimer/src/libraries/LinearVRGDALib.sol:

        targetTimeUnwrapped > 0 && decayConstantUnwrapped > type(int256).max / targetTimeUnwrapped


Found in line 62 at 2023-07-pooltogether/claimer/src/libraries/LinearVRGDALib.sol:

        targetTimeUnwrapped < 0 && decayConstantUnwrapped > type(int256).min / targetTimeUnwrapped


Found in line 270 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      observationDrawIdBeforeOrAtStart > 0 &&


Found in line 271 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      firstObservationDrawIdOccurringAtOrAfterStart > 0 &&


Found in line 287 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      firstObservationDrawIdOccurringAtOrAfterStart > 0 &&

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] Use shift Right/Left instead of division/multiplication if possible
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 72 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

      currentIndex = (leftSide + rightSide) / 2;


Found in line 443 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      currentIndex = (leftSide + rightSide) / 2;

------------------------------------------------------------------------ 

### Mitigation 

Use shift Right/Left instead of division/multiplication if possible.










# VULN 3 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 68 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    for (uint i = 0; i < winners.length; i++) {


Found in line 144 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    for (uint i = 0; i < _claimCount; i++) {


Found in line 618 at 2023-07-pooltogether/vault/src/Vault.sol:

    for (uint w = 0; w < _winners.length; w++) {


Found in line 620 at 2023-07-pooltogether/vault/src/Vault.sol:

      for (uint p = 0; p < prizeIndicesLength; p++) {


Found in line 441 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

      canaryClaimCount++;


Found in line 443 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

      claimCount++;


Found in line 361 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    for (uint8 i = start; i < end; i++) {


Found in line 630 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    for (uint8 i = start; i < end; i++) {


Found in line 138 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    for (uint8 i = 0; i < _numberOfTiers; i++) {

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 4 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 68 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    for (uint i = 0; i < winners.length; i++) {


Found in line 144 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    for (uint i = 0; i < _claimCount; i++) {


Found in line 618 at 2023-07-pooltogether/vault/src/Vault.sol:

    for (uint w = 0; w < _winners.length; w++) {


Found in line 620 at 2023-07-pooltogether/vault/src/Vault.sol:

      for (uint p = 0; p < prizeIndicesLength; p++) {


Found in line 137 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint32 count = 0;


Found in line 138 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    for (uint8 i = 0; i < _numberOfTiers; i++) {

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 5 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 653 at 2023-07-pooltogether/vault/src/Vault.sol:

  function setHooks(VaultHooks memory hooks) external {

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 6 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 68 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    for (uint i = 0; i < winners.length; i++) {


Found in line 618 at 2023-07-pooltogether/vault/src/Vault.sol:

    for (uint w = 0; w < _winners.length; w++) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 7 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 266 at 2023-07-pooltogether/vault/src/Vault.sol:

    if (address(twabController_) == address(0)) revert TwabControllerZeroAddress();


Found in line 267 at 2023-07-pooltogether/vault/src/Vault.sol:

    if (address(yieldVault_) == address(0)) revert YieldVaultZeroAddress();


Found in line 268 at 2023-07-pooltogether/vault/src/Vault.sol:

    if (address(prizePool_) == address(0)) revert PrizePoolZeroAddress();


Found in line 269 at 2023-07-pooltogether/vault/src/Vault.sol:

    if (address(owner_) == address(0)) revert OwnerZeroAddress();


Found in line 668 at 2023-07-pooltogether/vault/src/Vault.sol:

    if (address(liquidationPair_) == address(0)) revert LPZeroAddress();


Found in line 538 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

        _to == address(0) ||


Found in line 544 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

          _to == address(0) ? _amount : 0,


Found in line 545 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

          (_to == address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) ||


Found in line 569 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

        _from == address(0) ||


Found in line 574 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

          _from == address(0) ? _amount : 0,


Found in line 575 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

          (_from == address(0) && _toDelegate != SPONSORSHIP_ADDRESS) ||


Found in line 597 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

      if (_userDelegate == address(0)) {


Found in line 624 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

      if (_toDelegate == address(0) || _toDelegate == SPONSORSHIP_ADDRESS) {


Found in line 635 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

      if (_fromDelegate == address(0) || _fromDelegate == SPONSORSHIP_ADDRESS) {

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 8 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 62 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    uint8 tier,


Found in line 64 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    uint32[][] calldata prizeIndices,


Found in line 89 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  function computeTotalFees(uint8 _tier, uint _claimCount) external view returns (uint256) {


Found in line 104 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    uint8 _tier,


Found in line 161 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  function computeMaxFee(uint8 _tier) public view returns (uint256) {


Found in line 169 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  function _computeMaxFee(uint8 _tier, uint8 _numTiers) internal view returns (uint256) {


Found in line 170 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    uint8 _canaryTier = _numTiers - 1;


Found in line 340 at 2023-07-pooltogether/vault/src/Vault.sol:

  function decimals() public view virtual override(ERC4626, ERC20) returns (uint8) {


Found in line 431 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint8 _v,


Found in line 462 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint8 _v,


Found in line 498 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint8 _v,


Found in line 608 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint8 _tier,


Found in line 610 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint32[][] calldata _prizeIndices,


Found in line 1045 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint8 _tier,


Found in line 1046 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint32 _prizeIndex,


Found in line 1099 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint8 _v,


Found in line 24 at 2023-07-pooltogether/vault/src/interfaces/IVaultHooks.sol:

    uint8 tier,


Found in line 25 at 2023-07-pooltogether/vault/src/interfaces/IVaultHooks.sol:

    uint32 prizeIndex


Found in line 36 at 2023-07-pooltogether/vault/src/interfaces/IVaultHooks.sol:

    uint8 tier,


Found in line 37 at 2023-07-pooltogether/vault/src/interfaces/IVaultHooks.sol:

    uint32 prizeIndex,


Found in line 27 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  uint32 public immutable PERIOD_LENGTH;


Found in line 31 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  uint32 public immutable PERIOD_OFFSET;


Found in line 142 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  constructor(uint32 _periodLength, uint32 _periodOffset) {


Found in line 232 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 targetTime


Found in line 244 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  function getTotalSupplyAt(address vault, uint32 targetTime) external view returns (uint256) {


Found in line 261 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 startTime,


Found in line 262 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 endTime


Found in line 285 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 startTime,


Found in line 286 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 endTime


Found in line 309 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  ) external view returns (uint16, ObservationLib.Observation memory) {


Found in line 324 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  ) external view returns (uint16, ObservationLib.Observation memory) {


Found in line 337 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  ) external view returns (uint16, ObservationLib.Observation memory) {


Found in line 350 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  ) external view returns (uint16, ObservationLib.Observation memory) {


Found in line 360 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  function getTimestampPeriod(uint32 time) external view returns (uint32) {


Found in line 373 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  function isTimeSafe(address vault, address user, uint32 time) external view returns (bool) {


Found in line 392 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 startTime,


Found in line 393 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 endTime


Found in line 415 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  function isTotalSupplyTimeSafe(address vault, uint32 time) external view returns (bool) {


Found in line 432 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 startTime,


Found in line 433 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

    uint32 endTime


Found in line 16 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

uint16 constant MAX_CARDINALITY = 365; // 1 year


Found in line 25 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

  using OverflowSafeComparatorLib for uint32;


Found in line 36 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

    uint32 timestamp;


Found in line 57 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

    uint24 _newestObservationIndex,


Found in line 58 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

    uint24 _oldestObservationIndex,


Found in line 59 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

    uint32 _target,


Found in line 60 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

    uint16 _cardinality,


Found in line 61 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

    uint32 _time


Found in line 74 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

      beforeOrAt = _observations[uint16(RingBufferLib.wrap(currentIndex, _cardinality))];


Found in line 75 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

      uint32 beforeOrAtTimestamp = beforeOrAt.timestamp;


Found in line 79 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

        leftSide = uint16(RingBufferLib.nextIndex(leftSide, _cardinality));


Found in line 83 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

      afterOrAt = _observations[uint16(RingBufferLib.nextIndex(currentIndex, _cardinality))];


Found in line 15 at 2023-07-pooltogether/twab-controller/src/libraries/OverflowSafeComparatorLib.sol:

  function lt(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (bool) {


Found in line 31 at 2023-07-pooltogether/twab-controller/src/libraries/OverflowSafeComparatorLib.sol:

  function lte(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (bool) {


Found in line 47 at 2023-07-pooltogether/twab-controller/src/libraries/OverflowSafeComparatorLib.sol:

  function checkedSub(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (uint32) {


Found in line 55 at 2023-07-pooltogether/twab-controller/src/libraries/OverflowSafeComparatorLib.sol:

    return uint32(aAdjusted - bAdjusted);


Found in line 37 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

  using OverflowSafeComparatorLib for uint32;


Found in line 50 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint16 nextObservationIndex;


Found in line 51 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint16 cardinality;


Found in line 76 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_LENGTH,


Found in line 77 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 86 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 currentTime = uint32(block.timestamp);


Found in line 87 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 index;


Found in line 105 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

        accountDetails.nextObservationIndex = uint16(


Found in line 145 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_LENGTH,


Found in line 146 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 168 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 currentTime = uint32(block.timestamp);


Found in line 169 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 index;


Found in line 189 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

        accountDetails.nextObservationIndex = uint16(


Found in line 226 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

  ) internal view returns (uint16 index, ObservationLib.Observation memory observation) {


Found in line 247 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

  ) internal view returns (uint16 index, ObservationLib.Observation memory observation) {


Found in line 248 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    index = uint16(


Found in line 264 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 267 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _targetTime


Found in line 289 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 292 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _startTime,


Found in line 293 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _endTime


Found in line 332 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _time


Found in line 353 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_LENGTH,


Found in line 354 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 360 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    returns (uint16 index, ObservationLib.Observation memory newestObservation, bool isNew)


Found in line 362 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 currentTime = uint32(block.timestamp);


Found in line 363 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint16 newestIndex;


Found in line 371 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 currentPeriod = _getTimestampPeriod(PERIOD_LENGTH, PERIOD_OFFSET, currentTime);


Found in line 372 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 newestObservationPeriod = _getTimestampPeriod(


Found in line 383 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

        uint16(RingBufferLib.wrap(_accountDetails.nextObservationIndex, MAX_CARDINALITY)),


Found in line 401 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _timestamp


Found in line 419 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_LENGTH,


Found in line 420 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 421 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _timestamp


Found in line 422 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

  ) internal pure returns (uint32 period) {


Found in line 436 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_LENGTH,


Found in line 437 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 438 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _timestamp


Found in line 439 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

  ) private pure returns (uint32 period) {


Found in line 457 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 460 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _targetTime


Found in line 474 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 477 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _targetTime


Found in line 479 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 currentTime = uint32(block.timestamp);


Found in line 481 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint16 oldestTwabIndex;


Found in line 482 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint16 newestTwabIndex;


Found in line 545 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 548 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _targetTime


Found in line 562 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 565 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _targetTime


Found in line 567 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 currentTime = uint32(block.timestamp);


Found in line 569 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint16 oldestTwabIndex;


Found in line 594 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

      uint16 newestTwabIndex,


Found in line 628 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 631 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _targetTime


Found in line 664 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_LENGTH,


Found in line 665 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 668 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _time


Found in line 683 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_LENGTH,


Found in line 684 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 687 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _time


Found in line 694 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 period = _getTimestampPeriod(PERIOD_LENGTH, PERIOD_OFFSET, _time);


Found in line 718 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 preOrAtPeriod = _getTimestampPeriod(


Found in line 723 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 postPeriod = _getTimestampPeriod(


Found in line 744 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_LENGTH,


Found in line 745 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 PERIOD_OFFSET,


Found in line 748 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _startTime,


Found in line 749 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    uint32 _endTime


Found in line 28 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint8 tier,


Found in line 29 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint32 prizeIndex,


Found in line 43 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

error DidNotWin(address vault, address winner, uint8 tier, uint32 prizeIndex);


Found in line 76 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

error InvalidPrizeIndex(uint32 invalidPrizeIndex, uint32 prizeCount, uint8 tier);


Found in line 84 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

error InvalidTier(uint8 tier, uint8 numberOfTiers);


Found in line 109 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint32 drawPeriodSeconds;


Found in line 111 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint8 numberOfTiers;


Found in line 112 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint8 tierShares;


Found in line 113 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint8 canaryShares;


Found in line 114 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint8 reserveShares;


Found in line 142 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 drawId,


Found in line 143 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 tier,


Found in line 144 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint32 prizeIndex,


Found in line 159 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 indexed drawId,


Found in line 161 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 numTiers,


Found in line 162 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 nextNumTiers,


Found in line 182 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  event ContributePrizeTokens(address indexed vault, uint16 indexed drawId, uint256 amount);


Found in line 206 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))


Found in line 225 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint32 public immutable drawPeriodSeconds;


Found in line 240 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint32 public claimCount;


Found in line 243 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint32 public canaryClaimCount;


Found in line 246 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint8 public largestTierClaimed;


Found in line 348 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager returns (uint16) {


Found in line 357 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _numTiers = numberOfTiers;


Found in line 358 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _nextNumberOfTiers = _numTiers;


Found in line 409 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _tier,


Found in line 410 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint32 _prizeIndex,


Found in line 421 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    (SD59x18 _vaultPortion, SD59x18 _tierOdds, uint16 _drawDuration) = _computeVaultTierDetails(


Found in line 521 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _startDrawIdInclusive,


Found in line 522 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _endDrawIdInclusive


Found in line 537 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _startDrawIdInclusive,


Found in line 538 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _endDrawIdInclusive


Found in line 551 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  function getTierAccrualDurationInDraws(uint8 _tier) external view returns (uint16) {


Found in line 553 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

      uint16(TierCalculationLib.estimatePrizeFrequencyInDraws(_tierOdds(_tier, numberOfTiers)));


Found in line 608 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _numTiers = numberOfTiers;


Found in line 609 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _nextNumberOfTiers = _numTiers;


Found in line 638 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _tier,


Found in line 639 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint32 _prizeIndex


Found in line 663 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _tier,


Found in line 664 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint32 _prizeIndex


Found in line 666 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    (SD59x18 vaultPortion, SD59x18 tierOdds, uint16 drawDuration) = _computeVaultTierDetails(


Found in line 681 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _tier


Found in line 683 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _numberOfTiers = numberOfTiers;


Found in line 725 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _startDrawId,


Found in line 726 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _endDrawId


Found in line 735 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  function nextNumberOfTiers() external view returns (uint8) {


Found in line 753 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  function _checkValidTier(uint8 _tier, uint8 _numTiers) internal pure {


Found in line 781 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  function _computeNextNumberOfTiers(uint8 _numTiers) internal view returns (uint8) {


Found in line 784 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _nextNumberOfTiers = largestTierClaimed + 2; // canary tier, then length


Found in line 815 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  function _contributionsForDraw(uint16 _drawId) internal view returns (uint256) {


Found in line 845 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _tier,


Found in line 846 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint32 _prizeIndex,


Found in line 849 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _drawDuration


Found in line 851 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _numberOfTiers = numberOfTiers;


Found in line 852 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint32 tierPrizeCount = _getTierPrizeCount(_tier, _numberOfTiers);


Found in line 892 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _tier,


Found in line 893 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint8 _numberOfTiers,


Found in line 894 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _lastClosedDrawId


Found in line 895 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  ) internal view returns (SD59x18 vaultPortion, SD59x18 tierOdds, uint16 drawDuration) {


Found in line 902 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    drawDuration = uint16(TierCalculationLib.estimatePrizeFrequencyInDraws(tierOdds));


Found in line 905 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

      uint16(drawDuration > _lastClosedDrawId ? 0 : _lastClosedDrawId - drawDuration + 1),


Found in line 925 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint32 _endTimestamp = uint32(_lastClosedDrawStartedAt + drawPeriodSeconds);


Found in line 926 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint32 _startTimestamp = uint32(_endTimestamp - _drawDuration * drawPeriodSeconds);


Found in line 947 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _startDrawId,


Found in line 948 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint16 _endDrawId,


Found in line 15 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint16 drawId;


Found in line 22 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

error NumberOfTiersLessThanMinimum(uint8 numTiers);


Found in line 26 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

error NumberOfTiersGreaterThanMaximum(uint8 numTiers);


Found in line 45 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 internal constant MINIMUM_NUMBER_OF_TIERS = 3;


Found in line 46 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 internal constant MAXIMUM_NUMBER_OF_TIERS = 15;


Found in line 66 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint16 internal constant GRAND_PRIZE_PERIOD_DRAWS = 365;


Found in line 69 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_2_TIERS = 4;


Found in line 70 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_3_TIERS = 16;


Found in line 71 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_4_TIERS = 66;


Found in line 72 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_5_TIERS = 270;


Found in line 73 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_6_TIERS = 1108;


Found in line 74 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_7_TIERS = 4517;


Found in line 75 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_8_TIERS = 18358;


Found in line 76 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_9_TIERS = 74435;


Found in line 77 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_10_TIERS = 301239;


Found in line 78 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_11_TIERS = 1217266;


Found in line 79 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_12_TIERS = 4912619;


Found in line 80 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_13_TIERS = 19805536;


Found in line 81 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint32 internal constant ESTIMATED_PRIZES_PER_DRAW_FOR_14_TIERS = 79777187;


Found in line 205 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  mapping(uint8 => Tier) internal _tiers;


Found in line 208 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 public immutable tierShares;


Found in line 211 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 public immutable canaryShares;


Found in line 214 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 public immutable reserveShares;


Found in line 220 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 public numberOfTiers;


Found in line 223 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint16 internal lastClosedDrawId;


Found in line 235 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  constructor(uint8 _numberOfTiers, uint8 _tierShares, uint8 _canaryShares, uint8 _reserveShares) {


Found in line 331 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _nextDraw(uint8 _nextNumberOfTiers, uint96 _prizeTokenLiquidity) internal {


Found in line 336 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 numTiers = numberOfTiers;


Found in line 339 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

      uint16 closedDrawId,


Found in line 350 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 start;


Found in line 351 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 end;


Found in line 361 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    for (uint8 i = start; i < end; i++) {


Found in line 385 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _numberOfTiers,


Found in line 386 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _nextNumberOfTiers,


Found in line 388 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  ) internal view returns (uint16 closedDrawId, uint104 newReserve, UD60x18 newPrizeTokenPerShare) {


Found in line 407 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _numberOfTiers,


Found in line 408 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _nextNumberOfTiers,


Found in line 411 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  ) internal view returns (uint16 closedDrawId, uint104 newReserve, UD60x18 newPrizeTokenPerShare) {


Found in line 437 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function getTierPrizeSize(uint8 _tier) external view returns (uint96) {


Found in line 444 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function getTierPrizeCount(uint8 _tier) external view returns (uint32) {


Found in line 452 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function getTierPrizeCount(uint8 _tier, uint8 _numberOfTiers) external view returns (uint32) {


Found in line 460 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _getTierPrizeCount(uint8 _tier, uint8 _numberOfTiers) internal view returns (uint32) {


Found in line 464 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

        : uint32(TierCalculationLib.prizeCount(_tier));


Found in line 471 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _getTier(uint8 _tier, uint8 _numberOfTiers) internal view returns (Tier memory) {


Found in line 473 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint16 _lastClosedDrawId = lastClosedDrawId;


Found in line 497 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _getTotalShares(uint8 _numberOfTiers) internal view returns (uint256) {


Found in line 509 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _computeShares(uint8 _tier, uint8 _numTiers) internal view returns (uint8) {


Found in line 520 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _tier,


Found in line 523 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _shares = _computeShares(_tier, numberOfTiers);


Found in line 558 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _tier,


Found in line 559 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _numberOfTiers,


Found in line 594 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _shares


Found in line 604 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _isCanaryTier(uint8 _tier, uint8 _numberOfTiers) internal pure returns (bool) {


Found in line 613 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _numberOfTiers,


Found in line 614 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _nextNumberOfTiers,


Found in line 619 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 start;


Found in line 620 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 end;


Found in line 630 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    for (uint8 i = start; i < end; i++) {


Found in line 632 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

      uint8 shares = _computeShares(i, _numberOfTiers);


Found in line 646 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function getTierRemainingLiquidity(uint8 _tier) external view returns (uint256) {


Found in line 647 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    uint8 _numTiers = numberOfTiers;


Found in line 677 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function getOpenDrawId() external view returns (uint16) {


Found in line 683 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function estimatedPrizeCount() external view returns (uint32) {


Found in line 690 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function estimatedPrizeCount(uint8 numTiers) external pure returns (uint32) {


Found in line 697 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function canaryPrizeCountFractional(uint8 numTiers) external view returns (UD60x18) {


Found in line 703 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function canaryPrizeCount() external view returns (uint32) {


Found in line 710 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _canaryPrizeCount(uint8 _numberOfTiers) internal view returns (uint32) {


Found in line 711 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    return uint32(fromUD60x18(_canaryPrizeCountFractional(_numberOfTiers).floor()));


Found in line 717 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function canaryPrizeCount(uint8 _numTiers) external view returns (uint32) {


Found in line 730 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _estimatedPrizeCount(uint8 numTiers) internal pure returns (uint32) {


Found in line 764 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _canaryPrizeCountFractional(uint8 numTiers) internal view returns (UD60x18) {


Found in line 799 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function getTierOdds(uint8 _tier, uint8 _numTiers) external pure returns (SD59x18) {


Found in line 807 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  function _tierOdds(uint8 _tier, uint8 _numTiers) internal pure returns (SD59x18) {


Found in line 18 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint8 _tier,


Found in line 19 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint8 _numberOfTiers,


Found in line 20 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint16 _grandPrizePeriod


Found in line 39 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

  function prizeCount(uint8 _tier) internal pure returns (uint256) {


Found in line 53 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint8 _numberOfTiers,


Found in line 54 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint8 _canaryShares,


Found in line 55 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint8 _reserveShares,


Found in line 56 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint8 _tierShares


Found in line 106 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint8 _tier,


Found in line 107 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint32 _prizeIndex,


Found in line 134 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint8 _numberOfTiers,


Found in line 135 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint16 _grandPrizePeriod


Found in line 136 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

  ) internal pure returns (uint32) {


Found in line 137 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    uint32 count = 0;


Found in line 138 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    for (uint8 i = 0; i < _numberOfTiers; i++) {


Found in line 139 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

      count += uint32(


Found in line 14 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

error DrawClosed(uint16 drawId, uint16 newestDrawId);


Found in line 19 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

error InvalidDrawRange(uint16 startDrawId, uint16 endDrawId);


Found in line 23 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

error InvalidDisbursedEndDrawId(uint16 endDrawId);


Found in line 37 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

  uint24 internal constant MAX_CARDINALITY = 366;


Found in line 41 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 nextIndex;


Found in line 42 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 cardinality;


Found in line 48 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16[366] drawRingBuffer;


Found in line 54 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 first;


Found in line 55 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 second;


Found in line 67 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _drawId,


Found in line 76 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 newestDrawId_ = accumulator.drawRingBuffer[newestIndex];


Found in line 95 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      uint16 nextIndex = uint16(RingBufferLib.nextIndex(ringBufferInfo.nextIndex, MAX_CARDINALITY));


Found in line 96 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      uint16 cardinality = ringBufferInfo.cardinality;


Found in line 122 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _startDrawId,


Found in line 130 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 newestDrawId_ = accumulator.drawRingBuffer[newestIndex];


Found in line 168 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _startDrawId,


Found in line 169 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _endDrawId,


Found in line 228 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 lastObservationDrawIdOccurringAtOrBeforeEnd;


Found in line 235 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

        uint16(RingBufferLib.offset(indexes.second, 1, ringBufferInfo.cardinality))


Found in line 239 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 observationDrawIdBeforeOrAtStart;


Found in line 240 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 firstObservationDrawIdOccurringAtOrAfterStart;


Found in line 277 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      uint16 headStartDrawId = _startDrawId - observationDrawIdBeforeOrAtStart;


Found in line 278 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      uint16 headEndDrawId = headStartDrawId +


Found in line 317 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _startDrawId,


Found in line 318 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _endDrawId,


Found in line 319 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _lastObservationDrawIdOccurringAtOrBeforeEnd,


Found in line 325 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 tailRangeStartDrawId = (


Found in line 347 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

        first: uint16(


Found in line 354 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

        second: uint16(


Found in line 370 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

        first: uint16(accumulator.drawRingBuffer[indices.first]),


Found in line 371 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

        second: uint16(accumulator.drawRingBuffer[indices.second])


Found in line 421 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16[366] storage _drawRingBuffer,


Found in line 422 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _oldestIndex,


Found in line 423 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _newestIndex,


Found in line 424 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _cardinality,


Found in line 425 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 _targetLastClosedDrawId


Found in line 430 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      uint16 beforeOrAtIndex,


Found in line 431 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      uint16 beforeOrAtDrawId,


Found in line 432 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      uint16 afterOrAtIndex,


Found in line 433 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      uint16 afterOrAtDrawId


Found in line 436 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 leftSide = _oldestIndex;


Found in line 437 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 rightSide = _newestIndex < leftSide ? leftSide + _cardinality - 1 : _newestIndex;


Found in line 438 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    uint16 currentIndex;


Found in line 445 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      beforeOrAtIndex = uint16(RingBufferLib.wrap(currentIndex, _cardinality));


Found in line 448 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      afterOrAtIndex = uint16(RingBufferLib.nextIndex(currentIndex, _cardinality));

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 9 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 340 at 2023-07-pooltogether/vault/src/Vault.sol:

  function decimals() public view virtual override(ERC4626, ERC20) returns (uint8) {


Found in line 345 at 2023-07-pooltogether/vault/src/Vault.sol:

  function totalAssets() public view virtual override returns (uint256) {


Found in line 350 at 2023-07-pooltogether/vault/src/Vault.sol:

  function totalSupply() public view virtual override(ERC20, IERC20) returns (uint256) {


Found in line 375 at 2023-07-pooltogether/vault/src/Vault.sol:

  function maxDeposit(address) public view virtual override returns (uint256) {


Found in line 383 at 2023-07-pooltogether/vault/src/Vault.sol:

  function maxMint(address) public view virtual override returns (uint256) {


Found in line 407 at 2023-07-pooltogether/vault/src/Vault.sol:

  function deposit(uint256 _assets, address _receiver) public virtual override returns (uint256) {


Found in line 440 at 2023-07-pooltogether/vault/src/Vault.sol:

  function mint(uint256 _shares, address _receiver) public virtual override returns (uint256) {


Found in line 540 at 2023-07-pooltogether/vault/src/Vault.sol:

  function liquidatableBalanceOf(address _token) public view override returns (uint256) {

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 10 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 69 at 2023-07-pooltogether/claimer/src/Claimer.sol:

      claimCount += prizeIndices[i].length;


Found in line 145 at 2023-07-pooltogether/claimer/src/Claimer.sol:

      fee += _computeFeeForNextClaim(


Found in line 621 at 2023-07-pooltogether/vault/src/Vault.sol:

        totalPrizes += _claimPrize(


Found in line 853 at 2023-07-pooltogether/vault/src/Vault.sol:

    _yieldFeeTotalSupply += _shares;


Found in line 91 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    accountDetails.balance += _amount;


Found in line 92 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    accountDetails.delegateBalance += _delegateAmount;


Found in line 115 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

          accountDetails.cardinality += 1;


Found in line 199 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

          accountDetails.cardinality += 1;


Found in line 455 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

      claimerRewards[_feeRecipient] += _fee;


Found in line 499 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    _reserve += _amount;


Found in line 770 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

      _nextExpectedEndTime +=


Found in line 831 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    _totalWithdrawn += _amount;


Found in line 374 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

    _reserve += newReserve;


Found in line 139 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

      count += uint32(


Found in line 98 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

        cardinality += 1;


Found in line 281 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      total += amount;


Found in line 295 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

      total += amount;


Found in line 298 at 2023-07-pooltogether/prize-poool/src/libraries/DrawAccumulatorLib.sol:

    total += _computeTail(

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 11 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 336 at 2023-07-pooltogether/vault/src/Vault.sol:

    return _twabController.balanceOf(address(this), _account);


Found in line 435 at 2023-07-pooltogether/vault/src/Vault.sol:

    _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);


Found in line 468 at 2023-07-pooltogether/vault/src/Vault.sol:

    _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);


Found in line 502 at 2023-07-pooltogether/vault/src/Vault.sol:

    _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);


Found in line 562 at 2023-07-pooltogether/vault/src/Vault.sol:

    if (_tokenOut != address(this))


Found in line 563 at 2023-07-pooltogether/vault/src/Vault.sol:

      revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));


Found in line 570 at 2023-07-pooltogether/vault/src/Vault.sol:

    _prizePool.contributePrizeTokens(address(this), _amountIn);


Found in line 578 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));


Found in line 581 at 2023-07-pooltogether/vault/src/Vault.sol:

      _yieldVault.deposit(_vaultAssets, address(this));


Found in line 801 at 2023-07-pooltogether/vault/src/Vault.sol:

    return _yieldVault.maxWithdraw(address(this)) + super.totalAssets();


Found in line 809 at 2023-07-pooltogether/vault/src/Vault.sol:

    return _twabController.totalSupply(address(this));


Found in line 830 at 2023-07-pooltogether/vault/src/Vault.sol:

    if (_token != address(this)) revert LiquidationTokenOutNotVaultShare(_token, address(this));


Found in line 932 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint256 _vaultAssets = _asset.balanceOf(address(this));


Found in line 954 at 2023-07-pooltogether/vault/src/Vault.sol:

        address(this),


Found in line 959 at 2023-07-pooltogether/vault/src/Vault.sol:

    _yieldVault.deposit(_assets, address(this));


Found in line 986 at 2023-07-pooltogether/vault/src/Vault.sol:

      _twabController.delegateOf(address(this), _receiver) != _twabController.SPONSORSHIP_ADDRESS()


Found in line 1026 at 2023-07-pooltogether/vault/src/Vault.sol:

    _yieldVault.withdraw(_assets, address(this), address(this));


Found in line 1176 at 2023-07-pooltogether/vault/src/Vault.sol:

    uint256 _withdrawableAssets = _yieldVault.maxWithdraw(address(this));


Found in line 83 at 2023-07-pooltogether/vault/src/VaultFactory.sol:

    emit NewFactoryVault(_vault, VaultFactory(address(this)));


Found in line 312 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    uint256 _deltaBalance = prizeToken.balanceOf(address(this)) - _accountedBalance();


Found in line 500 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

    prizeToken.safeTransferFrom(msg.sender, address(this), _amount);

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 12 

## [GAS] Functions guaranteed to revert when called by normal users can be marked payable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 641 at 2023-07-pooltogether/vault/src/Vault.sol:

  function setClaimer(address claimer_) external onlyOwner returns (address) {


Found in line 691 at 2023-07-pooltogether/vault/src/Vault.sol:

  function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {


Found in line 704 at 2023-07-pooltogether/vault/src/Vault.sol:

  function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {

------------------------------------------------------------------------ 

### Mitigation 

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.










# VULN 13 

## [GAS] Do not calculate constants
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 16 at 2023-07-pooltogether/twab-controller/src/libraries/ObservationLib.sol:

uint16 constant MAX_CARDINALITY = 365; // 1 year

------------------------------------------------------------------------ 

### Mitigation 

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.










# VULN 14 

## [GAS] Using private rather than public for constants, saves gas
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 27 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  uint256 public immutable minimumFee;


Found in line 29 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  uint256 public immutable timeToReachMaxFee;

------------------------------------------------------------------------ 

### Mitigation 

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.










# VULN 15 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 210 at 2023-07-pooltogether/claimer/src/Claimer.sol:

    return fee > _maxFee ? _maxFee : fee;


Found in line 315 at 2023-07-pooltogether/vault/src/Vault.sol:

    return _sharesToAssets > _assets ? 0 : _assets - _sharesToAssets;


Found in line 14 at 2023-07-pooltogether/twab-controller/src/libraries/OverflowSafeComparatorLib.sol:

  /// @return bool Whether `_a` is chronologically < `_b`.


Found in line 22 at 2023-07-pooltogether/twab-controller/src/libraries/OverflowSafeComparatorLib.sol:

    return aAdjusted < bAdjusted;


Found in line 95 at 2023-07-pooltogether/prize-poool/src/libraries/TierCalculationLib.sol:

    return constrainedRandomNumber < winningZone;

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 16 

## [GAS] Use assembly to write address storage values
------------------------------------------------------------------------ 

### Proof of concept 

Found in lines 254 to 265 at 2023-07-pooltogether/vault/src/Vault.sol:

  constructor(
    IERC20 asset_,
    string memory name_,
    string memory symbol_,
    TwabController twabController_,
    IERC4626 yieldVault_,
    PrizePool prizePool_,
    address claimer_,
    address yieldFeeRecipient_,
    uint256 yieldFeePercentage_,
    address owner_
  ) ERC4626(asset_) ERC20(name_, symbol_) ERC20Permit(name_) Ownable(owner_) {

------------------------------------------------------------------------ 

### Mitigation 

Use assembly to write address storage values. Here are a few reasons: * Reduced opcode usage: When using assembly, you can directly manipulate storage values using lower-level instructions like sstore (storage store) instead of relying on higher-level Solidity storage assignments. These direct operations typically result in fewer opcode executions, reducing gas costs. * Avoiding unnecessary checks: Solidity storage assignments often involve additional checks and operations, such as enforcing security modifiers or triggering events. By using assembly, you can bypass these additional checks and perform the necessary storage operations directly, resulting in gas savings. * Optimized packing: Assembly provides greater flexibility in packing and unpacking data structures. By carefully arranging and manipulating the storage layout in assembly, you can achieve more efficient storage utilization and minimize wasted storage space. * Fine-grained control: Assembly allows for precise control over gas-consuming operations. You can optimize gas usage by selecting specific instructions and minimizing unnecessary operations or data copying.
