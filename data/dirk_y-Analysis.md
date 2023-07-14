# Summary
Overall I would say that the PoolTogetherV5 protocol achieves its goal of being autonomous and as decentralised as possible. The core architecture doesn't need any major changes in my opinion, and besides the prize pool lacking some comments around the tier system, there isn't anything basic I'd change.

Despite this, I have managed to find a fair number of significant vulnerabilities/issues that I encourage you to explore. I don't think the issues are a reflection of the quality of the architecture, but rather that the edge cases aren't sufficiently tested yet.

# Codebase quality
Overall I would consider the codebase to be of high quality. However, based on the number of findings I have submitted, I do think the test suite could be improved to look at more edge cases; at the moment the test cases mainly focus on "happy path" execution and don't really look at the extreme scenarios that could occur.

Also, given the codebase consists of a large amount of math, I would have expected some more comments to explain the choices behind some of the math. As a result of the lack of comments it took me longer to fully understand the protocol function compared to other protocols, and I relied heavily on the official documentation to supplement the code. I well commented codebase should make it easy to understand the entire functionality without having to refer to external sources.

Splitting the core maths and accumulator logic into specific libraries significantly improves the readability of the code, as well as making it easy to test each component in isolation.

The tiers could definitely be explained better in the codebase. It is currently confusing when the canary tier is included and when it is excluded, and having this special case tier does make the logic more confusing to follow.

# Architecture review
Splitting the core components of PoolTogetherV5 into completely independent projects is a smart architecture move in my opinion. It keeps the sections of the protocol that are logically unrelated separate and easier to reason.

The claimer contract is fairly simple by design and achieves its goal of incentivising prize claims without the need of an admin.

The vault logic is significantly more complicated given it utilises the ERC4626 standard, and uses the TWAB controller to keep track of balances. Conforming to the ERC4626 standard is a smart decision given it should increase the ability for the protocol to be interoperable with other protocols going forwards. The TWAB controller balance tracking significantly increases the complexity, but it does help reduce the ability for attackers to abuse the share mechanism whilst still keeping vaults separate from one another.

The TWAB controller is arguably the most important part of the protocol architecture, and given I couldn't find any way for users to manipulate their TWAB, I believe this is very well designed and tested. Keeping vault TWABs independent also makes a lot of sense when it comes to computing prize winners and ensures that only legitimate vaults can be used to significantly change user prize odds. The use of ring buffers both here and in the prize pool are a smart way of capping state usage over time and as far as I could tell are well implemented.

Finally, the prize pool is also well implemented in terms of minimising the number of entry and exit points. My biggest concern with the prize pool were that there was a way to break the balance tracking, however besides this bug, the logic of continually incrementing the distributed balances is a smart way to ensure you're distributing liquidity accurately over time. Particularly given the remainders were also accounted for an included in any calculations. As mentioned above, the complexity of the canary tier does make the prize pool harder to reason about isn't sufficiently commented in my opinion, but I can understand the purpose of the canary tier to solve the autonomy problem.

Overall there are no significant architecture changes I would make, besides including a registry of claimer contracts, which I talk more about in the following section.

# Centralisation risks
As intended, the PoolTogetherV5 contracts have no major centralisation risks. The contracts are all autonomous and adapt based on usage.

However, as mentioned in one of my reports, I believe there should be a registry of suitable claimer contracts that is managed by the PoolTogether team. This introduces a minimal amount of centralisation (which doesn't effect the day-to-day operation and success of the protocol) whilst protection users of vaults from malicious vault admins. The centralisation risk of this additional should be near 0, especially if the registry doesn't overwrite previous implementations. This way, even if the admin were compromised new vaults could still be deployed with previously approved claimer implementations.

# Recommendations
Besides resolving the issues I've submitted in dedicated reports, my main recommendations (that I've touched on above) would be to increase the comments in the prize pool contract regarding the tier system (in particular the canary tier) and to increase the number of edge case tests. Test coverage of 99.9% doesn't necessarily mean that a codebase is sufficiently tested, and this is a good example.

### Time spent:
24 hours