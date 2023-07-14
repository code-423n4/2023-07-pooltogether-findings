<h1>Functions guaranteed to revert when called by normal users can be marked payable</h1>

If a function modifier such as onlyDrawManager or onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

  function withdrawReserve(address _to, uint104 _amount) external onlyDrawManager {

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L335C1-L335C84

 function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager returns (uint16) {

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L348C2-L348C95

Vault.sol

 function setClaimer(address claimer_) external onlyOwner returns (address) {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L641C2-L641C79

 function setLiquidationPair(
    LiquidationPair liquidationPair_
  ) external onlyOwner returns (address) {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L665C2-L667C43

 function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L691C2-L691C101

 function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L704C2-L704C99

<h1>Using private rather than public for constants, saves gas</h1>

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table.

 address public constant SPONSORSHIP_ADDRESS = address(1);

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol#L24C2-L25C1

<h1>Use calldata instead of memory for function arguments that do not get mutated</h1>

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. Using calldata drastically saves gas.

function deployVault(
    IERC20 _asset,
    string memory _name,
    string memory _symbol,
    TwabController _twabController,
    IERC4626 _yieldVault,
    PrizePool _prizePool,
    address _claimer,
    address _yieldFeeRecipient,
    uint256 _yieldFeePercentage,
    address _owner
  ) external returns (address) {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol#L55C3-L66C33
