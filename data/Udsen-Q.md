## 1. NO INPUT VALIDATIONS FOR CRITICAL PARAMETER ASSIGNMENTS IN THE `PrizePool.Constructor()`

The input validation checks for the critical parameters should be performed before assignment to the state variables as shown below:

    require(params.drawPeriodSeconds != 0, "Can not be zero");

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L271-L276

## 2. VARIABLE NAME IS SHADOWED AND SHOULD BE REPLACED

In the `PrizePool.claimPrize` and `PrizePool._isWinner` the variable name `_tierOdds` is used. But there is a function of the same name in the inherited contract `TieredLiquidityDistributor.sol` as shown below:

  function _tierOdds(uint8 _tier, uint8 _numTiers) internal pure returns (SD59x18) {

Hence it is recommended to change the meomory variable name `_tierOdds` in the `PrizePool` contract functions mentioned above.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L421
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L429
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L876
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L807

## 3. NO INPUT VALIDATION IN CRITICAL PARAMETERS

In the `Vault.constructor` function is used to instantiate a new Vault for the protocol. But in this the following critical variables are not checked for `0` value. This could lead to loss of funds to the owner of the standard `Vault` if he makes a mistake.

    _setClaimer(claimer_);
    _setYieldFeeRecipient(yieldFeeRecipient_);
    _setYieldFeePercentage(yieldFeePercentage_);

For example if the `yieldFeeRecipient_` address is set to address(0), then the `owner` of the vault will not be able to recover the `Yield Fee` amount. And at the same time if the `claimer` address is set to `address(0)` the calim functionlity will be disfunctional.

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L275-L277