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

## Too many digits

### description:

Literals with many digits are difficult to read and review.


**There are `13` instances of this issue:**

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_12_13 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L171)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_14_15 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L200)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_7_8 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L116)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_4_5 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L95)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_9_10 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L135)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_11_12 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L158)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_2_3 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L86)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_6_7 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L108)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_3_4 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L90)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_10_11 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L146)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_13_14 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L185)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_8_9 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L125)

- [PrizePool.slitherConstructorConstantVariables()](src/PrizePool.sol#L124-L975) uses literals with too many digits:
	- [TIER_ODDS_5_6 = SD59x18.wrap(1000000000000000000)](src/abstract/TieredLiquidityDistributor.sol#L101)

#### Exploit scenario

```solidity
contract MyContract{
    uint 1_ether = 10000000000000000000; 
}
```

While `1_ether` looks like `1 ether`, it is `10 ether`. As a result, it's likely to be used incorrectly.


### recommendation:

Use:
- [Ether suffix](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#ether-units),
- [Time suffix](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#time-units), or
- [The scientific notation](https://solidity.readthedocs.io/en/latest/types.html#rational-and-integer-literals)


### locations:
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975
- src/PrizePool.sol#L124-L975

### severity:
Informational

## Unused state variable

### description:
Unused state variable.

**There is `1` instance of this issue:**

- [TieredLiquidityDistributor.GRAND_PRIZE_PERIOD_DRAWS](src/abstract/TieredLiquidityDistributor.sol#L66) is never used in [PrizePool](src/PrizePool.sol#L124-L975)


### recommendation:
Remove unused state variables.

### locations:
- src/abstract/TieredLiquidityDistributor.sol#L66

### severity:
Informational


## Events are missing sender information

### description:

When an action is triggered based on a user's action, not being able to filter based on 
who triggered the action makes event processing a lot more cumbersome. 
Including the `msg.sender` the events of these types of action will make events much more 
useful to end users.

**There is `1` instance of this issue:**

- [NewFactoryVault(_vault,VaultFactory(address(this)))](src/VaultFactory.sol#L83) should add `msg.sender` to event.

### recommendation:

Adding `msg.sender` to event.


### locations:
- src/VaultFactory.sol#L83

### severity:
Low
