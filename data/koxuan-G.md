# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 2 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 1 |
| [GAS-3](#GAS-3) | State variables should be cached in stack variables rather than re-reading them from storage | 2 |
| [GAS-4](#GAS-4) | Use calldata instead of memory for function arguments that do not get mutated | 1 |
| [GAS-5](#GAS-5) | Don't initialize variables with default value | 2 |
| [GAS-6](#GAS-6) | Functions guaranteed to revert when called by normal users can be marked `payable` | 2 |
| [GAS-7](#GAS-7) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 5 |
| [GAS-8](#GAS-8) | Use != 0 instead of > 0 for unsigned integer comparison | 3 |
| [GAS-9](#GAS-9) | `internal` functions not called by the contract should be removed | 9 |
### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (2)*:
```solidity
File: PrizePool.sol

279:     if (params.drawManager != address(0)) {

300:     if (drawManager != address(0)) {

```

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (1)*:
```solidity
File: PrizePool.sol

206:   mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))

```

### <a name="GAS-3"></a>[GAS-3] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (2)*:
```solidity
File: abstract/TieredLiquidityDistributor.sol

364:         prizeTokenPerShare: prizeTokenPerShare,

540:       _tierStruct.prizeTokenPerShare = prizeTokenPerShare;

```

### <a name="GAS-4"></a>[GAS-4] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (1)*:
```solidity
File: PrizePool.sol

259:     ConstructorParams memory params

```

### <a name="GAS-5"></a>[GAS-5] Don't initialize variables with default value

*Instances (2)*:
```solidity
File: libraries/TierCalculationLib.sol

137:     uint32 count = 0;

138:     for (uint8 i = 0; i < _numberOfTiers; i++) {

```

### <a name="GAS-6"></a>[GAS-6] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (2)*:
```solidity
File: PrizePool.sol

335:   function withdrawReserve(address _to, uint104 _amount) external onlyDrawManager {

348:   function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager returns (uint16) {

```

### <a name="GAS-7"></a>[GAS-7] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (5)*:
```solidity
File: PrizePool.sol

441:       canaryClaimCount++;

443:       claimCount++;

```

```solidity
File: abstract/TieredLiquidityDistributor.sol

361:     for (uint8 i = start; i < end; i++) {

630:     for (uint8 i = start; i < end; i++) {

```

```solidity
File: libraries/TierCalculationLib.sol

138:     for (uint8 i = 0; i < _numberOfTiers; i++) {

```

### <a name="GAS-8"></a>[GAS-8] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (3)*:
```solidity
File: libraries/DrawAccumulatorLib.sol

270:       observationDrawIdBeforeOrAtStart > 0 &&

271:       firstObservationDrawIdOccurringAtOrAfterStart > 0 &&

287:       firstObservationDrawIdOccurringAtOrAfterStart > 0 &&

```

### <a name="GAS-9"></a>[GAS-9] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (9)*:
```solidity
File: libraries/DrawAccumulatorLib.sol

64:   function add(

120:   function getTotalRemaining(

151:   function newestObservation(

166:   function getDisbursedBetween(

```

```solidity
File: libraries/TierCalculationLib.sol

32:   function estimatePrizeFrequencyInDraws(SD59x18 _tierOdds) internal pure returns (uint256) {

52:   function canaryPrizeCount(

73:   function isWinner(

104:   function calculatePseudoRandomNumber(

133:   function estimatedClaimCount(

```

