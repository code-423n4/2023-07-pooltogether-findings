In code instead of uint we need to write uint256 because in case of uint solidity compiler will do additional steps for casting it to uint256.

In for loops when we are doing initialization of index we don't need to explicitly write uint256 i = 0. In that case compiler will consume additional gas for =. Implicitly compiler will do initialization of variable with it's default value (in this case 0). Conclusion` we can write uint256 i;

In PrizePool.sol contract claimedPrizes mapping is very complicated. Instead of nested mappings we can simply write mapping(address => StructForTheseVariables{
                     account,
                     drawId,
                     tier,
                     prizeIndex,
                     claimed
                   })