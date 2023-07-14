# Table of Contents
| Number | Issues Details                                                                                         | Count |
| :----- | :----------------------------------------------------------------------------------------------------- | :---- |
| [L-1] | Unrestricted `for-`loops Making External Calls Could Lead to a DOS Attack | 1 |
| [L-2] | Extending the Timestamp Limit to Avoid Future Reverting | 4 |
| [L-3] | Validate Address Parameters to Avoid Issues | 12 |
| [L-4] | Utilize `_safeMint` for Safer Minting | 1 |
| [L-5] | Add Array Length Checks | 2 |
| [L-6] | Implement Reentrancy Guard for the `withdraw` Function | 5 |
| [L-7] | Limitations of `type(uint256).max` with Some Tokens | 1 |
| [L-8] | Adopt `safeTransfer` for Safer Token Transfers | 1 |
| [N-1] | Encourage Uniform Naming Convention for Immutables | 1 |
| [N-2] | Apply `delete` for Variable Clearance | 1 |
| [N-3] | Utilizing Underscores for Number Literals | 2 |
| [N-4] | Essential Modifications Should Follow a Two-stage Protocol | 4 |
| [N-5] | Emitting Events for Significant Parameter Changes | 3 |
| [N-6] | Upgrade to `Ownable2Step` for Improved Ownership Management | 1 |
| [N-7] | Adopt `safeTransferOwnership` for Safer Ownership Transfers | 1 |



