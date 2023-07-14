# Low-Level and Non-Critical issues

## Summary

### Low Level List

| Number | Issue Details                                                                                                      | Instances |
| :----: | :----------------------------------------------------------------------------------------------------------------- | :-------: |
| [L-01] | Unsafe downcasting can result in unwanted number                                                                   |     2     |
| [L-02] | All address should be checked for address(0) in constructor specially for immutables tokens                        |     1     |
| [L-03] | Check receiver address is not address(0) and amount is greater than zero before transferring or minting any tokens |     2     |

Total 3 Low Level Issues

### Non Critical List

| Number  | Issue Details                                                       | Instances |
| :-----: | :------------------------------------------------------------------ | :-------: |
| [NC-01] | Named return is confusing, use explicit returns                     |     2     |
| [NC-02] | Use `uint256` consistently . Do not mix `uint` and `uint256` both.  |     2     |
| [NC-03] | Named params of mapping is more readable                            |     1     |
| [NC-04] | Some Contracts are not following proper solidity style guide layout |     1     |

Total 4 Non Critical Issues

# Low-Level issues:

## [L-01] Unsafe downcasting can result in unwanted number.

_2 instances_

### Instance#1:

```solidity
File: src/libraries/LinearVRGDALib.sol
110: return ud2x18(uint64(uint256(wadExp(maxDiv / 1e18))));
    //@audit low unsafe down casting 256 to 64
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L110

Recommendation : Use openzeppelin SafeCast library.

### Instance#2: Unsafe downcasting from uint256 to uint96

```solidity
File: src/Claimer.sol
 72:   uint96 feePerClaim = uint96(
      _computeFeePerClaim(
        _computeMaxFee(tier, prizePool.numberOfTiers()),
        claimCount,
        prizePool.claimCount()
      )
 77:   );
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L71C1-L78C7

## [L-02] All address should be checked for address(0) in constructor specially for immutables

Once immutable set it is fixed if need to update or wrong set then deployer need to redeploy

### Instance#1: Check `_prizePool` for address(0) before assigning it to immutable `prizePool`

```solidity
File: src/Claimer.sol
37: constructor(
    PrizePool _prizePool,
    uint256 _minimumFee,
    uint256 _maximumFee,
    uint256 _timeToReachMaxFee,
    UD2x18 _maxFeePortionOfPrize
  ) {
44:   prizePool = _prizePool;
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L37-L44

## [L-03] Check receiver address is not address(0) and amount is greater than zero before transferring or minting any tokens

_2 Instances_

### Instance#1-2: Make sure `to` and `_prizeRecipient` is not address(0) before transfer and amount is non zero

```solidity
File: src/PrizePool.sol

340:   _transfer(_to, _amount);

      ...
473:  _transfer(_prizeRecipient, amount);
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L340

# Non Critcal Issues

## [NC-01] : Named return is confusing, use explicit returns.

return from function explicitly make code readable

### Instance#1 :

```solidity
File: src/Claimer.sol

60:  function claimPrizes(
    Vault vault,
    uint8 tier,
    address[] calldata winners,
    uint32[][] calldata prizeIndices,
    address _feeRecipient
66: ) external returns (uint256 totalFees) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L60C2-L66C43

### Instance#2 : Explicit return when named return defined is redundant and confusing

```solidity
File: src/Claimer.sol
66:  ) external returns (uint256 totalFees) {
        ...
82:   return feePerClaim * claimCount;
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L66-L82

## [NC-02] : Use `uint256` consistently . Do not mix `uint` and `uint256` both.

### Instance#1 :

```solidity
File: src/Claimer.sol

103: function computeTotalFees(
    uint8 _tier,
    uint _claimCount,
    uint _claimedCount
107:  ) external view returns (uint256) {
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L103C3-L107C38

### Instance#2 :

```solidity
File: src/Claimer.sol

120:  function computeFeePerClaim(uint256 _maxFee, uint _claimCount) external view returns (uint256)
```

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L120

## [NC-03] : Use named mapping params is more readable

### Instance#1:

```solidity
File: src/PrizePool.sol

206: mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))
    internal claimedPrizes;
```

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L206C3-L207C28

## [NC-04] :Some Contracts are not following proper solidity style guide layout

In some contract variables are declared after function and state changing functions are after view functions which is not proper code layout of solidity.
State variable declaration after functions is not proper code layout of solidity.
_1 Instance_

### Instance#1 :

```solidity
File: src/Vault.sol

383:  function maxMint(address) public view virtual override returns (uint256) {
    return _isVaultCollateralized() ? type(uint96).max : 0;
  }

  /**
   * @notice Mint Vault shares to the yield fee `_recipient`.
   * @dev Will revert if the Vault is undercollateralized
   *      or if the `_shares` are greater than the accrued `_yieldFeeTotalSupply`.
   * @param _shares Amount of shares to mint
   * @param _recipient Address of the yield fee recipient
   */
  function mintYieldFee(uint256 _shares, address _recipient) external {
    _requireVaultCollateralized();
    if (_shares > _yieldFeeTotalSupply) revert YieldFeeGTAvailable(_shares, _yieldFeeTotalSupply);

    _yieldFeeTotalSupply -= _shares;
    _mint(_recipient, _shares);

    emit MintYieldFee(msg.sender, _recipient, _shares);
402:   }

```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L383C3-L402C4

Recommendation: Update using below guide layout of variables
Inside each contract, library or interface, use the following order:

Type declarations
State variables
Events
Errors
Modifiers
Functions

https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout

Order of Functions
Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private

https://docs.soliditylang.org/en/latest/style-guide.html#order-of-functions
