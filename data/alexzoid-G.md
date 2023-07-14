## [G-01] Public functions not used internally can be marked as external

`isVaultCollateralized()` function visibility at line https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L367 could be changed to `external`.
```solidity
  function isVaultCollateralized() external view returns (bool) {
```

## [G-02] Use `calldata` instead of `memory` in function parameters

In `setHooks()` for `VaultHooks` parameter at line https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L653
```solidity
  function setHooks(VaultHooks calldata hooks) external {
```

In `deployVault()` for `_name` and `_symbol` parameters at line https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L57-L58
```solidity
  function deployVault(
    IERC20 _asset,
    string calldata _name,
    string calldata _symbol,
    TwabController _twabController,
    ...
```