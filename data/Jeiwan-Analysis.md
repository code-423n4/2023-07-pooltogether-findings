The audited PoolTogether protocol can be described as an advanced ERC4626 vault with extra mechanics and features. The main `Vault` contract is compatible with the ERC4626 standard, however it adds multiple unique and tricky features.

Foremost, the `Vault` sets the share-to-asset exchange rate to 1 or less than 1. This means that a deposited amount of assets cannot be later withdrawn as a bigger amount. During withdrawing, users receive the exact same amount of assets as they deposited, besides the situations when the exchange rate goes below 1 (i.e. when the vault becomes undercollateralized).

Second, the yield generated and collected by the vault is not distributed among the depositors of the vault: it can only be liquidated. Liquidating yield means swapping it for an amount of prize tokens and contributing the prize tokens into the prize pool. The liquidator converts the yield into prize tokens and deposits them to the prize pool, while receiving an equal amount of shares as reward.

Another big change is that the vault contract doesn't mint and manage shares–this functionality is delegated to the `TwabController` contract.

The `PrizePool` contract is another core contract of the protocol. The contract receives the yield generated by the vault and converted to prize tokens. The prize tokens are then distributed among the depositors of the vault via daily draws. During each draw, the total amount of prize tokens is split into multiple tiers, with each tier having a unique odds of winning, a unique prize size, and a unique number of prizes. A draw is triggered by the draw manager, which is a restricted role that provides a winning random number for each draw. The number randomizes the winners of a draw, however the main factor that contributes to the chances of a depositor to win is the historical vault shares balance of the depositor.

The `TwabController` contract glues the vault and the prize pool. Its main purpose is to store the vault share balances of depositors and keep the record of their historical balances. The latter is implemented via TWAB, Time-Weighted Average Balances, a mechanism that's identical to Uniswap's Time-Weighted Average Price oracle.

The final core contract is `Claimer`. The contract allows anyone to claim prizes for all winners in a draw for a fee. The size of the fee is determined via a Linear Variable Rate Gradual Dutch Auction, which was invented by Paradigm. The auction increases the fee as more time passes since the time the draw was closed at. The claimers thus get a higher reward the later they claim prizes.


The codebase as a whole is a solid engineering construction that relies on the standards and battle-tested algorithms, while providing novelty in how they're used together. The vault implements the ERC4626 standard, while significantly modifying it. The prize pool and the TWAB controller contracts implement the battle-tested historical oracle algorithm invented and used by Uniswap. The claimer contract cleverly uses a Variable Rate Gradual Dutch Auction to incentivize prize payouts for winners of draws.

The protocol doesn't expose significant centralization risks. All the audited contracts are permissionless and don't implement restricted functions. The only exception is the `PrizePool.closeDraw` function that can only be called by the draw manager. To avoid draw results manipulation, a centralized actor is required to provide a random number.

The audit was performed using the standard technique of reading the entire codebase line-by-line, validating each line against the whitepaper and using the common sense, as well as bird's eye view reviewing of the codebase with the goal of finding discrepancies in the integrations of the contracts.

### Time spent:
30 hours