1.Combine the calculation of `lastClosedDrawId + 1` and store it in a local variable to reduce gas cost consumption.
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L311#L330

2.delete local variable to save more gas.
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L364#L273

```
-   uint64 openDrawStartedAt_ = _openDrawStartedAt();

    _nextDraw(_nextNumberOfTiers, uint96(_contributionsForDraw(lastClosedDrawId + 1)));

    _winningRandomNumber = winningRandomNumber_;
    claimCount = 0;
    canaryClaimCount = 0;
    largestTierClaimed = 0;
-   _lastClosedDrawStartedAt = openDrawStartedAt_;
+   _lastClosedDrawStartedAt = _openDrawStartedAt();
    _lastClosedDrawAwardedAt = uint64(block.timestamp);
```

3.i++ into unchecked{} save more gas.
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144#L154
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68#L70