## [L-01] - no check on the ``deployedVaults`` mapping in the VaultFactory can lead to duplicate vaults being created
Mitigation - add an if check to not recreate the same vault: ``` if(deployedVaults[_vault]) revert....```

## [L-02] - cases of the ``claimPrizes`` functions across the contracts do not check if the sizes of the arrays ``winners`` and ``prizeIndices`` are the same

## [I-01] - fix the comments on the PrizePool#withdrawClaimRewards to say 'rewards', not 'fees'