**claimer/src/libraries/LinearVRGDALib.sol**
- L6 - toWadUnsafe, unsafeWadDiv functions are imported, but they are never used, therefore they generate more gas expense in the deploy and also generate a low understanding in the blockchain explorer.

- L50 - A division is performed by an input of the function, this should include a prior validation that the input != 0, so as not to generate an unhandled exception.


**claimer/src/Claimer.sol**
- L37 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0x.

- L63/64/68/69 - An array is used to traverse another, since it is not validated that they have the same length, this could generate problems, lack of traversal or generate an unhandled exception.


**prize-pool/src/libraries/TierCalculationLib.sol**
- L23/33 - A division is performed by an input of the function, this should include a prior validation that the input != 0, so as not to generate an unhandled exception.


**prize-pool/src/libraries/UD34x4.sol**
- L5 - uMAX_UD60x18 function is imported, but it is never used, therefore it generate more gas expense in the deploy and also generate a low understanding in the blockchain explorer.


**prize-pool/src/PrizePool.sol**
- L4 - console2 is imported and this type of code for testing should be removed in a final version.

- L8/12 - fromSD59x18, fromUD60x18toUD34x4 and toUD34x4 functions are imported, but they are never used, therefore they generate more gas expense in the deploy and also generate a low understanding in the blockchain explorer.

- L258 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0x.


**vault/src/Vault.sol**
- L609/610/618/619 - An array is used to traverse another, since it is not validated that they have the same length, this could generate problems, lack of traversal or generate an unhandled exception.

- L873/886/904 - A division is performed by the _rounding input and it is not validated that it is != 0. This should be validated so as not to generate an unhandled exception.


**prize-pool/src/abstract/TieredLiquidityDistributor.sol**
- L235 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0x.

- L599 - A division is made by the _fractionalPrizeCount input and it is not validated that it is != 0. This should be validated so as not to generate an unhandled exception.
