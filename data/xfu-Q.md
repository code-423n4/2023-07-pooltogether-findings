## Missing array length equality checks may lead to incorrect or undefined behavior

### description:

If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array

**There is `1` instance of this issue:**

- Missing check lengths of parameters below in function [Claimer.claimPrizes(Vault,uint8,address[],uint32[][],address)](src/Claimer.sol#L60-L83):
  - [Claimer.claimPrizes(Vault,uint8,address[],uint32[][],address).winners](src/Claimer.sol#L63)
  - [Claimer.claimPrizes(Vault,uint8,address[],uint32[][],address).prizeIndices](src/Claimer.sol#L64)

### recommendation:

Check if the lengths of the array parameters are equal before use.

### locations:

- src/Claimer.sol#L60-L83

### severity:

Low