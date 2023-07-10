Gas Issue #1. Use indexed events rather than non indexed events.

Events that have indexed variables actually use less gas than events that have non-indexed variables.
Several of the events that are used throughout the protocol only have 1 or 2 indexed variables.  Maximizing the use of indexed variables (up to 3) within these events can lower the cost of execution for the functions that emit them.  

All-in-all, there are: 
8 events within PrizePool.sol.  6 of these events could utilize the indexed keyword more than they currently do (line 158, 171, 176, 182, 188, 193). 
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L158-L193
 
1 unindexed event in TieredLiquidityDistributor.sol (line 41).
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L41

7 events inside of TwabController.sol,  6 of them can utilize indexed variables more than they currently do.  (line 53, 67, 83, 106, 114, 124)
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L53-L130
  
9 events within Vault.sol, 8 of these events can utilize the indexed keyword more than they currently do (line 147, 154, 160, 168, 175, 182, 191, 198).
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L147-L198


Total number of instances within protocol where events can use more indexed variables: 21.
Amount of gas savings per indexed event: 16 gas units.


=======================================================

Gas Issue #2. Use of memory instead of calldata within function parameters.

When dynamic variables are passed into a function, they are more efficient when labeled as calldata rather than memory. (Of course, these parameters can only be calldata if they are not altered within the function)  There are 2 instances within the protocol where function parameters marked as memory can be altered to calldata.

The deployVault function inside of VaultFactory.sol has two string parameters marked as memory, and these can be changed to calldata (this would make each deployment of Vault.sol cheaper).
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L57-L58

Amount of gas savings per instance:  690 units of gas.
