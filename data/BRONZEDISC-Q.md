## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

```solidity
// place this modifier before the constructor
287:  modifier onlyDrawManager() {
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.


https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol

```solidity
// external functions by the new standards should now come before public functions
394:  function mintYieldFee(uint256 _shares, address _recipient) external {
427:  function depositWithPermit(
458:  function mintWithPermit(
480:  function sponsor(uint256 _assets, address _receiver) external returns (uint256) {
494:  function sponsorWithPermit(
590:  function targetOf(address _token) external view returns (address) {
607:  function claimPrizes(
641:  function setClaimer(address claimer_) external onlyOwner returns (address) {
653:  function setHooks(VaultHooks memory hooks) external {
665:  function setLiquidationPair(
691:  function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {
704:  function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {
786:  function getHooks(address _account) external view returns (VaultHooks memory) {
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol

```solidity
// place all these private functions for last
330:  function _calculateTemporaryObservation(
352:  function _getNextObservationIndex(
399:  function _extrapolateFromBalance(
435:  function _getTimestampPeriod(
473:  function _getPreviousOrAtObservation(
561:  function _getNextOrNewestObservation(
627:  function _getSurroundingOrAtObservations(
682:  function _isTimeSafe(
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol

```solidity
// place all these internal functions last.
331:  function _nextDraw(uint8 _nextNumberOfTiers, uint96 _prizeTokenLiquidity) internal {
384: function _computeNewDistributions(
406:  function _computeNewDistributions(
460:  function _getTierPrizeCount(uint8 _tier, uint8 _numberOfTiers) internal view returns (uint32) {
471:  function _getTier(uint8 _tier, uint8 _numberOfTiers) internal view returns (Tier memory) {
497:  function _getTotalShares(uint8 _numberOfTiers) internal view returns (uint256) {
509:  function _computeShares(uint8 _tier, uint8 _numTiers) internal view returns (uint8) {
518:  function _consumeLiquidity(
557:  function _computePrizeSize(
590:  function _computePrizeSize(
604:  function _isCanaryTier(uint8 _tier, uint8 _numberOfTiers) internal pure returns (bool) {
612:  function _getTierLiquidityToReclaim(
663:  function _getTierRemainingLiquidity(
710:  function _canaryPrizeCount(uint8 _numberOfTiers) internal view returns (uint32) {
730:  function _estimatedPrizeCount(uint8 numTiers) internal pure returns (uint32) {
764:  function _canaryPrizeCountFractional(uint8 numTiers) internal view returns (UD60x18) {
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

```solidity
311:  function contributePrizeTokens(address _prizeVault, uint256 _amount) external returns (uint256) {

// @params missing
520:  function getTotalContributedBetween(
535:  function getContributedBetween(
551:  function getTierAccrualDurationInDraws(uint8 _tier) external view returns (uint16) {

753:  function _checkValidTier(uint8 _tier, uint8 _numTiers) internal pure {

// @return missing
760:  function _openDrawEndsAt() internal view returns (uint64) {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol

```solidity
// private and internal `variables` should preppend with `underline`
36:  mapping(address => mapping(address => TwabLib.Account)) internal userObservations;
39:  mapping(address => TwabLib.Account) internal totalSupplyObservations;
42:  mapping(address => mapping(address => address)) internal delegates;
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
75:  function increaseBalances(
144:  function decreaseBalances(
223:  function getOldestObservation(
244:  function getNewestObservation(
263:  function getBalanceAt(
288:  function getTwabBetween(
418:  function getTimestampPeriod(
456:  function getPreviousOrAtObservation(
544:  function getNextOrNewestObservation(
663:  function isTimeSafe(
743:  function isTimeRangeSafe(
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
15:  function lt(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (bool) {
31:  function lte(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (bool) {
47:  function checkedSub(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (uint32) {
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
55:  function binarySearch(
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

```solidity
// private and internal `variables` should preppend with `underline`
202:  mapping(address => DrawAccumulatorLib.Accumulator) internal vaultAccumulator;
206:  mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))
    internal claimedPrizes;
210:  mapping(address => uint256) internal claimerRewards;
231:  DrawAccumulatorLib.Accumulator internal totalAccumulator;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
17:  function getTierOdds(
32:  function estimatePrizeFrequencyInDraws(SD59x18 _tierOdds) internal pure returns (uint256) {
39:  function prizeCount(uint8 _tier) internal pure returns (uint256) {
52:  function canaryPrizeCount(
73:  function isWinner(
104:  function calculatePseudoRandomNumber(
118:  function calculateWinningZone(
133:  function estimatedClaimCount(
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
64:  function add(
120:  function getTotalRemaining(
141:  function newestDrawId(Accumulator storage accumulator) internal view returns (uint256) {
151:  function newestObservation(
166:  function getDisbursedBetween(
315:  function _computeTail(
342:  function computeIndices(
364:  function readDrawIds(
380:  function integrateInf(SD59x18 _alpha, uint _x, uint _k) internal pure returns (uint256) {
390:  function integrate(
406:  function computeC(SD59x18 _alpha, uint _x, uint _k) internal pure returns (SD59x18) {
420:  function binarySearch(
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol

```solidity
// private and internal `variables` should preppend with `underline`
223:  uint16 internal lastClosedDrawId;
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol

```solidity
// private and internal `functions` should preppend with `underline`
16:  function getDecayConstant(UD2x18 _priceDeltaScale) internal pure returns (SD59x18) {
24:  function getPerTimeUnit(
39:  function getVRGDAPrice(
102:  function getMaximumPriceDeltaScale(
```