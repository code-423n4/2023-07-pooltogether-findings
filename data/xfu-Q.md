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


## Setters should check the input value

### description:

Setters should have initial value check to prevent assigning wrong value to the variable.
Assignment of wrong value can lead to unexpected behavior of the contract.

**There are `2` instances of this issue:**

- [Claimer.constructor(PrizePool,uint256,uint256,uint256,UD2x18).\_minimumFee](src/Claimer.sol#L39) lacks an upper limit check on :

  - [minimumFee = \_minimumFee](src/Claimer.sol#L49)

- [Claimer.constructor(PrizePool,uint256,uint256,uint256,UD2x18).\_timeToReachMaxFee](src/Claimer.sol#L41) lacks an upper limit check on :
  - [timeToReachMaxFee = \_timeToReachMaxFee](src/Claimer.sol#L50)

### recommendation:

Add an upper limit check to the setters function.

### locations:

- src/Claimer.sol#L39
- src/Claimer.sol#L41

### severity:

Low

