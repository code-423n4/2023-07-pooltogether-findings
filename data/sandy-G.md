## [G-01] Use ``do while`` loops instead of ``for`` loops


##### Description:
A ``do while`` loop will cost less gas since the condition is not being checked for the first iteration. Also, using do while loop with ++i is more gas efficient and wrapping pre-increment with  unchecked keyword makes the whole iteration the most gas efficient of all.


## Summary For ``Claimer`` submodule:

|Number | Problems  | instance |
|------ |-----------|----------|
|1 |Use do while loops instead of for loops | 2

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

## Summary For ``Vault`` submodule:

|Number | Problems  | instance |
|------ |-----------|----------|
|1 |Use do while loops instead of for loops | 1

##### Instance Number ``1``:

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

