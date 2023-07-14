# 🛠️ Analysis - PoolTogether Project 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Architectural | Architecture feedback |
|e) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|f) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|g) |Systemic risks | Potential systemic risks in the project |
|h) |Competition analysis| What are similar projects? |
|i) |Security Approach of the Project | Audit approach of the Project |
|j) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|k) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|l) |Project team | Details about the project team and approach |
|m) |Other recommendations | What is unique? How are the existing patterns used? |
|n) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-07-pooltogether#scope

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-07-pooltogether#setup)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [PoolTogether V5 Protocol Design](https://dev.pooltogether.com/protocol/next/design/) |
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-07-pooltogether#tests)|The Slither output did not indicate a direct security risk to us|
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-07-pooltogether#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-07-pooltogether#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-07-pooltogether#overview)|The specific focus areas |

## b) Analysis of the code base
Intense mathematical formulas and structures were used in the project, I used the following document to control them.

https://paragraph.xyz/@spearbit/numerical-analysis


#### TieredLiquidityDistributor.sol:
The Tiered Liquidity Distributor contract works by dividing the liquidity pool into different tiers, each with its own minimum deposit requirement. The higher the minimum deposit requirement, the higher the share of the liquidity pool that the tier receives. This ensures that all liquidity providers, regardless of their size, have a fair chance to earn rewards from the prize pool.

This contract divides the liquidity pool into different tiers, each with its own minimum deposit requirement.
Ensures that all liquidity providers, regardless of their size, have a fair chance to earn rewards from the prize pool.
Includes a number of features to prevent front-running and other malicious attacks.
It is a complex and sophisticated piece of code, but it is essential for the smooth operation of the PoolTogether V5 prize pool.

#### DrawAccumulatorLib.sol:
The Draw Accumulator Library is used in the PoolTogether V5 prize pool, which is a decentralized finance (DeFi) application that allows users to earn interest on their deposits by entering a lottery. The library provides a number of functions for managing the draw accumulator, including:

Incrementing the number of draws that have been made.
Associating a prize with a particular draw.
Calculating the total amount of prize money that has been awarded.
Verifying that the draw accumulator is in a valid state

####  PrizePool.sol:
Here are some of the key features of the Prize Pool contract:
Manages the liquidity pool.
Manages the draw accumulator.
It keeps track of the prize winners.
It distributes the prize money to the winners.




#### TwabController.sol:
The TWAB Controller contract works by keeping track of the total amount of time that each liquidity provider has deposited their funds. The contract then uses this information to calculate the TWAB reward for each liquidity provider. The TWAB rewards are then distributed to the liquidity providers at the end of each prize pool period.



#### Vault.sol:
ERC-4626 compatible vault that uses a Twab Controller to track balances. The contract is bound to an ERC-4626 yield source and exposes yield to a liquidator.
Uses Openzeppelin's ERC4626 vault contract (Latest version is used) (The latest version takes precautions against the famous inflation attack in erc4626)
https://blog.openzeppelin.com/introducing-openzeppelin-contracts-v4.9

## c) Test analysis

What did the project do differently? ;
The project, while designing the tests, modeled it with the Threat Model, wrote the tests and documented it and presented it to the auditors, which increases the auditability and gives us the project team's view of the threat models specifically for the project.

What could they have done better?;
Other than Code4rena, I recommend having it audited by a reputable audit firm.

## d) Architectural 

First of all, to understand the LinearVRGDALIb design of his project, it is necessary to know 4 concepts;


<img width="390" alt="image" src="https://github.com/code-423n4/2023-07-pooltogether/assets/104318932/b839f24c-6bc5-4cd3-a66e-db7ace2f38ed">

- Dutch auction   
https://en.wikipedia.org/wiki/Dutch_auction
For example, a business may auction a used company car with a starting bid of €15,000. If no one accepts the first offer, the seller will reduce the price in increments of 1,000 € in succession. When the price reaches €10,000, a particular bidder who thinks the price is acceptable and someone else may soon bid, quickly accepts the offer and pays €10,000 for the car.
Dutch auctions are a competitive alternative to a traditional auction, where customers bid at increasing value until they are not willing to bid higher.

