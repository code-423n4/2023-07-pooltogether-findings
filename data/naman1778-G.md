## [G-01] Using *calldata* instead of *memory* for read-only arguments in *external* functions saves gas

When a function with a *memory* array is called externally, the *abi.decode()* step has to use a for-loop to copy each index of the *calldata* to the *memory* index. Each iteration of this for-loop costs at least 60 gas (i.e. *60 * <mem_array>.length*). Using *calldata* directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having *memory* arguments, it’s still valid for implementation contracs to use *calldata* arguments instead.

If the array is passed to an *internal* function which passes the array to another internal function where the array is modified and therefore *memory* is used in the *external* call, it’s still more gass-efficient to use *calldata* when the *external* function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is *public* but can be marked as *external* since it’s not called by the contract, and cases where a constructor is involved

There are 2 instances of this issue in 2 files:

    File: vault/src/Vault.sol	

    653: function setHooks(VaultHooks memory hooks) external {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol

    File: vault/src/VaultFactory.sol	

    55: function deployVault(
    56:   IERC20 _asset,
    57:   string memory _name,
    58:   string memory _symbol,
    59:   TwabController _twabController,
    60:   IERC4626 _yieldVault,
    61:   PrizePool _prizePool,
    62:   address _claimer,
    63:   address _yieldFeeRecipient,
    64:   uint256 _yieldFeePercentage,
    65:   address _owner
    66: ) external returns (address) {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/VaultFactory.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 100747                                    | 535             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 790             | 790 | 790    | 790 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 66917                                     | 366             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 556             | 556 | 556    | 556 | 1       |


## [G-02] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

Consider using abi.encodePacked() here:

Use abi.encodePacked() where possible to save gas

There is 1 instance of this issue in 1 file:

    File: prize-pool/src/libraries/TierCalculationLib.sol	

    110: return uint256(keccak256(abi.encode(_user, _tier, _prizeIndex, _winningRandomNumber)));

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-03] Use *assembly* to write *address* storage value

There are 4 instances of this issue 2 files:

    File: prize-pool/src/PrizePool.sol	

    278: drawManager = params.drawManager;

    303: drawManager = _drawManager;

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

    File: vault/src/Vault.sol	

    1210: _claimer = claimer_;

    1230: _yieldFeeRecipient = yieldFeeRecipient_;

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }

    contract Contract0 {

        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }

    }

    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }

    }

#### Gas Report

|  Contract0 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |


|  Contract1 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |

## [G-04] Using XOR (^) and AND (&) bitwise equivalents

On Remix, given only uint256 types, the following are logical equivalents, but don’t cost the same amount of gas:

(a != b || c != d || e != f) costs 571
((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas)
Consider rewriting as following to save gas:

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.
Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.

Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas:

However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values

There are 2 instances of this issue in 1 file:

    File: twab-controller/src/TwabController.sol	

    624: if (_toDelegate == address(0) || _toDelegate == SPONSORSHIP_ADDRESS) {

    635: if (_fromDelegate == address(0) || _fromDelegate == SPONSORSHIP_ADDRESS) {

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/TwabController.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-05] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up.

There are 7 instances of this issue in 4 files:

    File: claimer/src/Claimer.sol	

    67: for (uint i = 0; i < winners.length; i++) {

    144: for (uint i = 0; i < _claimCount; i++) {

https://github.com/GenerationSoftware/pt-v5-claimer/blob/57a381aef690a27c9198f4340747155a71cae753/src/Claimer.sol

    File: prize-pool/src/abstract/TieredLiquidityDistributor.sol	

    361: for (uint8 i = start; i < end; i++) {

    630: for (uint8 i = start; i < end; i++) {

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/abstract/TieredLiquidityDistributor.sol

    File: prize-pool/src/libraries/TierCalculationLib.sol	

    138: for (uint8 i = 0; i < _numberOfTiers; i++) {

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/libraries/TierCalculationLib.sol

    File: vault/src/Vault.sol	

    618: for (uint w = 0; w < _winners.length; w++) {

    620: for (uint p = 0; p < prizeIndicesLength; p++) {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-06] Functions guaranteed to revert when called by normal users can be marked *payable*

If a function modifier such as *onlyOwner* is used, the function will revert if a normal user tries to pay the function. Marking the function as *payable* will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are *CALLVALUE*(2),*DUP1*(3),*ISZERO*(3),*PUSH2*(3),*JUMPI*(10),*PUSH1*(3),*DUP1*(3),*REVERT*(0),*JUMPDEST*(1),*POP*(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

There are 6 instances of this issue in 2 files:

    File: prize-pool/src/PrizePool.sol	

    335: function withdrawReserve(address _to, uint104 _amount) external onlyDrawManager {

    348: function closeDraw(uint256 winningRandomNumber_) external onlyDrawManager returns (uint16) {

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

    File: vault/src/Vault.sol	

    641: function setClaimer(address claimer_) external onlyOwner returns (address) {

    665: function setLiquidationPair(
    666:   LiquidationPair liquidationPair_
    667: ) external onlyOwner returns (address) {

    691: function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {

    704: function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol