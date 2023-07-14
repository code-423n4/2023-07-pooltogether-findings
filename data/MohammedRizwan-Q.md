  ## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | Use of outdated prb-math library | 1 |
| [L&#x2011;02] | draft-ERC20Permit.sol is deprecated by openzeppelin | 1 |

### [L&#x2011;01]  Use of outdated prb-math library
The contracts have heavily used prb-math library but the library version used in contracts is v3.3.0 which is outdated. prb-math library is updated to latest version v4.01 which comes with features and optimizations.

There is 1 instance of this issue:
All contracts: https://github.com/PaulRBerg/prb-math/blob/1edf08dd73eb1ace0042459ba719b8ea4a55c0e0/package.json#L4

### Recommended Mitigation steps
Update the prb-math library to v4.01.

### [L&#x2011;02]  draft-ERC20Permit.sol is deprecated by openzeppelin
Vault.sol has used draft-ERC20Permit.sol in contract but Openzeppelin has deprecated draft-ERC20Permit.sol contract in v4.9.0. 
Reference link:- https://github.com/OpenZeppelin/openzeppelin-contracts/releases

There is 1 instance of this issue:
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L5

### Recommended Mitigation steps
Use ERC20Permit.sol instead of draft-ERC20Permit.sol.
