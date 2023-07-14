# PoolTogether Quality Assurance Report

**Note: Some issues are mentioned in the automated findings report but the report missed to mentioned all of these instances in the code.**

# Low Risk Findings

## Table Of Content

| Number                                                                                                                                   | Issue                                                                                                                                                                                                                                                        | Instances |
| :--------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------: |
| [L-01](#l-01---safeapprove-is-deprecated-and-its-usage-is-discouraged)                                                                   | [safeApprove is deprecated and its usage is discouraged.](#l-01---safeapprove-is-deprecated-and-its-usage-is-discouraged)                                                                                                                                    |         1 |
| [L-02](#l-02---users-can-accidentally-lose-funds-due-to-the-ignorance-of-zero-addess-check-in-these-functions-parameters)                | [Users can accidentally lose funds due to the ignorance of zero addess check in these functions parameters.](#l-02---users-can-accidentally-lose-funds-due-to-the-ignorance-of-zero-addess-check-in-these-functions-parameters)                              |         7 |
| [L-03](#l-03---allowing-anyone-to-call-sethooks-function-can-be-risky)                                                                   | [Allowing anyone to call `setHooks` function can be risky](#l-03---allowing-anyone-to-call-sethooks-function-can-be-risky)                                                                                                                                   |         1 |
| [L-04](#l-04---contract-logic-depends-on-the-erc20-token-balance-which-can-be-manipulated-by-transfering-assets-to-the-contract-address) | [Contract logic depends on the ERC20 token balance which can be manipulated by transfering assets to the contract address](#l-04---contract-logic-depends-on-the-erc20-token-balance-which-can-be-manipulated-by-transfering-assets-to-the-contract-address) |         1 |
| [L-05](#l-05---shadow-variables)                                                                                                         | [Shadow Variables](#l-05---shadow-variables)                                                                                                                                                                                                                 |         3 |

### [L-01] - safeApprove is deprecated and its usage is discouraged.

**Details**

Using this deprecated function can lead to unintended reverts and potentially the locking of funds. A deeper discussion on the deprecation of this function is in [OZ issue #2219](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2219). The OpenZeppelin ERC20 `safeApprove()` function has been deprecated, as seen in the comments of the OpenZeppelin code.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 282](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L282)

</details>

**Recommended Mitigation Steps**

As suggested by the OpenZeppelin comment, replace `safeApprove()` with `safeIncreaseAllowance()` or `safeDecreaseAllowance()` instead.

### [L-02] - Users can accidentally lose funds due to the ignorance of zero addess check in these functions parameters.

**Details**

These all functions ignore the zero address check for the token reciever address, we know nobody will pass the zero address intentionally but accidentally it can happen.

Zero address check will increase some gas cost but save user from lose.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 407](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L407)

[Vault.sol - Line 427](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L427)

[Vault.sol - Line 458](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L458)

[Vault.sol - Line 480](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L480)

[Vault.sol - Line 494](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L494)

[Vault.sol - Line 509](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L509)

[Vault.sol - Line 1154](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1154)

</details>

**Recommended Mitigation Steps**

Add the condition to check the token reciever address should not be zero address.

### [L-03] - Allowing anyone to call `setHooks` function can be risky.

**Details**

The Vault's `setHooks` function sets hooks for users which will execute before or after the `claimPrizes` function but the `setHooks` is open for everyone, on the other hand only `_claimer` can call the `claimPrizes` fuction and `_hooks` is only used inside `claimPrizes` function.

It would be better to only allow `_claimer` to set the hooks to be safe from risks.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 654](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L654)

</details>

**Recommended Mitigation Steps**

It would be better to only allow `_claimer` to set the hooks to be safe from risks.

### [L-04] - Contract logic depends on the ERC20 token balance which can be manipulated by transfering assets to the contract address.

**Details**

I am writing this issue in the low-risks category to just mention that this could be risky.

But I have tested the code and found that anybody can mint free Vault shares upto the Vault balance without approving or transfering any collateral tokens into the Vault.

This could happen through frontrun (such as user transfer the funds and then call the deposit function and in between an attack mints the shares) or just delay between the transfer and deposit operations can do this.

But after discussing with the team members, they ensure that it will not happen.

here is their answer in the discord discussions;

> I'm seeing some confusion around how deposits are made into the Vault
> There are two ways of depositing into the Vault:

> 1. A contract can transfer USDC to the Vault then call the deposit() function to deposit the additional balance
> 2. A contract can approve a Vault allowance and call the deposit() function. The Vault will use safeTransferFrom to transfer-in the required tokens.

> This is our intention, at least. Let's see if you all can find issues.

I am not sure whether this will happen or not but if any Vault has any funds left then any user can mint free shares upto that balance.

<details>
  <summary>here is the POC;</summary>
  <p></p>
  Add this in the <b>Deposit.t.sol</b> and run using command <b>forge test --mt testMintFreeVaultSharesWithoutDeposit</b>

```solidity
function testMintFreeVaultSharesWithoutDeposit() external {
  uint256 _vaultAmount = 1000e18;
  underlyingAsset.mint(address(vault), _vaultAmount);

  // * Vault balance is now 1000.
  assertEq(underlyingAsset.balanceOf(address(vault)), _vaultAmount);

  vm.startPrank(alice);

  uint256 _amount = 1000e18;

  vm.expectEmit();
  emit Transfer(address(0), alice, _amount);

  vm.expectEmit();
  emit Deposit(alice, alice, _amount, _amount);

  // * before deposit alice balance is zero.
  assertEq(twabController.balanceOf(address(vault), alice), 0);

  vault.deposit(_amount, alice);

  // * after deposit her balance is 1000.
  assertEq(vault.balanceOf(alice), _amount);

  assertEq(twabController.balanceOf(address(vault), alice), _amount);
  assertEq(twabController.delegateBalanceOf(address(vault), alice), _amount);

  assertEq(underlyingAsset.balanceOf(alice), _amount - _vaultAmount);
  assertEq(underlyingAsset.balanceOf(address(vault)), 0);
  assertEq(underlyingAsset.balanceOf(address(yieldVault)), _amount);

  assertEq(yieldVault.balanceOf(address(vault)), _amount);
  assertEq(yieldVault.totalSupply(), _amount);

  vm.stopPrank();
}
```

</details>

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 932](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L932)

</details>

### [L-05] - Shadow Variables.

**Details**

Consider the compiler warnings into consideration and avoid the variable shadowing.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[PrizePool.sol - Line 421](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L421)

[PrizePool.sol - Line 429](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L429)

[PrizePool.sol - Line 876](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L876)

</details>

# Non Critical Findings

## Table Of Content

| Number                                                                 | Issue                                                                                                                    | Instances |
| :--------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------- | --------- |
| [N-01](#n-01---lack-of-indexed-events-parameters)                      | [Lack of Indexed Events Parameters](#n-01---lack-of-indexed-events-parameters)                                           | 7         |
| [N-02](#n-02---unnecessary-conversions)                                | [Unnecessary Conversions](#n-02---unnecessary-conversions)                                                               | 2         |
| [N-03](#n-03---unchecked-should-be-inside-if-statement)                | [Unchecked should be inside if statement](#n-03---unchecked-should-be-inside-if-statement)                               | 1         |
| [N-04](#n-04---immutable-variables-should-be-mixedcase)                | [`immutable` variables should be mixedcase](#n-04---immutable-variables-should-be-mixedcase)                             | 2         |
| [N-05](#n-05---magic-numbers-should-be-avoided-for-better-readability) | [Magic numbers should be avoided for better readability](#n-05---magic-numbers-should-be-avoided-for-better-readability) | 1         |
| [N-06](#n-06---remove-consolesol-imports-from-production-code)         | [Remove console.sol imports from production code](#n-06---remove-consolesol-imports-from-production-code)                | 1         |

### [N-01] - Lack of Indexed Events Parameters.

**Details**

To facilitate off-chain searching and filtering for specific events information try to utilize the 3 available `indexed` parameters for events.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 147](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L147)

```diff
  * @param previousClaimer Address of the previous claimer
  * @param newClaimer Address of the new claimer
  */
-  event ClaimerSet(address previousClaimer, address newClaimer);
+  event ClaimerSet(address indexed previousClaimer, address indexed newClaimer); // * @audit use indexed parameters.
```

[Vault.sol - Line 154](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L154)

```diff
  /**
  * @notice Emitted when an account sets new hooks
  * @param account The account whose hooks are being configured
  * @param hooks The hooks being set
  */
-  event SetHooks(address account, VaultHooks hooks);
+  event SetHooks(address indexed account, VaultHooks indexed hooks); // * @audit use indexed parameters.
```

[Vault.sol - Line 160](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L160)

```diff
  /**
  * @notice Emitted when a new LiquidationPair has been set.
  * @param newLiquidationPair Address of the new liquidationPair
  */
-  event LiquidationPairSet(LiquidationPair newLiquidationPair);
+  event LiquidationPairSet(LiquidationPair indexed newLiquidationPair); // * @audit use indexed parameters.
```

[Vault.sol - Line 175](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L175)

```diff
  /**
  * @notice Emitted when yield fee is minted to the yield recipient.
  * @param previousYieldFeeRecipient Address of the previous yield fee recipient
  * @param newYieldFeeRecipient Address of the new yield fee recipient
  */
-  event YieldFeeRecipientSet(address previousYieldFeeRecipient, address newYieldFeeRecipient);
+  // * @audit use indexed parameters.
+  event YieldFeeRecipientSet(
+    address indexed previousYieldFeeRecipient,
+    address indexed newYieldFeeRecipient
+  );
```

[Vault.sol - Line 182](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L182)

```diff
  /**
  * @notice Emitted when a new yield fee percentage has been set.
  * @param previousYieldFeePercentage Previous yield fee percentage
  * @param newYieldFeePercentage New yield fee percentage
  */
-  event YieldFeePercentageSet(uint256 previousYieldFeePercentage, uint256 newYieldFeePercentage);
+  // * @audit use indexed parameters.
+  event YieldFeePercentageSet(
+    uint256 indexed previousYieldFeePercentage,
+    uint256 indexed newYieldFeePercentage
+  );
```

[Vault.sol - Line 198](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L198)

```diff
  /**
  * @notice Emitted when a user sponsors the Vault.
  * @param exchangeRate The recorded exchange rate
  * @dev This happens on mint and burn of shares
  */
-  event RecordedExchangeRate(uint256 exchangeRate);
+  event RecordedExchangeRate(uint256 indexed exchangeRate); // * @audit use indexed paramete
rs.
```

[PrizePool.sol - Line 176](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L176)

```diff
-   event IncreaseReserve(address user, uint256 amount); // * @audit use indexed event parameters.
+   event IncreaseReserve(address indexed user, uint256 amount); // * @audit use indexed event parameters.
```

</details>

**Recommended Mitigation Steps**

Add `indexed` keyword with events paramets for useful information.

### [N-02] - Unnecessary Conversions.

**Details**

These mentioned conversions are unnecessary and affect the readability.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 269](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L269)

```diff
-    if (address(owner_) == address(0)) revert OwnerZeroAddress();
+    if (owner_ == address(0)) revert OwnerZeroAddress(); // * @audit owner is already an address no need to convert it.
```

[TwabLib.sol - Line 89](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L89)

```diff
-   isObservationRecorded = _delegateAmount != uint96(0); // * @audit unnessesary casting.
+   isObservationRecorded = _delegateAmount != 0; // * @audit unnessesary casting.
```

</details>

**Recommended Mitigation Steps**

Remove these unnecessary conversion for better readability.

### [N-03] - Unchecked should be inside if statement.

**Details**

It is a development error and not a big issue just want to mention so they can fix it.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Vault.sol - Line 945 - 949](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L945-L949)

```diff
+      // * @audit unchecked should be inside if.
-      unchecked {
-        if (_vaultAssets != 0) {
+      if (_vaultAssets != 0) {
+        unchecked {
           _assetsDeposit = _assets - _vaultAssets;
         }
       }
```

</details>

### [N-04] - `immutable` variables should be mixedcase.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[TwabController.sol - Line 27](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L27)

[TwabController.sol - Line 31](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L31)

</details>

### [N-05] - Magic numbers should be avoided for better readability.

**Details**

Magic numbers such as `365` creats ambiguity and readers do not fimiliar with the code logic can be lost. Try to store in a constant variable which defined the function of the magic number with its name.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

Here I think there refer to `MAX_CARDINALITY` which is also `365`, if yes then replace it here.

[TwabLib.sol - Line 62](https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L62)

</details>

### [N-06] - Remove console.sol imports from production code.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[PrizePool.sol - Line 04](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L4)

</details>
