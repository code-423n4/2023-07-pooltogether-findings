# Internal call can be avoided in Vault's permit implementation

Inside [`Vault.sol`](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol) contract, there is a [`_permit()`](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1103) function simply proxies all the arguments to actual implementation `asset._permit()`.

There is three instances where gas can be saved by replacing:
`_permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);` to
`IERC20Permit(asset())._permit(msg.sender, address(this), _assets, _deadline, _v, _r, _s);`.

Issue exists at [line 435](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L435), [line 468](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L468) and [line 502](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L502).

This function is only used in these three places, and in all occurrences `_owner` and `_spender` is set to `msg.sender` and `address(this)` respectively, so passing them as parameters is wasteful. 

