
**Any comments for the judge to contextualize your findings**

Previously being a software developer by trade, I'm not deeply experienced in the open-ended style of writing needed for these analyses,
which resulted in it being rather time-consuming.
There is no strong guidance on what is wanted in this analysis write up either, unfortunately leading to many hours of procrastination
(which I've out of the hour auditing) in deciding the level of detail and volume of content.
Hopefully the analysis is inline with expectations you have.

Time ran short on this one, many of the areas I'd flagged for further investigation/testing, sadly I simply didn't get to look into seriously.


**Approach taken in evaluating the codebase**

Started off aiming to gain a high level understanding, by reading the [protocol docs](https://github.com/code-423n4/2023-07-pooltogether) 
and then watching the [Loom video](https://www.loom.com/share/a884d5c3ac4e408f8541af151989c5c9?sid=6c36560f-5323-46c7-9af0-83b3057951a6),
which did an excellent job explaining parts not entirely covered in the focs, such as the tier resizing and prize eligibility calculations.

Before cloning the repo, I checked the bot report for an overview of the potential avenue for later investigation and 
familiarize myself with the already public known issues in addition to catching up on the pooltogether-jul07 Discord channel.

Cloned the repo, then proceeded to step through the files line by line, verifying the NatSpec matched the logic and marking with 
inline comments open questions and LOW and NC issues.



**Architecture recommendations**

The architecture is sensible, with stateless functions being in libraries, stateful contracts have logically grouped behaviours and state.
Coupling exists in some cases, but practically speaking, due to contract size limitation (i.e. Spurious Dragon) and actually getting 
the contract deployed on mainnet results means division must occur somewhere.

A possible addition to the architecture could be a generic mechanism to sweep accidentally received tokens from each of the contracts, 
but that is very much being kind to eco-system users who make mistakes.



**Codebase quality analysis**

The codebase is generally at a very high standard.

There's plenty of decent quality NatSpec comments, and when present they usually meaningfully describes the purpose 
or ranges of the function, parameter or return value rather than merely re-stating the name in a sentence 
adding no additional depth of meaning.

By using libraries to move the stateless (functional) behaviour out of the stateful Claimer, TWABController and PrizePool contracts, 
they each are effectively serving a single purpose, a single are of responsibility. THe result is easier to understand code, 
where usually mistakes are more easily identified.

Struts are all packed efficiently, with a number of the fields trimmed to their slimmest to minimize storage, particularly relevant for 
rhe TWABController, whose storage increases linearly with vault and users.

Test coverage is effectively total (over the audited Solidity files), with the only exception being branch coverage in a few of the files. 
The tests themselves are structured adhering to the Foundry best practices (i.e. including asserting state transitions) and also include
fuzzing tests.

Although beyond of the scope of the audit, it is great to see in the `package.json` commands and dependencies supporting 
automatic code formatting and Solidity static analysis on git events (with husky and lintstaged).

The only area of friction in reviewing the code was the frequent wrapping and unwrapping between Solidity primitives and decimal representations, 
they may be an argument for using conversions only at the boundary functions (external and public), while keeping internal to use the decimal representations, 
as it should reduce the volume of wrapping ...but only a minor suggestion though (e.g. smoothing in PrizePool).



**Centralization risks**

While the singleton contracts of `PrizePool` and `TWABController` are points of centralization for data storage and calculations in the system,
the risk is mitigated by having them as immutable, and without any ownership powers (i.e. by either contract or EOA) meaning they are not able to pause, nor upgraded. However `PrizePool` does have a `DrawManager` that does have exclusive access to certain function (closing draws and removing reserve), 
with the potential impact of a misbehaving `DrawManager` ultimately being the loss of the reserve portion or no draws occurring (locking up of deposited prize tokens).

A single point to create and track those vaults, `VaultFactory` is a similar central point, but being immutable, without any ownership controls,
it poses no centralisation risks.

The section of the audited code to be most wary of for centralisation risk are the vault instances, as the `Vault` does use the ownership model, 
granting the owner full control over vault settings of settings the yield fees, who receives the yield fees, liquidation pair and claimer. 
There is no risks to the vault depositors shares from the ownership powers, nor the underlying assets assuming a sound yield vault was chosen.



**Mechanism review**

Based on the design resources and code provided, this was my take-away on how the mechanisms involved tie together.

There is a prize token, the abstracted method of account for all `Vault` shares held by users that determines
the odds of winning a prize during a draw, and also is the reward unit for a winning payout.

Users interact with a `Vault` to deposit their assets for shares in the `Vault`. 
These shares are a claim on the underlying assets (the type deposited by users) 
and shares in the yield bearing vault that uses the underlying token, held by the `Vault`.  
The `Vault` being a liquidation source, allows a liquidator to interact and perform conversions of prize tokens into 
shares of the `Vault`. Whenever there is an update to the shareholdings of the `Vault`the `TWABController` is informed.

The singleton `TWABController` keeps a duplicate record of `Vault` shareholders, with the addition of a rolling 365 count of
observation entries (address level shareholder change events) and provides the ability of holders to delegate (giving their chance of winning to another address).
All of these data points enable the `TWABController` to answer questions on what proportion an addresses holds of their `Vault` total shares and 
what proportion of their `Vault` is of the total contribution of prize tokens, within a finite scope of history.

The `PrizePool` is primarily concerned with managing draws, including whether a particular address (who holds shares in a participating `Vault`) has won a prize, 
determining prize distribution across draws and adjusting prize tiers for the next draw. 
When a draw is closed, a random number is provided. If any address that holds shares in a `Vault` that registered with the `TWABController` prior to the draw beginning,
has a combination of constant values (tht includes the prize tier and index) that hash to a number below the random number, they win that prize.
Hashing algorithms have the potential for collision, to avoid multiple claims on the same prize, a prize is claimable only once, awarded to the first claimer.
Prizes come from the prize tokens contributed by a `Vault` (liquidation of yield from their yield bearing vault they deposit underlying assets with) 
and may be paid out over a number of draws,  depending on a smoothing function (set on `PrizePool` creation), while the actual amount comes from another share model that
splits the prize tokens for that draw across the tiers (amount for each tier total being equal, but the high level tiers consisting of a greater number of smaller prizes) 
and a portion for the reserve and canary tier. 
Whether the next draw has fewer or more prize tiers depends on the percentage of highest tier prizes and canary tier prizes claim (minimum of three tiers, maximum of fifteen), 
as if too fewer of these lowest value prizes are claim, there must be insufficient economic incentive to claim them (e.g. high network fees) and tiers are reduced to increase
the size of all prizes, inversely if most are claimed then they could be split for a greater number of winners.

A `Vault` only allows holders to claim prizes via a claimer, which can be an EOA or a contract and there is support for batching together claims that would reduce gas fees for multiple claims.

The `Claimer` contract is ones of a convenience, providing a single point of entry for claiming prizes and performing claim fee calculations.

  

**Systemic risks**

The risks beyond the scope of the audited contracts include:

Liquidity risk: every vault relies on being able to exchange between their underlying asset, yield bearing vault asset and prize token asset. 
More specifically there are 
depth of liquidity (amount available to trade, affect slippage), 
market volatility (high volatility can deplete liquidity), 
external factors (rush caused by lack of confidence in a pool asset).

Smart contract risk: as the external dependencies are on other smart contracts there exists risks in their implementations (possible bugs, maybe exploits)

Statistical risk: there is always statistical uncertainty, meaning the results obtained from a statistical analysis may 
differ from the actual results observed.
Given the modelling and monte carlo testing performed by the PooledTogether team, the remaining outstanding factor is variability. 
The inherent variability or randomness itself, where natural variations and random fluctuations can affect the reliability of statistical estimates and predictions.

RNG Oracle risk: besides the centralization risk (single point of censorship, manipulation, or failure), 
associated security vulnerabilities (Oracles being targets for hackers) and 
potential network connectivity problems (Oracle connection to the external source providing the RNG), there is 
the risk of the RNG not being adequately random enough. 

Draw Manager risk: the audited code relies on an honest and reliable Draw Manager. 
As the Draw Manager has the ability to 
withdraw the entire reserve they must be trusted to do so, only when necessary for the good of the Prize Pools.
The trigger to close the draw and starting a new one is only invokable by the Draw Manager, meaning the Prize Pools depend
entirely on the Draw Manager call the appropriate function at the correct time intervals for everything to work as expected.

### Time spent:
32 hours