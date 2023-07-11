Title: Incorrect Return Value in `delegateOf` Function
Summary:
The `delegateOf` function in the `TwabController` contract returns an address instead of the delegate balance, contrary to its intended purpose. This inconsistency between the comment and the actual implementation can lead to confusion and incorrect usage of the delegation mechanism within the contract.
```solidity
/**
 * @notice The current delegate of a user for a specific vault.
 * @param vault the vault for which the delegate balance is being queried
 * @param user the user whose delegate balance is being queried
 * @return The current delegate balance of the user
 */
function delegateOf(address vault, address user) external view returns (address) {
  return _delegateOf(vault, user);
}

```
[Link](
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L201-L209
)
Impact:
The major impact of this issue is *Incorrect Delegation* Usage. By returning an `address` instead of the delegate balance, the function fails to provide the necessary information for proper usage of the delegation mechanism. This can lead to incorrect calculations or mishandling of delegated balances throughout the contract, undermining the accuracy and integrity of the delegation feature.
Mitigation:
To resolve this issue, the implementation of the `delegateOf` function should be updated to return the delegate balance (a `uint256`) instead of the address. 