# gas optimization

# summary
|      |   ISSUE   |  INSTANCE   |
|------|-----------|-------------|
|[G-01]|Functions guaranteed to revert when called by normal users can be marked payable|6|
|[G-02]| Use assembly to write address storage values|1|
|[G-03]|Using bools for storage incurs overhead|2|
|[G-04]|Avoid contract existence checks by using low level calls|1|
|[G-05]|Amounts should be checked for 0 before calling a transfer|10|
|[G-06]|Make 3 event parameters indexed when possible|20|
|[G-07]|Do not calculate constants|116|
|[G-08]| Use hardcode address instead address(this)|18|
|[G-09]|use Mappings Instead of Arrays|1|
|[G-10]|Use Assembly To Check For address(0)|22|
|[G-11]|Use constants instead of type(uintx).max|11|
|[G-12]|Duplicated require()/if() checks should be refactored to a modifier or function|4|
|[G-13]|Not using the named return variables when a function returns, wastes deployment gas|3|
|[G-14]|Avoid emitting storage values|5|
|[G-15]|Using private rather than public for constants, saves gas|1|
|[G-16]| Can Make The Variable Outside The Loop To Save Gas |2|
|[G-17]|State variables only set in the constructor should be declared immutable|1|
|[G-18]|Use assembly to emit events|21|
|[G-19]|2**<n> should be re-written as type(uint<n>).max|7|
|[G-20]|Using XOR (^) and OR (pipe line) bitwise equivalents|2|


