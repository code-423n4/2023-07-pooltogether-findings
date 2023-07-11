# Use ``` do while ``` loops instead of ``` for ``` loops

A ``` do while ```  loop will cost less gas since the condition is not being checked for the first iteration. 

#### For Vault Sub Module:

Total Instances: ```1```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L618-L629

Gas Savings for ``` Vault.claimPrizes ``` , obtained via protocol’s tests: Avg 221 gas 


|     | Max    |
|-----|-------:|
|Before| 7475 |
|After | 7254 |

```solidity 

    for (uint w = 0; w < _winners.length; w++) {
      uint prizeIndicesLength = _prizeIndices[w].length;
      for (uint p = 0; p < prizeIndicesLength; p++) {
        totalPrizes += _claimPrize(
          _winners[w],
          _tier,
          _prizeIndices[w][p],
          _feePerClaim,
          _feeRecipient
        );
      }
    }

``` 
can be changed to:

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

