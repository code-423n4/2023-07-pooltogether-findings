# [L-01] Vault.transfer(address to, uint256 amount) treats `amount` as its modulo 2^96, but emits Transfer event with the original `amount` value.

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol#L1154-L1155

### Proof of Concept
Add to /vault/test/unit/Vault/Deposit.t.sol
```
  function testLargeTransfer() external {

    uint256 _amount = 1000e18; 

    vm.startPrank(alice);
    underlyingAsset.mint(alice, _amount);
    underlyingAsset.approve(address(vault), type(uint256).max);
    vault.mint(_amount, alice);

    console.log("Alice's balance before:", vault.balanceOf(alice));
    console.log("Bob's   balance before:", vault.balanceOf(bob));
    console.log();
    console.log("'Transferring' an enormous amount of shares...");
    console.log();

    uint256 enormousNumber = type(uint256).max - type(uint96).max + 1;
    vm.expectEmit();
    emit Transfer(alice, bob, enormousNumber); 
    vault.transfer(bob, enormousNumber); 
    console.log("Alice's balance  after:", vault.balanceOf(alice));
    console.log("Bob's   balance  after:", vault.balanceOf(bob));
  }
```
Output:
```  
  Alice's balance before: 1000000000000000000000
  Bob's   balance before: 0
  
  'Transferring' an enormous amount of shares...
  
  Alice's balance  after: 999999999999999999999
  Bob's   balance  after: 1
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 5.95ms
```
