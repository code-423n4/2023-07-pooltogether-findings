### Gas Optimizations List:
| Number | Optimization Details | Instances |
|:--:|:-------| :-----:|
| [G-01] |Use indexed events|1|
| [G-02] | Using storage instead of memory for structs/arrays saves gas |2|
| [G-03] | Use ``do while`` loops instead of ``for`` loops | 4 |

## [G-01] Use indexed events

##### Description:
Using indexed events can help save gas when working with events in Solidity. There can be max 3 indexed arguments in events but only relevant parameters should be indexed. 

##### Instance Number ``1``:

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L176

###### Savings:

| | Average  | Median | Max 
|------ |-----------|----------|----------|
|Before |13885  | 7968   | 27728
After|13826  | 7880   | 27640

Gas Savings for ``event IncreaseReserve``, obtained via tests can be up to ``88 gas``.

###### Implementation:
```solidity 
prize-pool/src/PrizePool.sol

176: event IncreaseReserve(address user, uint256 amount);
```

###### This can be changed to:
```solidity
     event IncreaseReserve(address indexed user, uint256 amount);
```


###### ``NOTE:`` In several instances, where all there indexed arguments available in solidity events can be used, are not used. I suggest to make those arguments indexed by carefully selecting relevant parameters as you can optimize gas usage and improve the efficiency of event filtering.


## [G-02] Using storage instead of memory for structs/arrays saves gas


##### Description:
Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don’t incur extra Gcoldsload (2100 gas) for fields that you will never use.

###### Note: These are instances that the automated report missed.

##### Instance Number ``1``:

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L84-L86

###### Savings:

| | Average  | Median | Max 
|------ |-----------|----------|----------|
|Before |30470  | 30605   | 56136
After|27992  | 28120   | 53651

Gas Savings for ``TwabLib.increaseBalances``, obtained via tests can be up to ``2485 gas``.

###### Implementation:
```solidity 
twab-controller/src/libraries/TwabLib.sol

84:  {
85:    AccountDetails memory accountDetails = _account.details;
86:    uint32 currentTime = uint32(block.timestamp);
```

###### This can be changed to:
```solidity
  {
    AccountDetails storage accountDetails = _account.details;
    uint32 currentTime = uint32(block.timestamp);
```

##### Instance Number ``2``:

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L154C1-L157C44

###### Savings:

| | Average  | Median | Max 
|------ |-----------|----------|----------|
|Before |25122  | 30427   | 30427
After|23370  | 28334   | 28334

Gas Savings for ``TwabLib.decreaseBalances``, obtained via tests can be up to ``2093 gas``.

###### Implementation:
```solidity 
twab-controller/src/libraries/TwabLib.sol

154:  {
155:    AccountDetails memory accountDetails = _account.details;
156:
157:    if (accountDetails.balance < _amount) {
```

###### This can be changed to:
```solidity
  {
    AccountDetails storage accountDetails = _account.details;

    if (accountDetails.balance < _amount) {
```



## [G-03] Use ``do while`` loops instead of ``for`` loops


##### Description:
A ``do while`` loop will cost less gas since the condition is not being checked for the first iteration. Also, using do while loop with ++i is more gas efficient and wrapping pre-increment with  unchecked keyword makes the whole iteration the most gas efficient of all.


## For ``Claimer`` Sub Module:

##### Instance Number ``1``:

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L68C1-L70C6

###### Savings:

| | Average  | Median | Max 
|------ |-----------|----------|----------|
|Before |10918 | 10096  | 13962 
After|10789 | 9987   | 13771

Gas Savings for ``Claimer.claimPrizes``, obtained via tests can be up to ``191 gas``.

###### Implementation:
```solidity 
claimer/src/Claimer.sol

68:    for (uint i = 0; i < winners.length; i++) {
69:      claimCount += prizeIndices[i].length;
70:    }
```

###### This can be changed to:
```solidity
        uint256 i;
        do {
            claimCount += prizeIndices[i].length;

            unchecked {
                ++i;
            }
        } while (i < winners.length);
```

##### Instance Number ``2``:

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144C1-L153C6

###### Savings:

Since, this is an internal function which is used in four different functions, i.e:
1. claimPrizes => saves up to ``191`` gas.
2. computeTotalFees(uint8 _tier, uint256 _claimCount) => saves up to ``191`` gas.
3. computeTotalFees(uint8 _tier, uint256 _claimCount, uint256 _claimedCount) => saves up to ``191`` gas.
4. computeFeePerClaim => saves up to ``109`` gas.

###### Implementation:
```solidity 
claimer/src/Claimer.sol

144:    for (uint i = 0; i < _claimCount; i++) {
145:      fee += _computeFeeForNextClaim(
146:        minimumFee,
147:        decayConstant,
148:        perTimeUnit,
149:        elapsed,
150:        _claimedCount + i,
151:        _maxFee
152:      );
153:    }
```

###### This can be changed to:
```solidity
        uint256 i;
        do {
            fee += _computeFeeForNextClaim(minimumFee, decayConstant, perTimeUnit, elapsed, _claimedCount + i, _maxFee);

            unchecked {
                ++i;
            }
        } while (i < _claimCount);
```

## For ``Vault`` submodule:

##### Instance Number ``3``:

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L618-L629

###### Savings:

| | Average  | Median | Max 
|------ |-----------|----------|----------|
|Before  | 4856   | 5770   | 7254  
After | 4989   | 5991   | 7475 

Gas Savings for ``` Vault.claimPrizes ```, obtained via tests can be up to ``221`` gas.

###### Implementation:
```solidity 
vault/src/Vault.sol

618:    for (uint w = 0; w < _winners.length; w++) {
619:      uint prizeIndicesLength = _prizeIndices[w].length;
620:      for (uint p = 0; p < prizeIndicesLength; p++) {
621:        totalPrizes += _claimPrize(
622:          _winners[w],
623:          _tier,
624:          _prizeIndices[w][p],
625:          _feePerClaim,
626:          _feeRecipient
627:        );
628:      }
629:    }

``` 

###### This can be changed to:
```solidity 

        uint256 w;
        do {
            uint256 prizeIndicesLength = _prizeIndices[w].length;

            uint256 p;
            do {
                totalPrizes += _claimPrize(_winners[w], _tier, _prizeIndices[w][p], _feePerClaim, _feeRecipient);
                unchecked {
                    ++p;
                }
            } while (p < prizeIndicesLength);
            unchecked {
                ++w;
            }
        } while (w < _winners.length);

```

## For ``prize-pool`` submodule:

##### Instance Number ``4``:

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L138C1-L144C6

###### Savings:

| | Average  | Median | Max 
|------ |-----------|----------|----------|
|Before  | 180531 | 180238 | 322688  
After | 179655 | 179362 | 321161

Gas Savings for ``` TierCalculationLib.estimatedClaimCount ```, obtained via tests can be up to ``1527`` gas.

###### Implementation:
```solidity 
prize-pool/src/libraries/TierCalculationLib.sol

138:    for (uint8 i = 0; i < _numberOfTiers; i++) {
139:      count += uint32(
140:        uint256(
141:          unwrap(sd(int256(prizeCount(i))).mul(getTierOdds(i, _numberOfTiers, _grandPrizePeriod)))
142:        )
143:      );
144:    }

``` 

###### This can be changed to:
```solidity 

        uint8 i;
        do {
            count += uint32(
                uint256(unwrap(sd(int256(prizeCount(i))).mul(getTierOdds(i, _numberOfTiers, _grandPrizePeriod))))
            );
            unchecked {
                ++i;
            }
        } while (i < _numberOfTiers);
```
