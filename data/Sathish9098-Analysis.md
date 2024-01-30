# Salty.IO Analysis

## Overview
Salty.IO is a Decentralized Exchange (DEX) built on the Ethereum blockchain, featuring an innovative Automatic Atomic Arbitrage (AAA) mechanism. This mechanism seeks to capitalize on market inefficiencies during token swaps, generating profits which are then distributed among liquidity providers and stakers. Salty.IO also introduces USDS, an overcollateralized ERC20 stablecoin, backed by WBTC/WETH LP tokens. The platform is wholly decentralized from inception, with governance, parameters, and contract operations controlled by a Decentralized Autonomous Organization (DAO). Salty.IO emphasizes zero fees on swaps, enhancing its attractiveness to users. It uses a mix of Chainlink, Uniswap v3 TWAP, and its own reserve data to maintain accurate price feeds, ensuring robustness and resilience against market manipulation.
  
## Systemic risks
  
### Oracle Reliability and Manipulation

#### Chainlink Oracle Risks (CoreChainlinkFeed Contract):

``Data Source Centralization``: Chainlink, while decentralized, still relies on a set of node operators to provide data. If these nodes are compromised or colluded, it might lead to false data being fed into the system.

 ``Time Lag in Data Updates``: Prices provided by Chainlink are not real-time; they have a certain latency. Rapid market movements might not be reflected immediately in the oracle data, leading to temporal arbitrage opportunities.

  ``Smart Contract Integration Risk``: The integration of Chainlink oracles within the CoreChainlinkFeed contract requires careful handling of data retrieval and error checking. Mismanagement here could lead to incorrect data being accepted as valid.
  
#### Uniswap v3 TWAP Oracle Risks (CoreUniswapFeed Contract):

``Manipulation via Pool``: TWAP (Time-Weighted Average Price) is derived from a Uniswap v3 pool. If a pool's liquidity is low or if there are large trades, it could temporarily skew the price, affecting the TWAP.

 ``Vulnerability to Flash Loans``: Uniswap TWAPs can be susceptible to manipulation using flash loans. An attacker could use a large loan to make substantial trades that alter the price in a pool, impacting the TWAP.

 ``Dependency on External Protocol Stability``: The integrity of the TWAP data is dependent on Uniswap v3's smart contract security and operational stability. Any vulnerabilities in Uniswap v3 could indirectly impact Salty.IO.
  
#### Using Own Reserves for Price Feeds:

``Internal Market Depth``: Using its own reserves, Salty.IO calculates internal prices. However, this is highly dependent on the depth and liquidity of its own markets. Thin or imbalanced markets can lead to inaccurate pricing.

 ``Arbitrage and Slippage`` : Salty.IO’s reliance on its own liquidity pools to derive pricing information may lead to prices that are out of sync with the broader market, creating arbitrage opportunities. This is particularly risky in the context of automated arbitrage systems. 
   
 ### Governance Risks
  Being a DAO-controlled platform, Salty.IO's parameters and policies are subject to changes by DAO proposals and votes, as seen in the DAO contract. This opens up risks related to governance attacks, where malicious actors might gain control over the governance process, influencing decisions in their favor.
   
  ### Stablecoin (USDS) Overcollateralization Mechanism
   
  The system's stablecoin, USDS, is overcollateralized by WBTC/WETH LP tokens. The Stable contract managing this could be at risk if there are sharp, adverse market movements, potentially leading to undercollateralization.
   
  #### Market Volatility Risks

- WBTC (Wrapped Bitcoin) and WETH (Wrapped Ethereum) are subject to the volatility of the cryptocurrency market. Sharp declines in the value of BTC and ETH can drastically reduce the value of the collateral, leading to undercollateralization.
- In extreme market conditions, the collateral might not be sufficient to maintain the peg of USDS, resulting in a loss of confidence and potential depegging.
   
#### Liquidity Pool (LP) Token Fluctuations:

- As the collateral consists of LP tokens from a WBTC/WETH pool, it's also exposed to risks associated with liquidity pools, such as impermanent loss. This occurs when the price of the pooled assets changes after depositing them, potentially leading to a loss compared to holding the assets outside the pool.
- The liquidity of the pool itself is a factor. In times of market stress, if the pool's liquidity is low, it might be challenging to liquidate enough collateral to maintain the stablecoin’s value.
  
 #### Mechanism for Handling Undercollateralization:

