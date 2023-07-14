# Gas Optimization

# Summary

| Number | Optimization Details                                                                                                                                          | Context |
| :----: | :------------------------------------------------------------------------------------------------------------------------------------------------------------ | :-----: |
| [G-01] | Avoid contract existence checks by using low level calls                                                                                                      |    1    |
| [G-02] | TwabController.sol::SPONSORSHIP_ADDRESS change the visibility to private to save some gas                                                                     |    1    |
| [G-03] | PrizePool.sol::withdrawReserve and PrizePool.sol::closeDraw Functions revert when called by normal users This can be marked payable to save some gas          |    2    |
| [G-04] | Multipel Address/Id mappings can be combined into a single mapping of an Address                                                                              |    3    |
| [G-05] | Before some functions, we should check some variables for possible gas save                                                                                   |    5    |
| [G-06] | Duplicated if() checks should be refactored to a modifier or function                                                                                         |    7    |
| [G-07] | Replace type(uintx).max or type(intx).max with constant value                                                                                                 |   12    |
| [G-08] | Use assembly to write address storage values                                                                                                                  |   14    |
| [G-09] | PrizePool.sol::claimPrize Multiple accesses of a `claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex]` should use a local variable cache |    4    |
| [G-10] | Use Modifiers Instead of `PrizePool.sol::_checkValidTier` and `Vault.sol::_requireVaultCollateralized` functions To Save Gas                                  |    2    |
| [G-11] | abi.encode() is less efficient than abi.encodePacked()                                                                                                        |    1    |
| [G-12] | Using bools for storage incurs overhead                                                                                                                       |    4    |
| [G-13] | Do not calculate constant only pass the value                                                                                                                 |   117   |
| [G-14] | Use do while  loops instead of for loops                                                                                                                      |    5    |
| [G-15] | Missing zero-address check in constructor                                                                                                                     |    2    |
| [G-16] | Ternary operation is cheaper than if-else statement                                                                                                           |    2    |
| [G-17] | Unnecessary casting as variable is already of the same type                                                                                                   |   23    |
| [G-18] | Use hardcode address instead address(this)                                                                                                                    |   21    |
| [G-19] | use mappings instead of arrays                                                                                                                                |    1    |
| [G-20] | Use assembly to emit events                                                                                                                                   |   39    |
| [G-21] | Using delete statement can save gas                                                                                                                           |    4    |
| [G-22] | Access mappings directly rather than using accessor functions                                                                                                 |    4    |
| [G-23] | Using a positive conditional flow to save a NOT opcode                                                                                                        |    6    |
| [G-24] | `2**<n>` should be re-written as `type(uint<n>).max`                                                                                                          |    7    |
| [G-25] | Internal/Private functions only called once can be inlined to save gas                                                                                        |   16    |
| [G-26] | Use assembly to perform efficient back-to-back calls                                                                                                          |    1    |

## [G-01] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L578

```solidity
File: src/Vault.sol

578    uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));

```

## [G-02] TwabController.sol::SPONSORSHIP_ADDRESS change the visibility to private to save some gas

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L24

```diff
File: src/TwabController.sol

-24     address public constant SPONSORSHIP_ADDRESS = address(1);

+       address private constant SPONSORSHIP_ADDRESS = address(1);
```

## [G-03] PrizePool.sol::withdrawReserve and PrizePool.sol::closeDraw Functions revert when called by normal users This can be marked payable to save some gas

The onlyDrawManager modifier makes a function revert if not called by the address registered as the Manager

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

```diff
File: src/PrizePool.sol

-335    function withdrawReserve(address _to, uint104 _amount) external onlyDrawManager {
+       function withdrawReserve(address _to, uint104 _amount) external payable onlyDrawManager {


-348    function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager returns (uint16) {
+       function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager payable returns (uint16) {

```

## [G-04] Multipel Address/Id mappings can be combined into a single mapping of an Address/Id to a struct

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L206

```solidity
File: src/PrizePool.sol

206  mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))

```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L36

```solidity
File:

36  mapping(address => mapping(address => TwabLib.Account)) internal userObservations;


42  mapping(address => mapping(address => address)) internal delegates;

```

## [G-05] Before some functions call, we should check some variables for possible gas save

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L458

```diff
File: src/TwabController.sol

- 458    _transferBalance(msg.sender, address(0), _to, _amount);
+ if (_amount != 0) { _transferBalance(msg.sender, address(0), _to, _amount); }



-470    _transferBalance(msg.sender, _from, address(0), _amount);
+ if (_amount != 0) { _transferBalance(msg.sender, address(0), _to, _amount);



-482    _transferBalance(msg.sender, _from, _to, _amount);
+ if (_amount != 0) { _transferBalance(msg.sender, address(0), _to, _amount);



-492    _delegate(_vault, msg.sender, _to);
+ if (_amount != 0) { _transferBalance(msg.sender, address(0), _to, _amount);
```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1139

```solidity
File: src/Vault.sol

1139    _twabController.burn(_owner, uint96(_shares));

```

