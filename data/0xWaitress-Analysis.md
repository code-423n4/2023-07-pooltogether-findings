I am submitting a report since i dont think this is an issue but would find it worth a second thought in terms of design:


## Summary: 
the smoothing curve combined with calculation of _getVaultPortion may create confusion on the initial lower-than-expected probability of winning (In the initial stage of joining, vault does not stand as much chance winning big prizes as vaults that join earlier with same contribution)

The vaultPortion (and its user collectively), is calculated by its contribution over the entire contribution for a particular draw. This vaultPortion represents the entitled expectedAmount and is translated into the probability of winning.

When a vault makes some contribution, the amount is split into contribution for multiple upcoming draws using the smoothing factor and formula. Generally speaking the contribution is bigger in the beginning and decays gradually with the passage of time.

https://dev.pooltogether.com/protocol/next/design/prize-pool#yield-smoothing-example

During the calculation of a particular tier prize, based on the expected occurrence, or the (expected) duration the prize, it would trace back up to the duration to find the cumulative contribution of a vault relative to the cumulative overall contribution to compute the winning probability.

a vault would basically have negligible change of winning grand prize at the beginning. Since its contribution for the entire duration is 0, except after its contribution which is recent. This is fair, since its TWAB contribution would then gradually move up with the passage of time, just like any other contributors has experienced. However, this delay due to smoothing factor + retrospective calculation of winning probability is not specified (explicitly), and might not be expected by contributor. 

## Recommendation
Consider evenly distribute the contribution of a newly joined vault over a period of X draws (where X covers some 1 prize cycle, 365 in this set up). 

The smoothing factor is nice, as it gets the new comer started winning smaller prizes as soon as it participated, however it seems to create an infinite residuals and potentially leave accounting problems for the future. It is advised to format the vesting/distribution schedule on a rigid percentage similar to the tier probability, such that the contribution runs completely at a finite number of draws.

### Time spent:
8 hours