# **PoolTogether QA Report**

## **Table of Contents**

|       | Issue                                                                        |
|-------|------------------------------------------------------------------------------|
| QA-01 | PrizePool.sol: `setDrawManager()` function should be removed since it's never going to be used |
| QA-02 | Users have no way of revoking their signatures                               |
| QA-03 | Measures should be taken to improve readability of code                      |
| QA-04 | `safeApprove()` is deprecated and should not be used when possible           |
| QA-05 | Team _specified_ deprecated functions should not reach main production       |
| QA-06 | Setter functions should include an equality check                            |

## QA-01 - PrizePool.sol: `setDrawManager()` function should be removed since it's never going to be used

### Vulnerability Details

The function itself checks if the drawManager has being set, take a look at [setDrawManager()](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L296-L306) below:

```solidity
  function setDrawManager(address _drawManager) external {
    if (drawManager != address(0)) {
      revert DrawManagerAlreadySet();
    }
    drawManager = _drawManager;

    emit DrawManagerSet(_drawManager);
  }
```

But the issue is that in the constructor the drawManager has already being set, check [L278](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L278) of prizepool.sol and as such there would be no need for this function since t

### Recommendation

If the function is not going to be used it should be removed

## QA-02 - Users have no way of revoking their signatures

### Vulnerability Details

In [Vault.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol), users can use permit to approve other users as spenders.
Currently, users have no way of revoking their signatures once it is signed.
This could result in a misuse of approvals the scenario could be in the lines of:

- Alice provides a signature to approve Bob to spend 100 tokens.
- Before `deadline` is passed, Bob's account is hacked.
- As Alice has no way of revoking the signature, Bob's account uses the signature to approve and spend Alice's tokens.

### Recommendation

Implement a function for users to revoke their own signatures.

## QA-03 - Measures should be taken to improve readability of code

### Vulnerability Details

Take a look at the [TwabLib.sol](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol) contract, and let's use the `getPreviousOrAtObservation()` and it's internal counterpart []() below:
Take a look at the [TwabLib.sol](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol) contract, and let's use the `getPreviousOrAtObservation()` and it's internal counterpart [`_getPreviousOrAtObservation()`](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L448-L534) below:

```solidity
  /**
   * @notice Looks up the newest observation before or at a given timestamp.
   * @dev If an observation is available at the target time, it is returned. Otherwise, the newest observation before the target time is returned.
   * @param _observations The circular buffer of observations
   * @param _accountDetails The account details to query with
   * @param _targetTime The timestamp to look up
   * @return prevOrAtObservation The observation
   */
  function getPreviousOrAtObservation(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) internal view returns (ObservationLib.Observation memory prevOrAtObservation) {
    return _getPreviousOrAtObservation(PERIOD_OFFSET, _observations, _accountDetails, _targetTime);
  }


  /**
   * @notice Looks up the newest observation before or at a given timestamp.
   * @dev If an observation is available at the target time, it is returned. Otherwise, the newest observation before the target time is returned.
   * @param _observations The circular buffer of observations
   * @param _accountDetails The account details to query with
   * @param _targetTime The timestamp to look up
   * @return prevOrAtObservation The observation
   */
  function _getPreviousOrAtObservation(
    uint32 PERIOD_OFFSET,
    ObservationLib.Observation[MAX_CARDINALITY] storage _observations,
    AccountDetails memory _accountDetails,
    uint32 _targetTime
  ) private view returns (ObservationLib.Observation memory prevOrAtObservation) {
    uint32 currentTime = uint32(block.timestamp);


    uint16 oldestTwabIndex;
    uint16 newestTwabIndex;


    // If there are no observations, return a zeroed observation
    if (_accountDetails.cardinality == 0) {
      return
        ObservationLib.Observation({ cumulativeBalance: 0, balance: 0, timestamp: PERIOD_OFFSET });
    }


    // Find the newest observation and check if the target time is AFTER it
    (newestTwabIndex, prevOrAtObservation) = getNewestObservation(_observations, _accountDetails);
    if (_targetTime >= prevOrAtObservation.timestamp) {
      return prevOrAtObservation;
    }


    // If there is only 1 actual observation, either return that observation or a zeroed observation
    if (_accountDetails.cardinality == 1) {
      if (_targetTime >= prevOrAtObservation.timestamp) {
        return prevOrAtObservation;
      } else {
        return
          ObservationLib.Observation({
            cumulativeBalance: 0,
            balance: 0,
            timestamp: PERIOD_OFFSET
          });
      }
    }


    // Find the oldest Observation and check if the target time is BEFORE it
    (oldestTwabIndex, prevOrAtObservation) = getOldestObservation(_observations, _accountDetails);
    if (_targetTime < prevOrAtObservation.timestamp) {
      return
        ObservationLib.Observation({ cumulativeBalance: 0, balance: 0, timestamp: PERIOD_OFFSET });
    }


    ObservationLib.Observation memory afterOrAtObservation;
    // Otherwise, we perform a binarySearch to find the observation before or at the timestamp
    (prevOrAtObservation, afterOrAtObservation) = ObservationLib.binarySearch(
      _observations,
      newestTwabIndex,
      oldestTwabIndex,
      _targetTime,
      _accountDetails.cardinality,
      currentTime
    );


    // If the afterOrAt is at, we can skip a temporary Observation computation by returning it here
    if (afterOrAtObservation.timestamp == _targetTime) {
      return afterOrAtObservation;
    }


    return prevOrAtObservation;
  }
```

