L-01: vault._mint() forcing conversion to uint96 may result in missing values

The current protocol `Vault.sol` has `uint256` for shares, but `uint96` is used in TwabController.sol.
So in `mint/burn/transfer`, it is forced to convert to `uint96
This causes the value to exceed `type(uint96).max` if `asset.decimals()>18`, resulting in a loss of value.
Suggest to add a method like `SaftToUint96()` to revert if it is larger than `uint96`.
```solidity
  function _burn(address _owner, uint256 _shares) internal virtual override {
-   _twabController.burn(_owner, uint96(_shares));
+   _twabController.burn(_owner, _shares.SafeToUint96());
    _updateExchangeRate();

    emit Transfer(_owner, address(0), _shares);
  }
```
note:mint()/transfer() same problem

L-02: targetOf() incorrectly uses `_token ! = _liquidationPair.tokenIn()`
Currently `targetOf()` uses `_liquidationPair.tokenIn()` but the actual comparison in `liquidate()` is `_tokenIn ! = address(_prizePool.prizeToken())`
So it's `_prizePool.prizeToken()` that should be used.

```solidity
  function targetOf(address _token) external view returns (address) {
-   if (_token != _liquidationPair.tokenIn()) revert TargetTokenNotSupported(_token);
+   if (_tokenIn != address(_prizePool.prizeToken())) revert TargetTokenNotSupported(_token);
    return address(_prizePool);
  }
```

L-03: setLiquidationPair()` may not be able to set the old `liquidationPair`

Use the `safeApprove()` method in `setLiquidationPair()` to set the allowance
The `safeApprove()` method requires the current allowance to be 0 for the setting to succeed.
If the `liquidationPair_` has been used before, and this is the second time it is used, and there are residual allowances, it may not be possible to set it.
suggest: clear the allowance to 0 before setting.

```solidity
  function setLiquidationPair(
    LiquidationPair liquidationPair_
  ) external onlyOwner returns (address) {
    if (address(liquidationPair_) == address(0)) revert LPZeroAddress();

    IERC20 _asset = IERC20(asset());
    address _previousLiquidationPair = address(_liquidationPair);

    if (_previousLiquidationPair != address(0)) {
      _asset.safeApprove(_previousLiquidationPair, 0);
    }

+   _asset.safeApprove(address(liquidationPair_), 0);
    _asset.safeApprove(address(liquidationPair_), type(uint256).max);

    _liquidationPair = liquidationPair_;

    emit LiquidationPairSet(liquidationPair_);
    return address(liquidationPair_);
  }
```


L-04: in _withdraw() after `_burn()` then execute `_yieldVault.withdraw()` ,it may lead `_updateExchangeRate()` inaccuracy

In `withdraw()` execute `_burn()` first, in `_yieldVault.withdraw()`, code as follows.
```solidity
  function _withdraw(
    address _caller,
    address _receiver,
    address _owner,
    uint256 _assets,
    uint256 _shares
  ) internal virtual override {
....

@>  _burn(_owner, _shares);

@>  _yieldVault.withdraw(_assets, address(this), address(this));
    SafeERC20.safeTransfer(IERC20(asset()), _receiver, _assets);

    emit Withdraw(_caller, _receiver, _owner, _assets, _shares);
  }
```

in `_burn()`method will execute `_updateExchangeRate()`
```solidity
  function _burn(address _owner, uint256 _shares) internal virtual override {
@>  _twabController.burn(_owner, uint96(_shares));
@>  _updateExchangeRate();

    emit Transfer(_owner, address(0), _shares);
  }
```

then in `_currentExchangeRate()` will read `_yieldVault.maxWithdraw`():
```solidity
  function _currentExchangeRate() internal view returns (uint256) {
    uint256 _totalSupplyAmount = _totalSupply();
    uint256 _totalSupplyToAssets = _convertToAssets(
      _totalSupplyAmount,
      _lastRecordedExchangeRate,
      Math.Rounding.Down
    );

@>  uint256 _withdrawableAssets = _yieldVault.maxWithdraw(address(this));

    if (_withdrawableAssets > _totalSupplyToAssets) {
      _withdrawableAssets = _withdrawableAssets - (_withdrawableAssets - _totalSupplyToAssets);
    }

    if (_totalSupplyAmount != 0 && _withdrawableAssets != 0) {
      return _withdrawableAssets.mulDiv(_assetUnit, _totalSupplyAmount, Math.Rounding.Down);
    }

    return _assetUnit;
  }  
```

This results in a wrong order :

1. vault.burn() reduces `shares`
2. read `shares` and `total assets` to calculate the exchange rate
3. `_yieldVault.withdraw()` reduces `total assets`

The result is that the second step of reading `total assets` is not correct, and the third step should be put before the second step.

Suggestion.

```solidity
  function _withdraw(
    address _caller,
    address _receiver,
    address _owner,
    uint256 _assets,
    uint256 _shares
  ) internal virtual override {
....

+   _yieldVault.withdraw(_assets, address(this), address(this));

    _burn(_owner, _shares);

-   _yieldVault.withdraw(_assets, address(this), address(this));
    SafeERC20.safeTransfer(IERC20(asset()), _receiver, _assets);

    emit Withdraw(_caller, _receiver, _owner, _assets, _shares);
  }
```

L-05: `liquidate()` at risk of sandwich attack

Currently `liquidate()` requires `_amountOut >= _vaultAssets` to execute `_yieldVault.deposit()` if there is a residual balance in the contract.
This presents some risk for a sandwich attack

Example.

1. A malicious user monitors `memory pool`, monitors `_liquidationPair` transfer assets to the contract, and executes `liquidate()`.
2. front-run the transaction in step 1, transferring an `asset` larger than `_amountOut`.
3. wait until the first transaction, because `_amountOut >= _vaultAssets`, will not execute `_yieldVault.deposit()`, the funds are still in the contract
4. Malicious users are using the balance left in the contract by executing `vault.deposit()`.

Suggest to remove `_amountOut >= _vaultAssets` restriction.

```solidity
  function liquidate(
    address _account,
    address _tokenIn,
    uint256 _amountIn,
    address _tokenOut,
    uint256 _amountOut
  ) public virtual override returns (bool) {
...

    uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));

-   if (_vaultAssets != 0 && _amountOut >= _vaultAssets) {
+   if (_vaultAssets != 0) {
      _yieldVault.deposit(_vaultAssets, address(this));
    }

    _mint(_account, _amountOut);

    return true;
  }  
```