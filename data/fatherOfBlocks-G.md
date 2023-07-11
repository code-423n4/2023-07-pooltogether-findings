**vault/src/Vault.sol**
- L315/1179 - The operation: _assets - _sharesToAssets could become unchecked since it was previously validated that overflow is not generated. It also happens with _withdrawableAssets - (_withdrawableAssets - _totalSupplyToAssets), per Line 1178.

- L719/728/737/745/753/761/769/777 - The getter functions are public, since they are not used within the contract, therefore gas would be saved by making them external.


**prize-pool/src/abstract/TieredLiquidityDistributor.sol**
- L534/538 - The operation: _liquidity - remainingLiquidity; could become unchecked since it was previously validated that overflow is not generated.