- The protocol must have a robust mechanism to handle scenarios where the collateral falls below the required threshold. This could include automatic liquidation of collateral, raising additional collateral requirements, or other stabilization measures.
- The effectiveness of these measures in a fast-moving market is critical to maintain the peg and confidence in USDS.
  
#### Governance and Parameter Adjustments:

The parameters governing overcollateralization, such as collateralization ratios and acceptable collateral types, are subject to DAO governance. Rapid or ill-informed changes to these parameters could inadvertently increase risk.
  
### Economic Incentive Misalignment
The Rewards and Emissions contracts provide incentives for various stakeholders. If these incentives are not properly aligned, it could lead to unfavorable behaviors like short-term profit maximization at the expense of long-term stability.
  
#### Short-Term Profit Maximization
If the rewards are too high or easily exploitable, users might engage in behaviors that maximize their short-term gains. For example, they might frequently shift assets or manipulate the system to farm rewards, potentially destabilizing the platform.
Excessive reward chasing can lead to a lack of commitment to the platform, with users quickly moving to other platforms as soon as more attractive opportunities appear.

#### Unsustainable Emission Rates
The Emissions contract determines the rate at which SALT tokens are distributed. If this rate is unsustainable (too high or too low), it can lead to inflationary pressures (devaluing the token) or reduced participation (if rewards are too low).
A poorly calibrated emission rate might not adequately compensate for the risks users take, leading to lower liquidity and participation in the long run.

#### Imbalance Between Different Stakeholder Groups
Incentives should be balanced between different groups such as liquidity providers, SALT stakers, and governance participants. If one group is disproportionately rewarded, it can lead to an imbalance, where certain platform functions are over-prioritized while others are neglected.
For instance, if liquidity providers are excessively rewarded compared to stakers, it could lead to an overemphasis on liquidity provision activities at the expense of the platform's governance and long-term development.

#### Gaming the System:
If the reward mechanism is not designed with safeguards against manipulation, users might find ways to game the system. This can include activities like sybil attacks, where a user creates multiple accounts to farm rewards, or circular trading to generate artificial volume and earn more rewards.
System gaming not only leads to unfair distribution of rewards but also can skew the metrics and data, leading to poor decision-making and policy adjustments.

#### Long-Term Viability and Token Value:
The value of the platform’s native token (SALT in this case) is closely tied to the perceived sustainability and effectiveness of the reward system. If the tokenomics are not sustainable, it can lead to a decline in token value.
Long-term viability requires a careful balance where the emission of new tokens and rewards aligns with the platform's growth, usage, and value creation.
  
## Technical risks
  
### Liquidity Pool and Arbitrage Mechanisms
The Pools contract and associated liquidity mechanisms are central to Salty.IO. Inaccurate management of pool reserves, flawed arbitrage logic, or miscalculations in reward distributions could lead to financial losses or imbalances in liquidity.
  
#### Management of Pool Reserves
The integrity of the Pools contract is vital. Any inaccuracies in the management of pool reserves could impact the overall liquidity available on the platform. For example, errors in reserve calculations or updates post-transactions (swaps, deposits, withdrawals) could distort the market dynamics.
These distortions can lead to slippage, price inaccuracies, or even provide opportunities for arbitrage that could drain the reserves if not managed correctly.

#### Arbitrage Logic and Execution

Arbitrage is critical for aligning prices with the broader market and ensuring no single pool is significantly out of sync. Flaws in the arbitrage logic, such as incorrect calculation of arbitrage opportunities or faulty execution (e.g., timing delays), can lead to missed opportunities or exploitation by savvy traders.
The interplay between Salty.IO’s internal pools and external market prices must be constantly monitored. Any lag or discrepancy can result in adverse market manipulation.

#### Reward Distribution Mechanisms

The calculation and distribution of rewards to liquidity providers must be accurate and timely. If the reward mechanism overpays, underpays, or fails to distribute rewards, it can lead to a loss of trust and a decrease in liquidity provision.
Mismanagement of rewards could also impact the tokenomics of Salty.IO, potentially leading to inflationary pressures or devaluation of rewards.

#### Liquidity Incentives Alignment:
Incentives for liquidity providers need to be aligned with the long-term health of the platform. Short-term, unsustainable incentives could lead to rapid changes in liquidity, affecting the platform's stability.
  
### Reentrancy Attacks
The use of nonReentrant modifier in contracts like Pools and Staking suggests awareness of reentrancy risks. However, any lapse in implementing these checks across all functions that transfer funds or alter contract states could lead to reentrancy attacks.
  
