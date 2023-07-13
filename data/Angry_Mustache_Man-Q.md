# [L-1] No checks implemented for time parameters in `getTwabBetween`
### Vulnerability Detail
There is no comparison check included for `_startTime` and `_endTime` in `getTwabBetween` function, which would lead to an unnecessary underflow error at the end of the function for cases of "_startTime">"_endTime" where return variable is calculated as : 
```solidity 
  return
      (endObservation.cumulativeBalance - startObservation.cumulativeBalance) /
      (_endTime - _startTime); 
```
### Code Snippet
As the value is passed on by user from here : 
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L258-#L273
It the goes to the following function and neither of them have the check need for time parameters
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L288-#L321
### Tool used
Manual Review
### Recommendation
Add a check at `getTwabBetween` function similar to this: 
```solidity
require(_startTime>_endTime,"Incorrect Time Parameters");
```

# [NC-1] - `uint256` CAN BE USED INSTEAD OF `uint`
Both `uint` and `uint256` are used in the protocol’s code. In favor of explicitness, please consider using `uint256` instead of `uint` in the code.
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L105
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L106
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L131
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L132
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L144