Grep the comments before each instance of the function, they've been completely duplicated, also note that this practise is rampant within the linked contract.

### Recommendation

One comment is enough and can easily be linked as is done in for example Vault.sol where most comments are referenced to the inherited IERC4626

## QA-04 - `safeApprove()` is deprecated and should not be used when possible

### Vulnerability Details

Thr [Vault.sol](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol) contract
OpenZeppelin's safeapprove() has issues similar to the ones found in [IERC20.approve()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-), and its usage is discouraged, and has been deprecated by OZ.

Whenever possible, use [safeIncreaseAllowance()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20-safeIncreaseAllowance-contract-IERC20-address-uint256-) and [safeDecreaseAllowance()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20-safeDecreaseAllowance-contract-IERC20-address-uint256-) instead.

### Recommendation

As stated in _Vulnerability Details_ section

## QA-05 - Team _specified_ deprecated functions should not reach main production

### Vulnerability Details

Take a look at these lines, [UD34x4.sol#L60-L71](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/UD34x4.sol#L60-L71)

```solidity


/// @notice Alias for the `convert` function defined above.
/// @dev Here for backward compatibility. Will be removed in V4.
function fromUD34x4(UD34x4 x) pure returns (uint128 result) {
  result = convert(x);
}


/// @notice Alias for the `convert` function defined above.
/// @dev Here for backward compatibility. Will be removed in V4.
function toUD34x4(uint128 x) pure returns (UD34x4 result) {
  result = convert(x);
}
```

As clearly stated the two aliases above should be removed in V4 but this is V5 and they still exist.

### Recommendation

Remove the aliases if they are _deprecated_

## [QA-06] - Setter functions should include an equality check

### Vulnerability Details

The `setClaimer` function in the provided contract lacks an equality check. This oversight could lead to unnecessary state changes and the emission of events, even when the incoming new value is the same as the old value. The function as it currently stands [is](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L636-L647):

```solidity
  /**
   * @notice Set claimer.
   * @param claimer_ Address of the claimer
   * @return address New claimer address
   */
function setClaimer(address claimer_) external onlyOwner returns (address) {
  address _previousClaimer = _claimer;
  _setClaimer(claimer_);

  emit ClaimerSet(_previousClaimer, claimer_);
  return claimer_;
}
```

In this configuration, the function is designed to update the `_claimer` variable and emit an event. However, there is no mechanism in place to check if the new value (`claimer_`) is different from the current value (`_claimer`). As a result, this function could initiate unnecessary state changes and emit events, leading to an inefficient use of gas.

### Recommendation

To avoid unnecessary state changes and events, an equality check should be included at the start of the function to enable an early return if `claimer_` equals `_claimer`. This would result in a more gas-efficient operation. The improved code could look something like this:

```diff
function setClaimer(address claimer_) external onlyOwner returns (address) {
+  if (claimer_ == _claimer) {
    return claimer_;
  }

  address _previousClaimer = _claimer;
  _setClaimer(claimer_);

  emit ClaimerSet(_previousClaimer, claimer_);
  return claimer_;
}
```
