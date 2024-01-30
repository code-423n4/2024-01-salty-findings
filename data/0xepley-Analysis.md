# 🛠️ Analysis - Salty.IO
***An Ethereum-based DEX with zero swap fees, yield-generating Automatic Arbitrage, and a native WBTC/WETH backed stablecoin.***

### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |Overview of the Salty.IO Project| Summary of the whole Protocol |
|b) |Technical Architecture| Architecture of the smart contracts |
|c) |The approach I would follow when reviewing the code | Stages in my code review and analysis |
|d) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|e) |Test analysis | Test scope of the project and quality of tests |
|f) |Security Approach of the Project | Audit approach of the Project |
|g) |Codebase Quality | Overall Code Quality of the Project |
|h) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|i) |Full representation of the project’s risk model| What are the risks associated with the project |
|j) |Packages and Dependencies Analysis | Details about the project Packages |
|k) |New insights and learning of project from this audit | Things learned from the project |



## a) Overview of the Salty Project

The Salty.IO project is a comprehensive ecosystem, focusing on token staking, liquidity provision, and efficient management of digital assets. The project is designed to incentivize users to participate actively in the ecosystem through staking and liquidity provision, while also ensuring secure and efficient management of crucial wallet addresses.

### Key Features and Functionalities:

1. **Staking Rewards Management:**
   - Manages the distribution of rewards for users staking SALT tokens or liquidity shares.
   - Provides mechanisms for users to claim accumulated rewards based on their share in the staking pool.

2. **Token Staking System:**
   - Allows users to stake SALT tokens and receive xSALT, representing their staked amount.
   - Implements a flexible unstaking process with varying durations, influencing the amount of SALT reclaimed.
   - Includes a feature for expedited unstaking with reduced returns, offering users a choice between speed and efficiency.

3. **Secure Wallet Management:**
   - Manages critical wallet addresses using a dual-wallet system (main and confirmation wallets).
   - Enables secure and controlled changes to wallet addresses through a proposal and confirmation process.
   - Incorporates a 30-day timelock for implementing confirmed changes, adding an extra layer of security.



