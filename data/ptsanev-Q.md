## [L-01] - no check on the ``deployedVaults`` mapping in the VaultFactory can lead to duplicate vaults being created
Mitigation - add an if check to not recreate the same vault: ``` if(deployedVaults[_vault]) revert....```

## [I-01] - fix the comments on the PrizePool#withdrawClaimRewards to say 'rewards', not 'fees'