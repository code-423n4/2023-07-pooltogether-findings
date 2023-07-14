# PoolTogether Analysis by CatellaTech

## Codebase Analysis (What's unique? What utilizes existing patterns?)
The PoolTogether V5 system introduces several unique aspects to the protocol, including full autonomy, automation, and permissionlessness. These characteristics eliminate the need for administrative controls, allow for the automatic adaptation of prize sizes and counts, and enable the addition of new assets or yield sources through new vaults. The implementation of existing patterns such as delegation and token distribution using shares is also observed in the codebase.

## Architecture Feedback
The provided information gives a general overview of the PoolTogether V5 architecture. The protocol consists of several key contracts, including the vault for deposit and withdrawal operations, the prize pool for prize distribution, the TWAP controller for measuring average user balances during draws, and the claimer for managing prize claims. This architecture enables users to interact with the system and participate in daily draws while maintaining security and fairness.

## Centralization Risks
We do not believe there are centralization issues with the protocol, except for the access control in the PrizePool contract, which is specified for the draw manager in the documentation.

## Systemic Risks
The risk we noticed relates to the PRBMath library, which is inherited in important contracts. Since it is an external library, it is essential to stay attentive to future changes that may be introduced to avoid compromising the PoolTogether protocol.

## Other Recommendations
It is recommended to maintain consistency in the documentation and stay vigilant about all external dependencies to stay informed about updates and bug fixes.

## New Insights and Learnings from the Audit
We found the innovations introduced by the PoolTogether team and their contributions to the web3 ecosystem to be impressive. The primary methodology used for the audit was manual auditing, thoroughly examining the entire in-scope code from various adversarial perspectives. Additionally, any dependencies on external code were reviewed.

## Time Spent
The time spent analyzing and gaining a deep understanding of the project's implementation by the team was 72 hours.

### Time spent:
72 hours