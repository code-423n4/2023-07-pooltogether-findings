## [G-01] - Vault.sol#_currentExchangeRate() - instead of having the calculation ``_withdrawableAssets = _withdrawableAssets - (_withdrawableAssets - _totalSupplyToAssets);`` which would just result in the ``_totalSupplyToAssets``, just set it to the total supply.

## [G-02] - Vault.sol#_currentExchangeRate() - you can move the if check on line 1182 further up, but reversing the condition to return the ``_assetUnit`` and avoid unnecessary computation like so:
``if (_totalSupplyAmount == 0 || _withdrawableAssets == 0) return _assetUnit;``

## [G-03] - Vault.sol#_liquidatableBalanceOf() - this is an internal function mainly called by the ``liquidate()`` function which already does the ``if (_tokenOut != address(this))`` check. Remove the same check from ``_liquidatableBalanceOf()`` to save gas.