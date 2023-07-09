a) _claimer for a vault is set explicitly by calling setClaimer() function. Until that point, the state variable is not initialised. But, on the other hand claimPrizers can be called any time on the vault.
It is best to initiaize this _claimer in the constructor.

b) 