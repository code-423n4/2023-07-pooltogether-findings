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


