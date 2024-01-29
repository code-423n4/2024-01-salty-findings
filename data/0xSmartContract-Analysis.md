# 🛠️ Analysis - Salty.IO Audit
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |Gas Optimizations | Gas usage approach of the project and alternative solutions |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2024-01-salty

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#build--test-instructions)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [SaltyIO](https://tech.salty.io/) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from initial|
|5|Test Suits|[Tests](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#build--test-instructions)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#scope)|
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit

In addition, the auditor can decide on the question of where should I end the audit by examining these metrics;

Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="778" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/8de913a1-3a8e-4a6c-8f4d-12506e01971f">








</br>
</br>

## Project - Overview;

Salty.IO is a decentralized exchange built on Ethereum and uses Automated Atomic Arbitrage (AAA) offering zero fees on all exchange transactions.

AAA generates profits by arbitraging market inefficiencies and distributes these profits to liquidity providers and bookmakers, as well as creating Protocol Owner Liquidity (POL) for the DAO.

Salty.IO offers a protocol-specific overcollateralized ERC20 stablecoin called USDS, which is collateralized by WBTC/WETH LP. The exchange is 100% decentralized, with the DAO controlling all parameters, regional restrictions, whitelisting and contracts.

The SALT exchange token (ERC20) is limited to a maximum of 100 million tokens and includes limited emissions and anti-inflation mechanisms. Investing SALT yields xSALT, which is used for both returns and management.

 Initially, it won't be available in the U.S. due to regulatory reasons.

![image](https://github.com/code-423n4/2024-01-salty/assets/104318932/bc35b4d9-a568-4b21-86eb-1ad2cb730af2)

##### Note: Please click on the image to enlarge it:


</br>
</br>
</br>
</br>

## The Salty.IO  All codebase ;

Unique Features of Salty.IO: Salty.IO features novel elements such as a built-from-scratch pool system, an internal arbitrage mechanism that balances pools and generates profit, and protocol-owned liquidity for the DAO. It also has a unique stablecoin, USDS, over-collateralized with wrapped BTC and ETH, and a liquidation mechanism.

Technical Aspects and Ecosystem: The project includes a token called SALT, with adjustable emissions controlled by the DAO. Salty.IO features several key components: an arbitrage folder for identifying profitable trading paths, a rewards system for emissions, pool math for depositing varying amounts, a price feed, and a liquidizer contract for handling liquidated collateral.

Security and Accessibility: The platform incorporates an off-chain signing mechanism for airdrop validation and uses global servers for IP-based user location verification. It has an upkeep contract allowing anyone to call for maintenance, ensuring system health. The project, described as complex yet innovative, aims to address both functional and regulatory challenges in the crypto exchange domain.


![image](https://github.com/code-423n4/2024-01-salty/assets/104318932/c4ab8400-1c55-4e94-a4cb-d2dc67ba6552)


##### Note: Please click on the image to enlarge it:

</br>
</br>
</br>
</br>

## Codebase Details;

<img width="1275" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/8e70af4d-52fd-4e1e-91ac-e20a482df41e">


</br>
</br>
</br>
</br>

<img width="220" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/944da619-0fdb-4e88-96a2-46476aa09278">

The launch of Salty.IO begins with the whitelisting of tens of thousands of users (who have taken part in prominent DeFi projects).

Whitelisted users can choose to retweet the launch announcement and then vote on the BootstrapBallot.sol.  The BootstrapBallot.vote() function includes a signature that confirms that voting users are both whitelisted and have retweeted the launch announcement.

Once whitelisted users vote, they are eligible to receive the airdrop. Airdrop recipients are able to claim their share of xSALT once the exchange is launched (5 million xSALT evenly divided amongst all eligible recipients).

The bootstrap ballot determines whether or not the DEX should be started and have SALT distributed to the exchange contracts via InitialDistribution.sol.


![image](https://github.com/code-423n4/2024-01-salty/assets/104318932/e907c958-be46-43c4-9685-399431b2c8d5)



</br>
</br>
</br>

<img width="219" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/03a8499d-44b1-4fb0-bea0-48df7192ab5a">

When swaps are made on a DEX there is price movement - which can create market imbalances and introduce opportunities for arbitrage profit.
Here's an example:
Imagine two markets for the liquidity pair ETH/USD. Both markets start out with the same price for ETH. 
If ETH is bought on market A the price there becomes higher. 
ETH can then be bought on market B and sold for more on market A to produce an arbitrage profit. 

ArbitrageSearch.sol searches for and returns profitable arbitrage trade paths that always start and end with WETH (meaning all arbitrage profits are then in WETH).


User Swap | Arbitrage Path
-- | --
WBTC->WETH | WETH->WBTC->SALT->WETH
WETH->WBTC | WETH->SALT->WBTC->WETH
WETH->token | WETH->WBTC->token->WETH
token->WETH | WETH->token->WBTC->WETH
token1->token2 | WETH->token2->token1->WETH





![image](https://github.com/code-423n4/2024-01-salty/assets/104318932/4552ff3c-7869-4a95-a225-33e7b4f430d3)

</br>
</br>
</br>
</br>


<img width="218" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/65deb254-7e17-46f9-9559-a6b2eaa646e0">


`Pool.sol` implements k=xy style AMM token pools, with every token on the exchange being paired with both WBTC and WETH.
Standard methods such as `addLiquidity` and `removeLiquidity` are present along with convenience functions such as dualZapInLiquidity (to fully deposit arbitrary amounts of tokens regardless of ratio by automatically swapping them first as necessary).

As a means to save gas, users are able to deposit tokens into the Pools contract for later use in multiple swaps - eliminating the token transfer normally required on each swap. 


There are two main functions for swapping: 
` swap()` : requires that the tokenIn has already been deposited by the user.

`depositSwapWithdraw()` : allows swaps without having to deposit tokenIn first (at a higher gas cost).


<img width="1337" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/3880470e-9783-4a37-a1b0-94426b81e30f">

</br>
</br>
</br>
</br>

<img width="222" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/2854ffaf-1ec0-4fbd-8a1d-11417128b194">


Price Feed Architecture of SALTY Project

**Entrance:**
The price feed architecture of the SALTY project is implemented through a smart contract running on the Ethereum network. This architecture combines Bitcoin (BTC) and Ethereum (ETH) prices by pulling them from three different price feed sources (Oracle). This report will examine the advantages and disadvantages of this architecture in detail.

**General Summary of Architecture:**
- Three different price feed sources (IPriceFeed) are used: `priceFeed1`, `priceFeed2`, `priceFeed3`.
- Prices are taken from these three sources and combined to reach a reliable average.
- There is a certain cooldown period for price updates and changes.
- A maximum percentage difference (`maximumPriceFeedPercentDifferenceTimes1000`) has been set to limit price differences.

**Advantages:**

1. **Reliability and Durability:**
    - Pulling prices from three different sources reduces the risk of relying on faulty or manipulated data from a single source.
    - If one source gives an error, the other two sources can still provide price information.

2. **Manipulation Resistance:**
    - Using three different sources provides extra protection against price manipulation. Manipulating a single resource does not affect the overall price.

3. **Flexibility and Updatability:**
    - Price feed sources can be updated with DAO recommendations. This allows the project to adapt to changing market conditions over time.

4. **Automatic Error Detection:**
    - When price differences exceed a certain threshold, the system automatically excludes erroneous data. This prevents erroneous data from influencing the overall price calculation.

**Disadvantages:**

1. **Complexity and Cost:**
    - Using three separate price sources increases system complexity and may lead to higher gas costs.

2. **Cooldown Risk:**
    -The cooling-off period required to change price feed sources may limit the ability to react quickly to sudden changes in the market.

3. **Possibility of Common Error:**
    - If all three sources are subject to similar errors or manipulations, this can lead to mispricing.

4. **Data Source Reliability:**
    - The reliability of price feed sources directly affects the overall reliability of the project. If these sources are unreliable, the entire system is at risk.


<img width="1268" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/dfe7a811-6d57-4f56-b7ec-dc92a460ae0d">


</br>
</br>
</br>
</br>
</br>

<img width="216" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/39c56975-2f68-45e9-8c6a-ff672011af69">


The Salty project's stable component, as represented by the four contracts (`StableConfig.sol`, `USDS.sol`, `Liquidizer.sol`, `CollateralAndLiquidity.sol`), demonstrates a sophisticated and integrated DeFi lending and liquidation ecosystem. 

1. **StableConfig.sol**: Acts as the configuration center for the system. It manages critical parameters like reward percentages for liquidation, collateral requirements, and ratios for borrowing. These parameters are adjustable, ensuring the system's adaptability to changing market conditions.

2. **USDS.sol**: This contract is responsible for the minting and burning of the USDS stablecoin. It allows USDS to be minted against deposited WBTC/WETH liquidity as collateral and burned upon repayment or liquidation. The contract integrates closely with the `CollateralAndLiquidity.sol` contract to control minting permissions and align with the system's collateralization rules.

3. **Liquidizer.sol**: Central to the liquidation process, this contract handles the conversion of collateral into USDS for burning, addressing undercollateralized liquidations. It interfaces with other system components to swap various tokens (like WBTC, WETH, DAI) into USDS and manages the Protocol Owned Liquidity (POL) for additional stability measures.

4. **CollateralAndLiquidity.sol**: A key component for managing user interactions with the system. It allows users to deposit and withdraw collateral (WBTC/WETH), borrow USDS based on their collateral, and repay borrowed USDS. It also facilitates the liquidation process for undercollateralized positions, integrating with `Liquidizer.sol` and `StableConfig.sol` to manage the mechanics and economics of liquidation.

<img width="1106" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/f6353ef8-e3b4-4cc2-b46a-e547e8fa7395">



</br>
</br>
</br>
</br>



<img width="217" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/334acc4a-21d0-4469-b8ed-6b625307ba09">

From launch, the DAO has complete ownership and comprehensive control over Salty.IO.

Voting power is obtained by staking SALT for xSALT with every xSALT equaling one vote.

Holding a minimum amount of xSALT is required for making new governance proposals (amount adjustable by the DAO).  To provide ample time for a staking baseline to be established proposals are only possible 45 days after deployment.

The DAO has the following capabilities:

1. Token whitelisting and unwhitelisting
2. Establishing and removing regional exclusions by country if deemed necessary
3. Sending SALT to specified wallets (up to 5% of DAO balance)
4. Calling external contracts (via an explicit callFromDAO method)
6. Updating the AccessManager and PriceFeed contracts
7. Updating the active IPFS link for the website
8. Making extensive parameter changes to the protocol 

![image](https://github.com/code-423n4/2024-01-salty/assets/104318932/3f7eb16a-3240-4c8e-8c98-0477f33433e1)

##### Note: Please click on the image to enlarge it:



| Number | DAO feature                        | Details                                                                                                                  |
|------------------|----------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| 1                | Proposal Creation and Uniqueness       | Salty.IO contract allows the creation of various proposal types and ensures each proposal is unique, preventing duplicates.  |
| 2                | Voting Power and Stake Requirements    | Proposals require a minimum amount of staked xSALT tokens, limiting proposal creation to significant stakeholders.         |
| 3                | Time Restrictions                      | A time restriction of 45 days post-deployment is set for proposal creation, allowing the staking ecosystem to stabilize.    |
| 4                | Proposal Confirmation                  | The DAO can create confirmation proposals, adding an extra layer of security and oversight.                                 |
| 5                | Quorum Requirements                    | Different quorum requirements are set for various ballot types, ensuring decisions have substantial community backing.     |
| 6                | Vote Casting and Recasting             | Users can cast and change their votes, with the system tracking the last vote cast for transparency.                        |
| 7                | Finalization of Ballots                | Only the DAO can finalize ballots, adding an additional oversight layer.                                                    |
| 8                | Limitations on Proposal Types          | Specific restrictions are placed on certain proposal types, such as limits on SALT amounts and token unwhitelisting.        |


### Recommendations for SALTY DAO :

1. **Implement a Proposal Review Process**: If not already in place, a formal review process for proposals before voting could help mitigate the risk of malicious proposals.

2. **Monitoring and Adaptation**: Regularly monitor the governance process and be ready to adapt it in response to emerging threats and governance attacks.

3. **Community Education**: Educate the community about the governance process to encourage informed participation and vigilance against potential threats.

</br>
</br>

### DAO - Proposals.sol :

Proposal.sol allows creating and voting on different types of voting proposals. Here are the main features of the contract:

1. **Governance Authority:** The agreement allows SALT token holders to submit and vote on various proposals.

2. **Creating Recommendations:** Users can create various types of recommendations. Recommendations can be in different categories such as parameter changes, token whitelisting/removal, token submission, contract invocation, and website URL updates.

3. **Bid Tracking:** The contract ensures uniqueness of bids and prevents duplication of bids with the same name by checking the name of existing bids.

4. **Voting and Quorum:** Users can vote on proposals and controls the minimum participation (quorum) requirements for each proposal.

5. **Change Vote:** Users can withdraw their previous votes and cast a new vote before voting.

6. **Proposal Approval:** Once Quorum conditions are met and votes are counted, proposals can be approved and become implemented.

7. **Users' Active Offers:** Tracks which users have active offers and ensures that each user has only one active offer at a time.

9. **Bid Statistics:** Provides statistics showing the status of bids and which bids received the most votes.

10. **Limitation on First Offer Date:** Determines the date on which the first offer can be made. This allows users to stake tokens for a period of time before applying.

![image](https://github.com/code-423n4/2024-01-salty/assets/104318932/b53c8e73-0801-4b4b-b11f-2c420622898a)

</br>
</br>

### DAO - Proposals.sol :

<img width="1676" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/0a9bb5ca-624d-4187-8838-3d32c8a8757d">

##### Note: Please click on the image to enlarge it:






</br>
</br>
</br>
</br>

<img width="220" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/04a89376-163d-411e-9ce3-12f7689b93f9">

### Upkeep.sol :

The `performUpkeep()` function is used to perform automatic maintenance and optimization operations in the project from within the `Upkeep.sol` contract. This function automatically executes various financial transactions such as providing liquidity, withdrawing arbitrage profits, and SALT token distribution. This process increases the efficiency and stability of the system and ensures the healthy functioning of the token economy.


<img width="1827" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/00385c1b-be37-4e8a-9b7d-1afad7b3b27b">


</br>
</br>
</br>
</br>


## c) Test analysis


### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.

The testing approach in the SaltyIO project demonstrates a comprehensive and well-rounded strategy. It covers a range of aspects from multi-contract integration and environmental adaptability to detailed scenario testing and gas optimization. This thoroughness is indicative of a robust and reliable contract system, crucial for a project operating in the dynamic and often unpredictable environment of blockchain and smart contracts.




### What could they have done better?





-  1) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


<img width="782" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/5641ab50-569f-477d-b4d3-601df63c3a20">

[Ref Approche Analysis](https://xin-xia.github.io/publication/icse194.pdf)

</br>
</br>

-   2) Overall line coverage percentage provided by your tests : 99 (As a generally accepted safety rule, 100% test coverage is recommended)


</br>
</br>



-  3) Test Tools/Technologies analysis of the project;

<img width="986" alt="image" src="https://github.com/code-423n4/2024-01-salty/assets/104318932/25945135-5974-439c-8c98-d39625df27d8">

</br>
</br>

-  4) Test suites do not test for attack vectors, especially re-entrancy

Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability


</br>
</br>
</br>
</br>

## d) Security Approach of the Project

### Successful current security understanding of the project;

1 - First they did the main audit from a reputable auditing organization like Trail of Bits  and ABDK resolved all the security concerns in the report
2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.



### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/


5- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

6- ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise

9- We also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

10 - As the project team, you can consider applying the multi-stage audit model.

<img width="498" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/7ad49e46-4998-4bf2-830e-711039ea97a8">

Read more about the MPA model;
https://mpa.solodit.xyz/

11 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19


12 -Another security approach I can recommend is 0xDjango's security approaches in a project. This approach divides security into layers (Test, Automatic Tool, Individual Audit, Corporate Audit, Economic Audit) and aims to have each part done separately by experts in the field. I can also recommend this approach to you;

https://x.com/0xDjangoOnChain/status/1748402771684909094?s=20

</br>
</br>

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-01-salty/blob/main/bot-report.md

**Audit - ABDK**
https://github.com/abdk-consulting/audits/tree/main/othernet_global_pte_ltd

>A summary of the key points from the Audit from ABDK Audit document:
The ABDK audit report for Salty.IO categorizes the issues into major, moderate, and minor, with many of them marked as "fixed." 
This indicates that Salty.IO has taken steps to rectify numerous vulnerabilities and weaknesses identified during the audit. 
However, the detailed nature of the vulnerabilities, their potential impact, and the specific resolutions are not included in this summary. This level of detail is essential for a comprehensive understanding of the audit's findings and Salty.IO's security posture.


**Audit - Trail of Bits**
https://github.com/trailofbits/publications/blob/master/reviews/2023-10-saltyio-securityreview.pdf


>A summary of the key points from the Audit from Trail of Bits Audit document:
The security assessment of Salty.IO by Trail of Bits identified several high-severity issues acrossed various components of the protocol. 
Key concerns included risks of denial-of-service attacks on the token whitelisting process, front-running vulnerabilities in liquidity transactions, and the possibility of the USDS stablecoin becoming undercollateralized. 
The assessment also pointed out insufficient event generation for monitoring, the potential for stale price data from Chainlink oracles, and the manipulation of the liquidation fee. 
 The audit emphasized the need for comprehensive testing, documentation improvement, and system resilience, especially in the face of significant market anomalies.



</br>
</br>
</br>
</br>

## Centralization Risk

#### Owner Role:

Centrality Risk High: The Owner role is highly centralized since it is singular and possesses significant administrative powers. 

Single Point of Failure: If the Owner's credentials are compromised, the entire system could be at risk. Similarly, if the Owner acts maliciously or negligently, it could have a substantial negative impact on the protocol.



</br>
</br>
</br>
</br>


##  f) Packages and Dependencies Analysis 📦


| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)                         | A lot of OZ contracts |-  Version `4.9.3` is used by the project, it is recommended to use the newest version `5.0.1`|     
| [`v3-core`](https://www.npmjs.com/package/@uniswap/v3-core)         | [![npm](https://img.shields.io/npm/v/@uniswap/v3-core)](https://www.npmjs.com/package/@uniswap/v3-core)                         | FixedPoint96.sol, IUniswapV3Pool.sol, FullMath.sol, TickMath.sol |-  Version `1.0.2-solc-0.8-simulate` is used by the project, it is recommended to use the newest version `1.0.1`|        
| [`chainlink`](https://www.npmjs.com/package/@chainlink/contracts)         | [![npm](https://img.shields.io/npm/v/@chainlink/contracts)](https://www.npmjs.com/package/@chainlink/contracts)    |AggregatorV3Interface.sol |-  The latest updated version is used|  




</br>
</br>



</br>
</br>




## g) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ All staff accounts for the project should have control policies that require 2FA and must use 2FA wherever possible.
100% comprehensive security cannot be achieved based on smart contract codes alone.
Implement a more comprehensive policy to enforce non-SMS 2FA.
You can find the latest Simswap attack on Code4rena and details about it in this article: 
https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d


✅ I recommend you to set up a `system.sol` basic architecture where all contracts are registered.
The entire system can revolve around a single contract, like SystemRegistry. This is the contract that ties all the other contracts together, and from this contract we should be able to list all the other contracts in the system. It's almost like a registry. 



✅ Analyze the gas usage of each function under various conditions in tests to ensure efficiency and identify potential optimizations.	





✅  I recommend that you classify functions such as Public, External, Internal as follows, this is the most effective method to classify functions ; 
<img width="467" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/f6661c98-7595-4ae2-bc5b-b086dc4f3018">

https://x.com/PaulRBerg/status/1657098652987318292?s=20

</br>
</br>


✅ This listing makes imports more understandable;


```diff
Pools.sol

- import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
- import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
- import "openzeppelin-contracts/contracts/utils/math/Math.sol";
- import "openzeppelin-contracts/contracts/access/Ownable.sol";
- import "../interfaces/IExchangeConfig.sol";
- import "../arbitrage/ArbitrageSearch.sol";
- import "./interfaces/IPoolsConfig.sol";
- import "./interfaces/IPools.sol";
- import "./PoolStats.sol";
- import "./PoolMath.sol";
- import "./PoolUtils.sol";


+ // Security
+import {ReentrancyGuard} from "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";

+// Tokens
+import {IERC20} from "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";

+// Utilities
+import {Math} from "openzeppelin-contracts/contracts/utils/math/Math.sol";

+// Access Control
+import {Ownable} from "openzeppelin-contracts/contracts/access/Ownable.sol";

+// Interfaces for Exchange Configuration
+import {IExchangeConfig} from "../interfaces/IExchangeConfig.sol";

+// Arbitrage
+import {ArbitrageSearch} from "../arbitrage/ArbitrageSearch.sol";

+// Interfaces for Pools
+import {IPoolsConfig} from "./interfaces/IPoolsConfig.sol";
+import {IPools} from "./interfaces/IPools.sol";

+// Pool Operations
+import {PoolStats} from "./PoolStats.sol";
+import {PoolMath} from "./PoolMath.sol";
+import {PoolUtils} from "./PoolUtils.sol";




```



## h) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size


When the project is analyzed in terms of Gas Optimization, there is a very important gas optimization; "Using Mapping instead of Openzeppelin's EnumerableSet library provides high gas optimization"

```solidity
4 results - 4 files

src/dao/Proposals.sol:
  27  	using SafeERC20 for IERC20;
  28:     using EnumerableSet for EnumerableSet.UintSet;
  29  

src/launch/Airdrop.sol:
  15      {
  16:     using EnumerableSet for EnumerableSet.AddressSet;
  17  

src/pools/PoolsConfig.sol:
  24  
  25:     using EnumerableSet for EnumerableSet.Bytes32Set;
  26  

src/stable/CollateralAndLiquidity.sol:
  31  	using SafeERC20 for IUSDS;
  32:     using EnumerableSet for EnumerableSet.AddressSet;
  33  


```

The use of EnumerableSet takes place in the following steps; 1- Define Openzeppelin library with import 2- With using EnumerableSet for EnumerableSet.AddressSet; it can be used directly without specifying the library name with its definition. 3- In EnumerableSet.AddressSet private s_priceUpdaters; a private s_priceUpdaters type variable is declared using the EnumerableSet library



using Mapping in this project seems more efficient

Because while EnumerableSet provides enumeration capabilities, iterating over a large set consumes a significant amount of gas, especially if you need to perform operations on each item, which is exactly what happens in this project. It's more efficient to use a mapping if accessing them by key has a fixed gas cost

Gas optimization could not be measured due to the complete change of the architecture, but it is obvious that it will provide a significant gas gain in the context of use cases.


### Time spent:
32 hours