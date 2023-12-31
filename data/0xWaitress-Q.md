[L-1] At PrizePool, `_isWinner` passes a uint16 of `_drawDuration` to `_getVaultUserBalanceAndTotalSupplyTwab()` which takes a uint256 of `_drawDuration`.

```solidity
  function _isWinner(
...
    uint16 _drawDuration
  )
...
    (uint256 _userTwab, uint256 _vaultTwabTotalSupply) = _getVaultUserBalanceAndTotalSupplyTwab(
      _vault,
      _user,
      _drawDuration
    );

```

_getVaultUserBalanceAndTotalSupplyTwab
```solidity
  function _getVaultUserBalanceAndTotalSupplyTwab(
    address _vault,
    address _user,
    uint256 _drawDuration
  )
```

Since the maximum value of _drawDuration is only 366 so the severity is low.

Recommendation
Please align data type for consistency.


[L-2] estimatePrizeFrequencyInDraws returns uin256 but casted to uint16 at getTierAccrualDurationInDraws.

```solidity
  function getTierAccrualDurationInDraws(uint8 _tier) external view returns (uint16) {
    return
      uint16(TierCalculationLib.estimatePrizeFrequencyInDraws(_tierOdds(_tier, numberOfTiers)));
  }
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L551-L554

```solidity
  function estimatePrizeFrequencyInDraws(SD59x18 _tierOdds) internal pure returns (uint256) {
    return uint256(fromSD59x18(sd(1e18).div(_tierOdds).ceil()));
  }
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L32-L34

[QA-1] _tierOdds of tier 0 at every tier_num is the same, thus can be reduced to 1 variable

all tier odds at tier 0, for example TIER_ODDS_0_3 and TIER_ODDS_0_4 and so on so forth, has the same value `2739726027397260`, it's advisable to reduce the number of variable to TIER_ODDS_0 and return this for all _tierOdds(0, x).

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L807

## Recommendation
```solidity
function _tierOdds() {
if (_tier == 0) return TIER_ODDS_0;
...
...

```


[QA-2] this line in Vault can be re-written for better clarity
```solidity
if (_withdrawableAssets > _totalSupplyToAssets) {
      _withdrawableAssets = _withdrawableAssets - (_withdrawableAssets - _totalSupplyToAssets);
    }
```

is no different from making `_withdrawableAssets` as `_totalSupplyToAssets`
```solidity
if (_withdrawableAssets > _totalSupplyToAssets) {
      _withdrawableAssets = _totalSupplyToAssets;
    }
````