1 - The Vault.setClaimer() should validate is not a zero address
==

The function [Vault.setClaimer()](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L641) should validate that the claimer address is not a zero address. Since the claimer is the unique entity who claims the winner prizers, it is crucial for the winner that the claimer is set up correctly to claim the winners prizes.

2 - Validates that the `hooks.implementation.beforeClaimPrize()` returns a non zero address
==

The [hooks.implementation.beforeClaimPrize()](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1053C19-L1053C85) is managed by an external user and it may return a zero address, causing that the claim price proccess to fail because the [safeTransfer to zero address](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L832) will fail. If the [hooks.implementation.beforeClaimPrize()](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1053C19-L1053C85) function returns a zero address it should use the `_recipient` [given by the claimer](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1055).


3 - When the `prizePool` is assigned to the `Vault`, validates that `prizePool` and `Vault` are using the same `TwabController`
==

The [prizePool is registered](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L273) when the vault is created. The problem is that there is not any validation that the registered [twabController](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L272) in the `prizePool` is the same that the the [vault is using](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L271). If the vault is using another `TwabController` the winners can't claim prizes because their balances are not counted since `prizePool` manages another `TwabController`.

4 - The liquidator can be frontrunned while is depositing `prizeTokens` to the `prizePool`
==

The liquidator should introduce `prizeTokens` to the `prizePool` before he calls the [Vault.liquidate()](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L570) function. The problem is that a malicious actor can be frontrunned and make the introduced [prizeTokens to be accumulated to another vault](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L317). The liquidator can lost his `prizeTokens`.