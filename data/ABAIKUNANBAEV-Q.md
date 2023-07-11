Problem detail: _tokenOut parameter in liquidate() function can be omitted as it's used only for checks.

In Vault.sol, when trying to liquidate() yield, liquidator has to specify _tokenOut parameter that's supposed to be always address(this). But address(this) is not the address of the token, it's just the Vault smart contract address and this is used only for checks that are not needed at all as shares of the smart contract are not the ERC standard token or some other token. 

Code snippet:

Check on whether _tokenOut is address(this)
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L562

Then it's used as the parameter for getting liquidatableBalanceOf(_tokenOut):

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L829-837

And this function itself just checks _availableYield to liquidate and _tokenOut is again used only for checks

Recommendation: remove _tokenOut parameter as user will not get any ERC standard token but minted shares of the Vault