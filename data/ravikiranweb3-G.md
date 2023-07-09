1) ClaimPrizes:
Claimer contract's claimPrizes() accepts array for winners and prize Indices. Claim Prices is a heavy process as there could be indices from 1 to 15 and a number of winners. With more people on boarded into the platform, as winner count also grow, the size of overall computation can also grow.

it will be better if protocol enforces a limit on number of winners and price Indicies acceptable for processing in a single call.

