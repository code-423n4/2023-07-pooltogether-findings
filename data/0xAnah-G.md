# Gas Optimisations

## [G-01] Unused Imports
Some files were imported and were not used these files costs gas during deployment and generally this is bad coding practice
### 11 Instances

In the file below `E` was imported with the statement `import { SD59x18, toSD59x18, E } from "prb-math/SD59x18.sol";` but was not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L5

In the file below `wadMul`, `toWadUnsafe` and `unsafeWadDiv` were imported with the statement `import { wadExp, wadLn, wadMul, unsafeWadMul, toWadUnsafe, unsafeWadDiv, wadDiv } from "solmate/utils/SignedWadMath.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L6

In the file below `fromUD60x18` was imported with the statement `import { UD60x18, toUD60x18, fromUD60x18 } from "prb-math/UD60x18.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L6


In the file below `uMAX_UD60x18` was imported with the statement `import { UD60x18, uMAX_UD60x18 } from "prb-math/UD60x18.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/UD34x4.sol#L5


In the file below `E`, `toSD59x18` and `fromSD59x18` were imported with the statement `import { E, SD59x18, sd, toSD59x18, fromSD59x18 } from "prb-math/SD59x18.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L8


In the file below `UD60x18` and `ud` were imported with the statement `import { UD60x18, ud, toUD60x18, fromUD60x18, intoSD59x18 } from "prb-math/UD60x18.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L9


In the file below `toUD34x4` was imported with the statement `import { UD34x4, fromUD60x18 as fromUD60x18toUD34x4, intoUD60x18 as fromUD34x4toUD60x18, toUD34x4 } from "./libraries/UD34x4.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L12


In the file below `LiquidationPair` was imported with the statement `import { LiquidationPair } from "v5-liquidator/LiquidationPair.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L6


In the file below `E`, `toSD59x18` and `fromSD59x18` were imported with the statement `import { E, SD59x18, sd, toSD59x18, fromSD59x18 } from "prb-math/SD59x18.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L5


In the file below `intoSD59x18` was imported with the statement `import { UD60x18, ud, toUD60x18, fromUD60x18, intoSD59x18 } from "prb-math/UD60x18.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L6


In the file below `UD2x18` and `intoUD60x18` were imported with the statement `import { UD2x18, intoUD60x18 } from "prb-math/UD2x18.sol";` but were not used you should consider removing it from the imports.
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L7


In the file below `SD1x18` and `UNIT` were imported with the statement `import { SD1x18, unwrap, UNIT } from "prb-math/SD1x18.sol";` but were not used you should consider removing it from the imports.
- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L8


## [G-02] Making constant and Immutable variables private will save gas during deployment. 
When constants and immutable are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple.
### 3 Instances

- https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L18-#L29
```solidity
  /// @notice The Prize Pool that this Claimer is claiming prizes for
  PrizePool public immutable prizePool;

  /// @notice The maximum fee that can be charged as a portion of the smallest prize size. Fixed point 18 number
  UD2x18 public immutable maxFeePortionOfPrize;

  /// @notice The VRGDA decay constant computed in the constructor
  SD59x18 public immutable decayConstant;

  /// @notice The minimum fee that will be charged
  uint256 public immutable minimumFee;

  uint256 public immutable timeToReachMaxFee;

```
The code above should be refactored to:
```solidity
  /// @notice The Prize Pool that this Claimer is claiming prizes for
  PrizePool private immutable prizePool;

  /// @notice The maximum fee that can be charged as a portion of the smallest prize size. Fixed point 18 number
  UD2x18 private immutable maxFeePortionOfPrize;

  /// @notice The VRGDA decay constant computed in the constructor
  SD59x18 private immutable decayConstant;

  /// @notice The minimum fee that will be charged
  uint256 private immutable minimumFee;

  uint256 private immutable timeToReachMaxFee;

  function getConstants() public view
  returns (
    PrizePool,
    UD2x18,
    SD59x18,
    uint256,
    uint256
  )
  {
    return (
        prizePool,
        maxFeePortionOfPrize,
        decayConstant,
        minimumFee,
        timeToReachMaxFee
    )
  }

```


- https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L213-#L228
```solidity
  SD1x18 public immutable smoothing;

  /// @notice The token that is being contributed and awarded as prizes.
  IERC20 public immutable prizeToken;

  /// @notice The Twab Controller to use to retrieve historic balances.
  TwabController public immutable twabController;

  /// @notice The draw manager address.
  address public drawManager;

  /// @notice The number of seconds between draws.
  uint32 public immutable drawPeriodSeconds;

  /// @notice Percentage of prizes that must be claimed to bump the number of tiers.
  UD2x18 public immutable claimExpansionThreshold;