### Emission Rates and Reward Calculations
The Emissions contract controls the distribution of rewards. Any miscalculations or oversights in the emission logic could lead to rapid inflation or depletion of rewards, affecting the token economy.

#### Miscalculation of Emission Rates
The emission rate determines how quickly rewards are released into the ecosystem. If this rate is set too high, it could lead to rapid inflation of the SALT token, diminishing its value. Conversely, too low a rate might not provide adequate incentives for participation.
Calculations for emissions must account for the current supply, expected platform growth, and overall tokenomics. Errors in these calculations can disrupt the balance between incentivizing users and maintaining token value.

#### Automated Emission Adjustments:
Salty.IO's emissions strategy might involve automated adjustments based on certain triggers or market conditions. Any flaws in the logic that governs these adjustments could lead to unintended consequences, such as abrupt changes in emission rates that destabilize the ecosystem.

#### Integration with Market Conditions
The Emissions contract should ideally be responsive to broader market conditions. Failure to integrate real-time market data or lag in response times can lead to a misalignment between the platform's needs and the emissions strategy.

#### Reward Distribution Logic
The mechanism for distributing rewards must be equitable and transparent. Any errors in how rewards are calculated, apportioned, or distributed can lead to dissatisfaction among participants and might incentivize gaming the system.

#### Long-term Sustainability
The emissions strategy must be sustainable in the long term. Short-sighted emission policies that don’t consider the long-term viability can lead to token devaluation and reduced platform engagement.

#### Impact on Liquidity Providers and Staker
Changes in emission rates directly impact liquidity providers and stakers. Sudden changes can cause significant shifts in user behavior, such as mass unstaking or withdrawal of liquidity, affecting the platform's stability.

#### Token Burn Mechanisms
If the platform includes token burn mechanisms to control inflation, the interplay between emission rates and burning rates needs careful calibration. Misalignment here could either lead to token scarcity or oversupply.
  
### Overcollateralization and Stablecoin Peg:
The mechanism to maintain the peg of USDS and its overcollateralization can be complex. The risk lies in rapid market movements that could destabilize the peg, especially if the liquidation mechanisms or collateral assessments are not responsive enough.

## Integration risks
  
### Protocol Dependency and Cascading Failures
Salty.IO's functionality may rely on other DeFi protocols for liquidity pools, oracle services, or other features. If one of these integrated protocols faces an issue like a smart contract bug, liquidity crisis, or governance attack, it can cascade into Salty.IO, potentially disrupting its operations or causing financial losses.

### Interface and API Risks
Interfacing with external protocols through APIs or smart contract calls carries risks. Incompatibility issues, such as changes in external contract interfaces or API endpoints, can break integrations, requiring immediate attention and updates to maintain operational continuity.

### Liquidity Pool Integration Challenges
Salty.IO’s interaction with external liquidity pools (e.g., for arbitrage opportunities) can be problematic if these pools face liquidity shortages, high slippage, or impermanent loss. These external market conditions can affect Salty.IO's internal algorithms for pricing, arbitrage, or rewards.

### Cross-Protocol Oracle Dependency
Salty.IO might utilize price feeds or other data from external protocols. If these protocols suffer oracle manipulation or provide incorrect data, Salty.IO’s internal mechanisms (like pricing or liquidations) might execute on false premises, leading to erroneous transactions.

### Smart Contract Interaction Limitations
Given the immutable nature of smart contracts, Salty.IO's ability to adapt to changes in integrated protocols might be limited. For instance, if an integrated protocol upgrades its contracts or changes its operational model, Salty.IO might not be able to promptly align due to the rigidity of its own smart contracts.

### Inter-Protocol Transaction Ordering Risks
In DeFi ecosystems, the order of transactions can be crucial. Salty.IO's dependency on transaction sequencing in integrated protocols (like DEXes or lending platforms) can pose risks. Front-running or transaction ordering manipulation in these protocols can adversely affect Salty.IO's operations.

### Decentralized Governance Complications
If Salty.IO integrates with protocols governed by DAOs, changes in their governance models or policy decisions can indirectly impact Salty.IO. For instance, a change in fees, rewards, or operational parameters in an integrated protocol might necessitate adjustments in Salty.IO's strategies or configurations.
  
