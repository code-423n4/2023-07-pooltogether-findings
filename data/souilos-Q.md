# VULN 1 

## [LOW] Use .call instead of .transfer to send ether
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 1155 at 2023-07-pooltogether/vault/src/Vault.sol:

    _twabController.transfer(_from, _to, uint96(_shares));


Found in line 17 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

            Every token.transfer() creates a new TWAB observation. The new TWAB observation is

------------------------------------------------------------------------ 

### Mitigation 

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues.Replace .transfer with .call. Note that the result of .call need to be checked.










# VULN 2 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 399 at 2023-07-pooltogether/vault/src/Vault.sol:

    _mint(_recipient, _shares);


Found in line 584 at 2023-07-pooltogether/vault/src/Vault.sol:

    _mint(_account, _amountOut);


Found in line 960 at 2023-07-pooltogether/vault/src/Vault.sol:

    _mint(_receiver, _shares);


Found in line 1122 at 2023-07-pooltogether/vault/src/Vault.sol:

  function _mint(address _receiver, uint256 _shares) internal virtual override {

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().










# VULN 3 

## [LOW] Open TODOs
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 378 at 2023-07-pooltogether/twab-controller/src/libraries/TwabLib.sol:

    // TODO: Could skip this check for period 0 if we're sure that the PERIOD_OFFSET is in the past.

------------------------------------------------------------------------ 

### Mitigation 

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment.










# VULN 4 

## [LOW] safeApprove of openZeppelin is deprecated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 282 at 2023-07-pooltogether/vault/src/Vault.sol:

    asset_.safeApprove(address(yieldVault_), type(uint256).max);


Found in line 674 at 2023-07-pooltogether/vault/src/Vault.sol:

      _asset.safeApprove(_previousLiquidationPair, 0);


Found in line 677 at 2023-07-pooltogether/vault/src/Vault.sol:

    _asset.safeApprove(address(liquidationPair_), type(uint256).max);

------------------------------------------------------------------------ 

### Mitigation 

Deprecated, its usage is discouraged. https://docs.openzeppelin.com/contracts/3.x/api/token/erc20. Whenever possible, use safeIncreaseAllowance and safeDecreaseAllowance instead. https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeIncreaseAllowance-contract-IERC20-address-uint256- & https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeDecreaseAllowance-contract-IERC20-address-uint256-










# VULN 5 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 18 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  PrizePool public immutable prizePool;


Found in line 21 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  UD2x18 public immutable maxFeePortionOfPrize;


Found in line 24 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  SD59x18 public immutable decayConstant;


Found in line 27 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  uint256 public immutable minimumFee;


Found in line 29 at 2023-07-pooltogether/claimer/src/Claimer.sol:

  uint256 public immutable timeToReachMaxFee;


Found in line 203 at 2023-07-pooltogether/vault/src/Vault.sol:

  TwabController private immutable _twabController;


Found in line 206 at 2023-07-pooltogether/vault/src/Vault.sol:

  IERC4626 private immutable _yieldVault;


Found in line 209 at 2023-07-pooltogether/vault/src/Vault.sol:

  PrizePool private immutable _prizePool;


Found in line 27 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  uint32 public immutable PERIOD_LENGTH;


Found in line 31 at 2023-07-pooltogether/twab-controller/src/TwabController.sol:

  uint32 public immutable PERIOD_OFFSET;


Found in line 213 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  SD1x18 public immutable smoothing;


Found in line 216 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  IERC20 public immutable prizeToken;


Found in line 219 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  TwabController public immutable twabController;


Found in line 225 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  uint32 public immutable drawPeriodSeconds;


Found in line 228 at 2023-07-pooltogether/prize-poool/src/PrizePool.sol:

  UD2x18 public immutable claimExpansionThreshold;


Found in line 48 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_2_TIERS;


Found in line 49 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_3_TIERS;


Found in line 50 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_4_TIERS;


Found in line 51 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_5_TIERS;


Found in line 52 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_6_TIERS;


Found in line 53 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_7_TIERS;


Found in line 54 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_8_TIERS;


Found in line 55 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_9_TIERS;


Found in line 56 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_10_TIERS;


Found in line 57 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_11_TIERS;


Found in line 58 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_12_TIERS;


Found in line 59 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_13_TIERS;


Found in line 60 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  UD60x18 internal immutable CANARY_PRIZE_COUNT_FOR_14_TIERS;


Found in line 208 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 public immutable tierShares;


Found in line 211 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 public immutable canaryShares;


Found in line 214 at 2023-07-pooltogether/prize-poool/src/abstract/TieredLiquidityDistributor.sol:

  uint8 public immutable reserveShares;

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.