```
The code above should be refactored to:
```solidity
  SD1x18 private immutable smoothing;

  /// @notice The token that is being contributed and awarded as prizes.
  IERC20 private immutable prizeToken;

  /// @notice The Twab Controller to use to retrieve historic balances.
  TwabController private immutable twabController;

  /// @notice The number of seconds between draws.
  uint32 private immutable drawPeriodSeconds;

  /// @notice Percentage of prizes that must be claimed to bump the number of tiers.
  UD2x18 private immutable claimExpansionThreshold;

  function getConstants() public view
  returns (
    SD1x18,
    IERC20,
    TwabController,
    uint32,
    UD2x18
  ) {
    return (
        smoothing,
        prizeToken,
        twabController,
        drawPeriodSeconds,
        claimExpansionThreshold
    )
  }
  
```

- https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L23-#L31
```solidity
  /// @notice Allows users to revoke their chances to win by delegating to the sponsorship address.
  address public constant SPONSORSHIP_ADDRESS = address(1);

  /// @notice Sets the minimum period length for Observations. When a period elapses, a new Observation is recorded, otherwise the most recent Observation is updated.
  uint32 public immutable PERIOD_LENGTH;

  /// @notice Sets the beginning timestamp for the first period. This allows us to maximize storage as well as line up periods with a chosen timestamp.
  /// @dev Ensure that the PERIOD_OFFSET is in the past.
  uint32 public immutable PERIOD_OFFSET;
```
the above code should be refactored to:
```solidity
  /// @notice Allows users to revoke their chances to win by delegating to the sponsorship address.
  address private constant SPONSORSHIP_ADDRESS = address(1);

  /// @notice Sets the minimum period length for Observations. When a period elapses, a new Observation is recorded, otherwise the most recent Observation is updated.
  uint32 priavte immutable PERIOD_LENGTH;

  /// @notice Sets the beginning timestamp for the first period. This allows us to maximize storage as well as line up periods with a chosen timestamp.
  /// @dev Ensure that the PERIOD_OFFSET is in the past.
  uint32 private immutable PERIOD_OFFSET;


  function getConstants() public view 
  returns (
    address,
    uint32,
    uint32
  ) {
    return (SPONSORSHIP_ADDRESS, PERIOD_LENGTH, PERIOD_OFFSET)
  }
```

## [G-03] Declaring redundant variables
Some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.
### 3 Instances
- https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L704-#L710
```solidity
  function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {
    address _previousYieldFeeRecipient = _yieldFeeRecipient;
    _setYieldFeeRecipient(yieldFeeRecipient_);

    emit YieldFeeRecipientSet(_previousYieldFeeRecipient, yieldFeeRecipient_);
    return yieldFeeRecipient_;
  }
```
In the above code `_previousYieldFeeRecipient` variable is unnecessary we could save gas cost for `MLOAD` and `MSTORE` operations if we refactor the code to:
```solidity
  function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {

    emit YieldFeeRecipientSet(_yieldFeeRecipient, yieldFeeRecipient_);
    _setYieldFeeRecipient(yieldFeeRecipient_);
    return yieldFeeRecipient_;
  }
```
- https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L691-#L697
```solidity
  function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {
    uint256 _previousYieldFeePercentage = _yieldFeePercentage;
    _setYieldFeePercentage(yieldFeePercentage_);

    emit YieldFeePercentageSet(_previousYieldFeePercentage, yieldFeePercentage_);
    return yieldFeePercentage_;
  }
```
In the above code `_previousYieldFeePercentage` variable is unnecessary we could save gas cost for `MLOAD` and `MSTORE` operations if we refactor the code to:
```solidity
  function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {

    emit YieldFeePercentageSet(_yieldFeePercentage, yieldFeePercentage_);
    _setYieldFeePercentage(yieldFeePercentage_);
    return yieldFeePercentage_;
  }
```
- https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L641-#L647
```solidity
  function setClaimer(address claimer_) external onlyOwner returns (address) {
    address _previousClaimer = _claimer;
    _setClaimer(claimer_);

    emit ClaimerSet(_previousClaimer, claimer_);
    return claimer_;
  }
```
In the above code `_previousClaimer` variable is unnecessary we could save gas cost for `MLOAD` and `MSTORE` operations if we refactor the code to:
```solidity
  function setClaimer(address claimer_) external onlyOwner returns (address) {
    
    emit ClaimerSet(_claimer, claimer_);
    _setClaimer(claimer_);
    return claimer_;
  }
```