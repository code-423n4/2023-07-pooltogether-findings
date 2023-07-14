# QA Report

| Id                                                          | Issue                                         |
|-------------------------------------------------------------|-----------------------------------------------|
| [L-1](#l-1-enforce-offset-is-in-the-past)                   | Enforce offset is in the past                 |
| [L-2](#l-2-natspec-external-function-incomplete)            | NatSpec external function incomplete          |
| [L-3](#l-3-prevent-transfer-amount-of-zero)                 | Prevent transfer amount of zero               |
| [NC-1](#nc-1-natspec-typographical-error)                   | NatSpec typographical error                   |
| [NC-2](#nc-2-natspec-inconsistent-whitespace)               | NatSpec inconsistent whitespace               |
| [NC-3](#nc-3-natSpec-ambiguous-parameter-definition)        | NatSpec ambiguous parameter definition        |
| [NC-4](#nc-4-redundant-inline-comment)                      | Redundant inline comment                      |
| [NC-5](#nc-5-natspec-description-does-not-match-function)   | NatSpec description does not match function   |
| [NC-6](#nc-6-natspec-dev-comment-warning-function-removal)  | NatSpec dev comment warning function removal  |



## Low Risk 

### L-1 Enforce offset is in the past

The constructor for the `TwabController` contains the following NatSpec:

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L136
```
* @dev Ensure the periods offset is in the past, otherwise underflows will occur whilst calculating periods.
```

The requirement of the offset being in the past is not enforced programatically.

#### Recommendation

Insert a require statement to check `_periodOffset` is in the past (probably using a custom error rather than a string):
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L143
```
+   require( _periodOffset < block.timestamp, "Offest must be in the past");
```



### L-2 NatSpec external function incomplete

The NatSpec for the `external` function `getTierAccrualDurationInDraws()` is incomplete. 
When NatSpec is present for function, it is best to be complete and reasonably describe the purpose of the function beyond
what can be gleamed from the signature.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L549-L552
```
  /// @notice Returns the
  /// @return The number of draws
  function getTierAccrualDurationInDraws(uint8 _tier) external view returns (uint16) {
```

#### Recommendation
```
-  /// @notice Returns the
+  /// @notice Statistical estimation of time between tier payout, in draws.
```



### L-3 Prevent transfer amount of zero

The external function `withdrawClaimRewards` allows a claimer to withdraw any fees they have accrued. 
There is a check that the `_amount` is not greater than the rewards accrued by the claimer, an amount of zero passes through to the transfer,
where a transfer on the `prizeToken` would be performed.

Certain ERC20 token are know to revert on zero transfers, which ends up wasting even more gas than a pre-condition check.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L483-L493
```
  function withdrawClaimRewards(address _to, uint256 _amount) external {
```

#### Recommendation
Add a `require` check before starting any of the function internals, better to use a custom error, but the idea is:

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L484
```
+  require( _amount != 0, "Claim more than nothing");
```






## Non-Critical

### NC-1 NatSpec typographical Error

Instances: 3

Function NatSpec contains the incorrect spelling of emitted (missing a 't').

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L61
```
   * @notice Emited when a balance or delegateBalance is decreased.
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L75
```
   * @notice Emited when an Observation is recorded to the Ring Buffer.
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L117
```
   * @notice Emited when a Total Supply Observation is recorded to the Ring Buffer.
```

#### Recommendation
`Emited` -> `Emitted`



### NC-2 NatSpec inconsistent whitespace

Function NatSpec contains two whitespaces separates words, when a single space is consistently being used everywhere else.

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L300
```
   * @notice Looks up the newest observation  for a user.
```

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L315
```
   * @notice Looks up the oldest observation  for a user.
```

#### Recommendation
` ` -> ``



### NC-3 NatSpec ambiguous parameter definition

The function NatSpec is ambiguous regarding whether `_tierShares` is the total for all tiers or the shares per a tier.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L50
```
  /// @param _tierShares The number of shares allocated to prize tiers
```

`_tierShares` is shares per a tier.

#### Recommendation
```
-  /// @param _tierShares The number of shares allocated to prize tiers
+  /// @param _tierShares The number of shares allocated to each prize tier
```



### NC-4 Redundant inline comment

The comment line above `_consumeLiquidity` references `amount`, a variable that is not the subsequent line.
`amount` does appear later in the function.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L449-L452
```

    // `amount` is a snapshot of the reserve before consuming liquidity
    _consumeLiquidity(tierLiquidity, _tier, tierLiquidity.prizeSize);

```

#### Recommendation
Delete the comment.



### NC-5 NatSpec description does not match function

The NatSpec of the library function `fromUD60x18()` has the types around the wrong way, as it convert from `UD60x18` into `UD34x4`.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/UD34x4.sol#L26-L29
```
/// @notice Casts an UD34x4 number into UD60x18.
/// @dev Requirements:
/// - x must be less than or equal to `uMAX_UD2x18`.
function fromUD60x18(UD60x18 x) pure returns (UD34x4 result) {
```

#### Recommendation
```
- /// @notice Casts an UD34x4 number into UD60x18.
+ /// @notice Casts an UD60x18 number into UD34x4.
```



### NC-6 NatSpec dev comment warning function removal

The NatSpec for two library function contain a dev comment about removing them in v4, this version is v5 ...should they have been removed?

Their only references are in test files.

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/UD34x4.sol#L61-L71
```
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

#### Recommendation
Move the functions into the test files that depend on them, as it will reduce contract gas deployment.