## [G-01] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
File: src/PrizePool.sol
335   function withdrawReserve(address _to, uint104 _amount) external onlyDrawManager {

348   function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager returns (uint16) {        
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L335

```solidity
File: /src/Vault.sol
641   function setClaimer(address claimer_) external onlyOwner returns (address) {

667   )onlyOwner returns (address) {

691   function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {

704    function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {                
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L641

## [G-02] Use assembly to write address storage values
By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```

```solidity
File: src/PrizePool.sol
303   drawManager = _drawManager;
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L303



## [G-03] Using bools for storage incurs overhead


```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

    Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from 'false' to 'true', after having been 'true' in the past
```

```solidity
File: /src/VaultFactory.sol
36    mapping(Vault => bool) public deployedVaults;
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L36


```solidity
File: /src/PrizePool.sol
206     mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))
    internal claimedPrizes;
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L206

## [G‑04] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

```solidity
File: /src/Vault.sol
578     uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L578


## [G-05] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
File: src/PrizePool.sol
340    _transfer(_to, _amount);

473     _transfer(_prizeRecipient, amount);

491      _transfer(_to, _amount);

500    prizeToken.safeTransferFrom(msg.sender, address(this), _amount);

832   prizeToken.safeTransfer(_to, _amount);
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L340


```solidity
File: /src/TwabController.sol
458   _transferBalance(msg.sender, address(0), _to, _amount);

470   _transferBalance(msg.sender, _from, address(0), _amount);

482   _transferBalance(msg.sender, _from, _to, _amount);
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L458


```solidity
File: /src/Vault.sol
1027  SafeERC20.safeTransfer(IERC20(asset()), _receiver, _assets);

1155  _twabController.transfer(_from, _to, uint96(_shares));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1027

## [G-03] Make 3 event parameters indexed when possible
 events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows for more efficient searching of events.
Exmple:
```
• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);

• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes indexed value);
```

```solidity
File: /src/PrizePool.sol
158   event DrawClosed(
    uint16 indexed drawId,
    uint256 winningRandomNumber,
    uint8 numTiers,
    uint8 nextNumTiers,
    uint104 reserve,
    UD34x4 prizeTokensPerShare,
    uint64 drawStartedAt
  );

171    event WithdrawReserve(address indexed to, uint256 amount);

176     event IncreaseReserve(address user, uint256 amount);

182    event ContributePrizeTokens(address indexed vault, uint16 indexed drawId, uint256 amount);

188    event WithdrawClaimRewards(address indexed to, uint256 amount, uint256 available);

193    event IncreaseClaimRewards(address indexed to, uint256 amount);
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L158


```solidity
File: /src/abstract/TieredLiquidityDistributor.sol
41   event ReserveConsumed(uint256 amount);
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L41


```solidity
File: /src/TwabController.sol
53    event IncreasedBalance(
    address indexed vault,
    address indexed user,
    uint96 amount,
    uint96 delegateAmount
  );

67     event DecreasedBalance(
    address indexed vault,
    address indexed user,
    uint96 amount,
    uint96 delegateAmount
  );


83    event ObservationRecorded(
    address indexed vault,
    address indexed user,
    uint112 balance,
    uint112 delegateBalance,
    bool isNew,
    ObservationLib.Observation observation
  );


106    event IncreasedTotalSupply(address indexed vault, uint96 amount, uint96 delegateAmount);

114   event DecreasedTotalSupply(address indexed vault, uint96 amount, uint96 delegateAmount);

124   event TotalSupplyObservationRecorded(
    address indexed vault,
    uint112 balance,
    uint112 delegateBalance,
    bool isNew,
    ObservationLib.Observation observation
  );

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L53


```solidity
File: /src/Vault.sol
147   event ClaimerSet(address previousClaimer, address newClaimer);

154   event SetHooks(address account, VaultHooks hooks);


168    event MintYieldFee(address indexed caller, address indexed recipient, uint256 shares);

175     event YieldFeeRecipientSet(address previousYieldFeeRecipient, address newYieldFeeRecipient);

182    event YieldFeePercentageSet(uint256 previousYieldFeePercentage, uint256 newYieldFeePercentage);

191     event Sponsor(address indexed caller, address indexed receiver, uint256 assets, uint256 shares);

198    event RecordedExchangeRate(uint256 exchangeRate);
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L147


## [G-07] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
File: /src/abstract/TieredLiquidityDistributor.sol
84  SD59x18 internal constant TIER_ODDS_0_3 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_3 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_2_3 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_4 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_4 = SD59x18.wrap(19579642462506911);
  SD59x18 internal constant TIER_ODDS_2_4 = SD59x18.wrap(139927275620255366);
  SD59x18 internal constant TIER_ODDS_3_4 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_5 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_5 = SD59x18.wrap(11975133168707466);
  SD59x18 internal constant TIER_ODDS_2_5 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_3_5 = SD59x18.wrap(228784597949733865);
  SD59x18 internal constant TIER_ODDS_4_5 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_6 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_6 = SD59x18.wrap(8915910667410451);
  SD59x18 internal constant TIER_ODDS_2_6 = SD59x18.wrap(29015114005673871);
  SD59x18 internal constant TIER_ODDS_3_6 = SD59x18.wrap(94424100034951094);
  SD59x18 internal constant TIER_ODDS_4_6 = SD59x18.wrap(307285046878222004);
  SD59x18 internal constant TIER_ODDS_5_6 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_7 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_7 = SD59x18.wrap(7324128348251604);
  SD59x18 internal constant TIER_ODDS_2_7 = SD59x18.wrap(19579642462506911);
  SD59x18 internal constant TIER_ODDS_3_7 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_4_7 = SD59x18.wrap(139927275620255366);
  SD59x18 internal constant TIER_ODDS_5_7 = SD59x18.wrap(374068544013333694);
  SD59x18 internal constant TIER_ODDS_6_7 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_8 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_8 = SD59x18.wrap(6364275529026907);
  SD59x18 internal constant TIER_ODDS_2_8 = SD59x18.wrap(14783961098420314);
  SD59x18 internal constant TIER_ODDS_3_8 = SD59x18.wrap(34342558671878193);
  SD59x18 internal constant TIER_ODDS_4_8 = SD59x18.wrap(79776409602255901);
  SD59x18 internal constant TIER_ODDS_5_8 = SD59x18.wrap(185317453770221528);
  SD59x18 internal constant TIER_ODDS_6_8 = SD59x18.wrap(430485137687959592);
  SD59x18 internal constant TIER_ODDS_7_8 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_9 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_9 = SD59x18.wrap(5727877794074876);
  SD59x18 internal constant TIER_ODDS_2_9 = SD59x18.wrap(11975133168707466);
  SD59x18 internal constant TIER_ODDS_3_9 = SD59x18.wrap(25036116265717087);
  SD59x18 internal constant TIER_ODDS_4_9 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_5_9 = SD59x18.wrap(109430951602859902);
  SD59x18 internal constant TIER_ODDS_6_9 = SD59x18.wrap(228784597949733865);
  SD59x18 internal constant TIER_ODDS_7_9 = SD59x18.wrap(478314329651259628);
  SD59x18 internal constant TIER_ODDS_8_9 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_10 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_10 = SD59x18.wrap(5277233889074595);
  SD59x18 internal constant TIER_ODDS_2_10 = SD59x18.wrap(10164957094799045);
  SD59x18 internal constant TIER_ODDS_3_10 = SD59x18.wrap(19579642462506911);
  SD59x18 internal constant TIER_ODDS_4_10 = SD59x18.wrap(37714118749773489);
  SD59x18 internal constant TIER_ODDS_5_10 = SD59x18.wrap(72644572330454226);
  SD59x18 internal constant TIER_ODDS_6_10 = SD59x18.wrap(139927275620255366);
  SD59x18 internal constant TIER_ODDS_7_10 = SD59x18.wrap(269526570731818992);
  SD59x18 internal constant TIER_ODDS_8_10 = SD59x18.wrap(519159484871285957);
  SD59x18 internal constant TIER_ODDS_9_10 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_11 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_11 = SD59x18.wrap(4942383282734483);
  SD59x18 internal constant TIER_ODDS_2_11 = SD59x18.wrap(8915910667410451);
  SD59x18 internal constant TIER_ODDS_3_11 = SD59x18.wrap(16084034459031666);
  SD59x18 internal constant TIER_ODDS_4_11 = SD59x18.wrap(29015114005673871);
  SD59x18 internal constant TIER_ODDS_5_11 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_6_11 = SD59x18.wrap(94424100034951094);
  SD59x18 internal constant TIER_ODDS_7_11 = SD59x18.wrap(170338234127496669);
  SD59x18 internal constant TIER_ODDS_8_11 = SD59x18.wrap(307285046878222004);
  SD59x18 internal constant TIER_ODDS_9_11 = SD59x18.wrap(554332974734700411);
  SD59x18 internal constant TIER_ODDS_10_11 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_12 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_12 = SD59x18.wrap(4684280039134314);
  SD59x18 internal constant TIER_ODDS_2_12 = SD59x18.wrap(8009005012036743);
  SD59x18 internal constant TIER_ODDS_3_12 = SD59x18.wrap(13693494143591795);
  SD59x18 internal constant TIER_ODDS_4_12 = SD59x18.wrap(23412618868232833);
  SD59x18 internal constant TIER_ODDS_5_12 = SD59x18.wrap(40030011078337707);
  SD59x18 internal constant TIER_ODDS_6_12 = SD59x18.wrap(68441800379112721);
  SD59x18 internal constant TIER_ODDS_7_12 = SD59x18.wrap(117019204165776974);
  SD59x18 internal constant TIER_ODDS_8_12 = SD59x18.wrap(200075013628233217);
  SD59x18 internal constant TIER_ODDS_9_12 = SD59x18.wrap(342080698323914461);
  SD59x18 internal constant TIER_ODDS_10_12 = SD59x18.wrap(584876652230121477);
  SD59x18 internal constant TIER_ODDS_11_12 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_13 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_13 = SD59x18.wrap(4479520628784180);
  SD59x18 internal constant TIER_ODDS_2_13 = SD59x18.wrap(7324128348251604);
  SD59x18 internal constant TIER_ODDS_3_13 = SD59x18.wrap(11975133168707466);
  SD59x18 internal constant TIER_ODDS_4_13 = SD59x18.wrap(19579642462506911);
  SD59x18 internal constant TIER_ODDS_5_13 = SD59x18.wrap(32013205494981721);
  SD59x18 internal constant TIER_ODDS_6_13 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_7_13 = SD59x18.wrap(85581121447732876);
  SD59x18 internal constant TIER_ODDS_8_13 = SD59x18.wrap(139927275620255366);
  SD59x18 internal constant TIER_ODDS_9_13 = SD59x18.wrap(228784597949733866);
  SD59x18 internal constant TIER_ODDS_10_13 = SD59x18.wrap(374068544013333694);
  SD59x18 internal constant TIER_ODDS_11_13 = SD59x18.wrap(611611432212751966);
  SD59x18 internal constant TIER_ODDS_12_13 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_14 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_14 = SD59x18.wrap(4313269422986724);
  SD59x18 internal constant TIER_ODDS_2_14 = SD59x18.wrap(6790566987074365);
  SD59x18 internal constant TIER_ODDS_3_14 = SD59x18.wrap(10690683906783196);
  SD59x18 internal constant TIER_ODDS_4_14 = SD59x18.wrap(16830807002169641);
  SD59x18 internal constant TIER_ODDS_5_14 = SD59x18.wrap(26497468900426949);
  SD59x18 internal constant TIER_ODDS_6_14 = SD59x18.wrap(41716113674084931);
  SD59x18 internal constant TIER_ODDS_7_14 = SD59x18.wrap(65675485708038160);
  SD59x18 internal constant TIER_ODDS_8_14 = SD59x18.wrap(103395763485663166);
  SD59x18 internal constant TIER_ODDS_9_14 = SD59x18.wrap(162780431564813557);
  SD59x18 internal constant TIER_ODDS_10_14 = SD59x18.wrap(256272288217119098);
  SD59x18 internal constant TIER_ODDS_11_14 = SD59x18.wrap(403460570024895441);
  SD59x18 internal constant TIER_ODDS_12_14 = SD59x18.wrap(635185461125249183);
  SD59x18 internal constant TIER_ODDS_13_14 = SD59x18.wrap(1000000000000000000);
  SD59x18 internal constant TIER_ODDS_0_15 = SD59x18.wrap(2739726027397260);
  SD59x18 internal constant TIER_ODDS_1_15 = SD59x18.wrap(4175688124417637);
  SD59x18 internal constant TIER_ODDS_2_15 = SD59x18.wrap(6364275529026907);
  SD59x18 internal constant TIER_ODDS_3_15 = SD59x18.wrap(9699958857683993);
  SD59x18 internal constant TIER_ODDS_4_15 = SD59x18.wrap(14783961098420314);
  SD59x18 internal constant TIER_ODDS_5_15 = SD59x18.wrap(22532621938542004);
  SD59x18 internal constant TIER_ODDS_6_15 = SD59x18.wrap(34342558671878193);
  SD59x18 internal constant TIER_ODDS_7_15 = SD59x18.wrap(52342392259021369);
  SD59x18 internal constant TIER_ODDS_8_15 = SD59x18.wrap(79776409602255901);
  SD59x18 internal constant TIER_ODDS_9_15 = SD59x18.wrap(121589313257458259);
  SD59x18 internal constant TIER_ODDS_10_15 = SD59x18.wrap(185317453770221528);
  SD59x18 internal constant TIER_ODDS_11_15 = SD59x18.wrap(282447180198804430);
  SD59x18 internal constant TIER_ODDS_12_15 = SD59x18.wrap(430485137687959592);
  SD59x18 internal constant TIER_ODDS_13_15 = SD59x18.wrap(656113662171395111);
  SD59x18 internal constant TIER_ODDS_14_15 = SD59x18.wrap(1000000000000000000);
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L84



## [G-08] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)


```solidity
File: src/Vault.sol
336       return _twabController.balanceOf(address(this), _account);

335      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

468      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

502      _permit(IERC20Permit(asset()), msg.sender, address(this), _assets, _deadline, _v, _r, _s);

562      if (_tokenOut != address(this))

563      revert LiquidationTokenOutNotVaultShare(_tokenOut, address(this));

570      _prizePool.contributePrizeTokens(address(this), _amountIn);

578      uint256 _vaultAssets = IERC20(asset()).balanceOf(address(this));

581     _yieldVault.deposit(_vaultAssets, address(this));

801      return _yieldVault.maxWithdraw(address(this)) + super.totalAssets();

809      return _twabController.totalSupply(address(this));

830      if (_token != address(this)) revert LiquidationTokenOutNotVaultShare(_token, address(this));

932      uint256 _vaultAssets = _asset.balanceOf(address(this));

954      address(this),

959      _yieldVault.deposit(_assets, address(this));

986      _twabController.delegateOf(address(this), _receiver) != _twabController.SPONSORSHIP_ADDRESS()

1026     _yieldVault.withdraw(_assets, address(this), address(this));

1176      uint256 _withdrawableAssets = _yieldVault.maxWithdraw(address(this));
```

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L336


## [G-09]use Mappings Instead of Arrays
Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.


```solidity
File: /src/VaultFactory.sol
30   Vault[] public allVaults;
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L30


## [G-10] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary

```solidity
File: src/PrizePool.sol
279   if (params.drawManager != address(0)) {

300   if (drawManager != address(0)) {    
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L279



```solidity
File:  src/TwabController.sol
523   if (_from != address(0)) {

538    _to == address(0) ||

544     _to == address(0) ? _amount : 0,

545      (_to == address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) ||

554     if (_to != address(0)) {

569    _from == address(0) ||

574    _from == address(0) ? _amount : 0,

575    (_from == address(0) && _toDelegate != SPONSORSHIP_ADDRESS) ||

593    if (_user != address(0)) {

597    if (_userDelegate == address(0)) {

619    if (_fromDelegate != address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) {

624   if (_toDelegate == address(0) || _toDelegate == SPONSORSHIP_ADDRESS) {

630    if (_toDelegate != address(0) && _toDelegate != SPONSORSHIP_ADDRESS) {

635    if (_fromDelegate == address(0) || _fromDelegate == SPONSORSHIP_ADDRESS) {                    
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L523


```solidity
File: /src/Vault.sol
566     if (address(twabController_) == address(0)) revert TwabControllerZeroAddress();

567     if (address(yieldVault_) == address(0)) revert YieldVaultZeroAddress();

568     if (address(prizePool_) == address(0)) revert PrizePoolZeroAddress();

569     if (address(owner_) == address(0)) revert OwnerZeroAddress();

668     if (address(liquidationPair_) == address(0)) revert LPZeroAddress();

673     if (_previousLiquidationPair != address(0)) {    
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L566

## [G-11] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```solidity
File:  src/libraries/LinearVRGDALib.sol
56     targetTimeUnwrapped > 0 && decayConstantUnwrapped > type(int256).max / targetTimeUnwrapped

58     return type(uint256).max;

70     return type(uint256).max;

82     if (targetPriceInt > type(int256).max / expResult)

83     return type(uint256).max;

89     if (targetPriceInt > type(int256).max / extraPrecisionExpResult)

90     return type(uint256).max;
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/libraries/LinearVRGDALib.sol#L56

```solidity
File:   src/Vault.sol
282     asset_.safeApprove(address(yieldVault_), type(uint256).max);

376     return _isVaultCollateralized() ? type(uint96).max : 0;

384     return _isVaultCollateralized() ? type(uint96).max : 0;

677     _asset.safeApprove(address(liquidationPair_), type(uint256).max);

```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L282


## [G-12] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```

```solidity
File:  src/TwabController.sol
691   if (_isObservationRecorded) {

734   if (_isObservationRecorded) {

776   if (_isObservationRecorded) {

810   if (_isObservationRecorded) {            
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L691


## [G-13]Not using the named return variables when a function returns, wastes deployment gas
When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost

```solidity
File: /src/Claimer.sol
82    return feePerClaim * claimCount;
```
https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol#L82


```solidity
File: /src/libraries/TwabLib.sol
423   return _getTimestampPeriod(PERIOD_LENGTH, PERIOD_OFFSET, _timestamp);

441   return 0;
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/TwabLib.sol#L423

## [G-14] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-05-avoid-emitting-storage-values)
```solidity
File:  src/PrizePool.sol
375  emit DrawClosed(
      lastClosedDrawId,
      winningRandomNumber_,
      _numTiers,
      _nextNumberOfTiers,
      _reserve,
      prizeTokenPerShare,
      _lastClosedDrawStartedAt
    );

```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L375

```solidity
File:   /src/TwabController.sol

692    emit ObservationRecorded(
        _vault,
        _user,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

735    emit ObservationRecorded(
        _vault,
        _user,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

777   emit TotalSupplyObservationRecorded(
        _vault,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

811   emit TotalSupplyObservationRecorded(
        _vault,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L692




## [G‑15] Using private rather than public for constants, saves gas
When you declare a constant variable as public, Solidity generates a getter function that allows anyone to read the value of the constant. This getter function can consume gas, especially if the constant is read frequently or the contract is called by multiple users.

By using private instead of public for constants, you can prevent the generation of the getter function and reduce the overall gas cost of your contract.

```solidity
File:  src/TwabController.sol
24     address public constant SPONSORSHIP_ADDRESS = address(1);
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L24



## [G-16] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```


```solidity
File: /src/abstract/TieredLiquidityDistributor.sol
632    uint8 shares = _computeShares(i, _numberOfTiers);
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L632

```solidity
File: /src/Vault.sol
619    uint prizeIndicesLength = _prizeIndices[w].length;
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L619

## [G‑17] State variables only set in the constructor should be declared immutable
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).


```solidity
File: /src/Vault.sol
279   _assetUnit = 10 ** super.decimals();             
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L279

## [G‑18] Use assembly to emit events
We can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic emit event for eventSentAmountExample:
```
 // uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```
   assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }
```   


```solidity
File: /src/PrizePool.sol
280  emit DrawManagerSet(params.drawManager);


328   emit ContributePrizeTokens(_prizeVault, lastClosedDrawId + 1, _amount);    

341   emit WithdrawReserve(_to, _amount);    

375    emit DrawClosed(
      lastClosedDrawId,
      winningRandomNumber_,
      _numTiers,
      _nextNumberOfTiers,
      _reserve,
      prizeTokenPerShare,
      _lastClosedDrawStartedAt
    );

454   emit IncreaseClaimRewards(_feeRecipient, _fee);    

461   emit ClaimedPrize(
      msg.sender,
      _winner,
      _prizeRecipient,
      lastClosedDrawId,
      _tier,
      _prizeIndex,
      uint152(amount),
      _fee,
      _feeRecipient
    );

501   emit IncreaseReserve(msg.sender, _amount);
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L280


```solidity
File: /src/abstract/TieredLiquidityDistributor.sol
539   emit ReserveConsumed(excess);
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol#L539

```solidity
File: /src/TwabController.sol
688   emit IncreasedBalance(_vault, _user, _amount, _delegateAmount);

731    emit DecreasedBalance(_vault, _user, _amount, _delegateAmount);

692   emit ObservationRecorded(
        _vault,
        _user,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );
773    emit DecreasedTotalSupply(_vault, _amount, _delegateAmount);

777    emit TotalSupplyObservationRecorded(
        _vault,
        _account.details.balance,
        _account.details.delegateBalance,
        _isNewObservation,
        _observation
      );

807  emit IncreasedTotalSupply(_vault, _amount, _delegateAmount);       
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L688


```solidity
File: /src/Vault.sol
401  emit MintYieldFee(msg.sender, _recipient, _shares);

284     emit NewVault(
      asset_,
      name_,
      symbol_,
      twabController_,
      yieldVault_,
      prizePool_,
      claimer_,
      yieldFeeRecipient_,
      yieldFeePercentage_,
      owner_
    );

645   emit ClaimerSet(_previousClaimer, claimer_);

681   emit LiquidationPairSet(liquidationPair_);

695    emit YieldFeePercentageSet(_previousYieldFeePercentage, yieldFeePercentage_);

708   emit YieldFeeRecipientSet(_previousYieldFeeRecipient, yieldFeeRecipient_);

991   emit Sponsor(msg.sender, _receiver, _assets, _shares);

1111  emit RecordedExchangeRate(_lastRecordedExchangeRate);
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L401


```solidity
File: /src/VaultFactory.sol
83    emit NewFactoryVault(_vault, VaultFactory(address(this)));
```
https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L83

## [G-19] 2**<n> should be re-written as type(uint<n>).max
the expression 2**<n> is equivalent to calculating the maximum value of an unsigned integer of size n bits. Instead of using the 2**<n> expression, it is more efficient to use the type(uint<n>).max notation, which calculates the maximum value of an unsigned integer of size n bits at compile time. This can save gas during contract execution

```solidity
File:   src/libraries/TierCalculationLib.sol
40     uint256 _numberOfPrizes = 4 ** _tier;
```
https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol#L40

```solidity
File:    src/libraries/OverflowSafeComparatorLib.sol
19        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;

20        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

35        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;

36        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

52        uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;

53        uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol#L19


## [G-20] Using XOR (^) and OR (|) bitwise equivalents
This is a good optimization suggestion! Using XOR (^) and OR (|) bitwise operations can save gas compared to using || logical OR.
```solidity
File:   src/TwabController.sol
545     (_to == address(0) && _fromDelegate != SPONSORSHIP_ADDRESS) ||   

575     (_from == address(0) && _toDelegate != SPONSORSHIP_ADDRESS) ||
```
https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L545
