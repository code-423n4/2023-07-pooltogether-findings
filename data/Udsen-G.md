## 1. ARITHMETIC OPERATIONS CAN BE UNCHECKED DUE TO PREVIOUS CONDITIONAL CHECKS

`_reserve -= _amount;` can be unchecked to save gas, since previous conditional check makes sure that `_amount <= _reserve`. 

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L336-L339
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L671
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L538
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L459
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L490

## 2. REDUNDANT MSTORE ASSIGNMENT IN THE `PrizePool._computeNextNumberOfTiers()` function.

In the `PrizePool._computeNextNumberOfTiers()` the following code snippet is used to update the `_nextNumberOfTiers` value as shown below:

    _nextNumberOfTiers = _nextNumberOfTiers > MINIMUM_NUMBER_OF_TIERS
      ? _nextNumberOfTiers
      : MINIMUM_NUMBER_OF_TIERS;

There is a redundant assignment of the `_nextNumberOfTiers` variable to itself when `_nextNumberOfTiers > MINIMUM_NUMBER_OF_TIERS`.

This can be optimized as follows:

    if(!(_nextNumberOfTiers > MINIMUM_NUMBER_OF_TIERS)){
        _nextNumberOfTiers = MINIMUM_NUMBER_OF_TIERS;
    }

This will save an extra `MLOAD`.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L785-L787

## 3. USE LOCAL MEMORY VARIABLE INSTEAD OF STATE VARIABLE

The `PrizePool._computeVaultTierDetails` function is used to compute the data to determine the winner of a prize. In the function there is an internal call made to teh `_tierOdds` function as shown below:

    tierOdds = _tierOdds(_tier, numberOfTiers); 

The `numberOfTiers` used in this function call is a state variable. Inplace of this state variable the `_numberOfTiers` memory variable can be used instead. This will save an extra `SLOAD`.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L901

## 4. ARITHMETIC OPERATION CAN BE SIMPLIFIED TO SAVE GAS 

The `Vault._currentExchangeRate` function, is used to calculate the exchange rate between the amount of assets withdrawable from the YieldVault and the amount of shares minted by this Vault. In the function there is a calculation as follows to compute the `_withdrawableAssets` when `_withdrawableAssets > _totalSupplyToAssets`, as shown below:

      _withdrawableAssets = _withdrawableAssets - (_withdrawableAssets - _totalSupplyToAssets);

Here the `yield` is removed from the `_withdrawableAssets` amount. 

The above calculation can be simplified as follows to save gas: and the above calculation can be commented for ease of understanding later.

      _withdrawableAssets = _totalSupplyToAssets;

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1179

## 5. REDUNDANT CONDITIONAL CHECK CAN BE OMITTED TO SAVE GAS

In the `Vault.liquidate` function the following check is performed to verify that `_tokenOut` is the vault share.

    if (_tokenOut != address(this))
      revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));

But the same check check is performed inside the `_liquidatableBalanceOf(_tokenOut)` function call as well. Hence the above mentioned check inside the `Vault.liquidate` is a redundant check that can be omitted to save gas.

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L562-L566
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L830

## 6. ARRAY LENGTH MISMATCH IN THE `Vault.claimPrizes` COULD LEAD TO WASTE OF GAS

`Vault.claimPrizes` is used to calim the prizes for the winners. There are two arrays used in the function as shown below:

    address[] calldata _winners,
    uint32[][] calldata _prizeIndices,

These two arrays are used iterate through inside a nested `for` loop. But before the `for` loop the length of the above two arrays are not checked for equality. Hence if there is an array length mismath the transaction will revert after entering the `for` loop thus leading to a considerable gas loss to the `_claimer`.

Hence it is recommended to check `_winners.length == _prizeIndices.length` at the beginning of the function before entering the `for` loop.

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L618-L629
