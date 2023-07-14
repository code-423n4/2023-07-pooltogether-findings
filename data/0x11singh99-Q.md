## [L-01] Unsafe downcasting can result in unwanted number.

### Instance#1:

```solidity
File: src/libraries/LinearVRGDALib.sol
110: return ud2x18(uint64(uint256(wadExp(maxDiv / 1e18))));
    //@audit low unsafe down casting 256 to 64
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L110

Recommendation : Use openzeppelin SafeCast library.

### Instance#2: Unsafe downcasting from uint256 to uint96

```solidity

 72:   uint96 feePerClaim = uint96(
      _computeFeePerClaim(
        _computeMaxFee(tier, prizePool.numberOfTiers()),
        claimCount,
        prizePool.claimCount()
      )
 77:   );
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L71C1-L78C7

## [L-02] All address should be checked for address(0) in constructor specially for immutables

Once immutable set it is fixed if need to update or wrong set then deployer need to redeploy

### Instance#1: Check `_prizePool` for address(0) before assigning it to immutable `prizePool`

```solidity
File: src/Claimer.sol
37: constructor(
    PrizePool _prizePool,
    uint256 _minimumFee,
    uint256 _maximumFee,
    uint256 _timeToReachMaxFee,
    UD2x18 _maxFeePortionOfPrize
  ) {
44:   prizePool = _prizePool;
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L37-L44

# Non Critcal Issues

## [NC-01] : Named return is confusing

return from function explicitly make code readable

```solidity
File: src/Claimer.sol

60:  function claimPrizes(
    Vault vault,
    uint8 tier,
    address[] calldata winners,
    uint32[][] calldata prizeIndices,
    address _feeRecipient
66: ) external returns (uint256 totalFees) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L60C2-L66C43