- Revers Dutch auction
https://ensolva.com/dutch-reverse-auction/
Procurement recognized its benefits and adapted it to their needs. The Dutch reverse auction is the opposite image of the Dutch forward auction. The buyer defines an artificially low starting price at a point where the bid is believed to be zero. Periodically, the price rises and continues to rise until the supplier is ready to bid. The auction stops when the first bid is placed.
How does the Dutch reverse auction work?
Unlike the British auction, where bidders can enter their bids several times during the auction period, the Dutch auction is not allowed to enter any prices into the system.
Compared to the Japanese auction, bidders do not actively participate in the procedure. In this case, they observe the price increase not accepting the offered price until they are ready to offer their product or service at the offered price.
Only one bid is considered: The auction ends as soon as the first bidder accepts the bid price.
It is recommended that the buyer set the maximum price that he is willing to pay for the purchase or service item.

- GDA (Gradual Dutch Auctions)
https://www.paradigm.xyz/2022/04/gda
https://github.com/FrankieIsLost/gradual-dutch-auction
A mechanism for the efficient sale of non-liquid assets
Imagine that Alice wants to sell a 10,000 NFT collection. He's not sure what a fair price would be for his artwork, so he doesn't want to sell them at a fixed price.
Instead, he can choose to sell them all at a single Dutch auction - starting with a high asking price and gradually lowering it until all NFTs are sold. However, such an auction may be insufficient: there may not be enough interest from buyers to buy all the pieces at once.
Instead, Alice can auction one NFT at a time. For example, he might launch a new Dutch auction every minute for a new piece in his collection. This will allow more time for the market to find a fair price for their art.
Discrete GDAs are an extension of this idea, but support gas-efficient bulk purchases of multiple sub-auctions thanks to their mathematical properties


- VRGDA (Variable Rate GDAs)
https://www.paradigm.xyz/2022/08/vrgda

Variable Rate GDAs (VRGDAs), designed for Art Gobblers and used at 0xMonaco, allow you to sell tokens close to a specific schedule over time, raising prices when sales are ahead of schedule and lowering prices when sales are behind schedule


<img width="909" alt="image" src="https://github.com/code-423n4/2023-07-pooltogether/assets/104318932/b988c72a-45fa-45bc-a264-ebdec314f1e6">

<img width="992" alt="image" src="https://github.com/code-423n4/2023-07-pooltogether/assets/104318932/4f65d981-f3df-4eb6-940e-f6891f392b55">
https://dev.pooltogether.com/protocol/next/design/





## e) Documents 
 Documentation of the project for the new audit needs to be increased further, adding infographics and graphic-heavy documents showing the functioning of the functions will make the audit easier.

##  f) Centralization risks 
The owner role has a single point of failure and onlyOwner can use critical a few functions.

It is recommended to use timelock and multisign wallets to reduce the risk of centrality.

```js
vault/src/Vault.sol:
  641:   function setClaimer(address claimer_) external onlyOwner returns (address) {

  665    function setLiquidationPair(LiquidationPair liquidationPair_ ) external onlyOwner returns (address) {

  690:   function setYieldFeePercentage(uint256 yieldFeePercentage_) external onlyOwner returns (uint256) {

  703:   function setYieldFeeRecipient(address yieldFeeRecipient_) external onlyOwner returns (address) {
```



## g) Systemic risk 
The Claimant uses the Variable Rate Staged Dutch Auction to price the claim fees. The algorithm is used by the project as a reverse Dutch auction as the price drop is multiple, this is a very new and war-tested application, the biggest risk is that this system is novelty and has not yet been sufficiently tested in war

## h) Competition analysis
There is no other similar project managed by reverse Dutch auction.


## i) Security Approach of the Project


What the project should add in the understanding of Security;
1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)
2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

## j) Other Audit Reports and Automated Findings 

Especially 7 Medium findings in automatic finding are important, it should be taken into account.
**Automated Findings:**
https://gist.github.com/itsmetechjay/e7fd03943bbacff1984a33b9f89c4149


**Other Audit Reports :**
None

## k) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

##  l) Project team
No information found

## m) Other recommendations


## n) New insights and learning from this audit 
- I understand Variable-Rate Gradual Dutch Auction and reverse Dutch auction very well
https://www.paradigm.xyz/2022/08/vrgda

- In a reward savings protocol; I learned how to use fully autonomous, Automated and Permissionless mechanisms effectively




### Time spent:
20 hours