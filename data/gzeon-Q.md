# Low
## _transferBalance silently return when _from == _to
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L516-L518
This can be dangerous when other project integrating the TwabController assuming a transfer must have happened when it return without revert.