## GAS-1: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* DrawAccumulatorLib.sol (Line: 126, 178, 269, 286)
* OverflowSafeComparatorLib.sol (Line: 17, 17, 33, 33, 50, 50)
* PrizePool.sol (Line: 360, 440, 611)
* TieredLiquidityDistributor.sol (Line: 565, 731, 733, 735, 737, 739, 741, 743, 745, 747, 749, 751, 753, 755, 765, 767, 769, 771, 773, 775, 777, 779, 781, 783, 785, 787, 789, 809, 810, 811, 813, 814, 815, 816, 818, 819, 820, 821, 822, 824, 825, 826, 827, 828, 829, 831, 832, 833, 834, 835, 836, 837, 839, 840, 841, 842, 843, 844, 845, 846, 848, 849, 850, 851, 852, 853, 854, 855, 856, 858, 859, 860, 861, 862, 863, 864, 865, 866, 867, 869, 870, 871, 872, 873, 874, 875, 876, 877, 878, 879, 881, 882, 883, 884, 885, 886, 887, 888, 889, 890, 891, 892, 894, 895, 896, 897, 898, 899, 900, 901, 902, 903, 904, 905, 906, 908, 909, 910, 911, 912, 913, 914, 915, 916, 917, 918, 919, 920, 921, 923, 924, 925, 926, 927, 928, 929, 930, 931, 932, 933, 934, 935, 936)
* TwabController.sol (Line: 530, 561, 619, 630, 691, 734, 776, 810)
* TwabLib.sol (Line: 95, 103, 114, 179, 187, 198, 485, 492, 497, 498, 572, 587, 690, 696)
* Vault.sol (Line: 325, 572, 580, 946)

## GAS-2: Functions guaranteed to revert when called by normal users can be marked payable

### Description

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Affected file

* PrizePool.sol (Line: 335, 348)
* Vault.sol (Line: 641, 665, 691, 704)

## GAS-3: Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Description

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

### Affected file

* PrizePool.sol (Line: 202, 206, 210)
* TwabController.sol (Line: 36, 39, 42)

## GAS-4: Not using the named return variables when a function returns, wastes deployment gas

### Description

It is not necessary to have both a named return and a return statement.

### Affected file

* Claimer.sol (Line: 66)
* TieredLiquidityDistributor.sol (Line: 388)
* TwabLib.sol (Line: 361, 361, 361, 402, 422, 439, 439, 461, 478, 478, 478, 478, 478, 478, 478, 549, 566, 566, 566, 566, 566, 566)

## GAS-5: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* PrizePool.sol (Line: 287)

## GAS-6: Use ```assembly``` to write address storage values

### Affected file

* Claimer.sol (Line: 44, 45, 46, 49, 50)
* PrizePool.sol (Line: 271, 272, 273, 274, 275, 276, 278, 303, 368, 369, 370, 371, 372, 373, 447, 782)
* TieredLiquidityDistributor.sol (Line: 236, 237, 238, 239, 241, 247, 253, 259, 265, 271, 277, 283, 289, 295, 301, 307, 313, 337, 371, 372, 373, 473, 540, 543)
* TwabController.sol (Line: 143, 144, 234, 245, 264, 288, 310, 325, 338, 351, 374, 395, 416, 435, 679, 715, 757, 798)
* Vault.sol (Line: 271, 272, 273, 279, 679, 1110, 1210, 1222, 1230)

## GAS-7: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* LinearVRGDALib.sol (Line: 56, 58, 62, 70, 82, 83, 89, 90)
* Vault.sol (Line: 282, 376, 384, 677)

## GAS-8: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* PrizePool.sol (Line: 312, 500)
* Vault.sol (Line: 336, 435, 468, 502, 562, 563, 570, 578, 581, 801, 809, 830, 932, 954, 959, 986, 1026, 1176)
* VaultFactory.sol (Line: 83)

## GAS-9: Using private rather than public for constants saves gas

### Description

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

### Affected file

* TwabController.sol (Line: 24)

## GAS-10: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* TierCalculationLib.sol (Line: 110)