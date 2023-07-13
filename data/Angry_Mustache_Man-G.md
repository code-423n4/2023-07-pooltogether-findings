# [G-1] - Reducing Bytecode Size With Modifiers
We can constrain the bytecode size by refactoring a `modifier` so it calls an internal function. This approach is used in OpenZeppelin's [Ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/cd48b3eab380254b08d7893a5a7bf568a33c5259/contracts/access/Ownable.sol#LL45C9-L45C9)
### Code Snippet
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/main/src/PrizePool.sol#L287-#L292
```solidity  
modifier onlyDrawManager() {
    if (msg.sender != drawManager) {
      revert CallerNotDrawManager(msg.sender, drawManager);
    }
    _;
  }
```
### Recommendation 
```solidity
modifier onlyDrawManager() {
    _checkDrawManager();
    _;
}

function _checkDrawManager() internal view virtual {
   if (msg.sender != drawManager) {
      revert CallerNotDrawManager(msg.sender, drawManager);
    }
}
```
# [G-2] - Reading a value from Memory Struct can be Expensive
Reading a value from Memory struct can be very expensive , so avoid doing it as the following code: 
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/main/src/libraries/DrawAccumulatorLib.sol#L96-L99
```solidity
 uint16 cardinality = ringBufferInfo.cardinality;
      if (ringBufferInfo.cardinality < MAX_CARDINALITY) {
        cardinality += 1;
      }
```
instead of this use the `cardinality` variable for better gas optimizations. 