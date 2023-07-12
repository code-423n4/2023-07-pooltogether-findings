## Project description

The protocol fundamentally facilitates users with yield farming.
Users can create Vaults via a provided VaultFactory, and maintain or delegate their ownership.

Once a Vault is created (this happens via a VaultFactory contract), anybody can deposit ERC-20 assets (i.e. USDC), and receive back Vault-issued ERC-20 tokens in exchange, representing the depositor's share of assets managed by the vault.

When a user deposits assets to the Vault, the Vault immediately forwards them to the YieldVault (set at Vault creation).
Also, it outsources the keeping track of Vault token balances to a central contract, TwabController, that provides the additional functionality to keep track of historical balances.

TwabController tracks user balances of vault tokens. It allows for delegating tokens to another account.
Within TwabController, the sum of tokens held by an account and those delegated by others contribute is the baseline used to distribute the harvested yield.

Harvested yields are distributed by the PrizePool contract through a Claimer contract, whose primary goal is to aggregate prize claims to save gas, and prizes are randomly assigned to users with odds proportional to the ratio between their time-weighted balance of Vault tokens and the vault time-weighted Vault token supply.

## Suggestions
- as mentioned in a dedicated finding, the Vault ownership poses a serious centralization risk, potentially even on a non-trusted entity
- another more obvious centralization risk comes from the pool closing & random number generation. I acknowledge however how this is a tradeoff to avoid MEV

### Time spent:
20 hours