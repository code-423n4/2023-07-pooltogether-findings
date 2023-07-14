# PoolTogether Gas Optimization Report

### Note: Some issues are mentioned in the automated findings report but the report missed to mentioned all of these instances in the code.

## Table Of Content

| Number                                                                                                                         | Issue                                                                                                                                                                                                                                          | Instances |
| :----------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------: |
| [G-01](#g-01---use-calldata-data-location-for-functions-parameters-to-save-gas)                                                | [use `calldata` data location for functions parameters to save gas](#g-01---use-calldata-data-location-for-functions-parameters-to-save-gas)                                                                                                   |         2 |
| [G-02](#g-02---execute-the-maxdeposit_receiver-function-only-once-and-use-its-cached-value-in-the-error)                       | [Execute the `maxDeposit(_receiver)` function only once and use its cached value in the error](#g-02---execute-the-maxdeposit_receiver-function-only-once-and-use-its-cached-value-in-the-error)                                               |         1 |
| [G-03](#g-03---execute-the-maxwithdraw_owner-function-only-once-and-use-its-cached-value-in-the-error)                         | [Execute the `maxWithdraw(_owner)` function only once and use its cached value in the error](#g-03---execute-the-maxwithdraw_owner-function-only-once-and-use-its-cached-value-in-the-error)                                                   |         1 |
| [G-04](#g-04---execute-the-maxredeem_owner-function-only-once-and-use-its-cached-value-in-the-error)                           | [Execute the `maxRedeem(_owner)` function only once and use its cached value in the error](#g-04---execute-the-maxredeem_owner-function-only-once-and-use-its-cached-value-in-the-error)                                                       |         1 |
| [G-05](#g-05---multiple-reads-of-storage-variable-which-can-be-cached-to-save-users-gas-cost)                                  | [Multiple reads of `storage` variable which can be cached to save users gas cost](#g-05---multiple-reads-of-storage-variable-which-can-be-cached-to-save-users-gas-cost)                                                                       |         4 |
| [G-06](#g-06---use-do-while-loop-instead-of-for-loop-to-save-users-gas-cost)                                                   | [Use `do-while` loop instead of `for-loop` to save users gas cost](#g-06---use-do-while-loop-instead-of-for-loop-to-save-users-gas-cost)                                                                                                       |         3 |
| [G-07](#g-07---functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)                               | [Functions guaranteed to revert when called by normal users can be marked payable](#g-07---functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)                                                                   |         6 |
| [G-08](#g-08---_converttoshares-function-can-return-value-without-calculating-_currentexchangerate-when-_assets-value-is-zero) | [`_convertToShares` function can return value without calculating `_currentExchangeRate` when `_assets` value is zero](#g-08---_converttoshares-function-can-return-value-without-calculating-_currentexchangerate-when-_assets-value-is-zero) |         1 |
| [G-09](#g-09---_converttoassets-function-can-return-value-without-calculating-_currentexchangerate-when-_shares-value-is-zero) | [`_convertToAssets` function can return value without calculating `_currentExchangeRate` when `_shares` value is zero](#g-09---_converttoassets-function-can-return-value-without-calculating-_currentexchangerate-when-_shares-value-is-zero) |         1 |
| [G-10](#g-10---execute-the-maxmint_receiver-function-only-once-and-use-its-cached-value-in-the-error)                          | [Execute the `maxMint(_receiver)` function only once and use its cached value in the error](#g-10---execute-the-maxmint_receiver-function-only-once-and-use-its-cached-value-in-the-error)                                                     |         1 |
| [G-11](#g-11---structure-state-variables-to-save-user-gas-cost)                                                                | [Structure state variables to save user gas cost](#g-11---structure-state-variables-to-save-user-gas-cost)                                                                                                                                     |         1 |
| [G-12](#g-12---use-unchecked-where-possible-to-save-users-gas)                                                                 | [Use `unchecked` where possible to save users gas](#g-12---use-unchecked-where-possible-to-save-users-gas)                                                                                                                                     |         2 |

### [G-01] - use `calldata` data location for functions parameters to save gas.

**Details**

-   Solidity docs suggests that `If you can, try to use calldata as data location because it will avoid copies and also makes sure that the data cannot be modified.`
-   And it also save gas when compare to memory data location.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[VaultFactory.sol - Line 57 - 58](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L57-L58)

| Calculation Type | Before  | After   | Gas Saved |
| :--------------- | :------ | :------ | --------: |
| Avg              | 3737309 | 3736785 |       524 |

```diff
diff --git a/src/VaultFactory.sol b/src/VaultFactory.sol
index 559fd90..f7b7905 100644
--- a/src/VaultFactory.sol
+++ b/src/VaultFactory.sol
@@ -52,10 +52,11 @@ contract VaultFactory {
    * @param _owner Address that will gain ownership of this contract
    * @return address Address of the newly deployed Vault
    */
+    // * @audit use calldata to save gas.
   function deployVault(
     IERC20 _asset,
-    string memory _name,
-    string memory _symbol,
+    string calldata _name,
+    string calldata _symbol,
     TwabController _twabController,
     IERC4626 _yieldVault,
     PrizePool _prizePool,
```

[VaultFactory.sol - Line 57 - 58](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L57-L58)

</details>

### [G-02] - Execute the `maxDeposit(_receiver)` function only once and use its cached value in the error.

**Details**

In the Vault's `deposit` function we are calling the `maxDeposit(_receiver)` function twice, once for the condition check and once in the custom error information.

This will cost huge amount of gas to users when they deposit the amount more than the max deposit limit.

<details>
  <summary>Spotted Finding</summary>
  <p></p>

[Vault.sol - Line 408 - 409](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L408-L409)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 21267  | 18452 |      2815 |

```diff
   /* ============ Deposit Functions ============ */
   /// @inheritdoc ERC4626
   function deposit(uint256 _assets, address _receiver) public virtual override returns (uint
256) {
-    if (_assets > maxDeposit(_receiver))
-      revert DepositMoreThanMax(_receiver, _assets, maxDeposit(_receiver));
+    // * @audit we can call maxDeposit once and store it's value to a variable for use in the error.
+    uint256 maxDeposit_ = maxDeposit(_receiver);
+    if (_assets > maxDeposit_) revert DepositMoreThanMax(_receiver, _assets, maxDeposit_);
```

</details>

### [G-03] - Execute the `maxWithdraw(_owner)` function only once and use its cached value in the error.

**Details**

In the Vault's `withdraw` function we are calling the `maxWithdraw(_owner)` function twice, once for the condition check and once in the custom error information.

This will cost huge amount of gas to users when they withdraw the amount more than the max withdraw limit.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 514 - 515](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L514-L515)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 13853  | 7290  |      6563 |

```diff
-    if (_assets > maxWithdraw(_owner))
-      revert WithdrawMoreThanMax(_owner, _assets, maxWithdraw(_owner));
+    // * @audit we can call the maxWithdraw once and store the value in variable for use in error.
+    uint256 maxWithdraw_ = maxWithdraw(_owner);
+    if (_assets > maxWithdraw_) revert WithdrawMoreThanMax(_owner, _assets, maxWithdraw_);
```

</details>

### [G-04] - Execute the `maxRedeem(_owner)` function only once and use its cached value in the error.

**Details**

In the Vault's `redeem` function we are calling the `maxRedeem(_owner)` function twice, once for the condition check and once in the custom error information.

This will cost huge amount of gas to users when they redeem the amount more than the max redeem limit.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 529](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L529)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 3386   | 2046  |      1340 |

```diff
-    if (_shares > maxRedeem(_owner)) revert RedeemMoreThanMax(_owner, _shares, maxRedeem(_owner));
+    // * @audit maxRedeem can be called once and store the result in a variable for error.
+    uint256 maxRedeem_ = maxRedeem(_owner);
+    if (_shares > maxRedeem_) revert RedeemMoreThanMax(_owner, _shares, maxRedeem_);
```

</details>

### [G-05] - Multiple reads of `storage` variable which can be cached to save users gas cost.

**Details**

SLOAD (is the opcode used of read storage variable) cost 100 units of gas and MLOAD & MSTORE (are the opcodes used to read and write to memory respectivily) cost 3, 3 units of gas respectivily.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

In the Vault's `claimPrizes` function we are reading `_claimer` twice which can be cached in a local variable to use it later to save users gas.

[Vault.sol - Line 614](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L614)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 1447   | 1326  |       121 |

```diff
-    if (msg.sender != _claimer) revert CallerNotClaimer(msg.sender, _claimer);
+    // * @audit SLOAD cost 100 and MLOAD & MSTORE cost 3 + 3 = 6. Store _claimer in a local variable to use it twice to save users gas.
+    address claimer_ = _claimer;
+    if (msg.sender != claimer_) revert CallerNotClaimer(msg.sender, claimer_);
```

In the PrizePool's `onlyDrawManager` modifier we are reading `drawManager` twice which can be cached in a local variable to use it later to save users gas.

[PrizePool.sol - Line 288](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L288)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 2837   | 2734  |       103 |

```diff
   /// @notice Modifier that throws if sender is not the draw manager.
   modifier onlyDrawManager() {
-    if (msg.sender != drawManager) {
-      revert CallerNotDrawManager(msg.sender, drawManager);
+    // * @audit can draw manager be zero address?
+    // * @audit should not read twice from the storage.
+    address drawManager_ = drawManager;
+    if (msg.sender != drawManager_) {
+      revert CallerNotDrawManager(msg.sender, drawManager_);
     }
     _;
   }
```

In the PrizePool's `contributePrizeTokens` function we are reading `lastClosedDrawId` and `smoothing.intoSD59x18()` multiple times which can be cached in a local variables to use it later to save users gas.

[PrizePool.sol - Line 311 - 330](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L311-L330)

| Calculation Type | Before | After  | Gas Saved |
| :--------------- | :----- | :----- | --------: |
| Avg              | 132771 | 132516 |       255 |

```diff
   /// @notice Contributes prize tokens on behalf of the given vault. The tokens should have already been transferred to the prize pool.
   /// The prize pool balance will be checked to ensure there is at least the given amount to deposit.
   /// @return The amount of available prize tokens prior to the contribution.
     if (_deltaBalance < _amount) {
       revert ContributionGTDeltaBalance(_amount, _deltaBalance);
     }
+    uint16 lastClosedDrawId_ = lastClosedDrawId;
+    SD59x18 _alpha = smoothing.intoSD59x18();
+
     DrawAccumulatorLib.add(
       vaultAccumulator[_prizeVault],
       _amount,
-      lastClosedDrawId + 1,
-      smoothing.intoSD59x18()
+      lastClosedDrawId_ + 1, // * @audit-ok should be cached and reuse.
+      _alpha // * @audit should be cached and reuse.
     );
-    DrawAccumulatorLib.add(
-      totalAccumulator,
-      _amount,
-      lastClosedDrawId + 1,
-      smoothing.intoSD59x18()
-    );
-    emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);
+    DrawAccumulatorLib.add(totalAccumulator, _amount, lastClosedDrawId_ + 1, _alpha);
+    emit ContributePrizeTokens(_prizeVault, lastClosedDrawId_ + 1, _amount);
     return _deltaBalance;
   }
```

In the PrizePool's `closeDraw` function we are reading `_openDrawEndsAt()`, `lastClosedDrawId`, and `_lastClosedDrawStartedAt` multiple times which can be cached in a local variables to use it later to save users gas.

[PrizePool.sol - Line 348 - 386](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L348-L386)

**There are two calculations once when the error occur and once without**

**When `DrawNotFinished` Error occur**

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 5860   | 5255  |       605 |

**When execution works fine**

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 97221  | 96882 |       339 |

`Total Savings = 944 uint of gas`

```diff
-  function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager returns (uint16)
 {
+  // * @audit-ok Functions guaranteed to revert when called by normal users can be marked pa
yable.
+  function closeDraw(
+    uint256 winningRandomNumber_
+  ) external payable onlyDrawManager returns (uint16) {
     // check winning random number
     if (winningRandomNumber_ == 0) {
       revert RandomNumberIsZero();
     }
-    if (block.timestamp < _openDrawEndsAt()) {
-      revert DrawNotFinished(_openDrawEndsAt(), uint64(block.timestamp));
+    // * @audit should calculate `_openDrawEndsAt` once and cache it for second use.
+    // 5860
+    // 5255
+    // 605
+    uint64 openDrawEndsAt_ = _openDrawEndsAt();
+    if (block.timestamp < openDrawEndsAt_) {
+      revert DrawNotFinished(openDrawEndsAt_, uint64(block.timestamp));
     }

     uint8 _numTiers = numberOfTiers;
     uint8 _nextNumberOfTiers = _numTiers;

-    if (lastClosedDrawId != 0) {
+    // * @audit should be cached.
+    uint16 lastClosedDrawId_ = lastClosedDrawId;
+    if (lastClosedDrawId_ != 0) {
       _nextNumberOfTiers = _computeNextNumberOfTiers(_numTiers);
     }

     uint64 openDrawStartedAt_ = _openDrawStartedAt();

-    _nextDraw(_nextNumberOfTiers, uint96(_contributionsForDraw(lastClosedDrawId + 1)));
+    _nextDraw(_nextNumberOfTiers, uint96(_contributionsForDraw(lastClosedDrawId_ + 1)));

     _winningRandomNumber = winningRandomNumber_;
     claimCount = 0;
@@ -373,16 +416,16 @@ contract PrizePool is TieredLiquidityDistributor {
     _lastClosedDrawAwardedAt = uint64(block.timestamp);

     emit DrawClosed(
-      lastClosedDrawId,
+      lastClosedDrawId_,
       winningRandomNumber_,
       _numTiers,
       _nextNumberOfTiers,
       _reserve,
       prizeTokenPerShare,
-      _lastClosedDrawStartedAt
+      openDrawStartedAt_
     );

-    return lastClosedDrawId;
+    return lastClosedDrawId_;
   }
```

</details>

### [G-06] - Use `do-while` loop instead of `for-loop` to save users gas cost.

**Details**

do-while does not check the first condition and prevent assembly from executing losts of opcodes need for conditions checks and all these places are right for it because these all always execute the code inside loops on the first condition.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 618 - 629](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L618-L629)

**Note: These gas costs are estimated after fixing the multiple storage reads issue mentioned in the [G-05](#g-05---multiple-reads-of-storage-variable-which-can-be-cached-to-save-users-gas-cost)**

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 4943   | 4811  |       132 |

```diff
+    // * @audit use do while loop and unchecked here to save gas.
-    for (uint w = 0; w < _winners.length; w++) {
+    uint w = 0;
+    do {
       uint prizeIndicesLength = _prizeIndices[w].length;
-      for (uint p = 0; p < prizeIndicesLength; p++) {
-        totalPrizes += _claimPrize(
-          _winners[w],
-          _tier,
-          _prizeIndices[w][p],
-          _feePerClaim,
-          _feeRecipient
-        );
+      uint p = 0;
+      do {
+        totalPrizes =
+          totalPrizes +
+          _claimPrize(_winners[w], _tier, _prizeIndices[w][p], _feePerClaim, _feeRecipient);
+
+        unchecked {
+          ++p;
+        }
+      } while (p < prizeIndicesLength);
+
+      unchecked {
+        ++w;
       }
-    }
+    } while (w < _winners.length);
```

[Claimer.sol - Line 144 - 153](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144-L153)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 10101  | 9992  |       109 |

```diff
-    for (uint i = 0; i < _claimCount; i++) {
+    // * @audit-ok use do while and unchecked ++i.
+    uint i = 0;
+    do {
       fee += _computeFeeForNextClaim(
         minimumFee,
         decayConstant,
@@ -150,7 +164,11 @@ contract Claimer is Multicall {
         _claimedCount + i,
         _maxFee
       );
-    }
+
+      unchecked {
+        ++i;
+      }
+    } while (i < _claimCount);
```

[Claimer.sol - Line 68 - 70](https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68-L70)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 9992   | 9883  |       109 |

```diff
-    for (uint i = 0; i < winners.length; i++) {
+    // * @audit-ok use the do while loop.
+    uint i = 0;
+    do {
       claimCount += prizeIndices[i].length;
-    }
+      unchecked {
+        ++i;
+      }
+    } while (i < winners.length);
```

</details>

### [G-07] - Functions guaranteed to revert when called by normal users can be marked payable.

**Details**

It is advisable to always use `payable` keyword with functions which are only limited to owner or any other authority. This can save other users gas cost by deducting less execution cost because compiler will then not execute these opcodes `CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2),` which it does when payable is missing.

This saves 24 unit of gas per call excluded deployment cost.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 641](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L641)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 5875   | 5851  |        24 |

```diff
+  // * @audit Functions guaranteed to revert when called by normal users can be marked payable.
-  function setClaimer(address claimer_) external onlyOwner returns (address) {
+  function setClaimer(address claimer_) external payable onlyOwner returns (address) {
```

[Vault.sol - Line 665](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L665)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 2713   | 2689  |        24 |

```diff
+  // * @audit Functions guaranteed to revert when called by normal users can be marked payable.
-  function setLiquidationPair(LiquidationPair liquidationPair_) external onlyOwner returns (address) {
+  function setLiquidationPair(LiquidationPair liquidationPair_) external payable onlyOwner returns (address) {
```

[Vault.sol - Line 691](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L691)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 2660   | 2636  |        24 |

```diff
+  // * @audit Functions guaranteed to revert when called by normal users can be marked payable.
-  function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {
+  function setYieldFeePercentage(uint256 yieldFeePercentage_) external payable onlyOwner returns (uint256) {
```

[Vault.sol - Line 704](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L704)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 2712   | 2688  |        24 |

```diff
+ // * @audit Functions guaranteed to revert when called by normal users can be marked payable
-  function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {
+  function setYieldFeeRecipient(address yieldFeeRecipient_) external payable onlyOwner returns (address) {
```

[PrizePool.sol - Line 335](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L335)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 18510  | 18486 |        24 |

```diff
+  // * @audit Functions guaranteed to revert when called by normal users can be marked payable.
-  function withdrawReserve(address _to, uint104 _amount) external onlyDrawManager {
+  function withdrawReserve(address _to, uint104 _amount) external payable onlyDrawManager {
+    // * @audit should be cached and reuse.
   }
```

[PrizePool.sol - Line 348](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L348)

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 97221  | 97197 |        24 |

```diff
+   // * @audit Functions guaranteed to revert when called by normal users can be marked payable.
-   function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager returns (uint16) {
+   function closeDraw(uint256 winningRandomNumber_) external payable onlyDrawManager returns (uint16) {
```

</details>

### [G-08] - `_convertToShares` function can return value without calculating `_currentExchangeRate` when `_assets` value is zero.

**Details**

The Vault's `_convertToShares` is used on many places, it first calculate the `_currentExchangeRate` and then check the condition `(_assets == 0 || _exchangeRate == 0)` and return `_assets` if any of the condition is true.

If the `_assets` value will be zero then it will not check the second condition due to OR operator stops condition check when it found any single condition true.

In that case we do not need `_exchangeRate` value and we can return the `_assets` directly.

To save the gas we need to break these two conditions and use them saperatly. Checking the `_assets == 0` condition before calling `_currentExchangeRate` function can save `2608` units of gas per call.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 864 - 874](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L865-L874)

**Note: To calculate gas I called this function throw deposit function by passing zero value.**

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 79738  | 77130 |      2608 |

```diff
   /**
   * @inheritdoc ERC4626
   * @param _assets Amount of assets to convert
   * @param _rounding Rounding mode (i.e. down or up)
   * @return uint256 Amount of shares for the assets
   */
  function _convertToShares(
    uint256 _assets,
    Math.Rounding _rounding
  ) internal view virtual override returns (uint256) {
+    // * @audit if assets value is zero then we do not need to calculate exchange rate, this will save lot of gas.
+    if (_assets == 0) {
+      return _assets;
+    }
+
     uint256 _exchangeRate = _currentExchangeRate();

-    return
-      (_assets == 0 || _exchangeRate == 0)
-        ? _assets
-        : _assets.mulDiv(_assetUnit, _exchangeRate, _rounding);
+    return _exchangeRate == 0 ? _assets : _assets.mulDiv(_assetUnit, _exchangeRate, _rounding);
   }
```

</details>

### [G-09] - `_convertToAssets` function can return value without calculating `_currentExchangeRate` when `_shares` value is zero.

**Details**

The Vault's `_convertToAssets` is used on many places, it first calculate the `_currentExchangeRate` and then call the `a` which check the condition `(_shares == 0 || _exchangeRate == 0)` and return `_shares` if any of the condition is true.

If the `_shares` value will be zero then it will not check the second condition due to OR operator stops condition check when it found any single condition true.

In that case we do not need `_exchangeRate` value and we can return the `_shares` directly.

To save the gas we need to break these two conditions and use them saperatly. Checking the `_shares == 0` condition before calling `_currentExchangeRate` function can save `1329` units of gas per call.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 882 - 887](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L882-L887)

**Note: To calculate gas I called this function throw redeem function by passing zero value.**

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 57324  | 55995 |      1329 |

```diff
   /**
   * @inheritdoc ERC4626
   * @param _shares Amount of shares to convert
   * @param _rounding Rounding mode (i.e. down or up)
   * @return uint256 Amount of assets represented by the shares
   */
  function _convertToAssets(
    uint256 _shares,
    Math.Rounding _rounding
  ) internal view virtual override returns (uint256) {
+    // * @audit we do not need to calculate exchange rate if shares value is zero just return the _shares, this will save lots of gas.
-    return _convertToAssets(_shares, _currentExchangeRate(), _rounding);
+    if (_shares == 0) {
+      return _shares;
+    }
+
+    uint256 _exchangeRate = _currentExchangeRate();
+
+    return _exchangeRate == 0 ? _shares : _shares.mulDiv(_exchangeRate, _assetUnit, _rounding);
   }
```

</details>

### [G-10] - Execute the `maxMint(_receiver)` function only once and use its cached value in the error.

**Details**

In the Vault's `mint` and `mintWithPermit` functions we are calling the `maxMint(_receiver)` function twice, once for the condition check and once in the custom error information.

This will cost huge amount of gas to users when they mint the amount more than the max mint limit.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 971 - 974](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L971-L974)

**Note: To calculate gas I called this function throw mint function.**

| Calculation Type | Before | After | Gas Saved |
| :--------------- | :----- | :---- | --------: |
| Avg              | 21295  | 18480 |      2815 |

```diff
   /**
    * @notice Compute the amount of assets to deposit before minting `_shares`.
    * @param _shares Amount of shares to mint
    * @return uint256 Amount of assets to deposit.
    */
   function _beforeMint(uint256 _shares, address _receiver) internal view returns (uint256) {
-    if (_shares > maxMint(_receiver)) revert MintMoreThanMax(_receiver, _shares, maxMint(_receiver));
+    // * @audit maxMint can be call once and store vaule in a variable.
+    uint maxMint_ = maxMint(_receiver);
+    if (_shares > maxMint_) revert MintMoreThanMax(_receiver, _shares, maxMint_);
     return _convertToAssets(_shares, Math.Rounding.Up);
   }
```

</details>

### [G-11] - Structure state variables to save user gas cost.

**Details**

Padding variables together will allow the contract to add multiple variable at the same slot in the storage memory load multiple variables onces which will reduces lots of gas throught the execution of functions access these variables.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[PrizePool.sol - Line 199 - 253](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L199-L253)

**Note: This is just a development cost difference still reads and writes throughout this contract will save lost of gas because now contract load multiple variable at once from the same slot.**

| Calculation Type | Before  | After   | Gas Saved |
| :--------------- | :------ | :------ | --------: |
| Avg              | 5073048 | 5050336 |     22712 |

```diff
   /* ============ State ============ */
+  // * @audit structure the variables to save gas.

-  /// @notice The DrawAccumulator that tracks the exponential moving average of the contributions by a vault.
-  mapping(address => DrawAccumulatorLib.Accumulator) internal vaultAccumulator;
+  /// @notice The draw manager address.
+  address public drawManager; // * 20 bytes

-  /// @notice Records the claim record for a winner.
-  /// @dev vault => account => drawId => tier => prizeIndex => claimed
-  mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))
-    internal claimedPrizes;
+  /// @notice The number of prize claims for the last closed draw.
+  uint32 public claimCount; // * 4 bytes

-  /// @notice Tracks the total fees accrued to each claimer.
-  mapping(address => uint256) internal claimerRewards;
+  /// @notice The timestamp at which the last closed draw started.
+  uint64 internal _lastClosedDrawStartedAt; // * 4 bytes

-  /// @notice The degree of POOL contribution smoothing. 0 = no smoothing, ~1 = max smoothing. Smoothing spreads out vault contribution over multiple draws; the higher the smoothing the more draws.
-  SD1x18 public immutable smoothing;
+  // * ---------------- 1 slot complete here ------------------------

-  /// @notice The token that is being contributed and awarded as prizes.
-  IERC20 public immutable prizeToken;
+  /// @notice The largest tier claimed so far for the last closed draw.
+  uint8 public largestTierClaimed; // * 1 bytes

-  /// @notice The Twab Controller to use to retrieve historic balances.
-  TwabController public immutable twabController;
+  /// @notice The number of canary prize claims for the last closed draw.
+  uint32 public canaryClaimCount; // * 4 bytes

-  /// @notice The draw manager address.
-  address public drawManager;
+  /// @notice The timestamp at which the last closed draw was awarded.
+  uint64 internal _lastClosedDrawAwardedAt; // * 8 bytes
+
+  // * ---------------- above 3 will be in 1 slot ---------------------
+
+  /// @notice The total amount of prize tokens that have been claimed for all time.
+  uint256 internal _totalWithdrawn;
+
+  /// @notice The winner random number for the last closed draw.
+  uint256 internal _winningRandomNumber;

   /// @notice The number of seconds between draws.
   uint32 public immutable drawPeriodSeconds;
   /// @notice The exponential weighted average of all vault contributions.
   DrawAccumulatorLib.Accumulator internal totalAccumulator;

-  /// @notice The total amount of prize tokens that have been claimed for all time.
-  uint256 internal _totalWithdrawn;
-
-  /// @notice The winner random number for the last closed draw.
-  uint256 internal _winningRandomNumber;
+  /// @notice The degree of POOL contribution smoothing. 0 = no smoothing, ~1 = max smoothing. Smoothing spreads out vault contribution over multiple draws; the higher the smoothing themore draws.
+  SD1x18 public immutable smoothing;

-  /// @notice The number of prize claims for the last closed draw.
-  uint32 public claimCount;
+  /// @notice The token that is being contributed and awarded as prizes.
+  IERC20 public immutable prizeToken;

-  /// @notice The number of canary prize claims for the last closed draw.
-  uint32 public canaryClaimCount;
+  /// @notice The Twab Controller to use to retrieve historic balances.
+  TwabController public immutable twabController;

-  /// @notice The largest tier claimed so far for the last closed draw.
-  uint8 public largestTierClaimed;
+  /// @notice The DrawAccumulator that tracks the exponential moving average of the contributions by a vault.
+  mapping(address => DrawAccumulatorLib.Accumulator) internal vaultAccumulator;

-  /// @notice The timestamp at which the last closed draw started.
-  uint64 internal _lastClosedDrawStartedAt;
+  /// @notice Records the claim record for a winner.
+  /// @dev vault => account => drawId => tier => prizeIndex => claimed
+  mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 =>bool)))))
+    internal claimedPrizes;

-  /// @notice The timestamp at which the last closed draw was awarded.
-  uint64 internal _lastClosedDrawAwardedAt;
+  /// @notice Tracks the total fees accrued to each claimer.
+  mapping(address => uint256) internal claimerRewards;
```

</details>

### [G-12] - Use `unchecked` where possible to save users gas.

**Details**

When `unchecked` is used variable then compiler do not execute opcodes which check over/underflow errors, it can be risky some times but we should use it when we are sure it will not exceed the limits.

And here we have a huge limit for it.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[PrizePool.sol - Line 441](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L441)

[PrizePool.sol - Line 443](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L443)

| Calculation Type | Before | After  | Gas Saved |
| :--------------- | :----- | :----- | --------: |
| Avg              | 162074 | 161825 |       249 |

```diff
     if (_isCanaryTier(_tier, numberOfTiers)) {
-      canaryClaimCount++;
+      // * @audit should use unchecked and ++i;
+      unchecked {
+        ++canaryClaimCount;
+      }
     } else {
-      claimCount++;
+      // * @audit should use unchecked and ++i;
+      unchecked {
+        ++claimCount;
+      }
     }
```

</details>
