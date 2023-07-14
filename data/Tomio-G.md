1.
Title: `>=` is cheaper than `>`

Impact:
Strict inequalities (`>`) are more expensive than non-strict ones (`>=`). This is due to some supplementary checks (ISZERO, 3 gas)

Proof of Concept:
[Claimer.sol#L210](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L210)
[Vault.sol#L315](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L315)

Recommended Mitigation Steps:
Consider using `>=` instead of `>` to avoid some opcodes
________________________________________________________________________

2.
Title: function `estimatedClaimCount()` gas improvement on returning value

Proof of Concept:
[TierCalculationLib.sol#L137](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L137)

Recommended Mitigation Steps:
by set `count` in returns L#136 and delete L#137 can save gas

```
function estimatedClaimCount(
    uint8 _numberOfTiers,
    uint16 _grandPrizePeriod
  ) internal pure returns (uint32 count) { //@audit-info: set here
```
________________________________________________________________________
