## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#gas1-abiencode-is-less-efficient-than-abiencodepacked) | `abi.encode()` is less efficient than `abi.encodepacked()` | 1 | 53 |
| [GAS&#x2011;2](#gas2-use-assembly-to-emit-events) | Use assembly to emit events | 26 | 988 |
| [GAS&#x2011;3](#gas3-counting-down-in-for-statements-is-more-gas-efficient) | Counting down in `for` statements is more gas efficient | 7 | 1799 |
| [GAS&#x2011;4](#gas4-use-assembly-to-write-address-storage-values) | Use `assembly` to write address storage values | 22 | 1628 |
| [GAS&#x2011;5](#gas5-multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache) | Multiple accesses of a mapping/array should use a local variable cache | 9 | 720 |
| [GAS&#x2011;6](#gas6-remove-forgestd-import) | Remove `forge-std` import | 1 | 100 |
| [GAS&#x2011;7](#gas7-the-result-of-a-function-call-should-be-cached-rather-than-recalling-the-function) | The result of a function call should be cached rather than re-calling the function | 12 | 600 |
| [GAS&#x2011;8](#gas8-superfluous-event-fields) | Superfluous event fields | 1 | 34 |
| [GAS&#x2011;9](#gas9-use-nested-if-and-avoid-multiple-check-combinations) | Use nested `if` and avoid multiple check combinations | 12 | 72 |
| [GAS&#x2011;10](#gas10-using-xor-^-and-and-&-bitwise-equivalents) | Using XOR (^) and AND (&) bitwise equivalents | 196 | 2548 |

Total: 301 contexts over 10 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
File: TierCalculationLib.sol

110: return uint256(keccak256(abi.encode(_user, _tier, _prizeIndex, _winningRandomNumber)));
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L110



#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |





### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Use assembly to emit events

We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic `emit` event for `eventSentAmountExample`:
```solidity
// uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```solidity
        assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }
```

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: PrizePool.sol

305: emit DrawManagerSet(_drawManager);

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L305

```solidity
File: PrizePool.sol

328: emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L328

```solidity
File: PrizePool.sol

341: emit WithdrawReserve(_to, _amount);

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L341

```solidity
File: PrizePool.sol

454: emit IncreaseClaimRewards(_feeRecipient, _fee);

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L454

```solidity
File: PrizePool.sol

492: emit WithdrawClaimRewards(_to, _amount, _available);

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L492

```solidity
File: PrizePool.sol

501: emit IncreaseReserve(msg.sender, _amount);

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L501

```solidity
File: TieredLiquidityDistributor.sol

539: emit ReserveConsumed(excess);

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L539

```solidity
File: TwabController.sol

663: emit Delegated(_vault, _from, _to);

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L663

```solidity
File: TwabController.sol

688: emit IncreasedBalance(_vault, _user, _amount, _delegateAmount);

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L688

```solidity
File: TwabController.sol

731: emit DecreasedBalance(_vault, _user, _amount, _delegateAmount);

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L731

```solidity
File: TwabController.sol

773: emit DecreasedTotalSupply(_vault, _amount, _delegateAmount);

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L773

```solidity
File: TwabController.sol

807: emit IncreasedTotalSupply(_vault, _amount, _delegateAmount);

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L807

```solidity
File: Vault.sol

401: emit MintYieldFee(msg.sender, _recipient, _shares);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L401

```solidity
File: Vault.sol

645: emit ClaimerSet(_previousClaimer, claimer_);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L645

```solidity
File: Vault.sol

656: emit SetHooks(msg.sender, hooks);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L656

```solidity
File: Vault.sol

681: emit LiquidationPairSet(liquidationPair_);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L681

```solidity
File: Vault.sol

695: emit YieldFeePercentageSet(_previousYieldFeePercentage, yieldFeePercentage_);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L695

```solidity
File: Vault.sol

708: emit YieldFeeRecipientSet(_previousYieldFeeRecipient, yieldFeeRecipient_);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L708

```solidity
File: Vault.sol

962: emit Deposit(_caller, _receiver, _assets, _shares);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L962

```solidity
File: Vault.sol

991: emit Sponsor(msg.sender, _receiver, _assets, _shares);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L991

```solidity
File: Vault.sol

1029: emit Withdraw(_caller, _receiver, _owner, _assets, _shares);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1029

```solidity
File: Vault.sol

1111: emit RecordedExchangeRate(_lastRecordedExchangeRate);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1111

```solidity
File: Vault.sol

1126: emit Transfer(address(0), _receiver, _shares);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1126

```solidity
File: Vault.sol

1142: emit Transfer(_owner, address(0), _shares);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1142

```solidity
File: Vault.sol

1157: emit Transfer(_from, _to, _shares);

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1157

```solidity
File: VaultFactory.sol

83: emit NewFactoryVault(_vault, VaultFactory(address(this)));

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L83



</details>





### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Claimer.sol

68: for (uint i = 0; i < winners.length; i++) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68

```solidity
File: Claimer.sol

144: for (uint i = 0; i < _claimCount; i++) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144

```solidity
File: TieredLiquidityDistributor.sol

361: for (uint8 i = start; i < end; i++) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L361

```solidity
File: TieredLiquidityDistributor.sol

630: for (uint8 i = start; i < end; i++) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L630

```solidity
File: TierCalculationLib.sol

138: for (uint8 i = 0; i < _numberOfTiers; i++) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L138

```solidity
File: Vault.sol

618: for (uint w = 0; w < _winners.length; w++) {
620: for (uint p = 0; p < prizeIndicesLength; p++) {

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L618

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L620





</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |





### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Use `assembly` to write address storage values

#### <ins>Proof Of Concept</ins>

```solidity
File: PrizePool.sol

357: _numTiers = numberOfTiers;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L357

```solidity
File: PrizePool.sol

608: _numTiers = numberOfTiers;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L608

```solidity
File: PrizePool.sol

358: _nextNumberOfTiers = _numTiers;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L358

```solidity
File: PrizePool.sol

609: _nextNumberOfTiers = _numTiers;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L609

```solidity
File: PrizePool.sol

368: _winningRandomNumber = winningRandomNumber_;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L368

```solidity
File: PrizePool.sol

372: _lastClosedDrawStartedAt = openDrawStartedAt_;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L372

```solidity
File: PrizePool.sol

683: _numberOfTiers = numberOfTiers;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L683

```solidity
File: PrizePool.sol

851: _numberOfTiers = numberOfTiers;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L851

```solidity
File: PrizePool.sol

782: _claimExpansionThreshold = claimExpansionThreshold;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L782

```solidity
File: TieredLiquidityDistributor.sol

473: _lastClosedDrawId = lastClosedDrawId;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L473

```solidity
File: TieredLiquidityDistributor.sol

647: _numTiers = numberOfTiers;
```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L647

```solidity
File: TwabController.sol

598: _userDelegate = _user;
```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L598

```solidity
File: Vault.sol

271: _twabController = twabController_;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L271

```solidity
File: Vault.sol

272: _yieldVault = yieldVault_;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L272

```solidity
File: Vault.sol

273: _prizePool = prizePool_;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L273

```solidity
File: Vault.sol

642: _previousClaimer = _claimer;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L642

```solidity
File: Vault.sol

679: _liquidationPair = liquidationPair_;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L679

```solidity
File: Vault.sol

692: _previousYieldFeePercentage = _yieldFeePercentage;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L692

```solidity
File: Vault.sol

705: _previousYieldFeeRecipient = _yieldFeeRecipient;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L705

```solidity
File: Vault.sol

1210: _claimer = claimer_;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1210

```solidity
File: Vault.sol

1222: _yieldFeePercentage = yieldFeePercentage_;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1222

```solidity
File: Vault.sol

1230: _yieldFeeRecipient = yieldFeeRecipient_;
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1230





### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

#### <ins>Proof Of Concept</ins>


```solidity
File: PrizePool.sol

434: if (claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex]) {
438: claimedPrizes[msg.sender][_winner][lastClosedDrawId][_tier][_prizeIndex] = true;

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L434

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L438



```solidity
File: PrizePool.sol

484: uint256 _available = claimerRewards[msg.sender];
490: claimerRewards[msg.sender] -= _amount;

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L484

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L490



```solidity
File: DrawAccumulatorLib.sol

82: Observation memory newestObservation_ = accumulator.observations[newestDrawId_];
134: Observation memory newestObservation_ = accumulator.observations[newestDrawId_];
106: accumulator.observations[newestDrawId_] = Observation({

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L82

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L134

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L106



```solidity
File: TwabLib.sol

698: period != _getTimestampPeriod(PERIOD_LENGTH, PERIOD_OFFSET, _observations[0].timestamp)
700: : _time >= _observations[0].timestamp;

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L698

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L700






### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Remove import `forge-std`

It's used to print the values of variables while running tests to help debug and see what's happening inside your contracts but since it's a development tool, it serves no purpose on mainnet.
Also, the remember to remove the usage of calls that use `forge-std` when removing of the import of `forge-std`.

#### <ins>Proof Of Concept</ins>

```solidity
File: PrizePool.sol

4: import "forge-std/console2.sol";

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L4






### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


```solidity
File: PrizePool.sol

320: smoothing.intoSD59x18()
326: smoothing.intoSD59x18()
529: smoothing.intoSD59x18()
545: smoothing.intoSD59x18()
715: smoothing.intoSD59x18()
728: smoothing.intoSD59x18()
821: smoothing.intoSD59x18()
907: smoothing.intoSD59x18()

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L320

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L326

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L529

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L545

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L715

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L728

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L821

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L907



```solidity
File: PrizePool.sol

353: if (block.timestamp < _openDrawEndsAt()) {
354: revert DrawNotFinished(_openDrawEndsAt(), uint64(block.timestamp));

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L353-L354

```solidity
File: Vault.sol

560: if (_tokenIn != address(_prizePool.prizeToken()))
561: revert LiquidationTokenInNotPrizeToken(_tokenIn, address(_prizePool.prizeToken()));

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L560-L561







### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Superfluous event fields

`block.number` and `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.

#### <ins>Proof Of Concept</ins>


```solidity
File: Vault.sol

191: event Sponsor(address indexed caller, address indexed receiver, uint256 assets, uint256 shares);
```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L191



### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Use nested `if` and avoid multiple check combinations

Using nested `if`, is cheaper than using `&&` multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: DrawAccumulatorLib.sol

454: if (targetAtOrAfter && _targetLastClosedDrawId <= afterOrAtDrawId) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L454

```solidity
File: TwabController.sol

530: if (!_isFromDelegate && _fromDelegate != SPONSORSHIP_ADDRESS) {
561: if (!_isToDelegate && _toDelegate != SPONSORSHIP_ADDRESS) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L530

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L561



```solidity
File: TwabController.sol

619: if (_fromDelegate != address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) {
630: if (_toDelegate != address(0) && _toDelegate != SPONSORSHIP_ADDRESS) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L619

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L630



```solidity
File: ObservationLib.sol

88: if (targetAfterOrAt && _target.lte(afterOrAt.timestamp, _time)) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol#L88

```solidity
File: OverflowSafeComparatorLib.sol

17: if (_a <= _timestamp && _b <= _timestamp) return _a < _b;

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L17

```solidity
File: OverflowSafeComparatorLib.sol

33: if (_a <= _timestamp && _b <= _timestamp) return _a <= _b;

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L33

```solidity
File: OverflowSafeComparatorLib.sol

50: if (_a <= _timestamp && _b <= _timestamp) return _a - _b;

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L50

```solidity
File: Vault.sol

325: if (_availableYield != 0 && _yieldFeePercentage != 0) {

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L325

```solidity
File: Vault.sol

580: if (_vaultAssets != 0 && _amountOut >= _vaultAssets) {

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L580

```solidity
File: Vault.sol

1182: if (_totalSupplyAmount != 0 && _withdrawableAssets != 0) {

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1182



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }
    
    contract Contract0 {
    
        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }
    
    }
    
    contract Contract1 {
    
        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |




### <a href="#gas-summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Claimer.sol

134: if (_claimCount == 0) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L134

```solidity
File: LinearVRGDALib.sol

75: if (expResult == 0) {

```

https://github.com/GenerationSoftware/pt-v5-claimer/tree/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L75

```solidity
File: PrizePool.sol

350: if (winningRandomNumber_ == 0) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L350

```solidity
File: PrizePool.sol

763: (lastClosedDrawId == 0 ? 1 : 2) *

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L763

```solidity
File: PrizePool.sol

896: if (_lastClosedDrawId == 0) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L896

```solidity
File: TieredLiquidityDistributor.sol

605: return _tier == _numberOfTiers - 1;

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L605

```solidity
File: TieredLiquidityDistributor.sol

731: if (numTiers == 3) {
733: } else if (numTiers == 4) {
735: } else if (numTiers == 5) {
737: } else if (numTiers == 6) {
739: } else if (numTiers == 7) {
741: } else if (numTiers == 8) {
743: } else if (numTiers == 9) {
745: } else if (numTiers == 10) {
747: } else if (numTiers == 11) {
749: } else if (numTiers == 12) {
751: } else if (numTiers == 13) {
753: } else if (numTiers == 14) {
755: } else if (numTiers == 15) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L731

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L733

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L735

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L737

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L739

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L741

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L743

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L745

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L747

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L749

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L751

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L753

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L755



```solidity
File: TieredLiquidityDistributor.sol

765: if (numTiers == 3) {
767: } else if (numTiers == 4) {
769: } else if (numTiers == 5) {
771: } else if (numTiers == 6) {
773: } else if (numTiers == 7) {
775: } else if (numTiers == 8) {
777: } else if (numTiers == 9) {
779: } else if (numTiers == 10) {
781: } else if (numTiers == 11) {
783: } else if (numTiers == 12) {
785: } else if (numTiers == 13) {
787: } else if (numTiers == 14) {
789: } else if (numTiers == 15) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L765

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L767

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L769

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L771

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L773

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L775

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L777

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L779

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L781

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L783

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L785

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L787

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L789



```solidity
File: DrawAccumulatorLib.sol

70: if (_drawId == 0) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L70

```solidity
File: DrawAccumulatorLib.sol

126: if (ringBufferInfo.cardinality == 0) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L126

```solidity
File: DrawAccumulatorLib.sol

178: if (ringBufferInfo.cardinality == 0) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/DrawAccumulatorLib.sol#L178

```solidity
File: TierCalculationLib.sol

80: if (_vaultTwabTotalSupply == 0) {

```

https://github.com/generationsoftware/pt-v5-prize-pool/tree/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L80

```solidity
File: TwabController.sol

516: if (_from == _to) {
524: bool _isFromDelegate = _fromDelegate == _from;
538: _to == address(0) ||
539: (_toDelegate == SPONSORSHIP_ADDRESS && _fromDelegate != SPONSORSHIP_ADDRESS)
544: _to == address(0) ? _amount : 0,
545: (_to == address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) ||
539: (_toDelegate == SPONSORSHIP_ADDRESS && _fromDelegate != SPONSORSHIP_ADDRESS)
555: bool _isToDelegate = _toDelegate == _to;
569: _from == address(0) ||
570: (_fromDelegate == SPONSORSHIP_ADDRESS && _toDelegate != SPONSORSHIP_ADDRESS)

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L516

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L524

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L538

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L539

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L544

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L545

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L539

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L555

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L569

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L570




```solidity
File: TwabController.sol

597: if (_userDelegate == address(0)) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L597

```solidity
File: TwabController.sol

624: if (_toDelegate == address(0) || _toDelegate == SPONSORSHIP_ADDRESS) {
635: if (_fromDelegate == address(0) || _fromDelegate == SPONSORSHIP_ADDRESS) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L624

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L635



```solidity
File: TwabController.sol

650: if (_to == _currentDelegate) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L650

```solidity
File: ObservationLib.sol

78: if (beforeOrAtTimestamp == 0) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/ObservationLib.sol#L78

```solidity
File: TwabLib.sol

367: if (newestObservation.timestamp == currentTime) {
381: if (currentPeriod == 0 || currentPeriod > newestObservationPeriod) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L367

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L381



```solidity
File: TwabLib.sol

485: if (_accountDetails.cardinality == 0) {
497: if (_accountDetails.cardinality == 1) {
529: if (afterOrAtObservation.timestamp == _targetTime) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L485

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L497

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L529



```solidity
File: TwabLib.sol

572: if (_accountDetails.cardinality == 0) {
587: if (_accountDetails.cardinality == 1) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L572

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L587



```solidity
File: TwabLib.sol

690: if (_accountDetails.cardinality == 0) {
696: if (_accountDetails.cardinality == 1) {

```

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L690

https://github.com/generationsoftware/pt-v5-twab-controller/tree/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L696



```solidity
File: Vault.sol

564: if (_amountOut == 0) revert LiquidationAmountOutZero();

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L564

```solidity
File: Vault.sol

871: (_assets == 0 || _exchangeRate == 0)

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L871

```solidity
File: Vault.sol

902: (_shares == 0 || _exchangeRate == 0)

```

https://github.com/GenerationSoftware/pt-v5-vault/tree/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L902



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |




