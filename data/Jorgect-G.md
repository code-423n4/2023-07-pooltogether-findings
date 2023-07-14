# GAS OPTIMIZATION REPORT

## [G-01] THE CONTRACT CAN ADD UNCHECKED IN SEVERAL PART OF THE PROTOCOL.

The contract can add unchecked when it sure that the operation its no gonna revert:

1. Instance one

```
file: pt-v5-prize-pool/src/PrizePool.sol

function withdrawClaimRewards(address _to, uint256 _amount) external {
    uint256 _available = claimerRewards[msg.sender];

    if (_amount > _available) {
      revert InsufficientRewardsError(_amount, _available);
    }

    @audit add unchechked _amount it´s never gonna be major than  claimerRewards[msg.sender]

    claimerRewards[msg.sender] -= _amount;
    ...
  }
```
[Link](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L483C3-L493C4)

2. Instance two

```
file: pt-v5-prize-pool/src/abstract/TieredLiquidityDistributor.sol
function _consumeLiquidity(
    Tier memory _tierStruct,
    uint8 _tier,
    uint104 _liquidity
  ) internal returns (Tier memory) {
    ...
    if (_liquidity > remainingLiquidity) {

      @audit you can add uncheked _liquidity it´s never gonna be major than remainingLiquidity

      uint104 excess = _liquidity - remainingLiquidity;
      if (excess > _reserve) {
        revert InsufficientLiquidity(_liquidity);
      }

      @audit you can add uncheked _reserve it´s never gonna be major than excess

      _reserve -= excess;
    ...
  }

```
[Link](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L518C3-L550C1)

3. Instance three 

```
file: pt-v5-prize-pool/src/libraries/DrawAccumulatorLib.sol

function add(Accumulator storage accumulator, uint256 _amount, uint16 _drawId, SD59x18 _alpha)
        internal
        returns (bool)
    {
        ...
        if (_drawId < newestDrawId_) {
            revert DrawClosed(_drawId, newestDrawId_);
        }

        ...
        if (_drawId != newestDrawId_) {

            @audit you can add uncheked _drawId it´s never gonna be major than newestDrawId_

            uint256 relativeDraw = _drawId - newestDrawId_; 
```
[link](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L64C3-L112C4)

4. Instance four

```
file: pt-v5-vault/src/Vault.sol

 function mintYieldFee(uint256 _shares, address _recipient) external {
    _requireVaultCollateralized();
    if (_shares > _yieldFeeTotalSupply) revert YieldFeeGTAvailable(_shares, _yieldFeeTotalSupply);

    @audit you can add uncheked _yieldFeeTotalSupply it´s never gonna be major than _shares


    _yieldFeeTotalSupply -= _shares;
    _mint(_recipient, _shares);

    emit MintYieldFee(msg.sender, _recipient, _shares);
  }
```
[link](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L394C2-L402C4)

5. Instance five

```
 function _currentExchangeRate() internal view returns (uint256) {
        ...
       if (_withdrawableAssets > _totalSupplyToAssets) {

             @audit you can add uncheked _yieldFeeTotalSupply it´s never gonna be major than _shares

            _withdrawableAssets = _withdrawableAssets - (_withdrawableAssets - _totalSupplyToAssets); 
        }
       ...
    } 
```
[Link](https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1168C3-L1188C1)






