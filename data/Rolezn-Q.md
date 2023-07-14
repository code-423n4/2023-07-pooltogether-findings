## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#low1-approve-typeuint256max-not-work-with-some-tokens) | Approve `type(uint256).max` not work with some tokens | 1 |
| [LOW&#x2011;2](#low2-array-lengths-not-checked) | Array lengths not checked | 2 |
| [LOW&#x2011;3](#low3-decimals-not-part-of-erc20-standard) | `decimals()` not part of ERC20 standard | 2 |

Total: 5 contexts over 3 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#nc1-consider-adding-a-denylist) | Consider adding a deny-list | 1 |
| [NC&#x2011;2](#nc2-consistent-usage-of-require-vs-custom-error) | Consistent usage of require vs custom error | 20 |
| [NC&#x2011;3](#nc3-generate-perfect-code-headers-for-better-solidity-code-layout-and-readability) | Generate perfect code headers for better solidity code layout and readability | 1 |
| [NC&#x2011;4](#nc4-use-delete-to-clear-variables) | Use `delete` to Clear Variables | 4 |
| [NC&#x2011;5](#nc5-nonexternalpublic-function-names-should-begin-with-an-underscore) | Non-`external`/`public` function names should begin with an underscore | 38 |
| [NC&#x2011;6](#nc6-override-function-arguments-that-are-unused-should-have-the-variable-name-removed-or-commented-out-to-avoid-compiler-warnings) | `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings | 2 |
| [NC&#x2011;7](#nc7-empty-blocks-should-be-removed-or-emit-something) | Empty blocks should be removed or emit something | 1 |
| [NC&#x2011;8](#nc20-using-underscore-at-the-end-of-variable-name) | Using underscore at the end of variable name | 13 |
| [NC&#x2011;9](#nc25-use-smtchecker) | Use SMTChecker | 1 |

Total: 81 contexts over 9 issues

## Low Risk Issues

### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.



### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Approve `type(uint256).max` not work with some tokens

Source: https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers

#### <ins>Proof Of Concept</ins>

```solidity
File: Vault.sol

677: _asset.safeApprove(address(liquidationPair_), type(uint256).max);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L677






### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Array lengths not checked

If the length of the arrays are not required to be of the same length, user operations may not be fully executed

#### <ins>Proof Of Concept</ins>


```solidity
File: Claimer.sol


function claimPrizes(
    Vault vault,
    uint8 tier,
    address[] calldata winners,
    uint32[][] calldata prizeIndices,
    address _feeRecipient
  ) external returns (uint256 totalFees) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L60

```solidity
File: Vault.sol


function claimPrizes(
    uint8 _tier,
    address[] calldata _winners,
    uint32[][] calldata _prizeIndices,
    uint96 _feePerClaim,
    address _feeRecipient
  ) external returns (uint256) {

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L607







### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> `decimals()` not part of ERC20 standard

`decimals()` is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

#### <ins>Proof Of Concept</ins>


```solidity
File: Vault.sol

279: _assetUnit = 10 ** super.decimals()
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L279

```solidity
File: Vault.sol

341: return super.decimals()
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L341

















## Non Critical Issues

### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Consider adding a deny-list

Doing so will significantly increase centralization, but will help to prevent hackers from using stolen tokens

#### <ins>Proof Of Concept</ins>


```solidity
File: Vault.sol

110: contract Vault is ERC4626, ERC20Permit, ILiquidationSource, Ownable

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L110






### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Consistent usage of require vs custom error

Consider using the same approach throughout the codebase to improve the consistency of the code.

#### <ins>Proof Of Concept</ins>


```solidity
File: Vault.sol

266: if (address(twabController_) == address(0)) revert TwabControllerZeroAddress();
267: if (address(yieldVault_) == address(0)) revert YieldVaultZeroAddress();
268: if (address(prizePool_) == address(0)) revert PrizePoolZeroAddress();
269: if (address(owner_) == address(0)) revert OwnerZeroAddress();
396: if (_shares > _yieldFeeTotalSupply) revert YieldFeeGTAvailable(_shares, _yieldFeeTotalSupply);
409: revert DepositMoreThanMax(_receiver, _assets, maxDeposit(_receiver));
515: revert WithdrawMoreThanMax(_owner, _assets, maxWithdraw(_owner));
529: if (_shares > maxRedeem(_owner)) revert RedeemMoreThanMax(_owner, _shares, maxRedeem(_owner));
559: revert LiquidationCallerNotLP(msg.sender, address(_liquidationPair));
561: revert LiquidationTokenInNotPrizeToken(_tokenIn, address(_prizePool.prizeToken()));
563: revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));
564: if (_amountOut == 0) revert LiquidationAmountOutZero();
568: revert LiquidationAmountOutGTYield(_amountOut, _liquidableYield);
591: if (_token != _liquidationPair.tokenIn()) revert TargetTokenNotSupported(_token);
614: if (msg.sender != _claimer) revert CallerNotClaimer(msg.sender, _claimer);
668: if (address(liquidationPair_) == address(0)) revert LPZeroAddress();
830: if (_token != address(this)) revert LiquidationTokenOutNotVaultShare(_token, address(this));
972: if (_shares > maxMint(_receiver)) revert MintMoreThanMax(_receiver, _shares, maxMint(_receiver));
1200: if (!_isVaultCollateralized()) revert VaultUnderCollateralized();
1220: revert YieldFeePercentageGTPrecision(yieldFeePercentage_, FEE_PRECISION);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L266

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L267

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L268

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L269

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L396

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L409

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L515

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L529

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L559

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L561

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L563

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L564

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L568

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L591

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L614

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L668

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L830

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L972

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1200

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1220






### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Generate perfect code headers for better solidity code layout and readability

It is recommended to use pre-made headers for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```


#### <ins>Proof Of Concept</ins>


```solidity
File: Vault.sol

4: Vault.sol

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/test/contracts/mock/Vault.sol#L4






### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

```solidity
File: PrizePool.sol

369: claimCount = 0;
370: canaryClaimCount = 0;
371: largestTierClaimed = 0;

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L369-L371

```solidity
File: TwabLib.sol

229: index = 0;

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L229








### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Non-`external`/`public` function names should begin with an underscore

According to the Solidity Style Guide, Non-`external`/`public` function names should begin with an <a href="https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables">underscore</a>

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LinearVRGDALib.sol

16: function getDecayConstant(UD2x18 _priceDeltaScale) internal pure returns (SD59x18) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L16

```solidity
File: LinearVRGDALib.sol

24: function getPerTimeUnit(
    uint256 _count,
    uint256 _durationSeconds
  ) internal pure returns (SD59x18) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L24

```solidity
File: LinearVRGDALib.sol

39: function getVRGDAPrice(
    uint256 _targetPrice,
    uint256 _timeSinceStart,
    uint256 _sold,
    SD59x18 _perTimeUnit,
    SD59x18 _decayConstant
  ) internal pure returns (uint256) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L39

```solidity
File: LinearVRGDALib.sol

102: function getMaximumPriceDeltaScale(
    uint256 _minFee,
    uint256 _maxFee,
    uint256 _time
  ) internal pure returns (UD2x18) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L102

```solidity
File: DrawAccumulatorLib.sol

64: function add(
    Accumulator storage accumulator,
    uint256 _amount,
    uint16 _drawId,
    SD59x18 _alpha
  ) internal returns (bool) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L64

```solidity
File: DrawAccumulatorLib.sol

120: function getTotalRemaining(
    Accumulator storage accumulator,
    uint16 _startDrawId,
    SD59x18 _alpha
  ) internal view returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L120

```solidity
File: DrawAccumulatorLib.sol

141: function newestDrawId(Accumulator storage accumulator) internal view returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L141

```solidity
File: DrawAccumulatorLib.sol

151: function newestObservation(
    Accumulator storage accumulator
  ) internal view returns (Observation memory) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L151

```solidity
File: DrawAccumulatorLib.sol

166: function getDisbursedBetween(
    Accumulator storage _accumulator,
    uint16 _startDrawId,
    uint16 _endDrawId,
    SD59x18 _alpha
  ) internal view returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L166

```solidity
File: DrawAccumulatorLib.sol

342: function computeIndices(
    RingBufferInfo memory ringBufferInfo
  ) internal pure returns (Pair32 memory) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L342

```solidity
File: DrawAccumulatorLib.sol

364: function readDrawIds(
    Accumulator storage accumulator,
    Pair32 memory indices
  ) internal view returns (Pair32 memory) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L364

```solidity
File: DrawAccumulatorLib.sol

380: function integrateInf(SD59x18 _alpha, uint _x, uint _k) internal pure returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L380

```solidity
File: DrawAccumulatorLib.sol

390: function integrate(
    SD59x18 _alpha,
    uint _start,
    uint _end,
    uint _k
  ) internal pure returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L390

```solidity
File: DrawAccumulatorLib.sol

406: function computeC(SD59x18 _alpha, uint _x, uint _k) internal pure returns (SD59x18) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L406

```solidity
File: DrawAccumulatorLib.sol

420: function binarySearch(
    uint16[366] storage _drawRingBuffer,
    uint16 _oldestIndex,
    uint16 _newestIndex,
    uint16 _cardinality,
    uint16 _targetLastClosedDrawId
  )
    internal
    view
    returns (
      uint16 beforeOrAtIndex,
      uint16 beforeOrAtDrawId,
      uint16 afterOrAtIndex,
      uint16 afterOrAtDrawId
    )
  {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L420

```solidity
File: TierCalculationLib.sol

17: function getTierOdds(
    uint8 _tier,
    uint8 _numberOfTiers,
    uint16 _grandPrizePeriod
  ) internal pure returns (SD59x18) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L17

```solidity
File: TierCalculationLib.sol

32: function estimatePrizeFrequencyInDraws(SD59x18 _tierOdds) internal pure returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L32

```solidity
File: TierCalculationLib.sol

39: function prizeCount(uint8 _tier) internal pure returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L39

```solidity
File: TierCalculationLib.sol

52: function canaryPrizeCount(
    uint8 _numberOfTiers,
    uint8 _canaryShares,
    uint8 _reserveShares,
    uint8 _tierShares
  ) internal pure returns (UD60x18) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L52

```solidity
File: TierCalculationLib.sol

73: function isWinner(
    uint256 _userSpecificRandomNumber,
    uint128 _userTwab,
    uint128 _vaultTwabTotalSupply,
    SD59x18 _vaultContributionFraction,
    SD59x18 _tierOdds
  ) internal pure returns (bool) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L73

```solidity
File: TierCalculationLib.sol

104: function calculatePseudoRandomNumber(
    address _user,
    uint8 _tier,
    uint32 _prizeIndex,
    uint256 _winningRandomNumber
  ) internal pure returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L104

```solidity
File: TierCalculationLib.sol

118: function calculateWinningZone(
    uint256 _userTwab,
    SD59x18 _vaultContributionFraction,
    SD59x18 _tierOdds
  ) internal pure returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L118

```solidity
File: TierCalculationLib.sol

133: function estimatedClaimCount(
    uint8 _numberOfTiers,
    uint16 _grandPrizePeriod
  ) internal pure returns (uint32) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L133

```solidity
File: ObservationLib.sol

55: function binarySearch(
    Observation[MAX_CARDINALITY] storage _observations,
    uint24 _newestObservationIndex,
    uint24 _oldestObservationIndex,
    uint32 _target,
    uint16 _cardinality,
    uint32 _time
  ) internal view returns (Observation memory beforeOrAt, Observation memory afterOrAt) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol#L55

```solidity
File: OverflowSafeComparatorLib.sol

15: function lt(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (bool) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L15

```solidity
File: OverflowSafeComparatorLib.sol

31: function lte(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (bool) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L31

```solidity
File: OverflowSafeComparatorLib.sol

47: function checkedSub(uint32 _a, uint32 _b, uint32 _timestamp) internal pure returns (uint32) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L47

```solidity
File: TwabLib.sol

75: function increaseBalances(
    uint32 PERIOD_LENGTH,
    uint32 PERIOD_OFFSET,
    Account storage _account,
    uint96 _amount,
    uint96 _delegateAmount
  )
    internal
    returns (ObservationLib.Observation memory observation, bool isNew, bool isObservationRecorded)
  {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L75

```solidity
File: TwabLib.sol

144: function decreaseBalances(
    uint32 PERIOD_LENGTH,
    uint32 PERIOD_OFFSET,
    Account storage _account,
    uint96 _amount,
    uint96 _delegateAmount,
    string memory _revertMessage
  )
    internal
    returns (ObservationLib.Observation memory observation, bool isNew, bool isObservationRecorded)
  {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L144

```solidity
File: TwabLib.sol

223: function getOldestObservation(
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails
  ) internal view returns (uint16 index, ObservationLib.Observation memory observation) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L223

```solidity
File: TwabLib.sol

244: function getNewestObservation(
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails
  ) internal view returns (uint16 index, ObservationLib.Observation memory observation) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L244

```solidity
File: TwabLib.sol

263: function getBalanceAt(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) internal view returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L263

```solidity
File: TwabLib.sol

288: function getTwabBetween(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _startTime,
    uint32 _endTime
  ) internal view returns (uint256) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L288

```solidity
File: TwabLib.sol

418: function getTimestampPeriod(
    uint32 PERIOD_LENGTH,
    uint32 PERIOD_OFFSET,
    uint32 _timestamp
  ) internal pure returns (uint32 period) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L418

```solidity
File: TwabLib.sol

456: function getPreviousOrAtObservation(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) internal view returns (ObservationLib.Observation memory prevOrAtObservation) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L456

```solidity
File: TwabLib.sol

544: function getNextOrNewestObservation(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) internal view returns (ObservationLib.Observation memory nextOrNewestObservation) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L544

```solidity
File: TwabLib.sol

663: function isTimeSafe(
    uint32 PERIOD_LENGTH,
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _time
  ) internal view returns (bool) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L663

```solidity
File: TwabLib.sol

743: function isTimeRangeSafe(
    uint32 PERIOD_LENGTH,
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _startTime,
    uint32 _endTime
  ) internal view returns (bool) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L743



</details>






### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

#### <ins>Proof Of Concept</ins>


```solidity
File: Vault.sol

375: function maxDeposit : address

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L375

```solidity
File: Vault.sol

383: function maxMint : address

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L383








### <a href="#qa-summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Empty blocks should be removed or emit something

#### <ins>Proof Of Concept</ins>

```solidity
File: Vault.sol

7: constructor(
    IERC20 _asset,
    string memory _name,
    string memory _symbol,
    TwabController twabController_,
    IERC4626 yieldVault_,
    PrizePool prizePool_,
    address claimer_,
    address yieldFeeRecipient_,
    uint256 yieldFeePercentage_,
    address _owner
  )
    Vault(
      _asset,
      _name,
      _symbol,
      twabController_,
      yieldVault_,
      prizePool_,
      claimer_,
      yieldFeeRecipient_,
      yieldFeePercentage_,
      _owner
    )
  {}

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/test/contracts/mock/Vault.sol#L7-L31











### <a href="#qa-summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Using underscore at the end of variable name

The use of underscore at the end of the variable name is uncommon and also suggests that the variable name was not completely changed. Consider refactoring `variableName_` to `variableName`.

#### <ins>Proof Of Concept</ins>


```solidity
File: PrizePool.sol

368: winningRandomNumber_;
372: openDrawStartedAt_;

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L368

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L372



```solidity
File: DrawAccumulatorLib.sol

84: newestDrawId_;

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L84

```solidity
File: Vault.sol

271: twabController_;
272: yieldVault_;
273: prizePool_;
646: claimer_;
1210: claimer_;
679: liquidationPair_;
696: yieldFeePercentage_;
1222: yieldFeePercentage_;
709: yieldFeeRecipient_;
1230: yieldFeeRecipient_;

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L271

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L272

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L273

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L646

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1210

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L679

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L696

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1222

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L709

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1230





### <a href="#qa-summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19





