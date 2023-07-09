a) _claimer for a vault is set explicitly by calling setClaimer() function. Until it is set explicitly, the state variable is not initialised. But, on the other hand claimPrizers can be called any time on the vault.

It is best to initiaize this _claimer during the construction process as that vault and claimer contracts are enforcing valid state when claimPrizes is called.

b) feeRecipient of ClaimPrizes is passed by the claimer. there is a need to validate feeRecipient to be non zero address.