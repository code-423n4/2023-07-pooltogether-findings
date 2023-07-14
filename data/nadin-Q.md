# QA Report
## Summary
Total 02 Low and 07 Non-Critical
### Low Risk Issues
- [L-01] Upgrade to Solidity v0.8.19 and implement operators for UDVTs
- [L-02] In `Vault.sol` , `_currentExchangeRate()` should change the code to be more compact
## [L-01] Upgrade to Solidity v0.8.19 and implement operators for UDVTs
- Using the more updated version of Solidity can enhance security. As described in https://github.com/ethereum/solidity/releases, Version `0.8.19` is the latest version of Solidity, which "contains a fix for a long-standing bug that can result in code that is only used in creation code to also be included in runtime bytecode". To be more secured and more future-proofed, please consider using Version `0.8.19` for the following contracts.
- [Implement operators for UDVTs](https://github.com/PaulRBerg/prb-math/issues/167)

## [L-02] In `Vault.sol` , `_currentExchangeRate()` should change the code to be more compact
### Description:
```
File: Vault.sol
1178:    if (_withdrawableAssets > _totalSupplyToAssets) {
1179:      _withdrawableAssets = _withdrawableAssets - (_withdrawableAssets - _totalSupplyToAssets); // @audit _withdrawableAssets = _totalSupplyToAssets
```
[Vault.sol#L1179](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1179)
### Non-Critical Issues
- [N-01] Draft openzeppelin dependencies
- [N-02] SOLIDITY VERSION 0.8.19 CAN BE USED
- [N-03] INCLUDE THE PROJECT NAME AND DEVELOPMENT TEAM INFORMATION IN THE CONTRACT TO INCREASE THE POPULARITY AND TRUSR OF USERS
- [N-04] Use SMTChecker
- [N-05] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
- [N-06] Use named parameters for mapping type declarations
- [N-07] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
## [N-01] Draft openzeppelin dependencies
### Description:
- The current import of `draft-ERC20Permit.sol` is deprecated as [EIP-2612](https://eips.ethereum.org/EIPS/eip-2612) is final as of 2022-11-01. 
### Recommendation:
Consider using the latest version of the OpenZeppelin ERC20Permit contract.
```
File: Vault.sol
--- 5: import { ERC20Permit, IERC20Permit } from "openzeppelin/token/ERC20/extensions/draft-ERC20Permit.sol";
+++ 5: import { ERC20Permit, IERC20Permit } from "openzeppelin/token/ERC20/extensions/ERC20Permit.sol";
```
[Vault.sol#L5](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L5)

## [N-02] SOLIDITY VERSION 0.8.19 CAN BE USED
Using the more updated version of Solidity can enhance security. As described in https://github.com/ethereum/solidity/releases, Version `0.8.19` is the latest version of Solidity, which "contains a fix for a long-standing bug that can result in code that is only used in creation code to also be included in runtime bytecode". To be more secured and more future-proofed, please consider using Version `0.8.19` for the following contracts.
CONTEXTS: ALL CONTRACTS

## [N-03] INCLUDE THE PROJECT NAME AND DEVELOPMENT TEAM INFORMATION IN THE CONTRACT TO INCREASE THE POPULARITY AND TRUSR OF USERS
### Recommendation: Use form like FraxFinance project
https://github.com/FraxFinance/frxETH-public/blob/7f7731dbc93154131aba6e741b6116da05b25662/src/sfrxETH.sol#L4-L24

## [N-04] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [N-05] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
```
File: TieredLiquidityDistributor.sol
205:  mapping(uint8 => Tier) internal _tiers;
```
[TieredLiquidityDistributor.sol#L205](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L205)
```
File: DrawAccumulatorLib.sol
49:    mapping(uint256 => Observation) observations;
```
[DrawAccumulatorLib.sol#L49](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L49)
```
File: PrizePool.sol
202:  mapping(address => DrawAccumulatorLib.Accumulator) internal vaultAccumulator;
206:  mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))
210:  mapping(address => uint256) internal claimerRewards;
```
[PrizePool.sol#L202-L210](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L202-L210)
```
File: TwabController.sol
36:  mapping(address => mapping(address => TwabLib.Account)) internal userObservations;
39:  mapping(address => TwabLib.Account) internal totalSupplyObservations;
42:  mapping(address => mapping(address => address)) internal delegates;
```
[TwabController.sol#L36-L42](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L36-L42)
```
File: Vault.sol
236:  mapping(address => VaultHooks) internal _hooks;
```
[Vault.sol#L236](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L236)
```
File: VaultFactory.sol
36:  mapping(Vault => bool) public deployedVaults;
```
[VaultFactory.sol#L36](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L36)


## [N-06] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18
Cases already listed in N-05

## [N-07] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
### Description:
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.
#### constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.