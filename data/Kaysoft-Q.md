## [L-01] Avoid Variable Shadowing by renaming the `_tierOdds` return variable of the `_computeVaultTierDetails()` function call in the the PrizePool.sol::claimPrice function.

The `_tierOdds` return variable of the `_computeVaultTierDetails()` function call in the the PrizePool.sol::claimPrice function shadows a state variable TieredLiquidityDistributor._tierOdds.

File: https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L421

```
function claimPrize(
        address _winner,
        uint8 _tier,
        uint32 _prizeIndex,
        address _prizeRecipient,
        uint96 _fee,
        address _feeRecipient
    ) external returns (uint256) {
        Tier memory tierLiquidity = _getTier(_tier, numberOfTiers);

        if (_fee > tierLiquidity.prizeSize) {
            revert FeeTooLarge(_fee, tierLiquidity.prizeSize);
        }
          //@audit _tierOdds shadows TieredLiquidityDistributor._tierOdds
 @>       (SD59x18 _vaultPortion, SD59x18 _tierOdds, uint16 _drawDuration) = _computeVaultTierDetails( 
        msg.sender, _tier, numberOfTiers, lastClosedDrawId);

```
#### Recommended Mitigation Steps
Rename the return variable `tierOdds`, of the `_computeVaultTierDetails()` function call in the `claimPrice` function.

## [L-02] Paramater arrays `winners` and `prizeIndices` may be of different length.

The parameters `winners` and `prizeIndices` of the function Claimer.sol::claimPrizes() may be of different lengths leading to unexpected execution and state changes.


File: https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L69
```
function claimPrizes(
    Vault vault,
    uint8 tier,
    address[] calldata winners,
    uint32[][] calldata prizeIndices,
    address _feeRecipient
  ) external returns (uint256 totalFees) {
    uint256 claimCount;
    //@audit no check for same array length `winners.length == pricesIndices.length`
    for (uint i = 0; i < winners.length; i++) {
      claimCount += prizeIndices[i].length;
    }

    uint96 feePerClaim = uint96(
      _computeFeePerClaim(
        _computeMaxFee(tier, prizePool.numberOfTiers()),
        claimCount,
        prizePool.claimCount()
      )
    );

    vault.claimPrizes(tier, winners, prizeIndices, feePerClaim, _feeRecipient);

    return feePerClaim * claimCount;
  }
```
#### Recommended Mitigation Steps
Add a validation to check that the `winners` and `priceIndices` arrays are of same length.
```
require(winners.length == pricesIndices.length, "Unequal length");
```
## [L-03] Use latest stable version of Solidity
Version 0.8.17 pragma compiler used in the project. Consider using a latest stable version because it will have security bug fixes and updates.
Files: all files

#### Recommended Mitigation Steps
Consider using atleast Solidity version 0.8.19
