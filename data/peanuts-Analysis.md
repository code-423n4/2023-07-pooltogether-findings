### Context

Focused solely on the Vault, TWAB controller and TWAB library.

### Approach taken in evaluating the codebase

- Read through the documentation and watch the video walkthrough of the code
- Examine the interaction between the Vault, TWAB Library 
- Focus on the Vault and see if there is any ERC4626 complications
- Try to break the Vault 

### Architecture Recommendations

Check the resultant amount of shares/assets when depositing / withdrawing from the vault. Make sure that the shares and assets received after converting is not zero. If it is zero, make sure to revert. For example: 

```
  function deposit(uint256 _assets, address _receiver) public virtual override returns (uint256) {
    if (_assets > maxDeposit(_receiver))
      revert DepositMoreThanMax(_receiver, _assets, maxDeposit(_receiver));


    uint256 _shares = _convertToShares(_assets, Math.Rounding.Down);
+   require(_shares != 0, "Zero Shares");
    _deposit(msg.sender, _receiver, _assets, _shares);

Fee-on-transfer tokens, rebasing tokens and low-decimal tokens may affect the vault and accounting so maybe can check the decimal places of the asset during constructions, and set a whitelist of the assets used.

Also, safeApprove is used, and I believe USDT doesn't work through safeApprove but have to set the value to zero first.  
```

### Codebase quality analysis

Codebase is pretty complicated, so it'll be good if there is a step-by-step walkthrough for all the points and examples for the Math used. 

Good amount of NATSPECs, the external calls is a little hard to grasp, especially jumping from Vault to TwAB controller to TWAB library and back. I see that alot of thought and effort is put into this new version, with the extensive test coverage as well.


### Mechanism Review

The two-vault scenario is pretty novel and interesting, but extremely confusing to me. I believe that the Vault doesn't actually mint shares, but call the TWAB controller to store the amount of shares in the vault. The owner still receives shares from the yield vault and virtual shares from the TWAB controller. I would like to see how it plays out, especially since I am also a user of PoolTogether.

### Centralization Risks

The Vault is permissionless and the Yield Vault is also not checked. This invites alot of complication and trust issues with the user and the protocol because users has to trust that the Vault is not malicious in order to deposit and interact with the protocol. Would be good if there are some Protocol-initiated Vaults to increase user's trust.

### Systemic Risks

I don't see a failsafe measure to restore funds in case funds get stuck in the protocol. Maybe can introduce the emergency withdraw function through a multisig in case any part of the protocol gets compromised, especially so when the protocol is going permissionless



### Time spent:
10 hours