## Admin abuse risks
Analyzing the Salty.IO smart contracts, a notable single point of failure can be observed in the extensive use of the ``onlyOwner`` and ``onlySameContract`` modifier. This modifier restricts certain critical functions to be accessible only by the owner account, which can be a significant risk if the owner's private key is compromised or if the owner engages in malicious activities.
  
## onlyOwner 
  
```solidity
function changeBootstrappingRewards(bool increase) external onlyOwner
function changePercentPolRewardsBurned(bool increase) external onlyOwner
function changeBaseBallotQuorumPercent(bool increase) external onlyOwner
function changeBallotDuration(bool increase) external onlyOwner
function changeRequiredProposalPercentStake(bool increase) external onlyOwner
function changeMaxPendingTokensForWhitelisting(bool increase) external onlyOwner
function changeArbitrageProfitsPercentPOL(bool increase) external onlyOwner
function changeUpkeepRewardPercent(bool increase) external onlyOwner
function whitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
function unwhitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
function changeMaximumWhitelistedPools(bool increase) external onlyOwner
function changeMaximumInternalSwapPercentTimes1000(bool increase) external onlyOwner
function setContracts( IDAO _dao, ICollateralAndLiquidity _collateralAndLiquidity ) external onlyOwner
function setInitialFeeds( IPriceFeed _priceFeed1, IPriceFeed _priceFeed2, IPriceFeed _priceFeed3 ) public onlyOwner
function setPriceFeed( uint256 priceFeedNum, IPriceFeed newPriceFeed ) public onlyOwner
function changeMaximumPriceFeedPercentDifferenceTimes1000(bool increase) public onlyOwner
function changePriceFeedModificationCooldown(bool increase) public onlyOwner
function changeRewardsEmitterDailyPercent(bool increase) external onlyOwner
function changeEmissionsWeeklyPercent(bool increase) external onlyOwner
function changeStakingRewardsPercent(bool increase) external onlyOwner
function changePercentRewardsSaltUSDS(bool increase) external onlyOwner
function setCollateralAndLiquidity( ICollateralAndLiquidity _collateralAndLiquidity ) external onlyOwner
function changeRewardPercentForCallingLiquidation(bool increase) external onlyOwner
function changeMaxRewardValueForCallingLiquidation(bool increase) external onlyOwner
function changeMinimumCollateralValueForBorrowing(bool increase) external onlyOwner
function changeInitialCollateralRatioPercent(bool increase) external onlyOwner
function changeMinimumCollateralRatioPercent(bool increase) external onlyOwner
function changePercentArbitrageProfitsForStablePOL(bool increase) external onlyOwner
function setContracts( ICollateralAndLiquidity _collateralAndLiquidity, IPools _pools, IDAO _dao) external onlyOwner
function changeMinUnstakeWeeks(bool increase) external onlyOwner
function changeMaxUnstakeWeeks(bool increase) external onlyOwner
function changeMinUnstakePercent(bool increase) external onlyOwner
function changeModificationCooldown(bool increase) external onlyOwner

```
  
### Centralized Control Over Key Parameters
Functions like changeBootstrappingRewards, changePercentPolRewardsBurned, changeBaseBallotQuorumPercent, and others are governed by onlyOwner. This means the owner has unilateral control over crucial aspects like rewards, quorum requirements for DAO proposals, and ballot durations. Such control can be misused to destabilize the system.

### Governance and Whitelisting Power
The owner's authority to whitelist (whitelistPool) or unwhitelist (unwhitelistPool) pools puts a lot of power into a single account. This function impacts the liquidity and the overall health of the ecosystem. The owner can theoretically manipulate market dynamics by favoring certain pools.

### Security Risk of Owner Account
If the owner's private key is compromised, the attacker can alter critical system parameters, leading to potential loss of funds or trust in the system. This includes adjusting the maximumInternalSwapPercentTimes1000 or maximumPriceFeedPercentDifferenceTimes1000, which directly influence trade mechanics and oracle reliability.

### Smart Contract Upgrade Process
The methods like step1 to step11 under onlySameContract modifier indicate a multi-step contract upgrade process. If the owner account is compromised during this process, it could lead to incorrect upgrades, potentially introducing vulnerabilities or locking out the legitimate administrative access.

### Dependency on Owner for Routine Operations
Functions like setContracts and setInitialFeeds show that the owner is responsible for setting up fundamental components of the system. Continuous reliance on a single entity for such critical operations poses risks of centralization and operational delays.

