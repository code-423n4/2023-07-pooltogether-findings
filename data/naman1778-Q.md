## [N-01] Use delete to clear variables instead of zero assignment

You can use the delete keyword instead of setting the variable as zero.

There are 3 instances of this issue in 1 file:

    File: prize-pool/src/PrizePool.sol	

    369: claimCount = 0;

    370: canaryClaimCount = 0;

    371: largestTierClaimed = 0;

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

## [N-02] Use scientific notation (e.g. 1e32) rather than exponentiation (e.g. 10**32)

There are 6 instances of this issue in 1 file:

    File: twab-controller/src/libraries/OverflowSafeComparatorLib.sol	

    19: uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;

    20: uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

    35: uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;

    36: uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

    52: uint256 aAdjusted = _a > _timestamp ? _a : _a + 2 ** 32;
    
    53: uint256 bAdjusted = _b > _timestamp ? _b : _b + 2 ** 32;

https://github.com/GenerationSoftware/pt-v5-twab-controller/blob/0145eeac23301ee5338c659422dd6d69234f5d50/src/libraries/OverflowSafeComparatorLib.sol

## [N-03] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

There is 1 instance of this issue in 1 file:

    File: prize-pool/src/PrizePool.sol	

    206: mapping(address => mapping(address => mapping(uint16 => mapping(uint8 => mapping(uint32 => bool)))))

https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol

## [N-04] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

There are 2 instances of this issue in 1 file:

    File: vault/src/Vault.sol	

    558: if (msg.sender != address(_liquidationPair))
    559:   revert LiquidationCallerNotLP(msg.sender, address(_liquidationPair));

    614: if (msg.sender != _claimer) revert CallerNotClaimer(msg.sender, _claimer);

https://github.com/GenerationSoftware/pt-v5-vault/blob/b1deb5d494c25f885c34c83f014c8a855c5e2749/src/Vault.sol

## [N-05] Contract Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

-- splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
-- clearly documenting the functions and implementations the owner can change,
-- documenting the risks associated with privileged users and single points of failure, and
-- ensuring that users are aware of all the risks associated with the system.