## [G-06] Duplicated if() checks should be refactored to a modifier or function

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol

```solidity
File: src/libraries/DrawAccumulatorLib.sol

126    if (ringBufferInfo.cardinality == 0) {
178    if (ringBufferInfo.cardinality == 0) {

```

```diff
diff --git a/DrawAccumulatorLib.org.sol b/DrawAccumulatorLib.sol
index f0e2336..a8f844f 100644
--- a/DrawAccumulatorLib.org.sol
+++ b/DrawAccumulatorLib.sol
@@ -1,12 +1,14 @@
+ modifier requireCardinalityZero(RingBufferInfo memory info) {
+    require(info.cardinality == 0, "Cardinality must be zero");
+    _;
+}
+
  function getTotalRemaining(
     Accumulator storage accumulator,
     uint16 _startDrawId,
     SD59x18 _alpha
-  ) internal view returns (uint256) {
+  ) internal view requireCardinalityZero returns (uint256) {
     RingBufferInfo memory ringBufferInfo = accumulator.ringBufferInfo;
-    if (ringBufferInfo.cardinality == 0) {
-      return 0;
-    }
     uint256 newestIndex = RingBufferLib.newestIndex(ringBufferInfo.nextIndex, MAX_CARDINALITY);
     uint16 newestDrawId_ = accumulator.drawRingBuffer[newestIndex];
     if (_startDrawId < newestDrawId_) {
@@ -49,16 +51,13 @@
     uint16 _startDrawId,
     uint16 _endDrawId,
     SD59x18 _alpha
-  ) internal view returns (uint256) {
+  ) internal view requireCardinalityZero returns (uint256) {
     if (_startDrawId > _endDrawId) {
       revert InvalidDrawRange(_startDrawId, _endDrawId);
     }

     RingBufferInfo memory ringBufferInfo = _accumulator.ringBufferInfo;

-    if (ringBufferInfo.cardinality == 0) {
-      return 0;
-    }

     Pair32 memory indexes = computeIndices(ringBufferInfo);
     Pair32 memory drawIds = readDrawIds(_accumulator, indexes);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

```solidity
File: src/PrizePool.sol

