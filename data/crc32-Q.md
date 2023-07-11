## `_yieldFeeRecipient` is never used
Inside the `Vault` contract there is defined a state variable `_yieldFeeRecipient` and also defined some setter and getter functions for it but this variable is never used in project.
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L227
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L704-L710
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L719
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1229-L1231
Use it correctly or remove it if it is unusable.
Note: Something else may have been used in the project instead of the it by mistake.