## [L-1]</a><a name="L-1"> Unrestricted `for-`loops Making External Calls Could Lead to a DOS Attack

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: Vault.sol
for (uint w = 0; w < _winners.length; w++) {
for (uint p = 0; p < prizeIndicesLength; p++) {
```
</details>



## [L-2]</a><a name="L-2"> Extending the Timestamp Limit to Avoid Future Reverting

It has been observed that limiting the timestamp variable to fit within a `uint32` may lead to a call reverting in the year 2106. To prevent this issue and ensure the continued functionality of the contract beyond that timeframe, it is recommended to extend the timestamp limit.

<details>
<summary><i>There are 4 instances of this issue:</i></summary>

```solidity
File: TwabLib.sol
uint32 currentTime = uint32(block.timestamp);
```

```solidity
File: ObservationLib.sol
uint32 beforeOrAtTimestamp = beforeOrAt.timestamp;
```

```solidity
File: TwabLib.sol
uint32 currentTime = uint32(block.timestamp);
```

```solidity
File: TwabLib.sol
uint32 currentTime = uint32(block.timestamp);
```
</details>



## [L-3]</a><a name="L-3"> Validate Address Parameters to Avoid Issues

To prevent transaction reverts and gas wastage, validate address parameters to ensure they are not set to the zero address (0x0).

<details>
<summary><i>There are 18 instances of this issue:</i></summary>

```solidity
File: TwabController.sol
_to
```

```solidity
File: Vault.sol
_account
```

```solidity
File: PrizePool.sol
_to
```

```solidity
File: TwabController.sol
_from
```

```solidity
File: PrizePool.sol
_drawManager
```

```solidity
File: Vault.sol
_recipient
```

```solidity
File: PrizePool.sol
_winner
```

```solidity
File: PrizePool.sol
_prizeRecipient
```

```solidity
File: PrizePool.sol
_feeRecipient
```

```solidity
File: Vault.sol
yieldFeeRecipient_
```

```solidity
File: Vault.sol
_receiver
```

```solidity
File: TwabController.sol
_vault
```

```solidity
File: TwabController.sol
_to
```

```solidity
File: Vault.sol
_receiver
```

```solidity
File: Vault.sol
_owner
```

```solidity
File: Vault.sol
claimer_
```

```solidity
File: Vault.sol
_receiver
```
</details>



## [L-4]</a><a name="L-4"> Utilize `_safeMint` for Safer Minting

To enhance the safety and security of the minting process in your contract, it is recommended to use the `_safeMint` function instead of `_mint`. The `_safeMint` function incorporates additional checks and safeguards to mitigate potential risks.

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: Vault.sol
_mint(_account, _amountOut);
```

</details>



## [L-5]</a><a name="L-5"> Add Array Length Checks

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: Vault.sol

function claimPrizes
```

```solidity
File: Claimer.sol

function claimPrizes
```
</details>




## [L-6]</a><a name="L-6"> Implement Reentrancy Guard for the `withdraw` Function

To prevent reentrancy attacks and ensure the safety of funds in your contract, it is essential to implement a reentrancy guard in the `withdraw` function. Without a proper guard, malicious contracts or attackers can exploit potential vulnerabilities and repeatedly call the `withdraw` function to drain funds.

<details>
<summary><i>There are 5 instances of this issue:</i></summary>

```solidity
File: Vault.sol
function _withdraw(
    address _caller,
    address _receiver,
    address _owner,
    uint256 _assets,
    uint256 _shares
  ) internal virtual override {
    if (_caller != _owner) {
      _spendAllowance(_owner, _caller, _shares);
    }

    
    
    
    
    
    
    _burn(_owner, _shares);

    _yieldVault.withdraw(_assets, address(this), address(this));
    SafeERC20.safeTransfer(IERC20(asset()), _receiver, _assets);

    emit Withdraw(_caller, _receiver, _owner, _assets, _shares);
  }
```

```solidity
File: PrizePool.sol
function withdrawReserve(address _to, uint104 _amount) external onlyDrawManager {
    if (_amount > _reserve) {
      revert InsufficientReserve(_amount, _reserve);
    }
    _reserve -= _amount;
    _transfer(_to, _amount);
    emit WithdrawReserve(_to, _amount);
  }
```

```solidity
File: PrizePool.sol
function totalWithdrawn() external view returns (uint256) {
    return _totalWithdrawn;
  }
```

```solidity
File: Vault.sol
function withdraw(
    uint256 _assets,
    address _receiver,
    address _owner
  ) public virtual override returns (uint256) {
    if (_assets > maxWithdraw(_owner))
      revert WithdrawMoreThanMax(_owner, _assets, maxWithdraw(_owner));

    uint256 _shares = _convertToShares(_assets, Math.Rounding.Up);
    _withdraw(msg.sender, _receiver, _owner, _assets, _shares);

    return _shares;
  }
```

```solidity
File: PrizePool.sol
function withdrawClaimRewards(address _to, uint256 _amount) external {
    uint256 _available = claimerRewards[msg.sender];

    if (_amount > _available) {
      revert InsufficientRewardsError(_amount, _available);
    }

    claimerRewards[msg.sender] -= _amount;
    _transfer(_to, _amount);
    emit WithdrawClaimRewards(_to, _amount, _available);
  }
```
</details>



## [L-7]</a><a name="L-7"> Limitations of `type(uint256).max` with Some Tokens

It is important to note that using `type(uint256).max` as the allowance value when calling the `approve()` function may not work as expected with certain tokens. While it is commonly used to represent an infinite allowance, some token contracts do not interpret the maximum value as infinite and may have specific implementations or limitations.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: Vault.sol
_asset.safeApprove(address(liquidationPair_), type(uint256).max);
```
</details>



## [L-8]</a><a name="L-8"> Adopt `safeTransfer` for Safer Token Transfers

Consider using the `safeTransfer` function instead of the `transfer` function for safer token transfers within your contract.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: Vault.sol
_twabController.transfer(_from, _to, uint96(_shares));
```
</details>



## [N-1]</a><a name="N-1"> Encourage Uniform Naming Convention for Immutables 

In the current code, certain immutables are named with capital letters and underscores, while others are not. For enhanced code quality, it's recommended to adopt a consistent naming convention for all immutables.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_2_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_3_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_4_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_5_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_6_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_7_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_8_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_9_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_10_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_11_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_12_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_13_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_14_TIERS;
```

```solidity
File: TieredLiquidityDistributor.sol
uint8 public immutable tierShares;
```

```solidity
File: TieredLiquidityDistributor.sol
uint8 public immutable canaryShares;
```

```solidity
File: TieredLiquidityDistributor.sol
uint8 public immutable reserveShares;
```
</details>



## [N-2]</a><a name="N-2"> Apply `delete` for Variable Clearance

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: PrizePool.sol
claimCount = 0;
canaryClaimCount = 0;
largestTierClaimed = 0;
```
</details>




## [N-3]</a><a name="N-3"> Utilizing Underscores for Number Literals

To improve the readability of number literals, it is recommended to use underscores as separators within the numbers. This technique enhances code comprehension by visually breaking down large numbers into more manageable segments.

For example, instead of writing `1000000`, you can represent the same value as `1_000_000`. The underscores have no impact on the numerical value but make it easier for developers to read and understand the magnitude of the number at a glance.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: Vault.sol
/// @notice Yield fee percentage represented in integer format with 9 decimal places (i.e. 10000000 = 0.01 = 1%).
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_2_3 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_3_4 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_4_5 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_5_6 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_6_7 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_7_8 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_8_9 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_9_10 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_10_11 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_11_12 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_12_13 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_13_14 = SD59x18.wrap(1000000000000000000);
```

```solidity
File: TieredLiquidityDistributor.sol
SD59x18 internal constant TIER_ODDS_14_15 = SD59x18.wrap(1000000000000000000);
```
</details>




## [N-4]</a><a name="N-4"> Essential Modifications Should Follow a Two-stage Protocol

It's recommended that crucial alterations adhere to a two-stage procedure.

<details>
<summary><i>There are 4 instances of this issue:</i></summary>

```solidity
File: Vault.sol
function setClaimer(address claimer_) external onlyOwner returns (address) {
```

```solidity
File: Vault.sol
function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {
```

```solidity
File: Vault.sol
function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {
```

```solidity
File: PrizePool.sol
function setDrawManager(address _drawManager) external {
```
</details>


## [N-5]</a><a name="N-5"> Emitting Events for Significant Parameter Changes

It is important to emit events when modifying state variables to enable effective monitoring using off-chain monitoring tools. Emitting events facilitates the tracking of activities related to critical parameter changes.

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: Vault.sol
function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {
```

```solidity
File: PrizePool.sol
function setDrawManager(address _drawManager) external {
```

```solidity
File: Vault.sol
function setClaimer(address claimer_) external onlyOwner returns (address) {
```
</details>



## [N-6]</a><a name="N-6"> Upgrade to `Ownable2Step` for Improved Ownership Management

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: Vault.sol
import { Ownable } from "owner-manager-contracts/Ownable.sol";
```

```solidity
File: Vault.sol
contract Vault is ERC4626, ERC20Permit, ILiquidationSource, Ownable {
```

```solidity
File: Vault.sol
) ERC4626(asset_) ERC20(name_, symbol_) ERC20Permit(name_) Ownable(owner_) {
```
</details>



## [N-7]</a><a name="N-7"> Adopt `safeTransferOwnership` for Safer Ownership Transfers

Consider utilizing the `safeTransferOwnership` function instead of the standard `transferOwnership` function for more secure ownership transfers in your contract.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: Vault.sol
import { Ownable } from "owner-manager-contracts/Ownable.sol";
```
</details>


