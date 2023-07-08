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