## b) Technical Architecture
[![Screenshot-from-2024-01-30-23-20-24.png](https://i.postimg.cc/B6FHcCmy/Screenshot-from-2024-01-30-23-20-24.png)](https://postimg.cc/CBFRwbTH)
<br/>
<br/>
<br/>

| File Name               | Core Functionality                                      | Technical Characteristics                                                                                               | Importance and Management                                                 |
|-------------------------|---------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| `PoolMath.sol`          | Mathematical operations for liquidity pools             | Implements core math for pool operations, ensuring precise and efficient calculations for liquidity management         | Critical for accurate pool operations and ensuring financial stability    |
| `CoreChainlinkFeed.sol` | Price feed using Chainlink                              | Integrates Chainlink oracles for accurate BTC and ETH prices, crucial for asset valuation in various operations        | Essential for market-relevant pricing and reducing risks in valuations    |
| `CoreUniswapFeed.sol`   | TWAPs (Time-Weighted Average Prices) from Uniswap       | Provides TWAPs for WBTC and WETH, aiding in accurate and time-relevant pricing for these assets                        | Vital for maintaining up-to-date and fair asset pricing in the ecosystem  |
| `PriceAggregator.sol`   | Price aggregation and validation                        | Compares different price feeds for reliability, crucial for maintaining robust and accurate pricing in the ecosystem   | Ensures price integrity by filtering out anomalies and errors in feeds    |
| `CoreSaltyFeed.sol`     | Price retrieval using Salty.IO pools                    | Uses internal Salty.IO pools for asset pricing, adding an additional layer of pricing data                              | Adds a layer of internal validation for asset prices                      |
| `RewardsConfig.sol`     | Management of rewards configurations                    | Sets parameters for reward distribution, including daily percentages and allocation strategies                         | Key in managing how rewards are distributed, impacting user incentives    |
| `SaltRewards.sol`       | Handling of SALT rewards                                | Manages the distribution of SALT rewards from emissions and profits, crucial for incentivizing participation           | Central to the reward system, directly impacting user engagement          |
| `USDS.sol`              | USDS stablecoin management                              | Manages minting and burning of USDS, pivotal for maintaining its stablecoin properties                                  | Critical for the stablecoin's integrity and trustworthiness               |
| `StableConfig.sol`      | Configuration of stablecoin-related parameters          | Sets crucial parameters like collateral ratios and liquidation rewards, affecting the stablecoin's financial health    | Ensures the stablecoin system remains balanced and sustainable            |
| `CollateralAndLiquidity.sol` | Collateral and liquidity management              | Manages user collateral and liquidity provisions, fundamental for the platform's lending and borrowing features        | A cornerstone for the platform's financial activities and user trust      |
| `Liquidizer.sol`        | Token conversion and burning                            | Handles conversion of assets to USDS and burning excess tokens, important for maintaining token supply balance         | Plays a critical role in tokenomics and maintaining market equilibrium   |
| `StakingConfig.sol`     | Configuration of staking parameters                     | Sets key staking parameters like unstake periods and percentages, affecting how users interact with staking features  | Directly influences user staking behavior and platform liquidity          |
| `Liquidity.sol`         | Liquidity provision and management                      | Facilitates adding and withdrawing liquidity, crucial for the platform's liquidity pool operations                     | Key to ensuring sufficient liquidity in the platform's pools             |
| `StakingRewards.sol`    | Management of staking rewards                           | Oversees the distribution of rewards for staking, a major incentive for user participation                              | Central to the platform's staking mechanism and user retention           |
| `Staking.sol`           | SALT token staking functionalities                      | Handles the staking of SALT tokens, a fundamental aspect of the platform's tokenomics                                   | Critical for user engagement and maintaining the token's value stability |
| `ManagedWallet.sol`     | Secure management of crucial wallet addresses           | Ensures safe and controlled management of key wallets, a crucial aspect of platform security                            | Vital for maintaining trust and security in managing significant assets  |


<br/>
<br/>

## c) The approach I would follow when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://code4rena.com/audits/2024-01-saltyio

Accordingly, I would analyze and audit the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#build--test-instructions)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Salty](https://github.com/code-423n4/2024-01-salty/tree/main/src) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| Slither report of the project for some basic analysis|
|5|Test Suits|[Tests](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#build--test-instructions)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#scope)||
|7|Using Solodit for common vulnerabilities|[Solodit](https://solodit.xyz/)|Using solodit to find common vulnerabilites related to Lending Borrowing protocol|
|8|Infographic|[Figma](https://www.figma.com/)|Tried to make Visual drawings to understand the hard-to-understand mechanisms|
|9|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs)|Code where I should focus more|

<br/>
<br/>

## d) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics


-  **File:** This field contains the name or path of the source file being analyzed.

-  **Logic Contracts:** This field indicates the number of Contracts involves

-  **Interfaces:** This field indicated specify the number or details of interfaces defined in the source file.

-  **Lines:** This field represents the total number of lines in the source file, including code lines, comments, and blank lines.

-  **nLines:** nLines typically stands for "normalized lines" and represents the total number of lines in the source file excluding blank lines. 

-  **nSLOC:** nSLOC stands for "normalized source lines of code," and it further refines nLines by excluding both blank lines and comments. It gives a more accurate measure of the code's complexity.

-  **Comment Lines:** This field specifies the number of lines in the source file that contain comments.

-  **Complex. Score:** This field may indicate a complexity score or metric for the source file. 

## Analysis of sloc of `Dao` contracts

[![Screenshot-from-2024-01-30-12-40-36.png](https://i.postimg.cc/gk1md16Z/Screenshot-from-2024-01-30-12-40-36.png)](https://postimg.cc/bs0K9Cgy)

## Analysis of sloc of `Launch` contracts

[![Screenshot-from-2024-01-30-12-41-44.png](https://i.postimg.cc/TwhF478X/Screenshot-from-2024-01-30-12-41-44.png)](https://postimg.cc/47kWK1N8)

## Analysis of sloc of `Pools` contracts

[![Screenshot-from-2024-01-30-12-42-23.png](https://i.postimg.cc/PxLV4CZX/Screenshot-from-2024-01-30-12-42-23.png)](https://postimg.cc/47g14NBq)

## Analysis of sloc of `price_feed` contracts

[![Screenshot-from-2024-01-30-12-43-47.png](https://i.postimg.cc/y6v0fy35/Screenshot-from-2024-01-30-12-43-47.png)](https://postimg.cc/s1GBxSc9)

## Analysis of sloc of `Rewards` contracts

[![Screenshot-from-2024-01-30-12-44-35.png](https://i.postimg.cc/2SK4x2xW/Screenshot-from-2024-01-30-12-44-35.png)](https://postimg.cc/5X8Y45T9)

## Analysis of sloc of `Stable` contracts

[![Screenshot-from-2024-01-30-12-45-10.png](https://i.postimg.cc/2yGWLw6V/Screenshot-from-2024-01-30-12-45-10.png)](https://postimg.cc/rRdsv55k)

## Analysis of sloc of `Staking` contracts

[![Screenshot-from-2024-01-30-12-45-46.png](https://i.postimg.cc/RZyHtqMQ/Screenshot-from-2024-01-30-12-45-46.png)](https://postimg.cc/bZ0rcySs)

## Analysis of sloc of src contracts

[![Screenshot-from-2024-01-30-12-47-00.png](https://i.postimg.cc/1tTVQJ05/Screenshot-from-2024-01-30-12-47-00.png)](https://postimg.cc/LYt8kk0r)

## Comment-to-Source Ratio:

**`DAO` contracts:** On average there are **4.69** code lines per comment (lower=better).

**`Launch` contracts:** On average there are **4.39** code lines per comment (lower=better).

**`Pools` contracts:** On average there are **1.82** code lines per comment (lower=better).

**`price_feed` contracts:** On average there are **4.49** code lines per comment (lower=better).

**`Rewards` contracts:** On average there are **3.54** code lines per comment (lower=better).

**`Stable` contracts:** On average there are **3.26** code lines per comment (lower=better).

**`Staking` contracts:** On average there are **2.98** code lines per comment (lower=better).

**`src` contracts:** On average there are **3.22** code lines per comment (lower=better).

# Call Graph of Important Contracts

## Call graph of `Launch` contract
[![Screenshot-from-2024-01-30-12-52-14.png](https://i.postimg.cc/RVRdrJp6/Screenshot-from-2024-01-30-12-52-14.png)](https://postimg.cc/w71LLMpg)

## Call graph of `Pools` contracts

[![Screenshot-from-2024-01-30-12-53-45.png](https://i.postimg.cc/R09vW8Qb/Screenshot-from-2024-01-30-12-53-45.png)](https://postimg.cc/G4MWNXfk)

## Call graph of `Price_feed` contracts

[![Screenshot-from-2024-01-30-15-46-04.png](https://i.postimg.cc/SKZfSKhb/Screenshot-from-2024-01-30-15-46-04.png)](https://postimg.cc/R3HHG4hg)

## Call graph of `Staking` contracts

[![Screenshot-from-2024-01-30-15-53-21.png](https://i.postimg.cc/nL5TKLwY/Screenshot-from-2024-01-30-15-53-21.png)](https://postimg.cc/QBQphjDH)

## Contract Integration Graph

[![Screenshot-from-2024-01-30-15-54-15.png](https://i.postimg.cc/5yfz8d1S/Screenshot-from-2024-01-30-15-54-15.png)](https://postimg.cc/cg56Nzvv)


# High Level Domain Model

This domain model provides an overview of the key components  and how they are interconnected.

[![Screenshot-from-2024-01-30-22-20-55.png](https://i.postimg.cc/P51HHnpR/Screenshot-from-2024-01-30-22-20-55.png)](https://postimg.cc/RWCy18T7)

<br/>
<br/>

## e) Test analysis

 **Foundry Testing:**
   
   Foundry, a modern smart contract testing framework, was utilized to test the Salty contracts. This involved several key steps:
   
   a. **Installation and Setup:**
      - Foundry was installed using the command `curl -L https://foundry.paradigm.xyz | bash`, followed by `foundryup` to ensure the latest version was in use.
      - Dependencies were installed using `forge install`, ensuring all necessary components were available for the testing process.
      - Then to run the tests, I simply added the relevant files to the .env, referencing .env.example.
   
   b. **Execution of Tests:**
      - Tests were run using `fCOVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url  https://rpc.sepolia.org`, executing a suite of predefined test cases that covered various functionalities and scenarios.
   
   c. **Test Coverage and Documentation:**
      - The overview of the testing suite, as referred to in the provided documentation, likely details the scope, scenarios, and objectives of each test, ensuring a comprehensive assessment of the contracts.
   

### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content. In particular, tests have been written successfully.

-   2) Overall line coverage percentage provided by your tests : 99

### What could they have done better?

-  1) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


[![test-cases.jpg](https://i.postimg.cc/1zgD5wCt/test-cases.jpg)](https://postimg.cc/v1s40gdF)

Ref:https://xin-xia.github.io/publication/icse194.pdf

[![nabeel-1.jpg](https://i.postimg.cc/6qtBdLQW/nabeel-1.jpg)](https://postimg.cc/bDVXPnbW)

<br/>
<br/>


## Imp Test cases coverage with gas report

### ExchangeConfig.sol

| Function Name          | min   | avg    | median | max    | # calls |
|------------------------|-------|--------|--------|--------|---------|
| accessManager          | 390   | 390    | 390    | 390    | 11      |
| airdrop                | 370   | 592    | 370    | 2370   | 9       |
| dai                    | 237   | 237    | 237    | 237    | 1536    |
| dao                    | 370   | 378    | 370    | 2370   | 2548    |
| daoVestingWallet       | 391   | 1191   | 391    | 2391   | 30      |
| initialDistribution    | 370   | 397    | 370    | 2370   | 2515    |
| managedTeamWallet      | 238   | 238    | 238    | 238    | 47      |
| salt                   | 260   | 260    | 260    | 260    | 6304    |
| setAccessManager       | 620   | 23705  | 23829  | 23829  | 513     |
| setContracts           | 1104  | 133564 | 133824 | 133824 | 511     |
| teamVestingWallet      | 368   | 747    | 368    | 2368   | 58      |
| transferOwnership      | 2331  | 2331   | 2331   | 2331   | 509     |
| upkeep                 | 348   | 731    | 348    | 2348   | 334     |
| usds                   | 238   | 238    | 238    | 238    | 2580    |
| walletHasAccess        | 551   | 3343   | 1886   | 14386  | 2054    |
| wbtc                   | 283   | 283    | 283    | 283    | 2120    |
| weth                   | 239   | 239    | 239    | 239    | 3136    |

### ManagedWallet.sol

| Function Name                | min  | avg   | median | max   | # calls |
|------------------------------|------|-------|--------|-------|---------|
| activeTimelock               | 340  | 340   | 340    | 340   | 7       |
| changeWallets                | 373  | 1924  | 2373   | 2966  | 6       |
| confirmationWallet           | 303  | 303   | 303    | 303   | 2       |
| mainWallet                   | 325  | 805   | 325    | 2325  | 50      |
| proposeWallets               | 693  | 27356 | 46368  | 46368 | 12      |
| proposedConfirmationWallet   | 324  | 324   | 324    | 324   | 3       |
| proposedMainWallet           | 346  | 346   | 346    | 346   | 3       |
| receive                      | 0    | 324   | 381    | 381   | 8       |

### Salt.sol

| Function Name          | min   | avg   | median | max   | # calls |
|------------------------|-------|-------|--------|-------|---------|
| approve                | 2604  | 24431 | 24604  | 24604 | 3631    |
| balanceOf              | 583   | 752   | 583    | 2583  | 1474    |
| burnTokensInContract   | 3921  | 6131  | 6721   | 8721  | 38      |
| decimals               | 266   | 266   | 266    | 266   | 18      |
| name                   | 3198  | 3198  | 3198   | 3198  | 1       |
| symbol                 | 3263  | 3263  | 3263   | 3263  | 1       |
| totalBurned            | 562   | 673   | 562    | 2562  | 18      |
| totalSupply            | 349   | 820   | 349    | 2349  | 174     |
| transfer               | 3034  | 23949 | 24934  | 29734 | 3205    |
| transferFrom           | 879   | 21241 | 25342  | 32142 | 1178    |

### DAO.sol:

| Function Name               | min   | avg    | median | max    | # calls |
|-----------------------------|-------|--------|--------|--------|---------|
| collateralAndLiquidity      | 251   | 251    | 251    | 251    | 1       |
| countryIsExcluded           | 825   | 1396   | 825    | 2825   | 7       |
| daoConfig                   | 293   | 293    | 293    | 293    | 1       |
| exchangeConfig              | 274   | 274    | 274    | 274    | 1       |
| finalizeBallot              | 7296  | 85003  | 50499  | 520797 | 137     |
| formPOL                     | 5950  | 208196 | 208151 | 279651 | 50      |
| liquidityRewardsEmitter     | 272   | 272    | 272    | 272    | 1       |
| pools                       | 294   | 294    | 294    | 294    | 307     |
| poolsConfig                 | 294   | 294    | 294    | 294    | 1       |
| priceAggregator             | 251   | 251    | 251    | 251    | 1       |
| processRewardsFromPOL       | 5674  | 62314  | 74013  | 114013 | 30      |
| proposals                   | 251   | 251    | 251    | 251    | 1       |
| rewardsConfig               | 250   | 250    | 250    | 250    | 1       |
| stableConfig                | 250   | 250    | 250    | 250    | 1       |
| stakingConfig               | 295   | 295    | 295    | 295    | 1       |
| websiteURL                  | 1318  | 2489   | 3075   | 3075   | 3       |
| withdrawArbitrageProfits    | 2402  | 44638  | 59649  | 64449  | 34      |
| withdrawPOL                 | 661   | 74841  | 3930   | 179861 | 21      |

### DAOConfig.sol

| Function Name                           | min  | avg  | median | max  | # calls |
|-----------------------------------------|------|------|--------|------|---------|
| arbitrageProfitsPercentPOL              | 352  | 923  | 352    | 2352 | 42      |
| ballotMinimumDuration                   | 352  | 961  | 352    | 2352 | 220     |
| baseBallotQuorumPercentTimes1000        | 374  | 724  | 374    | 2374 | 194     |
| bootstrappingRewards                    | 373  | 887  | 373    | 2373 | 35      |
| changeArbitrageProfitsPercentPOL        | 1773 | 2866 | 2084   | 4884 | 17      |
| changeBallotDuration                    | 1772 | 2713 | 2072   | 4883 | 21      |
| changeBaseBallotQuorumPercent           | 1773 | 2509 | 2073   | 4884 | 31      |
| changeBootstrappingRewards              | 1771 | 2999 | 2071   | 6882 | 21      |
| changeMaxPendingTokensForWhitelisting   | 1772 | 2918 | 2077   | 8872 | 24      |
| changePercentPolRewardsBurned           | 1794 | 2736 | 2094   | 4905 | 21      |
| changeRequiredProposalPercentStake      | 1751 | 2390 | 2051   | 4862 | 40      |
| changeUpkeepRewardPercent               | 1773 | 2747 | 2073   | 4884 | 20      |
| maxPendingTokensForWhitelisting         | 351  | 1030 | 351    | 2351 | 53      |
| percentPolRewardsBurned                 | 350  | 850  | 350    | 2350 | 44      |
| requiredProposalPercentStakeTimes1000   | 329  | 919  | 329    | 2329 | 227     |
| transferOwnership                       | 2323 | 2323 | 2323   | 2323 | 466     |
| upkeepRewardPercent                     | 351  | 802  | 351    | 2351 | 62      |

### Proposals.sol

| Function Name                           | min   | avg    | median | max    | # calls |
|-----------------------------------------|-------|--------|--------|--------|---------|
| ballotForID                             | 4651  | 4703   | 4651   | 5180   | 515     |
| ballotIsApproved                        | 744   | 783    | 744    | 2744   | 51      |
| canFinalizeBallot                       | 3372  | 14390  | 14721  | 20728  | 145     |
| castVote                                | 6488  | 63761  | 77263  | 77651  | 181     |
| createConfirmationProposal              | 6595  | 255127 | 258680 | 314216 | 14      |
| lastUserVoteForBallot                   | 1153  | 1153   | 1153   | 1153   | 8       |
| markBallotAsFinalized                   | 4265  | 8326   | 8144   | 11332  | 141     |
| nextBallotID                            | 364   | 1252   | 364    | 2364   | 9       |
| openBallots                             | 833   | 1244   | 1256   | 1609   | 6       |
| openBallotsByName                       | 861   | 1210   | 861    | 2873   | 23      |
| openBallotsForTokenWhitelisting         | 1163  | 1269   | 1163   | 1398   | 11      |
| proposeCallContract                     | 5829  | 301459 | 375867 | 375867 | 5       |
| proposeCountryExclusion                 | 5925  | 245790 | 300803 | 300803 | 6       |
| proposeCountryInclusion                 | 5936  | 234925 | 291125 | 300825 | 10      |
| proposeParameterBallot                  | 6763  | 273206 | 281771 | 418283 | 107     |
| proposeSendSALT                         | 5767  | 165956 | 167908 | 322904 | 6       |
| proposeSetContractAddress               | 6046  | 254465 | 281400 | 326390 | 21      |
| proposeTokenUnwhitelisting              | 10259 | 126914 | 13406  | 382471 | 12      |
| proposeTokenWhitelisting                | 5649  | 334836 | 419569 | 459409 | 27      |
| proposeWebsiteUpdate                    | 6211  | 240083 | 301120 | 345580 | 8       |
| requiredQuorumForBallotType             | 1391  | 4323   | 3987   | 10399  | 11      |
| tokenWhitelistingBallotWithTheMostVotes | 5230  | 5901   | 5279   | 9205   | 8       |
| totalVotesCastForBallot                 | 4005  | 5795   | 6069   | 8069   | 7       |
| userHasActiveProposal                   | 563   | 563    | 563    | 563    | 2       |
| votesCastForBallot                      | 696   | 696    | 696    | 696    | 9       |
| winningParameterVote                    | 1047  | 1103   | 1049   | 5047   | 85      |

### Deployment.sol

| Function Name      | min  | avg  | median | max  | # calls |
|--------------------|------|------|--------|------|---------|
| DEPLOYER           | 316  | 316  | 316    | 316  | 6       |
| dai                | 459  | 459  | 459    | 459  | 6       |
| dao                | 438  | 2188 | 2438   | 2438 | 8       |
| exchangeConfig     | 440  | 440  | 440    | 440  | 14      |
| managedTeamWallet  | 415  | 415  | 415    | 415  | 6       |
| pools              | 459  | 977  | 459    | 2459 | 27      |
| poolsConfig        | 395  | 736  | 395    | 2395 | 41      |
| salt               | 415  | 415  | 415    | 415  | 6       |
| upkeep             | 416  | 1616 | 2416   | 2416 | 5       |
| usds               | 437  | 437  | 437    | 437  | 6       |
| wbtc               | 416  | 416  | 416    | 416  | 18      |
| weth               | 416  | 701  | 416    | 2416 | 7       |


### InitialDistribution.sol

| Function Name            | min  | avg    | median | max    | # calls |
|--------------------------|------|--------|--------|--------|---------|
| airdrop                  | 260  | 260    | 260    | 260    | 1       |
| bootstrapBallot          | 215  | 215    | 215    | 215    | 1574    |
| collateralAndLiquidity   | 217  | 217    | 217    | 217    | 1       |
| dao                      | 216  | 216    | 216    | 216    | 1       |
| daoVestingWallet         | 260  | 260    | 260    | 260    | 1       |
| distributionApproved     | 366  | 507020 | 508709 | 588609 | 318     |
| emissions                | 238  | 238    | 238    | 238    | 1       |
| poolsConfig              | 238  | 238    | 238    | 238    | 1       |
| salt                     | 237  | 237    | 237    | 237    | 1       |
| saltRewards              | 239  | 239    | 239    | 239    | 1       |
| teamVestingWallet        | 259  | 259    | 259    | 259    | 1       |

### Pools.sol

| Function Name              | min   | avg   | median | max    | # calls |
|----------------------------|-------|-------|--------|--------|---------|
| addLiquidity               | 1148  | 47913 | 44560  | 92398  | 1847    |
| arbitrageIndicies          | 838   | 1474  | 838    | 2838   | 22      |
| clearProfitsForPools       | 8086  | 12044 | 8086   | 63508  | 28      |
| deposit                    | 5600  | 28509 | 29913  | 58801  | 995     |
| depositDoubleSwapWithdraw  | 56379 | 85017 | 85017  | 113656 | 2       |
| depositSwapWithdraw        | 5686  | 37728 | 29973  | 84156  | 546     |
| depositedUserBalance       | 743   | 1014  | 743    | 2743   | 140     |
| exchangeIsLive             | 410   | 1410  | 1410   | 2410   | 4       |
| getPoolReserves            | 1178  | 1560  | 1197   | 3197   | 574     |
| profitsForWhitelistedPools | 9285  | 37491 | 26180  | 224644 | 29      |
| removeLiquidity            | 6070  | 24070 | 7874   | 57472  | 203     |
| setContracts               | 670   | 46745 | 46836  | 46836  | 512     |
| startExchangeApproved      | 13838 | 38664 | 37843  | 93465  | 307     |
| swap                       | 3688  | 28232 | 29662  | 84559  | 35      |
| updateArbitrageIndicies    | 8135  | 57382 | 37157  | 922812 | 5773    |
| withdraw                   | 3865  | 20890 | 27721  | 32521  | 41      |

### PoolsConfig.sol

| Function Name                             | min  | avg   | median | max    | # calls |
|-------------------------------------------|------|-------|--------|--------|---------|
| changeMaximumInternalSwapPercentTimes1000 | 1816 | 2908  | 2116   | 4927   | 17      |
| changeMaximumWhitelistedPools             | 1793 | 3390  | 2104   | 8904   | 30      |
| isWhitelisted                             | 333  | 657   | 510    | 2510   | 6266    |
| maximumInternalSwapPercentTimes1000       | 317  | 1182  | 317    | 2317   | 67      |
| maximumWhitelistedPools                   | 362  | 1028  | 362    | 2362   | 57      |
| numberOfWhitelistedPools                  | 393  | 1133  | 393    | 2393   | 100     |
| tokenHasBeenWhitelisted                   | 1084 | 3454  | 3616   | 5617   | 51      |
| transferOwnership                         | 2352 | 2352  | 2352   | 2352   | 509     |
| underlyingTokenPair                       | 780  | 963   | 840    | 4840   | 47666   |
| unwhitelistPool                           | 743  | 67735 | 54909  | 155188 | 12      |
| whitelistPool                             | 1056 | 151797| 132033 | 1015267| 5765    |
| whitelistedPools                          | 1118 | 3317  | 2528   | 45822  | 6533    |


### CoreSaltyFeed.sol

| Function Name | min   | avg  | median | max  | # calls |
|---------------|-------|------|--------|------|---------|
| getPriceBTC   | 2047  | 4215 | 4215   | 6384 | 10      |
| getPriceETH   | 1863  | 2898 | 2026   | 6363 | 9       |
| pools         | 248   | 248  | 248    | 248  | 1       |
| usds          | 204   | 204  | 204    | 204  | 1       |
| wbtc          | 227   | 227  | 227    | 227  | 1       |
| weth          | 249   | 249  | 249    | 249  | 1       |

### PriceAggregator.sol

| Function Name                                | min   | avg   | median | max    | # calls |
|----------------------------------------------|-------|-------|--------|--------|---------|
| changeMaximumPriceFeedPercentDifferenceTimes1000 | 1771 | 2821  | 2071   | 6882   | 26      |
| changePriceFeedModificationCooldown          | 1773  | 3296  | 2084   | 4884   | 11      |
| getPriceBTC                                  | 3430  | 7078  | 4293   | 32333  | 264     |
| getPriceETH                                  | 3304  | 4587  | 4107   | 10743  | 246     |
| maximumPriceFeedPercentDifferenceTimes1000   | 352   | 610   | 352    | 2352   | 31      |
| setInitialFeeds                              | 871   | 67035 | 67163  | 67163  | 521     |
| setPriceFeed                                 | 638   | 13023 | 7538   | 33415  | 20      |
| transferOwnership                            | 2352  | 2352  | 2352   | 2352   | 509     |

### ForcedPriceFeed.sol

| Function Name | min  | avg  | median | max  | # calls |
|---------------|------|------|--------|------|---------|
| getPriceBTC   | 453  | 740  | 506    | 4506 | 66      |
| getPriceETH   | 398  | 840  | 462    | 2462 | 15      |
| setBTCPrice   | 522  | 2525 | 522    | 7422 | 31      |
| setRevertNext | 3198 | 4698 | 5198   | 5198 | 8       |

### TestChainlinkAggregator.sol

| Function Name    | min   | avg   | median | max   | # calls |
|------------------|-------|-------|--------|-------|---------|
| latestRoundData  | 362   | 2162  | 2886   | 2896  | 9       |
| setPrice         | 543   | 5471  | 7443   | 7443  | 7       |
| setShouldFail    | 24462 | 24462 | 24462  | 24462 | 1       |
| setShouldTimeout | 24441 | 24441 | 24441  | 24441 | 1       |

### RewardsConfig.sol:RewardsConfig contract

| Function Name                         | min   | avg  | median | max  | # calls |
|---------------------------------------|-------|------|--------|------|---------|
| changeEmissionsWeeklyPercent          | 1794  | 2952 | 2078   | 4886 | 9       |
| changePercentRewardsSaltUSDS          | 1750  | 2749 | 2034   | 4842 | 11      |
| changeRewardsEmitterDailyPercent      | 1771  | 2576 | 2055   | 6863 | 19      |
| changeStakingRewardsPercent           | 1751  | 2450 | 2035   | 4843 | 19      |
| emissionsWeeklyPercentTimes1000       | 351   | 551  | 351    | 2351 | 10      |
| percentRewardsSaltUSDS                | 306   | 472  | 306    | 2306 | 12      |
| rewardsEmitterDailyPercentTimes1000   | 329   | 429  | 329    | 2329 | 20      |
| stakingRewardsPercent                 | 330   | 430  | 330    | 2330 | 20      |

### SaltRewards.sol:SaltRewards contract

| Function Name              | min   | avg    | median | max     | # calls |
|----------------------------|-------|--------|--------|---------|---------|
| liquidityRewardsEmitter    | 237   | 237    | 237    | 237     | 28      |
| performUpkeep              | 4193  | 134574 | 44767  | 2330225 | 28      |
| sendInitialSaltRewards     | 299526| 301139 | 299526 | 336126  | 315     |
| stakingRewardsEmitter      | 215   | 215    | 215    | 215     | 28      |

### TestSaltRewards.sol

| Function Name                  | min    | avg   | median | max   | # calls |
|--------------------------------|--------|-------|--------|-------|---------|
| performUpkeep                  | 3645   | 42520 | 4518   | 157401| 4       |
| sendInitialLiquidityRewards    | 115396 | 128460| 128460 | 141525| 2       |
| sendInitialSaltRewards         | 3414   | 65756 | 65756  | 128098| 2       |
| sendInitialStakingRewards      | 60174  | 60174 | 60174  | 60174 | 2       |
| sendLiquidityRewards           | 10121  | 69770 | 84078  | 88328 | 6       |
| sendStakingRewards             | 12968  | 44424 | 60152  | 60152 | 3       |



## f) Security Approach of the Project

### Successful current security understanding of the project;

1- The project hasn't underwent any audits(nothing stated in the docs), this innovative assessments on Code4rena is the first, where multiple auditors are scrutinizing the code.

### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

3- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

4- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

5- I also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

6 - As the project team, you can consider applying the multi-stage audit model.

[![sla.png](https://i.postimg.cc/nhR0kN3w/sla.png)](https://postimg.cc/Sn96Q1FW)

Read more about the MPA model;
https://mpa.solodit.xyz/

7 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19


## g) Codebase Quality

Overall, I consider the quality of the Salty.io protocol codebase to be Good. The code appears to be mature and well-developed, though there are areas for improvement, particularly in code commenting. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The codebase demonstrates a high level of maintainability and reliability. It is clear that the developers have focused on creating a robust and scalable architecture. The use of established Ethereum development patterns and adherence to Solidity best practices contributes significantly to the code's overall reliability. |
| **Code Comments**                        | The codebase shows a lack of comprehensive comments, particularly in complex logic areas. This can make it challenging to understand the purpose and functionality of certain sections, which might hinder the onboarding of new developers and code audits. Improving the comments would significantly enhance the codebase's clarity and maintainability. |
| **Documentation**                        | The project includes comprehensive documentation. It covers the overall architecture, functionality, and specific details about key components like staking mechanisms and wallet management. This level of documentation is essential for both developers and end-users to understand and interact with the protocol effectively. |
| **Testing**                              | The protocol demonstrates a strong emphasis on testing, which is evident from the extensive test cases covering various scenarios. Regular and thorough testing enhances the code's stability and reduces the likelihood of unforeseen issues in a live environment. |
| **Code Structure and Formatting**        | The code is well-structured and consistently formatted. It follows a logical structure that makes it easy to navigate and understand. Consistent formatting across the codebase not only improves readability but also indicates a professional development approach. |

While the codebase is robust and well-structured, the lack of detailed comments in the code is a notable area for improvement. Enhancing the code commenting would further elevate the overall quality and accessibility of the project.


## h) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-01-salty/blob/main/bot-report.md

**Previous Audits**
[ABDK Audit](https://github.com/abdk-consulting/audits/blob/main/othernet_global_pte_ltd/ABDK_OthernetGlobalPTELTD_SaltyIO_v_2_0.pdf)
[Trail of Bits](https://github.com/trailofbits/publications/blob/master/reviews/2023-10-saltyio-securityreview.pdf)

**4naly3er report**
https://github.com/code-423n4/2024-01-salty/blob/main/4naly3er-report.md

## i) Full representation of the project’s risk model

### 1. Admin Abuse Risks:
- **Centralized Control Points**: The project's governance is heavily reliant on smart contracts like `ManagedWallet.sol`, `ExchangeConfig.sol`, and the `DAO`. While these contracts ostensibly distribute control, there's a risk of centralization if few actors hold significant control.
- **Upgrade and Proposal Approval**: The `DAO` and `ManagedWallet.sol` contracts have functionalities to approve upgrades and changes. If these mechanisms are compromised or controlled by a small group, they could be used maliciously.

### 2. Systemic Risks:
- **Interconnected Contract Dependencies**: The project's DeFi ecosystem comprises various interdependent contracts (like `Liquidity.sol`, `Staking.sol`, `Emissions.sol`). A malfunction or exploitation in one contract could ripple through the entire system.\
- **Liquidity Risks**: The liquidity pools (`Pools.sol`) are central to the ecosystem. Any liquidity crunch or imbalance can pose systemic risks.

### 3. Technical Risks:
- **Smart Contract Vulnerabilities**: Given the complexity of contracts like `Upkeep.sol`, `Staking.sol`, and others, there's a risk of bugs or vulnerabilities that could be exploited, despite thorough auditing.
- **Oracle Failures**: The system relies on `PriceAggregator.sol` for market data. Any failure or manipulation in the price feeds can lead to incorrect valuations and system responses.

By continuously monitoring these risk factors and implementing robust mitigation strategies, Salty.IO can aim to ensure a secure and resilient DeFi ecosystem for its users.

##  j) Packages and Dependencies Analysis 📦

| Package | Version | Usage | 
| --- | --- | --- | 
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts) | [![npm]([![images.png](https://i.postimg.cc/MK89GbgX/images.png)](https://postimg.cc/pyqfG8Wt))](https://www.npmjs.com/package/@openzeppelin/contracts) |  Project uses version `4.9.3` while the recommended version is latest i.e: `5.0.1` 

## k) New insights and learning of project from this audit:

After thoroughly reviewing the Salty.io project's codebase and documentation, several new insights and learnings have emerged.

1. **Use of Uniswap and Chainlink**: The project utilizes Uniswap for decentralized trading and Chainlink for secure and reliable external data. This combination indicates an emphasis on robustness and security in obtaining market data and executing trades.

2. **Stablecoin Implementation**: The `USDS.sol` contract and related `StableConfig.sol` suggest the implementation of a stablecoin (`USDS`), backed by crypto assets like WBTC and WETH. This approach is critical for maintaining stability in a volatile crypto market.

3. **Rewards and Incentivization**: Contracts like `SaltRewards.sol` and `StakingRewards.sol` indicate a mechanism to reward users for participating in the ecosystem, such as through liquidity provision or staking. This is a common practice in DeFi to encourage user participation and liquidity.

4. **Governance and DAO**: The use of a DAO (Decentralized Autonomous Organization) structure for governance, as seen in `DAO.sol` and `ExchangeConfig.sol`, indicates a decentralized governance model. This aligns with the broader ethos of the DeFi sector promoting community-driven decision-making.

5. **Risk Management**: Contracts like `CollateralAndLiquidity.sol` and `Liquidizer.sol` show mechanisms for managing risks associated with collateralized debt positions and liquidity provision. This is essential for maintaining the system's health and user trust.

Overall, your project presents a sophisticated and multifaceted DeFi ecosystem, incorporating key elements like liquidity provision, stablecoin implementation, rewards, governance, and compliance. It shows a strong alignment with DeFi's principles of open finance and community governance, while also considering critical aspects like security and risk management.

Note: I didn't tracked the time, the time I mentioned is just an estimate

### Time spent:
5 hours