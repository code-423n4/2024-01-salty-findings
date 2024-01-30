## Overview
Salty is a decentralized exchange with a unique value proposition of zero fees trading. Zero fees are made possible by an Automatic Atomic Arbitrage (AAA) mechanism whose generated profits are used to incentivise liquidity providers. In addition to the exchange, Salty also offers a dollar pegged over collateralized stable token (USDS) that uses the liquidity of the Salty WBTC/WETH pool as collateral. Salty also includes an exchange token (Salt) that is the main form of rewards on the platform, and whose staked token (XSalt) grants voting rights on the Salty DAO.

Salty is unique in the DeFi exchange landscape in its no-compromises full decentralization policy that applies from day one of the platforms launch. All settings and parameters underlying the protocol can only be changed through a DAO vote. There are no trusted admins or team safety switches. The only exception is an offchain external verifier in charge of authorizing exchange access based on a geolocation whitelist, however this function is also under the control of the DAO in that the DAO can vote to replace the AccessControl contract and with it the entire ACL mechanism. 

## Scope

#### ArbitrageSearch.sol
Implements an internal arbitrage search for Salty. For each pool trade, defines a fixed potential path (always with three legs that start and end with Weth) and runs an 8 step iterative binary search for the optimal input amount (if any) to maximize arbitrage profit on the path.
#### Dao.sol
Represents the Dao controlling Salty. Holds DAO funds and in charge with finalizing voted actions.
#### DaoConfig.sol
Holds values of core system parameters such as voting quorum, staking rewards etc and enables the DAO to change these parameters.
#### Proposals.sol
Manages Dao proposals/Ballots: enables opening ballots, voting on ballots and reporting if a ballot can be finalized or not 
#### Parameters.sol
Defines the different system parameters that can be tweaked by the dao and channels change requests to the right config contracts
#### Airdrop.sol
Implements the airdrop mechanism activated when the bootstrap ballot is accepted. Allows BootstrapBallot to authorize wallets that voted and are eligible for the airdrop, enables claiming when bootstrap ballot is accepted. Claiming users receive an equal share of XSalt of the airdrop allocation (5M Salt)
#### BootstrapBallot.sol
Manages the initial vote to launch the system/DAO
#### InitialDistribution.sol
Defines and performs the initial allocation of Salt then the system is launched.
#### PoolUtils.sol
Helper functions for the pools contract
#### PoolsConfig.sol
Parameters (configurable by the DAO) related to Salty pools
#### PoolStats.sol
Registry of arbitrage profits per pool for arbitrage rewards distribution
#### Pools.sol
Implements Salty exchange with pools that implement swaps with atomic internal arbitrage
#### PoolMath.sol
Helper functions for pools related math
#### CoreChainlinkFeed.sol
Feed implementation encaplusationg Chainlink price feeds (BTC/USD, ETH/USD) used by PriceAggregator
#### CoreUniswapFeed.sol
Feed implementation encaplusationg UniswapV3 TWAP prices of last 30 minutes (BTC/USD, ETH/USD) used by PriceAggregator
#### PriceAggregator.sol
PriceFeed implementation aggregating results from three sources (initially UniswapV3 TWAP, Chainlink and Salty but can be replaced by DAO). Used for collateral evaluation in USDS borrowing and liquidations
#### CoreSaltyFeed.sol
Feed implementation encapsulating Salty spot prices(BTC/USD, ETH/USD) used by PriceAggregator
#### RewardsConfig.sol
Parameters (configurable by the DAO) related to reward distributions
#### Emissions.sol
A contract that receives an initial amount of Salt at launch and releases the funds gradually over time (exponential decay pattern) to further emitters and eventually to staking and LP rewards contracts.
#### RewardsEmitter.sol
Emmits Salt rewards acording to a configurable schedule (exponential decay pattern) to a StakingRewards target (either CollateralAndLiquidity or Staking)
#### SaltRewards.sol
Receives emissions from Emissions.sol and distributed between the staking RewardsEmitter and the liquidity RewardsEmitter according to a configurable ratio parameter.
#### USDS.sol
A stable token that can be borrowed against Weth/WBTC pool liquidity. Initial collateralization ratio of 200% and liquidation (min) ratio of 110%
#### StableConfig.sol
Parameters (configurable by the DAO) related to the stable token USDS
#### CollateralAndLiquidity.sol
Derives from Liquidity, manages collateral balances for USDS borrowing and liquidity providing balances for Salty pools.
#### Liquidizer.sol
Manages liquidations of USDS CDPs 
#### StakingConfig.sol
Parameters (configurable by the DAO) related to the Salt staking mechanism in Salty
#### Liquidity.sol
Derives from StakingRewards (base of CollateralAndLiquidity). Implements the part related to pools liquidity providing.
#### StakingRewards.sol
Base contract used for both pools LP rewards and staking rewards. Registers user rewards shares and enables rewards claiming.
#### Staking.sol
Derives from StakingRewards, implements Salt staking and rewards distribution.
#### ManagedWallet.sol
A contract that represents a wallet that can be changed subject to the approval of a confirmation wallet. The main wallet can propose a change (of both itself and the confirmation wallet). The confirmation wallet can either accept the change (in which case the proposed new main wallet can finalize the change after a wait period of 30 days form acceptance) or reject the change.
#### AccessManager.sol
Grants wallet access to add liquidity/collateral and borrow USDS. Based on an offchain verifier that checks for whitelisted geo location.
Salt.sol    16  @openzeppelin/*
SigningTools.sol    20  -
Upkeep.sol  169 @openzeppelin/*
#### ExchangeConfig.sol
Serves as a registry of core protocol addresses such as tokens (salt, dai, wbtc, weth etc.) accessManager, DAO, upkeep, airdrop and teamWallet. DAO contrtolled. Of these, only the accessManager can be updated after the first initialization. 



## Approach Taken in evaluating the codebase

In analysing the codebase we took the following steps:

**1. Architecture Review**  
    Reading the provided documentation to understand the architecture and main functionality of Salty.

**2. Compiling code and running provided tests**

**3. Detailed Code Review**  
    A thorough code review of the contracts in scope.

**4. Security Analysis**  
    Defining the main attack surfaces of the codebase i.e. DAO voting manipulation, pool liquidity funds exfiltration, Rewards Distribution Attacks, external manipulation (front/back running trader and LPs), price manipulation of the Salty price feed etc.

**5. Additional Testing**  
    Augmenting the existing test scheme with new tests derived from the potential attack surfaces outlined in the security analysis conducted in the previous step.    

## Mechanism Review
Below is a description of the main mechanisms at the core of Salty's protocol:

### Lanuch and Rewards Distribution
Salty's launch is orchestrated using a set of contracts (Airdrop, BootsrapBallot and InitialDistribution) with full decentralization in mind. The process starts with a vote on weather or not to launch the protocol, the duration of which is set by the platform deployer. Voters are authorized by an offchain entity that provides them with a signature to be verified before voting (authorization is based on geolocation and having twitted about the airdrop). An authorized voter gets approved for an airdrop of Salt that takes place at launch. If the vote is successful, the system is kickstarted with an initial distribution of Salt that cascade to different actors through various scheduling and reward distribution mechanisms. The diagram below shows the flow of rewards in Salty at launch and during its ongoing operation.

![Salty Launch Flow](https://raw.githubusercontent.com/nirohgo/images/main/SaltyFlowsLaunch.jpg "Salty Launch Flow")


### Automatic Atomic Arbitrage
As mentioned above, Salty is able to offer zero-fee trade by capruting arbitrage opportunities atomically and using the gains to reward liquidity providers. Automatic Atomic Arbitrage (AAA) is implemented based on the following components:
A. New tokens on Salty can only be added to two predefined pools: NewToken/Weth and NewToken/WBTC. This, coupled with the core pools that are pre-added to Salty at the time of deployment enables the system to map a single arbitrage path for any trade, that start and ends with Weth.
B. As part of each trade transaction, the exchange attempts to find a profitable arbitrage on the mapped path by running an 8 step binary search for an optimal input amount that generates maximum profit. If such a trade exists it is carried out, and the profit is accredited to the pools involved in the path for the purpose of AAA reward distribution.

### DAO Voting
As mentioned, Salty imposes strict DAO control on all system configurable components. The DAO voting system is based on XSalt (staked Salt) voting power and enable several predefined types of proposals, namely parameter changes, token whitelisting and approval proposals (such as replacing price feed sources, adding/removing approved countries etc). The voting mechanism includes several mechanisms to prevent hasty/malicious changes: 
1. parameters can only be changed incrementally (in fixed steps) and between pre-set min/max values.
2. The default voting period is 10 days (though this can be changed by the DAO down to 3 days)
3. Some sensitive changes (such at contract replacements) require an additional vote (confirmation) once the original proposal is approved.
When a proposal quorum is reached and it's minimal duration elapses, anyone can call a finalize function which executes the proposal if the vote is successful, and removes it from the open ballot queue.

### Access Control
Access control to add liquidity and to stake Salt is restricted through an external offchain verifier that verifies an address meets the geo restrictions set in the DAO. Once the verification signature is validated the address is marked as whitelisted for future access. If the DAO decides to remove a previously approved country from the list, a new version of verification is formed and users need to re-verify in order to interact with the access-restricted actions.

### Stable Token and Liquidations
Salty's stable token (USDS) can be minted based on provided collateral in the form of WBTC/WETH pool liquidity. The collateral value is determined through the system's price feed (see next item) and affect the borrowing limit (requires 200% collateralization) and liquidations (enabled below 110% collateralization). When a user CDP can be liquidated anyone can call a liquidate function and receive a reward of 5% of the liquidated collateral value. The remaining collateral is sent to the Liquidizer contract to be converted to USDS, and the forgone amount of USDS is marked for burning. In each Upkeep cycle the liquidizer tries to burn all USDS marked for burning, and complements missing UDSD (due to undercollateralized liquidations) from the protocol owned liquidity.

### Price Feeds
To evaluate collateral for USDS borrowing and liquidations, Salty uses a price aggregator for BTC/USD, ETH/UDS that relies on three external sources (UniswapV3 TWAP, Chainlink and Salty's own pools). The price aggregator averages the two closest prices of the three and reverts if all three reported prices diverge from each other by more than a predefined max difference (3% by default). All three price sources can be replaced by the DAO.

### Upkeep
An upkeep transaction that can be called by anyone in the driving engine behind many of Salty's inner mechanics. It is in charge initiating liquidates USDS burns, distributing Weth AAA profits between POL, LPs, stakers and the function caller, distributing POL rewards, releasing scheduled rewards to LPs/stakers and more. Upkeep calls are incentivised by 5% of accumulated AAA Weth profits since the last call. 
![Salty Upkeep Flow](https://raw.githubusercontent.com/nirohgo/images/main/SaltyFlowUpkeep.jpg "Salty Upkeep Flow")

## Systemic Risks

Salty's system architecture facilitates many of Salty's features and unique advantages. However, as with any DeFi protocol, it also entails several risks:
 

### Slow response time to sudden market changes or security threats
As mentioned above, Salty's voting system takes a cautious approach with mechanisms such as long minimal ballot periods, incremental and limited parameter changes, confirmation ballots in some cases etc. This approah however also poses a risk in cases where a more swift response is required. For example:
1. Changing the Upkeep reward percent to match sudden market condition changes where arbitrage profits are unable to cover the upkeep gas cost. As an example, currently increasing the rate to 10% takes 5 voting cycles with a minimal total duration of 50 days. Leaving the upkeep process (which is crucial for Satly's system health) negatively incentivized for that long can be problematic.
2. Changing a disfunctional/compromized price feed that disrupts the correct functioning of USDS borrowing and liquidations. (Which would take atleast 20 days to perform with default params)
3. Unwhitelisting a token that turns out to be rogue and exploitative towards users.

### Exposure to rogue tokens
While rogue tokens (fee-on-transfer/reentrancy attacks and other variants) are a problem that affects most dexes, Salty's exposure to this risk is slightly elevated for two reasons:
1. Salty pool collateral is unsegregated opening the door for rogue token exploits that affect pools other than the ones hosting the rogue token.
2. The AAA mechanism potentially couples every trade with an arbitrage trade involving other pools. This too raises the risk for cross-pool exploits.

While tokens need to pass a DAO vote to be whitelisted, some rogue tokens are hard to identify even for experts and the task of filtering them out might prove to be beyond the capabilities of a DAO vote.

### Dependancy on SALT token value
Salty's intricate incentive system is at the core of the protocol, especially given that no initial liquidity is provided through investors or founders. In this context it is important to note that all rewards in Salty are awarded in the exchange token SALT (while many dexes incentivize LPs with fees paid in the provided liquidity tokens). This creates a strong dependency on the value of SALT, raising the risk of death-by-lack-of-incentive in the case of an extreme devaluation of Salt.

### high levels of inter-dependencies
Many of Salty's inner mechanics are intertwined in a way that can cause a cascading effect from a disruption in one service to the others. Some examples:
1. The WBTC/WETH pool is used both as collateral for the USDS token and as a participant in almost all arbitrage paths. In the event of a "bank run" on USDS causing large liquidity withdrawals from this pool, arbitrage profits will also be affected (since low liquidity reduces the potential arbitrage profit).
2. Salty relies on its on pools for various maintenance tasks. For example: Weth AAA profits are traded for Salt/Usds/Dai before they are distributed or added as POL, POL is liquidated and Weth/WBTC/Dai are traded by the liquidizer for USDS to cover for USDS burning deficiencies.
This creates the risk of a problem in one of these pools (such as liquidity shortage or price divergence from market prices) causing disruptions in the above mechanisms and causing a snowball effect in the system.

### Lack of contract upgradeability options
In the event that a critical vulnerability is discovered after launch, the protocol currently does not include any mechanism to upgrade contracts (other than the limited option to replace the price-feeds and AccessControl with a DAO vote). This creates the risk of future exploits necessitating a full system relaunch as a remedy.


## Centralization Risks
Due to its fully decentralized approach, Salty does not present many centralization risks. The only potential centralization risk is around the external access control authority

### Access Control Validator Risk
Salty's system is deployed with a trusted external validator that is in charge of verifying users geolocation and provides signatures that enable users to add liquidity/stake of Salty. While the AccessControl contract can be replaced by the DAO, the process for doing takes a minimum of 20 days which gives ample time for a malicious verifier to perform various exploits such as DOSing the system by blocking everyone. 




## Recommendations

Below is a list of recommendations, some of which are general architecture/structure recommendations and some are meant to address one or more of the risks mentioned above.

### Adding a limited fast-track for changes to address urgent scenarios
Given the relative slow reaction time of the dao to different situations, salty could add a fast-track for changes, limited to specific situations that call for immediate reaction. This can be achieved with a trusted multisig address that is allowed to open fast-response ballots (i.e. shorter duration, smaller quorum) or even perform certain actions directly (such as unwhitelist tokens or change upkeep reward ratio). Alternatively (and keeping in line with Salty's decentralization), certain emergency actions can be defined that can be voted on with a much shorter minimum duration coupled with a larger quorum to enable the DAO to act on emergencies

### Adding an upgradability mechanism controlled by the DAO
To address the current lack of upgradability mechanics, it is recommended to include such a mechanism, controlled by a new type of DAO proposal (possibly requiring a very high quorum as a security measure) to enable remediation of potential exploits or bugs that might be discovered in the future.

### Adding a standard Access Control mechanism
Currently Salty relies on hard-coded checks to limit access to protected functionality. This might prove to be too limited a mechanism, when in the future access control requirements might become more elaborate (for example if implementing the above recommendations). A standard ACL mechanism such as OpenZeppelin can be more flexible (i.e. enabling the DAO to dynamically grant certain privileges to specific actors) and more robust, and help prevent future ACL vulnerabilities.




### Time spent:
50 hours