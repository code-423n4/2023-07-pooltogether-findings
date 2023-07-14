## Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

### description:

> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

More detail see [this.](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html)

Each operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/0xxfu/3672fec07eb3031cd5da14ac015e04a1) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

**There are `8` instances of this issue:**

- `uint8 `[Claimer.claimPrizes(Vault,uint8,address[],uint32[][],address).tier](src/Claimer.sol#L62) should be used `uint256/int256`.

- `uint96 `[Claimer.claimPrizes(Vault,uint8,address[],uint32[][],address).feePerClaim](src/Claimer.sol#L72-L78) should be used `uint256/int256`.

- `uint8 `[Claimer.computeTotalFees(uint8,uint256).\_tier](src/Claimer.sol#L89) should be used `uint256/int256`.

- `uint8 `[Claimer.computeTotalFees(uint8,uint256,uint256).\_tier](src/Claimer.sol#L104) should be used `uint256/int256`.

- `uint8 `[Claimer.computeMaxFee(uint8).\_tier](src/Claimer.sol#L161) should be used `uint256/int256`.

- `uint8 `[Claimer.\_computeMaxFee(uint8,uint8).\_tier](src/Claimer.sol#L169) should be used `uint256/int256`.

- `uint8 `[Claimer.\_computeMaxFee(uint8,uint8).\_numTiers](src/Claimer.sol#L169) should be used `uint256/int256`.

- `uint8 `[Claimer.\_computeMaxFee(uint8,uint8).\_canaryTier](src/Claimer.sol#L170) should be used `uint256/int256`.

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

### severity:

Optimization