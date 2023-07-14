## Advanced Analysis Report for [PoolTogether](https://github.com/code-423n4/2023-07-pooltogether)

### Overview 
- [PoolTogether](https://github.com/code-423n4/2023-07-pooltogether) is a prize savings protocol where the yield on deposits is awarded periodically as random prizes. It's a gamification layer that can be added to any yield-bearing asset. The version 5 of the protocol is fully autonomous, automated, and permissionless.

### Understanding the Ecosystem: 
- [PoolTogether](https://github.com/code-423n4/2023-07-pooltogether) V5 allows anyone to add new assets or yield sources to the protocol by adding new vaults. The yield from each Vault is converted to ``POOL`` by the Liquidator. The Draw Auction is the mechanism that pushes new draws to the Prize Pool.

### Codebase Quality Analysis:
- The codebase is well-structured and modular, with clear separation of concerns. It uses libraries like ``prb-math``, ``openzeppelin``, ``solmate``, and ``ring-buffer-lib``. The codebase has a high test coverage of ~99.9%, indicating a robust testing framework.

### Architecture Recommendations: 
- The architecture is well-designed with a focus on modularity and reusability. However, the complexity of the system necessitates further testing and auditing to ensure the continued security of the system.

### Centralization Risks: 
- The system is fully autonomous with no admin controls, which reduces centralization risks. However, the Draw Auction has the privileged ability to withdraw reserve from the Prize Pool, which could be a potential centralization risk.

### Mechanism Review: 
- The protocol uses a Variable-Rate Gradual Dutch Auction to price the claiming fees. The Prize Pool receives liquidity from Vaults and distributes the liquidity across future draws with a low-pass filter. The Prize Pool distributes prizes by generating pseudo-random numbers for each vault/account/prize combination.

### Systemic Risks:
- The complexity of the system and the interaction of its components pose potential systemic risks. Unexpected behaviour or vulnerabilities could arise from the interaction of the components.

### Areas of Concern
- The main areas of concern are ensuring the prize pool doesn't award too much prize liquidity, precise tracking of prize liquidity by the prize pool, potential manipulation of the Twab Controller to improve odds of winning, and ensuring the security of Vaults.


### Codebase Analysis
- The codebase consists of 14 contracts with a total of 3335 SLoC. It uses 4 external imports and has about 20 separate interfaces and struct definitions. The codebase uses composition over inheritance and has 12 external calls.

### Recommendations
- Continued regular audits and further testing are recommended due to the complexity of the system. It's also recommended to monitor the Draw Auction's ability to withdraw reserve from the Prize Pool to mitigate centralization risks. Use [Defender](https://defender.openzeppelin.com/) and [tenderly](https://dashboard.tenderly.co/login) for continued monitoring. 

### Contract Details
The main contracts in scope are:

- [Claimer.sol](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol): Incentivizes prize claims for a Prize Pool.
- [TieredLiquidityDistributor.sol](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol): Distributes liquidity as tiered prizes.
- [DrawAccumulatorLib.sol](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol): Distributes liquidity over future draws using exponential smoothing.
- [TierCalculationLib.sol](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol): Computes tier odds, prize counts, and winners.
- [PrizePool.sol](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol): Distributes liquidity as tiered prizes.
- [ObservationLib.sol](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol): Tracks a ring buffer of balance observations.
- [TwabLib.sol](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol): Provides an accounts struct and logic to mint, burn and transfer balance and measure twabs.
- [TwabController.sol](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol): Allows contracts to track balances and look up the time-weighted average balance of an account.
- [Vault.sol](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol): An ERC-4626 compatible vault that uses a Twab Controller to track balances.
- [VaultFactory.sol](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol): Provides a mean to easily create new Vaults and find them.

### Conclusion
- [PoolTogether](https://github.com/code-423n4/2023-07-pooltogether) V5 is a complex, well-structured, and autonomous protocol. While it has been designed with security in mind, the complexity of the system necessitates regular audits and testing. The protocol's unique mechanisms and mathematical models add to its novelty, but also require careful scrutiny to ensure they function as intended.

### Time spent:
16 hours