## [01] Unused Import

An import statement is present in the source file, but the imported file is not used:

- https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L6
```solidity
import { LiquidationPair } from "v5-liquidator/LiquidationPair.sol";
```
- https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L6
```solidity
import { toWadUnsafe, unsafeWadDiv } from "solmate/utils/SignedWadMath.sol";
```
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L5
```solidity
import { toSD59x18, fromSD59x18 } from "prb-math/SD59x18.sol";
```
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L7-L10
```solidity
import { UD2x18, intoUD60x18 } from "prb-math/UD2x18.sol";
import { SD1x18, unwrap, UNIT } from "prb-math/SD1x18.sol";
import { toUD34x4 } from "../libraries/UD34x4.sol";
```
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L6
```solidity
import { fromUD60x18 } from "prb-math/UD60x18.sol";
```
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/UD34x4.sol#L5
```solidity
import { uMAX_UD60x18 } from "prb-math/UD60x18.sol";
```
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L8-L9
```solidity
import { toSD59x18, fromSD59x18 } from "prb-math/SD59x18.sol";
import { UD60x18, intoSD59x18 } from "prb-math/UD60x18.sol";
```
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L12
```solidity
import { fromUD60x18 as fromUD60x18toUD34x4, intoUD60x18 as fromUD34x4toUD60x18, toUD34x4 } from "./libraries/UD34x4.sol";
```

To optimize the code and avoid redundancy, remove the unnecessary import.

## [02] Constructor's Input Parameters Check

The `_minimumFee` and `_maximumFee` parameters of the `constructor()` function at https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L47-L49 does not have any checks. Add check that `_maximumFee` is greater or equal to `_minimumFee`. For example:
```solidity
require(_maximumFee >= _minimumFee, "Invalid max fee");
```

Consider adding these checks to ensure that input parameters are within the expected range.

## [03] Getter for `allVaults` array

In the `VaultFactory` contract, the `allVaults` variable serves to store the addresses of all vaults deployed by this factory. While the `totalVaults()` function allows external contracts to know the total count of these vaults, it does not provide direct access to their addresses.

To provide an interface for accessing all vault addresses, we suggest adding a getter function for the `allVaults` array. This function can return the entire list of vaults, enabling other contracts to interact with this data:
```solidity
function getAllVaults() external view returns (Vault[] memory) {
    return allVaults;
}
```

## [04] Function documentation omitted

Missed documentation for `_isCanaryTier()` function at line https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L604
```solidity
  function _isCanaryTier(uint8 _tier, uint8 _numberOfTiers) internal pure returns (bool) {
```

`_checkValidTier()` function at line https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L753

## [05] Function parameter documentation omitted

Missed documentation for parameters in `contributePrizeTokens()` function at line https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L311

`getTotalContributedBetween()`, `getContributedBetween()`, `getTierAccrualDurationInDraws()` functions at lines https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L518-L554

`_prizeIndex` parameter in `isWinner()` at line https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L660

`_prizeIndex`, `_vaultPortion`, `_tierOdds`, `_drawDuration` parameters in `_isWinner()` at line https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L842