### Risk in Reward and Staking Configurations
The owner's ability to alter staking configurations (changeMinUnstakeWeeks, changeMaxUnstakeWeeks) and reward strategies (changeRewardPercentForCallingLiquidation) can impact the incentive structures. Mismanagement here can lead to loss of user confidence and liquidity.

### Lack of Decentralized Checks and Balances
Ideally, in a decentralized system, critical decisions should go through a multi-signature process or a DAO vote to distribute power among several stakeholders. The onlyOwner approach centralizes decision-making, which is contrary to the ethos of decentralization in DeFi.
  
## onlySameContract
  
```solidity
function step1() public onlySameContract
function step2(address receiver) public onlySameContract
function step3() public onlySameContract
function step4() public onlySameContract
function step5() public onlySameContract
function step6() public onlySameContract
function step7() public onlySameContract
function step8() public onlySameContract
function step9() public onlySameContract
function step10() public onlySameContract
function step11() public onlySameContract

```
Functions marked with onlySameContract, such as step1 to step11, seem to be part of a multi-step process, possibly for contract upgrades or intricate inter-contract workflows. This modifier might be ensuring that these functions are only callable by certain contracts within the Salty.IO system, perhaps for synchronization or staged execution purposes. 
  
## Testing suite Analysis
  
```
the overall line coverage percentage provided by your tests?: 99

```
  
Test coverage, particularly in the context of smart contracts like those in Salty.IO, is a metric used to measure the extent to which the smart contract's code is executed when the test suite runs. The "99" line coverage percentage indicates that 99% of the lines of code in the smart contracts are executed during testing.
  
### Meaning of 99% Line Coverage:
This implies that almost all lines of the smart contract code are tested by the test suite. Out of every 100 lines of code, 99 lines are executed in various test scenarios. This high percentage suggests a thorough testing process, aiming to uncover issues in almost every part of the code.
  
### Limitations of Coverage Metrics:
It's important to note that even 99% coverage doesn't guarantee the absence of bugs or vulnerabilities. Certain aspects, like complex interactions between contracts or unexpected user behaviors, might not be fully captured in tests.
  
## Software engineering considerations
  
### Refinement of Governance Functions
  
- ``Decentralization of onlyOwner Modifiers``: Analyze functions with onlyOwner modifiers (e.g., changeBootstrappingRewards, changeArbitrageProfitsPercentPOL, setContracts) to potentially decentralize control, reducing single points of failure and enhancing DAO governance.

- ``Improve Governance Contract Robustness``: Strengthen the DAO contract to mitigate governance risks, possibly by introducing time-locks for critical parameter changes or multi-signature requirements for sensitive operations.
  
### Optimization of Reward Mechanisms
- ``Emission Rate Adjustments``: In the Emissions contract, consider implementing adaptive emission rates based on network activity or token economics to maintain balance between incentivization and token value.

- ``Enhanced Reward Distribution Algorithms``: Analyze the StakingRewards contract for potential optimizations in reward calculations to improve gas efficiency and fairness in distribution.

  ### Enhancing Oracle Reliability
  
- ``Diversification of Oracle Sources``: In PriceAggregator, further diversify oracle sources or introduce additional fail-safes to mitigate risks associated with oracle failure or manipulation.

- ``Implement Oracle Failover Mechanisms``: Develop a system for detecting faulty oracle data and automatically switching to backup data sources or methodologies.

### Liquidity Pool Management
- ``Automated Pool Reserve Balancing``: In the Pools contract, explore algorithms for dynamic pool reserve balancing to optimize liquidity efficiency and mitigate risks from market volatility.
- ``Improve Arbitrage Logic``: Conduct a thorough review of the arbitrage mechanism to ensure it aligns with market dynamics and effectively mitigates risks related to sandwich attacks or price manipulation.
### Integration and Interoperability
- ``Strengthen Cross-Protocol Interaction Security``: Evaluate and fortify the security of cross-protocol interactions, especially where Salty.IO integrates with external DeFi platforms or liquidity sources.
- ``Compliance with Standards``: Ensure smart contracts, especially those dealing with tokens and liquidity (e.g., ISalt, IPoolsConfig), are compliant with industry standards (e.g., ERC-20) for broader compatibility and integration.
### Testing and Quality Assurance
- ``Expand Test Coverage``: Although the test coverage is high (99%), focus on edge cases and stress testing, especially for critical paths like Emissions and Staking.
- ``Simulation of Extreme Market Conditions``: Simulate platform behavior under extreme market conditions to test the resilience of mechanisms like overcollateralization and arbitrage.
### Documentation and Knowledge Sharing
- ``Update and Expand Documentation``: Regularly update the documentation to reflect changes in the codebase and provide detailed explanations of complex mechanisms.
- ``Developer Guides and API Documentation`` : Offer comprehensive guides for developers and thorough API documentation to encourage community development and integration.
 
