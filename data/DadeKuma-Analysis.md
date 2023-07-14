# PoolTogether Analysis

## Summary

---
|Id|Title|
|:--:|:-------|
|[01](#01-high-level-architecture)| High-level architecture |
|[02](#02-analysis-of-the-codebase)| Analysis of the codebase |
|[03](#03-architecture-feedback)| Architecture feedback |
|[04](#04-centralization-risks)| Centralization risks |
|[05](#05-systemic-risks)| Systemic risks |

---

## 01 High-level architecture

![Pool Together architecture](https://i.imgur.com/3bqD34u.png)

## 02 Analysis of the codebase

- The separation of concerns between Vaults, Prize Pool, TWAB, and other components allows for modularity and an easier code review.

- The codebase quality is **extremely high**, with extensive comments that explain every step in detail.

- Test quality is **extremely high**, with high coverage and usage of advanced testing techniques (e.g. fuzzing).


## 03 Architecture feedback

- The architecture of PoolTogether V5 is well thought out and designed for flexibility. It allows for the inclusion of any ERC20-compliant asset and the customization of any ERC-4626 Vault. 

- Some of the new features include a novel hand-off approach where users don't need to manually claim their prizes as there are incentives for third parties to do so.

- The use of a weighted average to smooth contributions over time provides a fair distribution of prizes and encourages participation from Vaults with a periodic yield generation.

- The architecture of the TWAB Controller appears well-designed to fulfill its purpose of accurately tracking and calculating time-weighted average balances. 

- The utilization of Observations to record relevant data points and the approach of overwriting Observations within periods for gas efficiency are clever design choices. This is ensured by aligning periods with draws, as the TWAB Controller ensures accurate tracking of balances during draw periods.


## 04 Centralization risks

- The PoolTogether V5 Protocol aims to be autonomous and permissionless, reducing centralization risks.

- It's important to evaluate the dependencies and external services used, such as the RNG service to extract the winners, to ensure they are decentralized and reliable.

## 05 Systemic risks

- Claimers should always have a high incentive to claim winner prizes as the new V5 version is marketed as a hand-off approach. Failure to do so might result in an expired prize, as (non-claimer) users don't expect to claim manually.

- The random number generation process for Draw completion is critical, and reliance on external RNG services introduces risks if those services are compromised or manipulated.

- Periods are not considered finalized until the full period length has passed. Balances and average balances within a period that has not ended may still be subject to changes due to Observations being overwritten. This can lead to inaccurate balances and average balances for that period.

- Historic balances and average balances are only guaranteed to be accurate within specific conditions. Historic balances are accurate for timestamps falling between the newest Observation recorded before the end of a finalized period and the end time of that period. Average balances across time ranges are accurate if temporary Observations can be accurately recreated on both ends of the range and if time-weighted balances have been captured throughout the range.

- Loss of historic information: due to the compression of information and overwriting of Observations, specific timestamps at which balances were held throughout a period may be lost, so external actors should be aware of it.

### Time spent:
12 hours