360    if (lastClosedDrawId != 0) {
611    if (lastClosedDrawId != 0) {

```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol

```solidity
File:

95    if (isObservationRecorded) {
179    if (isObservationRecorded) {

103      if (isNew) {
187      if (isNew) {


485    if (_accountDetails.cardinality == 0) {
572    if (_accountDetails.cardinality == 0) {


691    if (_isObservationRecorded) {
734    if (_isObservationRecorded) {


776    if (_isObservationRecorded) {
810    if (_isObservationRecorded) {



```

## [G-07] Replace type(uintx).max or type(intx).max with constant value

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol

```diff
File: src/libraries/LinearVRGDALib.sol

+   uint256 constant MAX_UINT256 = type(int256).max;
+   uint256 constant MIN_UINT256 = type(int256).min;
+   int256 constant MAX_INT256 = type(int256).max;



-56        targetTimeUnwrapped > 0 && decayConstantUnwrapped > type(int256).max / targetTimeUnwrapped
+        targetTimeUnwrapped > 0 && decayConstantUnwrapped > MAX_INT256 / targetTimeUnwrapped

-58        return type(uint256).max;
+          return MAX_UINT256;

-62        targetTimeUnwrapped < 0 && decayConstantUnwrapped > type(int256).min / targetTimeUnwrapped
+          targetTimeUnwrapped < 0 && decayConstantUnwrapped > MIN_UINT256 / targetTimeUnwrapped

-70        return type(uint256).max;
+          return MAX_UINT256;


-82        if (targetPriceInt > type(int256).max / expResult) {
+          if (targetPriceInt > MAX_INT256 / expResult) {


-83        return type(uint256).max;
+            return MAX_UINT256;



-89        if (targetPriceInt > type(int256).max / extraPrecisionExpResult) {
+        if (targetPriceInt > MAX_UINT256 / extraPrecisionExpResult) {


-90          return type(uint256).max;
+            return MAX_UINT256;

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol

```diff
File:  src/Vault.sol

+     uint256 constant MAX_UINT256 = type(int256).max;
+     uint96 constant MAX_UINT96 = type(uint96).max;



-282    asset_.safeApprove(address(yieldVault_), type(uint256).max);
+       asset_.safeApprove(address(yieldVault_), MAX_UINT256);



-376    return _isVaultCollateralized() ? type(uint96).max : 0;
+       return _isVaultCollateralized() ? MAX_UINT96 : 0;


-384    return _isVaultCollateralized() ? type(uint96).max : 0;
+       return _isVaultCollateralized() ? MAX_UINT96 : 0;


-677    _asset.safeApprove(address(liquidationPair_), type(uint256).max);
+       _asset.safeApprove(address(liquidationPair_), MAX_UINT256);


```

## [G-08] Use assembly to write address storage values

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

```solidity
File: src/PrizePool.sol

271    prizeToken = params.prizeToken;

272    twabController = params.twabController;

273    smoothing = params.smoothing;

274    claimExpansionThreshold = params.claimExpansionThreshold;

278    drawManager = params.drawManager;


303    drawManager = _drawManager;

```

```diff
diff --git a/PrizePool.org.sol b/PrizePool.sol
index b0ac8f8..29c4edc 100644
--- a/PrizePool.org.sol
+++ b/PrizePool.sol
@@ -11,12 +11,18 @@
     if (unwrap(params.smoothing) >= unwrap(UNIT)) {
       revert SmoothingGTEOne(unwrap(params.smoothing));
     }
-    prizeToken = params.prizeToken;
-    twabController = params.twabController;
-    smoothing = params.smoothing;
-    claimExpansionThreshold = params.claimExpansionThreshold;
+    assembly {
+        sstore(prizeToken.slot, params.prizeToken)
+        sstore(twabController.slot, params.twabController)
+        sstore(smoothing.slot, params.smoothing)
+        sstore(claimExpansionThreshold.slot, params.claimExpansionThreshold)
+        sstore(drawManager.slot,params.drawManager)
    }

     drawPeriodSeconds = params.drawPeriodSeconds;
     _lastClosedDrawStartedAt = params.firstDrawStartsAt;

-    drawManager = params.drawManager;
    if (params.drawManager != address(0)) {

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol

```solidity
File: src/Vault.sol

271    _twabController = twabController_;

272    _yieldVault = yieldVault_;


273    _prizePool = prizePool_;

679    _liquidationPair = liquidationPair_;

1210    _claimer = claimer_;

1230    _yieldFeeRecipient = yieldFeeRecipient_;

```

## [G-09] PrizePool.sol::claimPrize Multiple accesses of a `claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex]` should use a local variable cache

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L434

```solidity
File: src/PrizePool.sol

434    if (claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex]) {
438    claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex] = true;


484    uint256 _available = claimerRewards[msg.sender];
490    claimerRewards[msg.sender] -= _amount;

```

```diff
diff --git a/PrizePool.org.sol b/PrizePool.sol
index 5209f4e..8dcfad7 100644
--- a/PrizePool.org.sol
+++ b/PrizePoolt.sol
@@ -7,6 +7,8 @@
     address _feeRecipient
   ) external returns (uint256) {
     Tier memory tierLiquidity = _getTier(_tier, numberOfTiers);

+    bool prizeClaimed = claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex];

     if (_fee > tierLiquidity.prizeSize) {
       revert FeeTooLarge(_fee, tierLiquidity.prizeSize);
@@ -25,11 +27,11 @@
       revert DidNotWin(msg.sender, _winner, _tier, _prizeIndex);
     }

-    if (claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex]) {
+    if (prizeClaimed) {
       revert AlreadyClaimedPrize(msg.sender, _winner, _tier, _prizeIndex, _prizeRecipient);
     }

-    claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex] = true;
+    prizeClaimed = true;

     if (_isCanaryTier(_tier, numberOfTiers)) {
       canaryClaimCount++;
```

## [G-10] Use Modifiers Instead of `PrizePool.sol::_checkValidTier` and `Vault.sol::_requireVaultCollateralized` functions To Save Gas

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L753-L758

```solidity
File: src/PrizePool.sol

753  function _checkValidTier(uint8 _tier, uint8 _numTiers) internal pure {
754    if (_tier >= _numTiers) {
755      revert InvalidTier(_tier, _numTiers);
756    }
757  }

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1199-L1201

```solidity
File: src/Vault.sol

1199  function _requireVaultCollateralized() internal view {
1200    if (!_isVaultCollateralized()) revert VaultUnderCollateralized();
1201  }
```

## [G-11] abi.encode() is less efficient than abi.encodePacked()

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L110

```solidity
File: src/libraries/TierCalculationLib.sol

110    return uint256(keccak256(abi.encode(_user, _tier, _prizeIndex, _winningRandomNumber)));

```

## [G-12] Using bools for storage incurs overhead

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L206

```solidity
File: src/PrizePool.sol

206  mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/interfaces/IVaultHooks.sol#L6

```solidity
File: src/interfaces/IVaultHooks.sol

6  bool useBeforeClaimPrize;

8  bool useAfterClaimPrize;

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L36

```solidity
File:

36  mapping(Vault => bool) public deployedVaults;

```

## [G-13] Do not calculate constant only pass the value

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol

```solidity
File: src/abstract/TieredLiquidityDistributor.sol

  SD59x18 internal constant TIER_ODDS_0_3 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_3 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_2_3 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_4 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_4 = SD59x18.wrap(19579642462506911);
  SD59x18 internal constant TIER_ODDS_2_4 = SD59x18.wrap(139927275620255366);
  SD59x18 internal constant TIER_ODDS_3_4 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_5 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_5 = SD59x18.wrap(11975133168707466);
  SD59x18 internal constant TIER_ODDS_2_5 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_3_5 = SD59x18.wrap(228784597949733865);
  SD59x18 internal constant TIER_ODDS_4_5 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_6 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_6 = SD59x18.wrap(8915910667410451);
  SD59x18 internal constant TIER_ODDS_2_6 = SD59x18.wrap(29015114005673871);
  SD59x18 internal constant TIER_ODDS_3_6 = SD59x18.wrap(94424100034951094);
  SD59x18 internal constant TIER_ODDS_4_6 = SD59x18.wrap(307285046878222004);
  SD59x18 internal constant TIER_ODDS_5_6 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_7 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_7 = SD59x18.wrap(7324128348251604);
  SD59x18 internal constant TIER_ODDS_2_7 = SD59x18.wrap(19579642462506911);
  SD59x18 internal constant TIER_ODDS_3_7 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_4_7 = SD59x18.wrap(139927275620255366);
  SD59x18 internal constant TIER_ODDS_5_7 = SD59x18.wrap(374068544013333694);
  SD59x18 internal constant TIER_ODDS_6_7 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_8 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_8 = SD59x18.wrap(6364275529026907);
  SD59x18 internal constant TIER_ODDS_2_8 = SD59x18.wrap(14783961098420314);
  SD59x18 internal constant TIER_ODDS_3_8 = SD59x18.wrap(34342558671878193);
  SD59x18 internal constant TIER_ODDS_4_8 = SD59x18.wrap(79776409602255901);
  SD59x18 internal constant TIER_ODDS_5_8 = SD59x18.wrap(185317453770221528);
  SD59x18 internal constant TIER_ODDS_6_8 = SD59x18.wrap(430485137687959592);
  SD59x18 internal constant TIER_ODDS_7_8 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_9 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_9 = SD59x18.wrap(5727877794074876);
  SD59x18 internal constant TIER_ODDS_2_9 = SD59x18.wrap(11975133168707466);
  SD59x18 internal constant TIER_ODDS_3_9 = SD59x18.wrap(25036116265717087);
  SD59x18 internal constant TIER_ODDS_4_9 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_5_9 = SD59x18.wrap(109430951602859902);
  SD59x18 internal constant TIER_ODDS_6_9 = SD59x18.wrap(228784597949733865);
  SD59x18 internal constant TIER_ODDS_7_9 = SD59x18.wrap(478314329651259628);
  SD59x18 internal constant TIER_ODDS_8_9 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_10 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_10 = SD59x18.wrap(5277233889074595);
  SD59x18 internal constant TIER_ODDS_2_10 = SD59x18.wrap(10164957094799045);
  SD59x18 internal constant TIER_ODDS_3_10 = SD59x18.wrap(19579642462506911);
  SD59x18 internal constant TIER_ODDS_4_10 = SD59x18.wrap(37714118749773489);
  SD59x18 internal constant TIER_ODDS_5_10 = SD59x18.wrap(72644572330454226);
  SD59x18 internal constant TIER_ODDS_6_10 = SD59x18.wrap(139927275620255366);
  SD59x18 internal constant TIER_ODDS_7_10 = SD59x18.wrap(269526570731818992);
  SD59x18 internal constant TIER_ODDS_8_10 = SD59x18.wrap(519159484871285957);
  SD59x18 internal constant TIER_ODDS_9_10 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_11 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_11 = SD59x18.wrap(4942383282734483);
  SD59x18 internal constant TIER_ODDS_2_11 = SD59x18.wrap(8915910667410451);
  SD59x18 internal constant TIER_ODDS_3_11 = SD59x18.wrap(16084034459031666);
  SD59x18 internal constant TIER_ODDS_4_11 = SD59x18.wrap(29015114005673871);
  SD59x18 internal constant TIER_ODDS_5_11 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_6_11 = SD59x18.wrap(94424100034951094);
  SD59x18 internal constant TIER_ODDS_7_11 = SD59x18.wrap(170338234127496669);
  SD59x18 internal constant TIER_ODDS_8_11 = SD59x18.wrap(307285046878222004);
  SD59x18 internal constant TIER_ODDS_9_11 = SD59x18.wrap(554332974734700411);
  SD59x18 internal constant TIER_ODDS_10_11 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_12 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_12 = SD59x18.wrap(4684280039134314);
  SD59x18 internal constant TIER_ODDS_2_12 = SD59x18.wrap(8009005012036743);
  SD59x18 internal constant TIER_ODDS_3_12 = SD59x18.wrap(13693494143591795);
  SD59x18 internal constant TIER_ODDS_4_12 = SD59x18.wrap(23412618868232833);
  SD59x18 internal constant TIER_ODDS_5_12 = SD59x18.wrap(40030011078337707);
  SD59x18 internal constant TIER_ODDS_6_12 = SD59x18.wrap(68441800379112721);
  SD59x18 internal constant TIER_ODDS_7_12 = SD59x18.wrap(117019204165776974);
  SD59x18 internal constant TIER_ODDS_8_12 = SD59x18.wrap(200075013628233217);
  SD59x18 internal constant TIER_ODDS_9_12 = SD59x18.wrap(342080698323914461);
  SD59x18 internal constant TIER_ODDS_10_12 = SD59x18.wrap(584876652230121477);
  SD59x18 internal constant TIER_ODDS_11_12 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_13 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_13 = SD59x18.wrap(4479520628784180);
  SD59x18 internal constant TIER_ODDS_2_13 = SD59x18.wrap(7324128348251604);
  SD59x18 internal constant TIER_ODDS_3_13 = SD59x18.wrap(11975133168707466);
  SD59x18 internal constant TIER_ODDS_4_13 = SD59x18.wrap(19579642462506911);
  SD59x18 internal constant TIER_ODDS_5_13 = SD59x18.wrap(32013205494981721);
  SD59x18 internal constant TIER_ODDS_6_13 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_7_13 = SD59x18.wrap(85581121447732876);
  SD59x18 internal constant TIER_ODDS_8_13 = SD59x18.wrap(139927275620255366);
  SD59x18 internal constant TIER_ODDS_9_13 = SD59x18.wrap(228784597949733866);
  SD59x18 internal constant TIER_ODDS_10_13 = SD59x18.wrap(374068544013333694);
  SD59x18 internal constant TIER_ODDS_11_13 = SD59x18.wrap(611611432212751966);
  SD59x18 internal constant TIER_ODDS_12_13 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_14 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_14 = SD59x18.wrap(4313269422986724);
  SD59x18 internal constant TIER_ODDS_2_14 = SD59x18.wrap(6790566987074365);
  SD59x18 internal constant TIER_ODDS_3_14 = SD59x18.wrap(10690683906783196);
  SD59x18 internal constant TIER_ODDS_4_14 = SD59x18.wrap(16830807002169641);
  SD59x18 internal constant TIER_ODDS_5_14 = SD59x18.wrap(26497468900426949);
  SD59x18 internal constant TIER_ODDS_6_14 = SD59x18.wrap(41716113674084931);
  SD59x18 internal constant TIER_ODDS_7_14 = SD59x18.wrap(65675485708038160);
  SD59x18 internal constant TIER_ODDS_8_14 = SD59x18.wrap(103395763485663166);
  SD59x18 internal constant TIER_ODDS_9_14 = SD59x18.wrap(162780431564813557);
  SD59x18 internal constant TIER_ODDS_10_14 = SD59x18.wrap(256272288217119098);
  SD59x18 internal constant TIER_ODDS_11_14 = SD59x18.wrap(403460570024895441);
  SD59x18 internal constant TIER_ODDS_12_14 = SD59x18.wrap(635185461125249183);
  SD59x18 internal constant TIER_ODDS_13_14 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_15 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_15 = SD59x18.wrap(4175688124417637);
  SD59x18 internal constant TIER_ODDS_2_15 = SD59x18.wrap(6364275529026907);
  SD59x18 internal constant TIER_ODDS_3_15 = SD59x18.wrap(9699958857683993);
  SD59x18 internal constant TIER_ODDS_4_15 = SD59x18.wrap(14783961098420314);
  SD59x18 internal constant TIER_ODDS_5_15 = SD59x18.wrap(22532621938542004);
  SD59x18 internal constant TIER_ODDS_6_15 = SD59x18.wrap(34342558671878193);
  SD59x18 internal constant TIER_ODDS_7_15 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_8_15 = SD59x18.wrap(79776409602255901);
  SD59x18 internal constant TIER_ODDS_9_15 = SD59x18.wrap(121589313257458259);
  SD59x18 internal constant TIER_ODDS_10_15 = SD59x18.wrap(185317453770221528);
  SD59x18 internal constant TIER_ODDS_11_15 = SD59x18.wrap(282447180198804430);
  SD59x18 internal constant TIER_ODDS_12_15 = SD59x18.wrap(430485137687959592);
  SD59x18 internal constant TIER_ODDS_13_15 = SD59x18.wrap(656113662171395111);
  SD59x18 internal constant TIER_ODDS_14_15 = SD59x18.wrap(1000000000000000000);
```

## [G-14] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol

```solidity
File: src/Claimer.sol

68    for (uint i = 0; i < winners.length; i++) {

144    for (uint i = 0; i < _claimCount; i++) {

```

```diff
diff --git a/Claimer.org.sol b/Claimer.sol
index 9d8a801..1b76487 100644
--- a/Claimer.org.sol
+++ b/Claimer.sol
@@ -6,9 +6,11 @@
     address _feeRecipient
   ) external returns (uint256 totalFees) {
     uint256 claimCount;
-    for (uint i = 0; i < winners.length; i++) {
+    uint i;
+    do {
       claimCount += prizeIndices[i].length;
-    }
+    }while (i < winners.length);


     uint96 feePerClaim = uint96(
       _computeFeePerClaim(
@@ -82,7 +84,8 @@
     uint256 elapsed = block.timestamp - (prizePool.lastClosedDrawAwardedAt());
     uint256 fee;

-    for (uint i = 0; i < _claimCount; i++) {
+    uint i;
+    do{
       fee += _computeFeeForNextClaim(
         minimumFee,
         decayConstant,
@@ -91,7 +94,7 @@
         _claimedCount + i,
         _maxFee
       );
-    }
+    }while(i < _claimCount);

     return fee / _claimCount;
   }

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L361

```solidity
File: src/abstract/TieredLiquidityDistributor.sol

361    for (uint8 i = start; i < end; i++) {

630    for (uint8 i = start; i < end; i++) {

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L138

```solidity
File: src/libraries/TierCalculationLib.sol

138    for (uint8 i = 0; i < _numberOfTiers; i++) {

```

## [G-15] Missing zero-address check in constructor

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L37C1-L51C4

```solidity
File:
  constructor(
    PrizePool _prizePool,
    uint256 _minimumFee,
    uint256 _maximumFee,
    uint256 _timeToReachMaxFee,
    UD2x18 _maxFeePortionOfPrize
  ) {
    prizePool = _prizePool;
    maxFeePortionOfPrize = _maxFeePortionOfPrize;
    decayConstant = LinearVRGDALib.getDecayConstant(
      LinearVRGDALib.getMaximumPriceDeltaScale(_minimumFee, _maximumFee, _timeToReachMaxFee)
    );
    minimumFee = _minimumFee;
    timeToReachMaxFee = _timeToReachMaxFee;
  }
```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L254

```solidity
File:
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
    if (address(twabController_) == address(0)) revert TwabControllerZeroAddress();
    if (address(yieldVault_) == address(0)) revert YieldVaultZeroAddress();
    if (address(prizePool_) == address(0)) revert PrizePoolZeroAddress();
    if (address(owner_) == address(0)) revert OwnerZeroAddress();

    _twabController = twabController_;
    _yieldVault = yieldVault_;
    _prizePool = prizePool_;

    _setClaimer(claimer_);
    _setYieldFeeRecipient(yieldFeeRecipient_);
    _setYieldFeePercentage(yieldFeePercentage_);

    _assetUnit = 10 ** super.decimals();

    // Approve once for max amount
    asset_.safeApprove(address(yieldVault_), type(uint256).max);

```

## [G-16] Ternary operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L171C1-L176C6

```solidity
File: src/Claimer.sol

   if (_tier != _canaryTier) {
      // canary tier
      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1));
    } else {
      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
    }
```

```diff
diff --git a/Claimer.org.sol b/Claimer.sol
index 95c289f..d6f06df 100644
--- a/Claimer.org.sol
+++ b/Claimer.sol
@@ -1,9 +1,7 @@
  function _computeMaxFee(uint8 _tier, uint8 _numTiers) internal view returns (uint256) {
     uint8 _canaryTier = _numTiers - 1;
-    if (_tier != _canaryTier) {
-      // canary tier
-      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1));
-    } else {
-      return _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
-    }
+    return (_tier != _canaryTier)
+     ? _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier - 1))
+     : _computeMaxFee(prizePool.getTierPrizeSize(_canaryTier));
+
   }
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L958C1-L973C6

```solidity
File: src/PrizePool.sol

    if (totalContributed != 0) {
      // vaultContributed / totalContributed
      return
        sd(
          int256(
            DrawAccumulatorLib.getDisbursedBetween(
              vaultAccumulator[_vault],
              _startDrawId,
              _endDrawId,
              _smoothing
            )
          )
        ).div(sd(int256(totalContributed)));
    } else {
      return sd(0);
```

## [G-17] Unnecessary casting as variable is already of the same type

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L266C1-L269C66

```solidity
File:

266    if (address(twabController_) == address(0)) revert TwabControllerZeroAddress();

267    if (address(yieldVault_) == address(0)) revert YieldVaultZeroAddress();

268    if (address(prizePool_) == address(0)) revert PrizePoolZeroAddress();

269    if (address(owner_) == address(0)) revert OwnerZeroAddress();

558    if (msg.sender != address(_liquidationPair))

559      revert LiquidationCallerNotLP(msg.sender, address(_liquidationPair));

560    if (_tokenIn != address(_prizePool.prizeToken()))

561      revert LiquidationTokenInNotPrizeToken(_tokenIn, address(_prizePool.prizeToken()));

592    return address(_prizePool);

668    if (address(liquidationPair_) == address(0)) revert LPZeroAddress();

671    address _previousLiquidationPair = address(_liquidationPair);

677    _asset.safeApprove(address(liquidationPair_), type(uint256).max);

682    return address(liquidationPair_);

746    return address(_twabController);


754    return address(_yieldVault);

762    return address(_liquidationPair);

770    return address(_prizePool);


```

## [G-18] Use hardcode address instead address(this)

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

```solidity
File:

312    uint256 _deltaBalance = prizeToken.balanceOf(address(this)) - _accountedBalance();

500    prizeToken.safeTransferFrom(msg.sender, address(this), _amount);

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L668

```solidity
File:

336    return _twabController.balanceOf(address(this), _account);

435     _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

468    _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

502    _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

562    if (_tokenOut != address(this))

563      revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));

570    _prizePool.contributePrizeTokens(address(this), _amountIn);

578    uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));

581      _yieldVault.deposit(_vaultAssets, address(this));

801    return _yieldVault.maxWithdraw(address(this)) + super.totalAssets();

809    return _twabController.totalSupply(address(this));

830    if (_token != address(this)) revert LiquidationTokenOutNotVaultShare(_token, address(this));

932    uint256 _vaultAssets = _asset.balanceOf(address(this));

954        address(this),

959    _yieldVault.deposit(_assets, address(this));

986      _twabController.delegateOf(address(this), _receiver) != _twabController.SPONSORSHIP_ADDRESS()

1026    _yieldVault.withdraw(_assets, address(this), address(this));

1176    uint256 _withdrawableAssets = _yieldVault.maxWithdraw(address(this));

```

## [G-19] use mappings instead of arrays

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L30

```solidity
File:

30  Vault[] public allVaults;

```

## [G-20] Use assembly to emit events

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L539

```solidity
File:

539      emit ReserveConsumed(excess);

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L958C1-L973C6

```solidity
File:

280      emit DrawManagerSet(params.drawManager);

305    emit DrawManagerSet(_drawManager);

328    emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);

341    emit WithdrawReserve(_to, _amount);

375    emit DrawClosed(

454      emit IncreaseClaimRewards(_feeRecipient, _fee);

461    emit ClaimedPrize(

492    emit WithdrawClaimRewards(_to, _amount, _available);

501    emit IncreaseReserve(msg.sender, _amount);

```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L458C1-L459C1

```solidity
File:


663    emit Delegated(_vault, _from, _to);

688    emit IncreasedBalance(_vault, _user, _amount, _delegateAmount);

692      emit ObservationRecorded(

731    emit DecreasedBalance(_vault, _user, _amount, _delegateAmount);

735      emit ObservationRecorded(

773    emit DecreasedTotalSupply(_vault, _amount, _delegateAmount);

777      emit TotalSupplyObservationRecorded(

807    emit IncreasedTotalSupply(_vault, _amount, _delegateAmount);

811      emit TotalSupplyObservationRecorded(

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L668

```solidity
File:

284    emit NewVault(

401    emit MintYieldFee(msg.sender, _recipient, _shares);

645    emit ClaimerSet(_previousClaimer, claimer_);

656    emit SetHooks(msg.sender, hooks);

681    emit LiquidationPairSet(liquidationPair_);

695    emit YieldFeePercentageSet(_previousYieldFeePercentage, yieldFeePercentage_);

708    emit YieldFeeRecipientSet(_previousYieldFeeRecipient, yieldFeeRecipient_);

962    emit Deposit(_caller, _receiver, _assets, _shares);

991    emit Sponsor(msg.sender, _receiver, _assets, _shares);

1029    emit Withdraw(_caller, _receiver, _owner, _assets, _shares);

1111    emit RecordedExchangeRate(_lastRecordedExchangeRate);

1126    emit Transfer(address(0), _receiver, _shares);

1142    emit Transfer(_owner, address(0), _shares);

1157    emit Transfer(_from, _to, _shares);

```

## [G-21] Using delete statement can save gas

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L369

```solidity
File:

369    claimCount = 0;

370    canaryClaimCount = 0;

371    largestTierClaimed = 0;

```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L229

```solidity
File:

229      index = 0;

```

## [G-22] Access mappings directly rather than using accessor functions

Accessing mappings directly instead of using accessor functions can potentially save gas costs in some cases. This is because accessing a mapping directly involves a single storage read operation, while using an accessor function may involve multiple storage reads and writes.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L316-L317

```solidity
File: src/PrizePool.sol
316      DrawAccumulatorLib.add(
      vaultAccumulator[_prizeVault],
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L541-L542

```solidity
File

541     DrawAccumulatorLib.getDisbursedBetween(
        vaultAccumulator[_vault],
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

```solidity
File: src/PrizePool.sol

963     DrawAccumulatorLib.getDisbursedBetween(
              vaultAccumulator[_vault],
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L660

```solidity
File: src/TwabController.sol#L660

660     uint96(userObservations[_vault][_from].details.balance)

```

## [G-23] Using a positive conditional flow to save a NOT opcode

Accessing mappings directly rather than using accessor functions and using positive conditional flow to save a NOT opcode are both techniques that can be used to optimize smart contracts for gas efficiency.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L428

```solidity
File: src/PrizePool.sol

428         if (
      !_isWinner(msg.sender, _winner, _tier, _prizeIndex, _vaultPortion, _tierOdds, _drawDuration
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L459

```solidity
File: src/libraries/DrawAccumulatorLib.sol

459      if (!targetAtOrAfter)
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L530

```solidity
File: src/TwabController.sol

530   if (!_isFromDelegate && _fromDelegate != SPONSORSHIP_ADDRESS)
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L561

```solidity
File: src/TwabController.sol

561   if (!_isToDelegate && _toDelegate != SPONSORSHIP_ADDRESS)
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol#L93

```solidity
File: src/libraries/ObservationLib.sol

93      if (!targetAfterOrAt)
```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/#L1200

```solidity
File: src/Vault.sol

1200     if (!_isVaultCollateralized()) revert VaultUnderCollateralized();
```

## [G-24] `2**<n>` should be re-written as type(uint<n>).max

In Solidity, the expression 2**<n> is commonly used to represent the maximum value that can be stored in a variable of size n bits. However, this expression can be expensive in terms of gas cost, particularly for large values of n, because it requires a lot of computational steps to calculate the result.
To avoid this gas cost, the expression 2**<n> can be replaced with the built-in constant type(uint<n>).max. This constant represents the maximum value that can be stored in a variable of size n bits, and it is calculated by setting all bits in the variable to 1.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L40

```solidity
File:

40     uint256 _numberOfPrizes = 4 ** _tier;
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L19

```solidity
File: src/libraries/OverflowSafeComparatorLib.sol

19        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;

20        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

35        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;

36        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

52        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;

53        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;
```

## [G-25] Internal/Private functions only called once can be inlined to save gas

Inlining internal or private functions that are only called once can be an effective way to optimize gas costs in Solidity contracts.

When a function is inlined, its code is inserted directly into the calling function at compile time, rather than being called as a separate function at runtime. This can eliminate the gas cost associated with the function call, as well as any overhead associated with setting up the function's stack frame.

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L24-L28

```solidity
File: src/libraries/LinearVRGDALib.sol

24     function getPerTimeUnit(
    uint256 _count,
    uint256 _durationSeconds
  ) internal pure returns (SD59x18)


39       function getVRGDAPrice(
    uint256 _targetPrice,
    uint256 _timeSinceStart,
    uint256 _sold,
    SD59x18 _perTimeUnit,
    SD59x18 _decayConstant
  ) internal pure returns (uint256)

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L331

```solidity
File: src/abstract/TieredLiquidityDistributor.sol

331    function _nextDraw(uint8 _nextNumberOfTiers, uint96 _prizeTokenLiquidity) internal



518      function _consumeLiquidity(
       Tier memory _tierStruct,
       uint8 _tier,
       uint104 _liquidity
  )    internal returns (Tier memory)

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L120-L124

```solidity
File: src/libraries/DrawAccumulatorLib.sol

120       function getTotalRemaining(
          Accumulator storage accumulator,
          uint16 _startDrawId,
         SD59x18 _alpha
  )      internal view returns (uint256)


166        function getDisbursedBetween(
           Accumulator storage _accumulator,
           uint16 _startDrawId,
           uint16 _endDrawId,
           SD59x18 _alpha
      )   internal view returns (uint256)

```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L32

```solidity
File: src/libraries/TierCalculationLib.sol

32     function estimatePrizeFrequencyInDraws(SD59x18 _tierOdds) internal pure returns (uint256)
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L52-L57

```solidity
File: src/libraries/TierCalculationLib.sol

52       function canaryPrizeCount(
          uint8 _numberOfTiers,
          uint8 _canaryShares,
          uint8 _reserveShares,
          uint8 _tierShares
         ) internal pure returns (UD60x18)


73        function isWinner(
          uint256 _userSpecificRandomNumber,
          uint128 _userTwab,
          uint128 _vaultTwabTotalSupply,
          SD59x18 _vaultContributionFraction,
          SD59x18 _tierOdds
          )internal pure returns (bool)

104        function calculatePseudoRandomNumber(
           address _user,
           uint8 _tier,
           uint32 _prizeIndex,
           uint256 _winningRandomNumber
           )internal pure returns (uint256)


133        function estimatedClaimCount(
           uint8 _numberOfTiers,
           uint16 _grandPrizePeriod
           ) internal pure returns (uint32)
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol

```solidity
File: src/libraries/OverflowSafeComparatorLib.sol

31     function lte(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (bool)


47     function checkedSub(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (uint32)

```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L75-L82

```solidity
File:

75          function increaseBalances(
            uint32 PERIOD_LENGTH,
            uint32 PERIOD_OFFSET,
            Account storage _account,
            uint96 _amount,
            uint96 _delegateAmount
            )
          internal


144        function decreaseBalances(
          uint32 PERIOD_LENGTH,
          uint32 PERIOD_OFFSET,
          Account storage _account,
          uint96 _amount,
          uint96 _delegateAmount,
          string memory _revertMessage
       )
        internal



263      function getBalanceAt(
         uint32 PERIOD_OFFSET,
         ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
         AccountDetails memory _accountDetails,
         uint32 _targetTime
        )    internal view returns (uint256)

```

## [G-26] Use assembly to perform efficient back-to-back calls

Using assembly to perform efficient back-to-back calls can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, when a contract calls another contract, it typically uses the CALL opcode. However, using CALL can be expensive in terms of gas cost, particularly if the contract frequently makes back-to-back calls to the same contract.

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol

```solidity
File: src/Vault.sol

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