## In-depth architecture assessment of business logic
To provide an in-depth analysis of the architecture of Salty.IO, we'll explore each major component of the decentralized exchange (DEX) platform and its associated features as described earlier. This analysis will help you understand how the various parts of Salty.IO work together to create a decentralized trading ecosystem.

### arbitrage

This module handles the detection and execution of arbitrage opportunities. It's a crucial component for generating profits and maintaining efficient markets.
It's important to analyze the algorithms and mechanisms used to identify arbitrage opportunities and ensure they are accurate and efficient.
Consider the gas costs associated with arbitrage and whether they are manageable for users.

### ArbitrageSearch.sol

#### Architecture
  
![9W~G7O@L6$ZEG}C4E68TCRJ](https://gist.github.com/assets/58845085/4ec16692-2088-4466-ab37-81659cc8b6dc)

#### Sequence Flow
![A$~CR5{DH 5A{@X56WWTEVX](https://gist.github.com/assets/58845085/bceeb391-4b99-43e7-ab14-804220c5cca4)

### dao

The DAO module manages the governance aspects of Salty.IO, including proposal creation, voting, and execution.
Analyze the governance model to ensure that it is secure, resistant to manipulation, and provides a fair mechanism for decision-making.
Examine how DAO parameters are stored and adjusted and how they impact various parts of the platform.

### DAO.sol

![@9MAEA7I}H1)} KXQ`Z$}(X](https://gist.github.com/assets/58845085/338501fa-ac70-4008-99c8-256b45f7046f)


![UVE87{7N8KH3YUVN%KMM _A](https://gist.github.com/assets/58845085/247c47f3-b0a4-44ef-8468-bf522a4a3c45)


### pools

The pools module is a core part of the DEX and manages liquidity pools, swaps, arbitrage, and user token deposits.
Analyze the algorithms used for liquidity provision, swaps, and arbitrage to ensure they are efficient and secure.
Examine how user deposits are managed and how they impact gas costs and trade efficiency.
Consider how pools contribute to recent arbitrage trades and how rewards are distributed to liquidity providers.

### Pools.sol

![KRO{1$5%14PI3(8T RDDFOB](https://gist.github.com/assets/58845085/e0789130-8f77-498e-a2e3-d58c231706ee)


### staking

The staking module implements a rewards mechanism for users who stake SALT or deposit collateral and liquidity.
Understand the staking mechanisms and how user rewards are calculated and distributed.
Examine the role of "userShare" and its significance in the StakingRewards contracts.

### Staking.sol

![_TLL55~IP5YSXRE7SRU@LIY](https://gist.github.com/assets/58845085/f0c233a8-a26e-4b06-b28a-054dff770c18)

## Code Weak Spots
There are several areas that may be considered potential weak spots or areas of concern. It's essential to analyze and address these potential issues to ensure the security and reliability of the contract. Here are some of the potential weak spots in the code.

- ``Access Control``: Access to critical functions in the contract, such as stakeSALT and unstake, is currently controlled by checking if the sender has "exchange access." However, it's not entirely clear from the code what this access control mechanism entails. Depending on its implementation, it may be vulnerable to unauthorized access. Access control should be robust and well-documented.
- ``Unstake Timing``: The contract allows users to initiate unstaking of their tokens with a specified numWeeks. The calculation of claimable SALT is based on this duration. If there is an issue with the time calculation, it might lead to incorrect claimable amounts. 
- ``Array Length Limits``: The unstakesForUser function retrieves pending unstakes associated with a user using array indices. If the number of pending unstakes is extensive, it may lead to gas limitations and out-of-gas errors. You should consider adding pagination or limit the maximum number of unstakes returned in a single call.
- ``Cancellation of Unstakes``: Users can cancel their pending unstakes. While this is a useful feature, it introduces complexity. The contract should handle the cancellation of unstakes securely, ensuring that users cannot manipulate the system to their advantage.
- ``SALT Token Burn``: The contract burns SALT tokens when calculating expeditedUnstakeFee. Ensure that the token burning mechanism is correctly implemented and cannot be exploited.




### Time spent:
20 hours