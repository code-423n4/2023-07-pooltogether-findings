a) _claimer for a vault is set explicitly by calling setClaimer() function. Until it is set explicitly, the state variable is not initialised. But, on the other hand claimPrizers can be called any time on the vault.

It is best to initiaize this _claimer during the construction process so that vault and claimer contracts are enforcing valid state when claimPrizes is called.

b) feeRecipient of ClaimPrizes function in claimer contract is passed by the caller. there is a need to validate feeRecipient to be non zero address.

c) _setYieldFeePercentage should validate for a range to represent 0% to 100% to represent the valid range. Ideally, the real business viable percentage will make more sense.

d) setYieldFeeRecipient function should check for incoming address param to be non zero address.

e) setClaimer function should check for incoming address param to be non zero address