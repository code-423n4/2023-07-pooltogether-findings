## Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

### description:

> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

More detail see [this.](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html)

Each operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/0xxfu/3672fec07eb3031cd5da14ac015e04a1) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

**There are `121` instances of this issue:**

- `uint8 `[Claimer.claimPrizes(Vault,uint8,address[],uint32[][],address).tier](src/Claimer.sol#L62) should be used `uint256/int256`.

- `uint96 `[Claimer.claimPrizes(Vault,uint8,address[],uint32[][],address).feePerClaim](src/Claimer.sol#L72-L78) should be used `uint256/int256`.

- `uint8 `[Claimer.computeTotalFees(uint8,uint256).\_tier](src/Claimer.sol#L89) should be used `uint256/int256`.

- `uint8 `[Claimer.computeTotalFees(uint8,uint256,uint256).\_tier](src/Claimer.sol#L104) should be used `uint256/int256`.

- `uint8 `[Claimer.computeMaxFee(uint8).\_tier](src/Claimer.sol#L161) should be used `uint256/int256`.

- `uint8 `[Claimer.\_computeMaxFee(uint8,uint8).\_tier](src/Claimer.sol#L169) should be used `uint256/int256`.

- `uint8 `[Claimer.\_computeMaxFee(uint8,uint8).\_numTiers](src/Claimer.sol#L169) should be used `uint256/int256`.

- `uint8 `[Claimer.\_computeMaxFee(uint8,uint8).\_canaryTier](src/Claimer.sol#L170) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.lt(uint32,uint32,uint32).\_b](src/libraries/OverflowSafeComparatorLib.sol#L15) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.lt(uint32,uint32,uint32).\_timestamp](src/libraries/OverflowSafeComparatorLib.sol#L15) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.lt(uint32,uint32,uint32).\_a](src/libraries/OverflowSafeComparatorLib.sol#L15) should be used `uint256/int256`.

- `uint32 `[TwabController.PERIOD_LENGTH](src/TwabController.sol#L27) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.lte(uint32,uint32,uint32).\_b](src/libraries/OverflowSafeComparatorLib.sol#L31) should be used `uint256/int256`.

- `uint32 `[TwabController.PERIOD_OFFSET](src/TwabController.sol#L31) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.lte(uint32,uint32,uint32).\_timestamp](src/libraries/OverflowSafeComparatorLib.sol#L31) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.lte(uint32,uint32,uint32).\_a](src/libraries/OverflowSafeComparatorLib.sol#L31) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.checkedSub(uint32,uint32,uint32).](src/libraries/OverflowSafeComparatorLib.sol#L47) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.checkedSub(uint32,uint32,uint32).\_timestamp](src/libraries/OverflowSafeComparatorLib.sol#L47) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.checkedSub(uint32,uint32,uint32).\_b](src/libraries/OverflowSafeComparatorLib.sol#L47) should be used `uint256/int256`.

- `uint32 `[OverflowSafeComparatorLib.checkedSub(uint32,uint32,uint32).\_a](src/libraries/OverflowSafeComparatorLib.sol#L47) should be used `uint256/int256`.

- `uint24 `[ObservationLib.binarySearch(ObservationLib.Observation[365],uint24,uint24,uint32,uint16,uint32).\_newestObservationIndex](src/libraries/ObservationLib.sol#L57) should be used `uint256/int256`.

- `uint24 `[ObservationLib.binarySearch(ObservationLib.Observation[365],uint24,uint24,uint32,uint16,uint32).\_oldestObservationIndex](src/libraries/ObservationLib.sol#L58) should be used `uint256/int256`.

- `uint32 `[ObservationLib.binarySearch(ObservationLib.Observation[365],uint24,uint24,uint32,uint16,uint32).\_target](src/libraries/ObservationLib.sol#L59) should be used `uint256/int256`.

- `uint16 `[ObservationLib.binarySearch(ObservationLib.Observation[365],uint24,uint24,uint32,uint16,uint32).\_cardinality](src/libraries/ObservationLib.sol#L60) should be used `uint256/int256`.

- `uint32 `[ObservationLib.binarySearch(ObservationLib.Observation[365],uint24,uint24,uint32,uint16,uint32).\_time](src/libraries/ObservationLib.sol#L61) should be used `uint256/int256`.

- `uint32 `[ObservationLib.binarySearch(ObservationLib.Observation[365],uint24,uint24,uint32,uint16,uint32).beforeOrAtTimestamp](src/libraries/ObservationLib.sol#L75) should be used `uint256/int256`.

- `uint32 `[TwabLib.increaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96).PERIOD_LENGTH](src/libraries/TwabLib.sol#L76) should be used `uint256/int256`.

- `uint32 `[TwabLib.increaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96).PERIOD_OFFSET](src/libraries/TwabLib.sol#L77) should be used `uint256/int256`.

- `uint96 `[TwabLib.increaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96).\_amount](src/libraries/TwabLib.sol#L79) should be used `uint256/int256`.

- `uint96 `[TwabLib.increaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96).\_delegateAmount](src/libraries/TwabLib.sol#L80) should be used `uint256/int256`.

- `uint32 `[TwabLib.increaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96).currentTime](src/libraries/TwabLib.sol#L86) should be used `uint256/int256`.

- `uint32 `[TwabLib.increaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96).index](src/libraries/TwabLib.sol#L87) should be used `uint256/int256`.

- `uint32 `[TwabController.constructor(uint32,uint32).\_periodOffset](src/TwabController.sol#L142) should be used `uint256/int256`.

- `uint32 `[TwabController.constructor(uint32,uint32).\_periodLength](src/TwabController.sol#L142) should be used `uint256/int256`.

- `uint32 `[TwabLib.decreaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96,string).PERIOD_LENGTH](src/libraries/TwabLib.sol#L145) should be used `uint256/int256`.

- `uint32 `[TwabLib.decreaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96,string).PERIOD_OFFSET](src/libraries/TwabLib.sol#L146) should be used `uint256/int256`.

- `uint96 `[TwabLib.decreaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96,string).\_amount](src/libraries/TwabLib.sol#L148) should be used `uint256/int256`.

- `uint96 `[TwabLib.decreaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96,string).\_delegateAmount](src/libraries/TwabLib.sol#L149) should be used `uint256/int256`.

- `uint32 `[TwabLib.decreaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96,string).currentTime](src/libraries/TwabLib.sol#L168) should be used `uint256/int256`.

- `uint32 `[TwabLib.decreaseBalances(uint32,uint32,TwabLib.Account,uint96,uint96,string).index](src/libraries/TwabLib.sol#L169) should be used `uint256/int256`.

- `uint16 `[TwabLib.getOldestObservation(ObservationLib.Observation[365],TwabLib.AccountDetails).index](src/libraries/TwabLib.sol#L226) should be used `uint256/int256`.

- `uint32 `[TwabController.getBalanceAt(address,address,uint32).targetTime](src/TwabController.sol#L232) should be used `uint256/int256`.

- `uint32 `[TwabController.getTotalSupplyAt(address,uint32).targetTime](src/TwabController.sol#L244) should be used `uint256/int256`.

- `uint16 `[TwabLib.getNewestObservation(ObservationLib.Observation[365],TwabLib.AccountDetails).index](src/libraries/TwabLib.sol#L247) should be used `uint256/int256`.

- `uint32 `[TwabController.getTwabBetween(address,address,uint32,uint32).startTime](src/TwabController.sol#L261) should be used `uint256/int256`.

- `uint32 `[TwabController.getTwabBetween(address,address,uint32,uint32).endTime](src/TwabController.sol#L262) should be used `uint256/int256`.

- `uint32 `[TwabLib.getBalanceAt(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L264) should be used `uint256/int256`.

- `uint32 `[TwabLib.getBalanceAt(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).\_targetTime](src/libraries/TwabLib.sol#L267) should be used `uint256/int256`.

- `uint32 `[TwabController.getTotalSupplyTwabBetween(address,uint32,uint32).startTime](src/TwabController.sol#L285) should be used `uint256/int256`.

- `uint32 `[TwabController.getTotalSupplyTwabBetween(address,uint32,uint32).endTime](src/TwabController.sol#L286) should be used `uint256/int256`.

- `uint32 `[TwabLib.getTwabBetween(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L289) should be used `uint256/int256`.

- `uint32 `[TwabLib.getTwabBetween(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32,uint32).\_startTime](src/libraries/TwabLib.sol#L292) should be used `uint256/int256`.

- `uint32 `[TwabLib.getTwabBetween(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32,uint32).\_endTime](src/libraries/TwabLib.sol#L293) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_calculateTemporaryObservation(ObservationLib.Observation,uint32).\_time](src/libraries/TwabLib.sol#L332) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getNextObservationIndex(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails).PERIOD_LENGTH](src/libraries/TwabLib.sol#L353) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getNextObservationIndex(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails).PERIOD_OFFSET](src/libraries/TwabLib.sol#L354) should be used `uint256/int256`.

- `uint16 `[TwabLib.\_getNextObservationIndex(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails).index](src/libraries/TwabLib.sol#L360) should be used `uint256/int256`.

- `uint32 `[TwabController.getTimestampPeriod(uint32).time](src/TwabController.sol#L360) should be used `uint256/int256`.

- `uint32 `[TwabController.getTimestampPeriod(uint32).](src/TwabController.sol#L360) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getNextObservationIndex(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails).currentTime](src/libraries/TwabLib.sol#L362) should be used `uint256/int256`.

- `uint16 `[TwabLib.\_getNextObservationIndex(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails).newestIndex](src/libraries/TwabLib.sol#L363) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getNextObservationIndex(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails).currentPeriod](src/libraries/TwabLib.sol#L371) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getNextObservationIndex(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails).newestObservationPeriod](src/libraries/TwabLib.sol#L372-L376) should be used `uint256/int256`.

- `uint32 `[TwabController.isTimeSafe(address,address,uint32).time](src/TwabController.sol#L373) should be used `uint256/int256`.

- `uint32 `[TwabController.isTimeRangeSafe(address,address,uint32,uint32).startTime](src/TwabController.sol#L392) should be used `uint256/int256`.

- `uint32 `[TwabController.isTimeRangeSafe(address,address,uint32,uint32).endTime](src/TwabController.sol#L393) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_extrapolateFromBalance(ObservationLib.Observation,uint32).\_timestamp](src/libraries/TwabLib.sol#L401) should be used `uint256/int256`.

- `uint128 `[TwabLib.\_extrapolateFromBalance(ObservationLib.Observation,uint32).cumulativeBalance](src/libraries/TwabLib.sol#L402) should be used `uint256/int256`.

- `uint32 `[TwabController.isTotalSupplyTimeSafe(address,uint32).time](src/TwabController.sol#L415) should be used `uint256/int256`.

- `uint32 `[TwabLib.getTimestampPeriod(uint32,uint32,uint32).PERIOD_LENGTH](src/libraries/TwabLib.sol#L419) should be used `uint256/int256`.

- `uint32 `[TwabLib.getTimestampPeriod(uint32,uint32,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L420) should be used `uint256/int256`.

- `uint32 `[TwabLib.getTimestampPeriod(uint32,uint32,uint32).\_timestamp](src/libraries/TwabLib.sol#L421) should be used `uint256/int256`.

- `uint32 `[TwabLib.getTimestampPeriod(uint32,uint32,uint32).period](src/libraries/TwabLib.sol#L422) should be used `uint256/int256`.

- `uint32 `[TwabController.isTotalSupplyTimeRangeSafe(address,uint32,uint32).startTime](src/TwabController.sol#L432) should be used `uint256/int256`.

- `uint32 `[TwabController.isTotalSupplyTimeRangeSafe(address,uint32,uint32).endTime](src/TwabController.sol#L433) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getTimestampPeriod(uint32,uint32,uint32).PERIOD_LENGTH](src/libraries/TwabLib.sol#L436) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getTimestampPeriod(uint32,uint32,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L437) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getTimestampPeriod(uint32,uint32,uint32).\_timestamp](src/libraries/TwabLib.sol#L438) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getTimestampPeriod(uint32,uint32,uint32).period](src/libraries/TwabLib.sol#L439) should be used `uint256/int256`.

- `uint96 `[TwabController.mint(address,uint96).\_amount](src/TwabController.sol#L457) should be used `uint256/int256`.

- `uint32 `[TwabLib.getPreviousOrAtObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L457) should be used `uint256/int256`.

- `uint32 `[TwabLib.getPreviousOrAtObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).\_targetTime](src/libraries/TwabLib.sol#L460) should be used `uint256/int256`.

- `uint96 `[TwabController.burn(address,uint96).\_amount](src/TwabController.sol#L469) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getPreviousOrAtObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L474) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getPreviousOrAtObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).\_targetTime](src/libraries/TwabLib.sol#L477) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getPreviousOrAtObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).currentTime](src/libraries/TwabLib.sol#L479) should be used `uint256/int256`.

- `uint16 `[TwabLib.\_getPreviousOrAtObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).oldestTwabIndex](src/libraries/TwabLib.sol#L481) should be used `uint256/int256`.

- `uint96 `[TwabController.transfer(address,address,uint96).\_amount](src/TwabController.sol#L481) should be used `uint256/int256`.

- `uint16 `[TwabLib.\_getPreviousOrAtObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).newestTwabIndex](src/libraries/TwabLib.sol#L482) should be used `uint256/int256`.

- `uint96 `[TwabController.\_transferBalance(address,address,address,uint96).\_amount](src/TwabController.sol#L515) should be used `uint256/int256`.

- `uint32 `[TwabLib.getNextOrNewestObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L545) should be used `uint256/int256`.

- `uint32 `[TwabLib.getNextOrNewestObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).\_targetTime](src/libraries/TwabLib.sol#L548) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getNextOrNewestObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L562) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getNextOrNewestObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).\_targetTime](src/libraries/TwabLib.sol#L565) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getNextOrNewestObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).currentTime](src/libraries/TwabLib.sol#L567) should be used `uint256/int256`.

- `uint16 `[TwabLib.\_getNextOrNewestObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).oldestTwabIndex](src/libraries/TwabLib.sol#L569) should be used `uint256/int256`.

- `uint16 `[TwabLib.\_getNextOrNewestObservation(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).newestTwabIndex](src/libraries/TwabLib.sol#L594) should be used `uint256/int256`.

- `uint96 `[TwabController.\_transferDelegateBalance(address,address,address,uint96).\_amount](src/TwabController.sol#L616) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getSurroundingOrAtObservations(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L628) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_getSurroundingOrAtObservations(uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).\_targetTime](src/libraries/TwabLib.sol#L631) should be used `uint256/int256`.

- `uint32 `[TwabLib.isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_LENGTH](src/libraries/TwabLib.sol#L664) should be used `uint256/int256`.

- `uint32 `[TwabLib.isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L665) should be used `uint256/int256`.

- `uint32 `[TwabLib.isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).\_time](src/libraries/TwabLib.sol#L668) should be used `uint256/int256`.

- `uint96 `[TwabController.\_increaseBalances(address,address,uint96,uint96).\_amount](src/TwabController.sol#L676) should be used `uint256/int256`.

- `uint96 `[TwabController.\_increaseBalances(address,address,uint96,uint96).\_delegateAmount](src/TwabController.sol#L677) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_LENGTH](src/libraries/TwabLib.sol#L683) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L684) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).\_time](src/libraries/TwabLib.sol#L687) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).period](src/libraries/TwabLib.sol#L694) should be used `uint256/int256`.

- `uint96 `[TwabController.\_decreaseBalances(address,address,uint96,uint96).\_amount](src/TwabController.sol#L712) should be used `uint256/int256`.

- `uint96 `[TwabController.\_decreaseBalances(address,address,uint96,uint96).\_delegateAmount](src/TwabController.sol#L713) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).preOrAtPeriod](src/libraries/TwabLib.sol#L718-L722) should be used `uint256/int256`.

- `uint32 `[TwabLib.\_isTimeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32).postPeriod](src/libraries/TwabLib.sol#L723-L727) should be used `uint256/int256`.

- `uint32 `[TwabLib.isTimeRangeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32,uint32).PERIOD_LENGTH](src/libraries/TwabLib.sol#L744) should be used `uint256/int256`.

- `uint32 `[TwabLib.isTimeRangeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32,uint32).PERIOD_OFFSET](src/libraries/TwabLib.sol#L745) should be used `uint256/int256`.

- `uint32 `[TwabLib.isTimeRangeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32,uint32).\_startTime](src/libraries/TwabLib.sol#L748) should be used `uint256/int256`.

- `uint32 `[TwabLib.isTimeRangeSafe(uint32,uint32,ObservationLib.Observation[365],TwabLib.AccountDetails,uint32,uint32).\_endTime](src/libraries/TwabLib.sol#L749) should be used `uint256/int256`.

- `uint96 `[TwabController.\_decreaseTotalSupplyBalances(address,uint96,uint96).\_amount](src/TwabController.sol#L754) should be used `uint256/int256`.

- `uint96 `[TwabController.\_decreaseTotalSupplyBalances(address,uint96,uint96).\_delegateAmount](src/TwabController.sol#L755) should be used `uint256/int256`.

- `uint96 `[TwabController.\_increaseTotalSupplyBalances(address,uint96,uint96).\_amount](src/TwabController.sol#L795) should be used `uint256/int256`.

- `uint96 `[TwabController.\_increaseTotalSupplyBalances(address,uint96,uint96).\_delegateAmount](src/TwabController.sol#L796) should be used `uint256/int256`.

### recommendation:

Using `uint256/int256` replace `uint128/uint64/uint32/uint16/uint8` or `int128/int64/int32/int16/int8`

### locations:

- src/Claimer.sol#L62
- src/Claimer.sol#L72-L78
- src/Claimer.sol#L89
- src/Claimer.sol#L104
- src/Claimer.sol#L161
- src/Claimer.sol#L169
- src/Claimer.sol#L169
- src/Claimer.sol#L170
- src/libraries/OverflowSafeComparatorLib.sol#L15
- src/libraries/OverflowSafeComparatorLib.sol#L15
- src/libraries/OverflowSafeComparatorLib.sol#L15
- src/TwabController.sol#L27
- src/libraries/OverflowSafeComparatorLib.sol#L31
- src/TwabController.sol#L31
- src/libraries/OverflowSafeComparatorLib.sol#L31
- src/libraries/OverflowSafeComparatorLib.sol#L31
- src/libraries/OverflowSafeComparatorLib.sol#L47
- src/libraries/OverflowSafeComparatorLib.sol#L47
- src/libraries/OverflowSafeComparatorLib.sol#L47
- src/libraries/OverflowSafeComparatorLib.sol#L47
- src/libraries/ObservationLib.sol#L57
- src/libraries/ObservationLib.sol#L58
- src/libraries/ObservationLib.sol#L59
- src/libraries/ObservationLib.sol#L60
- src/libraries/ObservationLib.sol#L61
- src/libraries/ObservationLib.sol#L75
- src/libraries/TwabLib.sol#L76
- src/libraries/TwabLib.sol#L77
- src/libraries/TwabLib.sol#L79
- src/libraries/TwabLib.sol#L80
- src/libraries/TwabLib.sol#L86
- src/libraries/TwabLib.sol#L87
- src/TwabController.sol#L142
- src/TwabController.sol#L142
- src/libraries/TwabLib.sol#L145
- src/libraries/TwabLib.sol#L146
- src/libraries/TwabLib.sol#L148
- src/libraries/TwabLib.sol#L149
- src/libraries/TwabLib.sol#L168
- src/libraries/TwabLib.sol#L169
- src/libraries/TwabLib.sol#L226
- src/TwabController.sol#L232
- src/TwabController.sol#L244
- src/libraries/TwabLib.sol#L247
- src/TwabController.sol#L261
- src/TwabController.sol#L262
- src/libraries/TwabLib.sol#L264
- src/libraries/TwabLib.sol#L267
- src/TwabController.sol#L285
- src/TwabController.sol#L286
- src/libraries/TwabLib.sol#L289
- src/libraries/TwabLib.sol#L292
- src/libraries/TwabLib.sol#L293
- src/libraries/TwabLib.sol#L332
- src/libraries/TwabLib.sol#L353
- src/libraries/TwabLib.sol#L354
- src/libraries/TwabLib.sol#L360
- src/TwabController.sol#L360
- src/TwabController.sol#L360
- src/libraries/TwabLib.sol#L362
- src/libraries/TwabLib.sol#L363
- src/libraries/TwabLib.sol#L371
- src/libraries/TwabLib.sol#L372-L376
- src/TwabController.sol#L373
- src/TwabController.sol#L392
- src/TwabController.sol#L393
- src/libraries/TwabLib.sol#L401
- src/libraries/TwabLib.sol#L402
- src/TwabController.sol#L415
- src/libraries/TwabLib.sol#L419
- src/libraries/TwabLib.sol#L420
- src/libraries/TwabLib.sol#L421
- src/libraries/TwabLib.sol#L422
- src/TwabController.sol#L432
- src/TwabController.sol#L433
- src/libraries/TwabLib.sol#L436
- src/libraries/TwabLib.sol#L437
- src/libraries/TwabLib.sol#L438
- src/libraries/TwabLib.sol#L439
- src/TwabController.sol#L457
- src/libraries/TwabLib.sol#L457
- src/libraries/TwabLib.sol#L460
- src/TwabController.sol#L469
- src/libraries/TwabLib.sol#L474
- src/libraries/TwabLib.sol#L477
- src/libraries/TwabLib.sol#L479
- src/libraries/TwabLib.sol#L481
- src/TwabController.sol#L481
- src/libraries/TwabLib.sol#L482
- src/TwabController.sol#L515
- src/libraries/TwabLib.sol#L545
- src/libraries/TwabLib.sol#L548
- src/libraries/TwabLib.sol#L562
- src/libraries/TwabLib.sol#L565
- src/libraries/TwabLib.sol#L567
- src/libraries/TwabLib.sol#L569
- src/libraries/TwabLib.sol#L594
- src/TwabController.sol#L616
- src/libraries/TwabLib.sol#L628
- src/libraries/TwabLib.sol#L631
- src/libraries/TwabLib.sol#L664
- src/libraries/TwabLib.sol#L665
- src/libraries/TwabLib.sol#L668
- src/TwabController.sol#L676
- src/TwabController.sol#L677
- src/libraries/TwabLib.sol#L683
- src/libraries/TwabLib.sol#L684
- src/libraries/TwabLib.sol#L687
- src/libraries/TwabLib.sol#L694
- src/TwabController.sol#L712
- src/TwabController.sol#L713
- src/libraries/TwabLib.sol#L718-L722
- src/libraries/TwabLib.sol#L723-L727
- src/libraries/TwabLib.sol#L744
- src/libraries/TwabLib.sol#L745
- src/libraries/TwabLib.sol#L748
- src/libraries/TwabLib.sol#L749
- src/TwabController.sol#L754
- src/TwabController.sol#L755
- src/TwabController.sol#L795
- src/TwabController.sol#L796

### severity:

Optimization

## Using `x >> constant(uint)` with the right shift operator is a more gas-efficient

### description:

`<x> / 2` is the same as `<x> >> 1`. While the compiler uses the `SHR` opcode to accomplish both,
the version that uses division incurs an overhead of [**20 gas**](https://gist.github.com/0xxfu/84e3727f28e01f9b628836d5bf55d0cc)
due to `JUMP`s to and from a compiler utility function that introduces checks which can
be avoided by using `unchecked {}` around the division by two

**There are `2` instances of this issue:**

- [uint256(unsafeWadMul(targetPriceInt,extraPrecisionExpResult)) / 1e18](src/libraries/LinearVRGDALib.sol#L92) should use right shift `>>` operator to save gas.

- [ud2x18(uint64(uint256(wadExp(maxDiv / 1e18))))](src/libraries/LinearVRGDALib.sol#L110) should use right shift `>>` operator to save gas.

### recommendation:

Using bit shifting (`>>` operator) replace division divided by constant.

### locations:

- src/libraries/LinearVRGDALib.sol#L92
- src/libraries/LinearVRGDALib.sol#L110

### severity:

Optimization

## Cache the `<array>.length` for the loop condition

### description:

The overheads outlined below are _PER LOOP_, excluding the first loop

- storage arrays incur a Gwarmaccess (**100 gas**)
- memory arrays use `MLOAD` (**3 gas**)
- calldata arrays use `CALLDATALOAD` (**3 gas**)

Caching the length changes each of these to a `DUP<N>` (**3 gas**), and gets rid of the extra `DUP<N>` needed to store the stack offset.
More detail optimization see [this](https://gist.github.com/0xxfu/80fcbc39d2d38d85ae61b4b8838ef30b)

**There is `1` instance of this issue:**

- [i < winners.length](src/Claimer.sol#L68) `<array>.length` should be cached.

### recommendation:

Caching the `<array>.length` for the loop condition, for example:

```solidity
// gas save (-230)
function loopArray_cached(uint256[] calldata ns) public returns (uint256 sum) {
  uint256 length = ns.length;
  for (uint256 i = 0; i < length; ) {
    sum += ns[i];
    unchecked {
      i++;
    }
  }
}
```

### locations:

- src/Claimer.sol#L68

### severity:

Optimization

## Use indexed events for value types as they are less costly compared to non-indexed ones

### description:

Using the `indexed` keyword for [value types](https://docs.soliditylang.org/en/v0.8.20/types.html#value-types) (`bool/int/address/string/bytes`) saves gas costs, as seen in [this example](https://gist.github.com/0xxfu/c292a65ecb61cae6fd2090366ea0877e).

However, this is only the case for value types, whereas indexing [reference types](https://docs.soliditylang.org/en/v0.8.20/types.html#reference-types) (`array/struct`) are more expensive than their unindexed version.

**There are `6` instances of this issue:**

- The following variables should be indexed in [TwabController.IncreasedBalance(address,address,uint96,uint96)](src/TwabController.sol#L53-L58):

  - [delegateAmount](src/TwabController.sol#L57)

- The following variables should be indexed in [TwabController.DecreasedBalance(address,address,uint96,uint96)](src/TwabController.sol#L67-L72):

  - [amount](src/TwabController.sol#L70)

- The following variables should be indexed in [TwabController.ObservationRecorded(address,address,uint112,uint112,bool,ObservationLib.Observation)](src/TwabController.sol#L83-L90):

  - [balance](src/TwabController.sol#L86)

- The following variables should be indexed in [TwabController.IncreasedTotalSupply(address,uint96,uint96)](src/TwabController.sol#L106):

  - [amount](src/TwabController.sol#L106)

  - [delegateAmount](src/TwabController.sol#L106)

- The following variables should be indexed in [TwabController.DecreasedTotalSupply(address,uint96,uint96)](src/TwabController.sol#L114):

  - [delegateAmount](src/TwabController.sol#L114)

  - [amount](src/TwabController.sol#L114)

- The following variables should be indexed in [TwabController.TotalSupplyObservationRecorded(address,uint112,uint112,bool,ObservationLib.Observation)](src/TwabController.sol#L124-L130):

  - [delegateBalance](src/TwabController.sol#L127)

  - [balance](src/TwabController.sol#L126)

### recommendation:

Using the `indexed` keyword for values types `bool/int/address/string/bytes` in event

### locations:

- src/TwabController.sol#L53-L58
- src/TwabController.sol#L67-L72
- src/TwabController.sol#L83-L90
- src/TwabController.sol#L106
- src/TwabController.sol#L114
- src/TwabController.sol#L124-L130

### severity:

Optimization

## Using `private` rather than `public` for constants, saves gas

### description:

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants.

Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

**There is `1` instance of this issue:**

- [TwabController.SPONSORSHIP_ADDRESS](src/TwabController.sol#L24) should be used `private` visibility to save gas.

### recommendation:

Using `private` replace `public` with constant.

### locations:

- src/TwabController.sol#L24

### severity:

Optimization

## `++i` costs less gas than `i++`, especially when it's used in for-loops (`--i/i--` too)

### description:

`++i` costs less gas compared to `i++` or `i += 1` for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration).
This statement is true even with the optimizer enabled.

`i++` increments i and returns the initial value of i. Which means:

```
uint i = 1;
i++; // == 1 but i == 2
```

But ++i returns the actual incremented value:

```
uint i = 1;
++i; // == 2 and i == 2 too, so no need for a temporary variable
```

In the first case, the compiler has to create a temporary variable (when used)
for returning 1 instead of 2

**There are `2` instances of this issue:**

- [accountDetails.cardinality += 1](src/libraries/TwabLib.sol#L115) should use `++i`/`--i` instead of `i++`/`i--`/`i+=1`/`i-=1` operator to save gas.

- [accountDetails.cardinality += 1](src/libraries/TwabLib.sol#L199) should use `++i`/`--i` instead of `i++`/`i--`/`i+=1`/`i-=1` operator to save gas.

### recommendation:

Using `++i`/`--i` instead of `i++`/`i--`/`i+=1`/`i-=1` to operate the value of an uint variable.

### locations:

- src/libraries/TwabLib.sol#L115
- src/libraries/TwabLib.sol#L199

### severity:

Optimization

## Multiple address `mappings` can be combined into a single `mapping`

### description:

Saves a storage slot for the `mapping`.
Depending on the circumstances and sizes of types,
can avoid a `Gsset` (20000 gas) per `mapping` combined. Reads and subsequent writes can also
be cheaper when a function requires both values and they both fit in the same storage slot.
Finally, if both fields are accessed in the same function, can save `~42 gas` per access
due to not having to recalculate the key’s `keccak256` hash (Gkeccak256 - 30 gas) and
that calculation’s associated stack operations.

**There is `1` instance of this issue:**

- Following mappings should be combined into one:
  - [TwabController.userObservations](src/TwabController.sol#L36)
  - [TwabController.totalSupplyObservations](src/TwabController.sol#L39)
  - [TwabController.delegates](src/TwabController.sol#L42)

### recommendation:

Multiple address `mappings` can be combined into a single mapping of
an address to a struct, where appropriate.

### locations:

- src/TwabController.sol#L36

### severity:

Optimization

## Dead-code: functions not used should be removed to save deployment gas

### description:

Functions that are not sued.

**There is `1` instance of this issue:**

- [OverflowSafeComparatorLib.lt(uint32,uint32,uint32)](src/libraries/OverflowSafeComparatorLib.sol#L15-L23) is never used and should be removed

### recommendation:

Remove unused functions.

### locations:

- src/libraries/OverflowSafeComparatorLib.sol#L15-L23

### severity:

Optimization
