**1.** The extra layer of validation at https://github.com/GenerationSoftware/pt-v5-vault/blob/main/src/Vault.sol#L955 is a good one but might not be necessary as _assetsDeposit can be declared directly without condition as there is no way it is equal to zero based on the previous condition on Line 942 and the calculation on Line 947. An exception is the unchecked{} code at L945 to L949 which should not have any influence on the outcome.

**2.** This line of code 
_withdrawableAssets = _withdrawableAssets - (_withdrawableAssets - _totalSupplyToAssets);
@ https://github.com/GenerationSoftware/pt-v5-vault/blob/main/src/Vault.sol#L1179
can be more efficiently written as 
_withdrawableAssets = _totalSupplyToAssets