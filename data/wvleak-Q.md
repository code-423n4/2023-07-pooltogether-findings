# Non-Critical issues
> ## `PrizePool.sol`
## [N-01] Params not documented for `contributePrizeTokens()` NatSpec

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L308

```solidity
/// @notice Contributes prize tokens on behalf of the given vault. The tokens should have already been transferred to the prize pool.
  /// The prize pool balance will be checked to ensure there is at least the given amount to deposit.
  /// @return The amount of available prize tokens prior to the contribution.
  function contributePrizeTokens(address _prizeVault, uint256 _amount) external returns (uint256) 
```
> ## `Vault.sol`
## [N-O2] - Unnecessary getter functions
Consider changing the visibility of variables to public instead of writing getter function, therefore reducing contract size and improving overall readability.

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L712C2-L712C2

## [N-03] - uint for uint256
Consider setting uint256 instead of uint to increase clarity and prevent mistakes.
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L616
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L618
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L620
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L620
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1058