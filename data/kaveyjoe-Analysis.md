


# Salty Advanced Analysis Report 


## Introduction 




Salty.IO is a cutting-edge decentralized exchange (DEX) built on the Ethereum blockchain, pioneering a revolutionary approach to trading with its Automatic Atomic Arbitrage (AAA) technology. Currently undergoing  audit in  Code4rena, Salty.IO is set to redefine the landscape of decentralized finance (DeFi) when it launches on March 1, 2024.This Avanced Analysis report provide the detailed Overview & Mechanism of salty Smart conttracts. This report alo provides the associated Centralization and systematic risks related to The salty smart contrats.  

### Key Features og Salty.IO 

**Automatic Atomic Arbitrage (AAA)**


Salty.IO employs AAA to capitalize on market inefficiencies during swaps, ensuring that profits are generated in real-time. These profits are subsequently distributed to liquidity providers and stakers, forming the foundation of Protocol Owned Liquidity (POL) for the DAO.

**Zero Fees and Yield Generation**


One of the standout features of Salty.IO is the provision of zero fees on all swaps. This is made possible through the AAA mechanism, creating a unique incentive structure for liquidity providers and stakers to actively participate in the network and share in the generated yield.

**USDS Stablecoin**

Salty.IO introduces USDS, an overcollateralized ERC20 stablecoin native to the protocol. This stablecoin utilizes WBTC/WETH LP as collateral, adding an additional layer of stability and value to the Salty.IO ecosystem.

**Decentralization at its Core**

Salty.IO is committed to full decentralization right from its launch. The DAO controls all aspects, including parameters, regional exclusions, whitelisting, and contracts. This ensures a trustless and transparent platform where the community has a direct say in the governance and evolution of the protocol.

**SALT Token**

The native token of the Salty.IO ecosystem is SALT, an ERC20 token with a capped maximum supply of 100 million tokens. With limited emissions and deflationary mechanics, SALT is designed to be a scarce and valuable asset. Staking SALT enables users to acquire xSALT, a token that serves dual purposes for both yield generation and active participation in governance decisions.



## Scope Contracts

1 . ArbitrageSearch.sol	
2 . DAO.sol		
3 . DAOConfig.sol	
4 . Proposals.sol	
5 . Parameters.sol	
6 . Airdrop.sol	
7 . BootstrapBallot.sol	
8 . InitialDistribution.sol	
9 . PoolUtils.sol	
10 . PoolsConfig.sol	
11 . PoolStats.sol	
12 . Pools.sol	
13 . PoolMath.sol	
14 . CoreChainlinkFeed.sol	
15 . CoreUniswapFeed.sol	
16 . PriceAggregator.sol	
17 . CoreSaltyFeed.sol	
18 . RewardsConfig.sol	
19 . Emissions.sol		
20 . RewardsEmitter.sol	
21 . SaltRewards.sol		
22 . USDS.sol	
23 . StableConfig.sol	
24 . CollateralAndLiquidity.sol	
25 . Liquidizer.sol	
26 . StakingConfig.sol	
27 . Liquidity.sol	
28 . StakingRewards.sol	
29 . Staking.sol	
30 . ManagedWallet.sol	
31 . AccessManager.sol	
32 . Salt.sol	
33 . SigningTools.sol	
34 . Upkeep.sol	
35 . ExchangeConfig.sol	



## Contracts Overview 

### 1 . ArbitrageSearch.sol


ArbitrageSearch  is designed to identify profitable arbitrage opportunities within a trading system that involves swapping tokens through a series of pools. The contract is abstract, meaning it is intended to be inherited by other contracts that implement its functionality. The contract focuses on finding a circular trading path that starts and ends with WETH (Wrapped Ether), aiming to capitalize on price discrepancies between different trading pairs.

**Immutable Variables**

- The contract sets immutable references to wbtc, weth, and salt tokens, which are used to define the arbitrage paths. These are set once during contract deployment and cannot be changed, ensuring consistency in the arbitrage strategy.

**Arbitrage Path Logic**

- The _arbitragePath function calculates the middle tokens of an arbitrage path based on the tokens involved in an initial swap. It supports various scenarios, including direct swaps between wbtc and weth, as well as other tokens.

**Profitability Estimation**

- The _rightMoreProfitable function is used to estimate whether the profit is higher just to the right of the midpoint of a potential arbitrage amount. This heuristic is used to guide the search for the optimal arbitrage amount.

**Bisection Search Algorithm**

- The _bisectionSearch function uses a bisection search algorithm to find the best amount to input for arbitrage. It operates under the assumption that the profit function is unimodal and that token reserves are within certain limits.

**Constants and Assumptions**

- The contract uses a MIDPOINT_PRECISION constant for arbitrage calculations and relies on the DUST threshold from PoolUtils to determine the minimum profitable amount.


### 2 . DAO.sol

The DAO contract serves as a governance mechanism for a decentralized platform, allowing stakeholders to propose changes and vote on them. 

**Proposal and Voting Mechanism**
-  The contract interacts with an IProposals contract, which likely manages the creation and storage of governance proposals. Proposals can be for parameter changes, token whitelisting/unwhitelisting, sending tokens, calling contracts, and updating the website URL.

-  The contract does not include the voting logic itself but relies on the IProposals contract to track and tally votes. It distinguishes between parameter change votes (increase, decrease, no change) and approval votes (yes or no).

- The finalizeBallot function allows anyone to trigger the finalization of a proposal once it meets certain conditions (e.g., minimum end time and quorum). The contract then executes the proposal if it is approved.

**Execution of Approved Proposals**

-  If a parameter change is approved, the _finalizeParameterBallot function executes the change by calling _executeParameterChange, which likely adjusts configuration settings in other contracts.

- The _finalizeTokenWhitelisting function handles the approval of new tokens to be added to the platform. It ensures that the proposal with the most votes is executed and provides bootstrapping rewards from the DAO's funds.

-  The _finalizeApprovalBallot function executes various actions based on the type of approval ballot, such as sending tokens, calling contracts, or updating geographical exclusions.

**Protocol Owned Liquidity (POL)**

-  The formPOL function allows the DAO to create liquidity pairs using its funds. It assumes the tokens are already transferred to the DAO and uses a collateralAndLiquidity contract to deposit the liquidity.

-  The processRewardsFromPOL function claims rewards from the liquidity pools owned by the DAO and distributes them, burning a portion as per the DAO's configuration.

-  The withdrawPOL function allows the DAO to withdraw liquidity from the pools to cover shortfalls in liquidation processes, sending the underlying tokens to a liquidizer contract.

**Arbitrage Profits**

- The withdrawArbitrageProfits function allows the DAO to withdraw profits from arbitrage operations deposited in the Pools contract and send them to the caller, which must be the Upkeep contract.

**Access Control and Geographical Exclusions**

The contract maintains a list of excluded countries, which can be updated through governance proposals. This affects access to the DEX and is enforced by an AccessManager contract.


### 3 . DAOConfig.sol


The DAOConfig contract is a Solidity smart contract designed to be owned and controlled by a Decentralized Autonomous Organization (DAO). It contains a set of parameters that govern various aspects of the DAO's operations, such as rewards, quorums for voting, proposal requirements, and more. 

**bootstrappingRewards**
-  This parameter sets the amount of SALT tokens awarded as an incentive for adding a new token to the whitelist. It ensures that there is a financial motivation for expanding the ecosystem.

**percentPolRewardsBurned**
-  This defines the percentage of SALT tokens received as rewards by the DAO that are burned. The burn mechanism is a deflationary feature that can potentially increase the value of the remaining tokens.

**baseBallotQuorumPercentTimes1000**
-  This is a multiplier used to determine the minimum percentage of xSALT (staked SALT tokens) required for different types of voting quorums. It is a critical parameter for ensuring that decisions are made with sufficient community backing.

**ballotMinimumDuration**
-  This sets the minimum time a proposal must be open for voting before any action can be taken. It ensures that token holders have enough time to consider and vote on proposals.

**requiredProposalPercentStakeTimes1000**
-  This parameter specifies the minimum percentage of SALT tokens that must be staked by a user to submit a proposal. It is designed to prevent spam proposals by ensuring that only stakeholders with a significant investment can propose changes.

**maxPendingTokensForWhitelisting**
-  This limits the number of tokens that can be in the process of being whitelisted at any given time. It helps manage the workload and focus of the DAO's governance process.

**arbitrageProfitsPercentPOL**
-  This determines the portion of profits from WETH arbitrage that is allocated to the DAO for creating Protocol Owned Liquidity (POL). It is a financial strategy to increase the DAO's treasury and liquidity.

**upkeepRewardPercent**
-  This sets the percentage of WETH arbitrage profits that are awarded to the entity that calls the DAO.performUpkeep() function. It incentivizes the maintenance of the DAO's operations.


### 4 . Proposals.sol
The Proposals contract is part Salty system that allows stakeholders of the SALT token to propose and vote on various types of ballots. These include parameter changes, token whitelisting/unwhitelisting, sending tokens, calling contracts, and updating website URLs. The contract ensures the uniqueness of ballots, tracks and validates user voting power, enforces quorums, and allows users to alter their votes.


**Proposal Creation**
- Proposals can be created by SALT stakers with a minimum required stake, which is a percentage of the total staked SALT.
- The firstPossibleProposalTimestamp ensures no proposals can be made within the first 45 days after deployment, allowing time for the staking pool to grow.
- Users are limited to one active proposal at a time to prevent spam.
- Proposal names must be unique among open ballots to prevent duplication.
- Special confirmation proposals can be created by the DAO without the restrictions applied to regular users.

**Voting Mechanism**

- Votes are cast by stakers of the SALT token, with voting power proportional to their staked amount.
- Users can change their votes, with the contract tracking the last vote cast.
- Votes are tallied by type, with different types being valid for different ballot types (e.g., Vote.INCREASE, Vote.DECREASE, Vote.NO_CHANGE for parameter ballots, and Vote.YES, Vote.NO for approval ballots).

**Ballot Types**

- The contract supports various ballot types, including parameter changes, token whitelisting/unwhitelisting, sending SALT tokens, calling contracts, country inclusion/exclusion, setting contract addresses, and updating website URLs.
- Each ballot type has a corresponding function to propose that specific change.

**Quorum and Approval**
- The required quorum for a ballot to be valid is a percentage of the total SALT staked, with a minimum quorum set at 0.50% of the total SALT supply.
- The quorum required varies depending on the ballot type, with parameter changes requiring the least and other types requiring more.
- A ballot is considered approved if the quorum is met and, for approval ballots, if Vote.YES votes exceed Vote.NO votes. For parameter ballots, the winning vote type is determined by the highest vote count among Vote.INCREASE, Vote.DECREASE, and Vote.NO_CHANGE.

**Finalization**
- Ballots can only be marked as finalized by the DAO, which involves checking that the ballot is live, the minimum end time has passed, and the required quorum has been reached.
- Finalization effectively closes the ballot and enforces the result.

**Views and Helpers**
- The contract provides view functions to check ballot details, user votes, votes cast for a ballot, and whether a ballot can be finalized.
- It also includes functions to list all open ballots and those specifically for token whitelisting.



### 5 . Parameters.sol


Parameters.solincludes an enumeration ParameterTypes that lists various configuration parameters across different modules of the platform, such as PoolsConfig, StakingConfig, RewardsConfig, StableConfig, DAOConfig, and PriceAggregator.

The contract's primary function _executeParameterChange takes a ParameterTypes value, a boolean increase, and interfaces to various configuration modules. Depending on the parameterType provided, the function calls the appropriate method on the corresponding configuration module to either increase or decrease the parameter value.


### 6 . Airdrop.sol	

Airdrop.sol is  intended to distribute a token called SALT to users who have participated in certain promotional activities, such as retweeting a launch announcement and voting. The contract is designed to work in conjunction with other contracts, specifically an IExchangeConfig for configuration parameters, an IStaking contract for staking functionality, and a token contract ISalt representing the SALT token.


**Initialization**
-  The contract is initialized with references to IExchangeConfig and IStaking contracts. The IExchangeConfig contract provides access to the SALT token and other configuration details, while the IStaking contract handles the staking of SALT tokens.

**Authorization**
-  The authorizeWallet function allows the BootstrapBallot contract (part of the initial distribution process) to authorize wallets that have met the promotional criteria. This function can only be called by the BootstrapBallot contract and cannot be used after claiming has been enabled.

**Claiming Process**
-  The allowClaiming function is called by the InitialDistribution contract upon successful conclusion of the BootstrapBallot. It calculates the amount of SALT each user will receive by dividing the contract's SALT balance by the number of authorized users. It then approves the IStaking contract to handle this amount of SALT.

**Claiming Airdrop**
-  The claimAirdrop function allows an authorized user to claim their airdrop. It ensures that claiming is allowed, the user is authorized, and the user has not already claimed their airdrop. The function stakes the SALT on behalf of the user and then transfers the staked SALT to them.

**Views**
-  The contract includes view functions to check if a wallet is authorized (isAuthorized) and to get the number of authorized wallets (numberAuthorized).

### 7 . BootstrapBallot.sol

The BootstrapBallot contract is designed to facilitate a voting process for airdrop participants to decide whether to launch an exchange and establish initial geographic restrictions. The contract is linked to an IExchangeConfig interface to interact with exchange configuration settings and an IAirdrop interface to manage airdrop authorizations.


**Initialization**
-  The contract is initialized with references to the IExchangeConfig and IAirdrop contracts, as well as a ballotDuration which sets the time frame for the voting period.

**Voting**
-  The vote function allows whitelisted users to cast a single vote on whether to start the exchange. The vote is a binary choice (Yes/No), and each voter can only vote once, enforced by the hasVoted mapping. A signature is required to validate the voter's eligibility, which is checked off-chain and verified within the contract using SigningTools._verifySignature.

**Airdrop Authorization**
-  Upon voting, the voter is authorized to receive an airdrop by calling airdrop.authorizeWallet.

**Finalization**
-  Once the completionTimestamp is reached, any party can call finalizeBallot to conclude the voting process. If the majority voted "Yes," the contract calls exchangeConfig.initialDistribution().distributionApproved() and exchangeConfig.dao().pools().startExchangeApproved() to proceed with the exchange launch and distribution of SALT tokens.

**Event Logging**
-  The contract emits a BallotFinalized event indicating the outcome of the vote.


### 8 . InitialDistribution.sol


The InitialDistribution contract is designed to handle the initial distribution of the SALT token to various stakeholders and systems within a Salty ecosystem. The contract is responsible for allocating tokens to emissions, a DAO reserve, the initial development team, airdrop participants, and for liquidity and staking bootstrapping.


**The distributionApproved function is the core of this contract, which is called when the BootstrapBallot is approved. It performs the following actions**

- Validates that the caller is the bootstrapBallot contract.
- Checks that the contract still holds 100 million SALT tokens.
- Transfers 52 million SALT tokens to the emissions contract.
- Transfers 25 million SALT tokens to the daoVestingWallet.
- Transfers 10 million SALT tokens to the teamVestingWallet.
- Transfers 5 million SALT tokens to the airdrop contract and allows claiming.
- Retrieves the list of whitelisted pools from poolsConfig.
- Transfers 8 million SALT tokens to the saltRewards contract.
- Calls sendInitialSaltRewards on the saltRewards contract with 5 million SALT tokens and the list of pool IDs.


### 9 . PoolUtils.sol


The PoolUtils  contract  is intendeds managing liquidity pools and performing token swaps. The library includes functions for generating unique pool identifiers for pairs of ERC20 tokens, and for executing internal swaps within the protocol with certain restrictions to mitigate the profitability of sandwich attacks.

**Pool Identifier Generation**

- `_poolID`: Generates a unique identifier (poolID) for a pair of ERC20 tokens by sorting the addresses of the tokens and hashing them together. This ensures that the order of the tokens does not affect the poolID.
- `_poolIDAndFlipped`: Similar to _poolID, but also returns a boolean indicating whether the order of the tokens was flipped to generate the poolID.

**Internal Swap Mechanism**

- `_placeInternalSwap`: This function allows for swapping tokens within the protocol. It takes into account the current reserves of the input token (tokenIn) in the liquidity pool and limits the amountIn to a certain percentage of the reserves, defined by maximumInternalSwapPercentTimes1000. This is designed to reduce the effectiveness of sandwich attacks by limiting the size of the swap and allowing for atomic arbitrage within the same transaction.

### 10 . PoolsConfig.sol

The PoolsConfig contract is designed to manage the configuration of Salty liquidity pools. It is intended to be owned and controlled by a Decentralized Autonomous Organization (DAO), which has the exclusive authority to modify the contract's state. The contract provides functionality to whitelist and unwhitelist token pairs for liquidity pools, adjust the maximum number of whitelisted pools, and set limits on internal swap sizes as a percentage of the pool's reserves.


**Whitelisting Pools**
- The whitelistPool function allows the DAO to add a new token pair to the whitelist, enabling liquidity provision and trading for that pair within the protocol.
- It checks that the number of whitelisted pools does not exceed the maximumWhitelistedPools limit and that the two tokens in the pair are not the same.
- The function generates a poolID using PoolUtils._poolID and adds it to the _whitelist set. It also updates the underlyingPoolTokens mapping with the new token pair.
- After whitelisting, it calls updateArbitrageIndicies on the pools contract to refresh any cached data related to arbitrage opportunities.

**Unwhitelisting Pools**
- The unwhitelistPool function allows the DAO to remove a token pair from the whitelist.
- It generates a poolID for the given token pair and removes it from the _whitelist set and deletes the associated entry in underlyingPoolTokens.
- Similar to whitelisting, it calls updateArbitrageIndicies on the pools contract to update related data.

**Adjusting Maximum Whitelisted Pools**

- The changeMaximumWhitelistedPools function allows the DAO to increase or decrease the maximumWhitelistedPools within a specified range (20 to 100) and in increments of 10.

**Adjusting Maximum Internal Swap Size**

- The changeMaximumInternalSwapPercentTimes1000 function allows the DAO to adjust the maximumInternalSwapPercentTimes1000 within a specified range (0.25% to 2%) and in increments of 0.25%. The value is stored with a 1000x multiplier for precision.

**Views**

- The contract provides several view functions to check the number of whitelisted pools, whether a specific poolID is whitelisted, the list of all whitelisted poolIDs, the token pair associated with a poolID, and whether a token has been whitelisted with WBTC or WETH.


### 11 . PoolStats.sol

The PoolStats is designed to work with a set of whitelisted pools that participate in triangular arbitrage opportunities involving a wrapped Ether (WETH) token and two other arbitrary tokens (arbToken2 and arbToken3). The contract is responsible for recording profits from arbitrage, resetting these stats, updating indices of pools involved in arbitrage, and calculating the distribution of rewards based on the profits generated.



**Arbitrage Profit Tracking**
-  The contract maintains a mapping (_arbitrageProfits) that records the profits generated from arbitrage for each pool, identified by a unique poolID derived from the two non-WETH tokens involved in the arbitrage path (WETH -> arbToken2 -> arbToken3 -> WETH).

**Profit Resetting**
-  The clearProfitsForPools function is called to reset the arbitrage profits to zero for all whitelisted pools. This function can only be called by the Upkeep contract, ensuring that only authorized maintenance operations can clear the profits.

**Index Mapping**
-  The contract also maintains a mapping (_arbitrageIndicies) that stores the indices of each pool within the whitelistedPools array that would contribute to arbitrage for a given pool. This is used to track which pools are involved in a particular arbitrage opportunity.

**Updating Arbitrage Indices**
-  The updateArbitrageIndicies function is used to traverse the current whitelisted pool IDs and update the indices of each pool that would contribute to arbitrage for it. This function is public and can be called by any user.

**Calculating Arbitrage Profits**
-  The _calculateArbitrageProfits function is used to determine how much each pool contributed to the arbitrage profits since the last upkeep. It divides the profits equally among the three pools involved in the arbitrage path.

**View Functions**
-  The contract provides view functions like profitsForWhitelistedPools and arbitrageIndicies to allow external entities to query the calculated profits for pools and the arbitrage indices for a given pool ID, respectively.

### 12 . Pools.sol


The Pools contract is designed to manage liquidity pools for a Salty  (DEX). It handles the addition and removal of liquidity, token swaps, and arbitrage opportunities. The contract is built on the Ethereum blockchain using the Solidity programming language and is compatible with the ERC-20 token standard.


**Liquidity Management**
- Liquidity is added to pools using the addLiquidity function, which requires the caller to be the CollateralAndLiquidity contract. This function calculates the maximum amount of tokens that can be added to a pool while maintaining the current ratio of reserves.
- Liquidity is removed with the removeLiquidity function, which also requires the caller to be the CollateralAndLiquidity contract. It ensures that the removal of liquidity does not reduce the reserves below a minimum threshold (PoolUtils.DUST).

**Token Deposits and Withdrawals**

- Users can deposit tokens into the contract using the deposit function, which increases their internal balance without affecting liquidity.
- Withdrawals are made through the withdraw function, which decreases the user's internal balance.

**Swaps and Arbitrage**
- Swaps between tokens are performed by the _adjustReservesForSwap function, which updates the reserves according to the constant product formula.
- Arbitrage is attempted after every swap using the _attemptArbitrage function, which seeks to profit from imbalances in the exchange pools by executing a circular trade path starting and ending with WETH (Wrapped Ether).
- The contract includes convenience functions like swap, depositSwapWithdraw, and depositDoubleSwapWithdraw for users to perform swaps and manage their deposits in a single transaction.

**Contract Initialization and Management**
- The contract is initialized with references to IExchangeConfig and IPoolsConfig through the constructor.
- The setContracts function is used to set the IDAO and ICollateralAndLiquidity contracts, and it can only be called once as it renounces ownership after execution.
- The startExchangeApproved function is used to activate the exchange once approved by the bootstrapBallot.

### 13 . PoolMath.sol

The PoolMath cntract is designed to assist with calculations related to adding liquidity ,specifically for pools that follow the constant product formula (x * y = k). The library is intended to be used with pools where liquidity providers can "zap" in liquidity by depositing uneven amounts of two tokens. The goal is to determine the optimal amount of one token to swap for the other before adding liquidity to ensure that the deposited tokens maintain the same ratio as the pool's reserves, minimizing slippage and leftover funds.

**The library contains several functions**

- `_mostSignificantBit`: Finds the most significant bit (MSB) of a non-zero number, which is used to determine the bit length of the number.
- `_maximumMSB`: Determines the maximum MSB across four given values, which is used to normalize the values for calculations.
- `_zapSwapAmount`: Calculates the amount of token0 that needs to be swapped to token1 before adding liquidity to the pool. This function uses a quadratic equation derived from the constant product formula to find the optimal swap amount that aligns the ratio of the zapped amounts with the pool's reserves ratio.
- `_determineZapSwapAmount`: Determines whether a swap is necessary before adding liquidity and, if so, which token and how much of it needs to be swapped.

The library uses a quadratic equation to solve for the swap amount and employs bit-shifting to normalize values for calculations, ensuring that the operations do not exceed the 256-bit limit of Solidity integers.

### 14 . CoreChainlinkFeed.sol	



The CoreChainlinkFeed contract is designed to provide the latest price data for Bitcoin (BTC) and Ethereum (ETH) in USD using Chainlink's decentralized oracle network. It implements an interface IPriceFeed which is not shown in the provided code but is expected to define the functions for retrieving price data. The contract uses two immutable public variables, CHAINLINK_BTC_USD and CHAINLINK_ETH_USD, which are references to Chainlink's AggregatorV3Interface for BTC/USD and ETH/USD price feeds, respectively.


**Immutable Oracle References**
-  It initializes with immutable references to Chainlink's BTC/USD and ETH/USD price feed oracles.

**Price Retrieval**
-  It provides external functions to fetch the latest prices for BTC and ETH, converting them from Chainlink's 8 decimal format to the standard 18 decimal format used in Ethereum.

**Freshness Check**
-  It includes a check to ensure that the price data is not older than 60 minutes, returning zero if this condition is not met.

**Error Handling**
-  It uses a try-catch block to handle any errors during the oracle call, returning zero in case of failure.

**Negative Price Check**
-  It validates the sign of the price, returning zero if the price is negative.

### 15 . CoreUniswapFeed.sol

The CoreUniswapFeed contract provides time-weighted average price (TWAP) data for WBTC (Wrapped Bitcoin) and WETH (Wrapped Ether) using Uniswap v3 pools. It is designed to fetch prices with a 30-minute TWAP period to mitigate the risk of price manipulation. The contract uses two Uniswap v3 pools: one for WBTC/WETH and another for WETH/USDC (USD Coin).


**TWAP Calculation**
-  The contract calculates the TWAP by querying the Uniswap v3 pool's observe function, which returns cumulative tick data over specified time intervals. The tick data is then used to compute the square root of the price and subsequently the price itself, normalized to 18 decimals.

**Token Order Handling**
-  The contract accounts for the possibility that the tokens in a Uniswap pool could be in any order. It uses boolean flags (wbtc_wethFlipped and weth_usdcFlipped) to track the order of tokens in the pools and adjusts the price calculation accordingly.

**Price Retrieval Functions**
-  The contract provides public functions getTwapWBTC and getTwapWETH to retrieve the TWAP for WBTC and WETH, respectively. These functions call getUniswapTwapWei, which wraps the internal _getUniswapTwapWei function with a try/catch block to handle any potential errors by returning zero.

**External Interface**
-  The contract implements the IPriceFeed interface, providing getPriceBTC and getPriceETH functions that external contracts can call to get the 30-minute TWAP for WBTC and WETH.

### 16 . PriceAggregator.sol

The PriceAggregator contract is designed to provide reliable price feeds for BTC and ETH by aggregating data from three different external price feed sources. The contract is intended to mitigate the risk of a single point of failure by using multiple feeds and discarding outliers. The contract is updatable, meaning the price feeds can be changed, but with a cooldown period to ensure stability and allow for performance review.


**Initial Setup**
-  The contract owner sets the initial price feeds using setInitialFeeds. This can only be done once as it checks that priceFeed1 is not already set.

**Updating Price Feeds**
-  The owner can update the price feeds using setPriceFeed. This function checks if the cooldown period has passed before allowing an update. The cooldown expiration time is then reset.

**Cooldown Management**
-  The owner can adjust the cooldown period (priceFeedModificationCooldown) between updates within a range of 30 to 45 days using changePriceFeedModificationCooldown.

**Price Feed Difference Threshold**
- The contract has a threshold for the maximum percent difference between two price feeds (maximumPriceFeedPercentDifferenceTimes1000). This can be adjusted by the owner within a range of 1% to 7% using changeMaximumPriceFeedPercentDifferenceTimes1000.

**Price Aggregation**
-  The contract uses _aggregatePrices to calculate an average price from the two closest price feeds, provided they are within the acceptable difference threshold. If the threshold is exceeded or fewer than two feeds are available, the function returns zero, indicating a failure.

**Price Retrieval**
-  External functions getPriceBTC and getPriceETH are provided to fetch the current BTC and ETH prices, respectively. They use internal functions to try to get prices from each feed and handle exceptions gracefully.


### 17 . CoreSaltyFeed.sol

The CoreSaltyFeed contract is designed to provide price feeds for Bitcoin (BTC) and Ethereum (ETH) using the Salty.IO pools. It is intended to be used in a decentralized finance (DeFi) ecosystem where accurate and reliable price information is crucial for various financial instruments and smart contracts. The contract interacts with an external IPools interface to retrieve the reserves of wrapped Bitcoin (WBTC), wrapped Ethereum (WETH), and a stablecoin (USDS) from liquidity pools. The contract assumes that WBTC and WETH are ERC20 tokens with 8 and 18 decimals, respectively, while USDS also has 18 decimals.

**Immutable References**
-  The contract initializes immutable references to the liquidity pools and token contracts upon deployment, which cannot be changed later.
**Price Calculation**
-  The getPriceBTC() and getPriceETH() functions fetch the reserves from the liquidity pools and calculate the price by dividing the stablecoin reserves by the cryptocurrency reserves, adjusting for decimal differences.
**Liquidity Check**
-  Before calculating the price, the contract checks if the reserves are above a minimal threshold (DUST) to avoid price manipulation in low-liquidity scenarios. If the reserves are below this threshold, the function returns zero, indicating an invalid price.
**No Slippage or Fee Adjustment**
-  The price calculation does not account for trading slippage or fees, which could affect the actual exchange rate in a live trade.
**Single Source of Data**
-  The contract relies solely on the Salty.IO pools for price information, without any fallback or alternative data sources.

### 18 . RewardsConfig.sol	

The RewardsConfig  is designed to manage the distribution of rewards.RewardsConfig includes mechanisms to set the daily and weekly percentages of rewards distribution, the split of rewards between stakers and liquidity providers, and the specific allocation to the SALT/USDS pool.


**rewardsEmitterDailyPercentTimes1000**
-  This variable represents the daily percentage of rewards distributed by the rewards emitters, with a multiplier of 1000 to allow for decimal percentages. The range is 0.25% to 2.5%, adjustable in increments of 0.25%.

**emissionsWeeklyPercentTimes1000**
-  This variable represents the weekly percentage of SALT emissions distributed to reward emitters. The range is 0.25% to 1.0%, adjustable in increments of 0.25%.

**stakingRewardsPercent**
-  This variable represents the percentage of emissions and arbitrage profits allocated to xSALT holders versus liquidity providers. The range is 25% to 75%, adjustable in increments of 5%.

**percentRewardsSaltUSDS**
-  This variable represents the percentage of SALT rewards sent to the SALT/USDS pool. The range is 5% to 25%, adjustable in increments of 5%.

RewardsConfig  provides four functions to adjust these parameters, each of which can only be called by the owner (the DAO):

- `changeRewardsEmitterDailyPercent(bool increase)`
- `changeEmissionsWeeklyPercent(bool increase)`
- `changeStakingRewardsPercent(bool increase)`
- `changePercentRewardsSaltUSDS(bool increase)`

Each function checks whether the increase flag is true or false and adjusts the corresponding variable within the allowed range. After the change, an event is emitted to log the new value.


### 19 . Emissions.sol


The Emissions contract is designed to manage and distribute the emissions of the SALT token over time. It interacts with several other contracts, including ISaltRewards, IExchangeConfig, IRewardsConfig, and ISalt. it is intended to distribute SALT tokens to two types of emitters: stakingRewardsEmitter and liquidityRewardsEmitter, which are managed through the SaltRewards contract.


 The performUpkeep function is the core of the contract, which is called to distribute the SALT tokens. The function takes timeSinceLastUpkeep as an argument, which is used to calculate the amount of SALT to distribute.

**The distribution logic works as follows:**

- The function can only be called by the Upkeep contract, as verified by the require statement checking the msg.sender.
- If timeSinceLastUpkeep is zero, the function exits early, as there is no need to distribute tokens.
- The timeSinceLastUpkeep is capped at one week (MAX_TIME_SINCE_LAST_UPKEEP) to prevent distributing more than 0.50% of the SALT balance in a single transaction.
- The contract calculates the SALT balance it holds and then determines the amount of SALT to send based on the timeSinceLastUpkeep and the configured weekly emissions percentage (emissionsWeeklyPercentTimes1000).
- If the calculated saltToSend is non-zero, it transfers that amount of SALT to the saltRewards contract.

### 20 . RewardsEmitter.sol	

The RewardsEmitter  is designed to manage and distribute SALT token rewards to participants in a staking ecosystem. It supports two types of reward distributions:

- **Staking.sol:** For users who stake SALT tokens to acquire xSALT.
- **Liquidity.sol:** For liquidity providers who deposit and stake collateral and liquidity.
The contract is intended to provide a stable yield by gradually emitting rewards at a default rate of 1% per day. This gradual emission rate helps to mitigate the natural fluctuation of rewards and creates a more predictable yield environment.

**Reward Addition (addSALTRewards)**
-  Allows the addition of SALT rewards for later distribution. It requires that the pools where rewards are added are whitelisted by the poolsConfig. The rewards are transferred from the sender to the contract.

**Reward Distribution (performUpkeep)**
-  Distributes a percentage of the pending rewards to the staking pools. The percentage is based on the time elapsed since the last upkeep, with a maximum cap of 1% per day. The performUpkeep function can only be called by the upkeep contract specified in the exchangeConfig.

**Views**
-  The contract provides a view function pendingRewardsForPools to check the pending rewards for specific pools.

### 21 . SaltRewards.sol


The SaltRewards contract is a utility contract designed to distribute SALT token rewards to participants . It interacts with two types of reward emitters: stakingRewardsEmitter and liquidityRewardsEmitter. The contract is responsible for temporarily holding SALT rewards that are generated from emissions and arbitrage profits, and then distributing these rewards to stakers and liquidity providers.



**Initialization:**
 Saltrewards  is initialized with references to the stakingRewardsEmitter, liquidityRewardsEmitter, exchangeConfig, and rewardsConfig. It also caches the SALT token contract and the SALT/USDS pool ID for efficiency. The contract sets an unlimited approval for the SALT token to both reward emitters to facilitate reward distribution without requiring repeated approvals.

**Reward Distribution**

- **Initial Rewards**: The sendInitialSaltRewards function allows the InitialDistribution contract to trigger the distribution of initial rewards. It divides a specified liquidityBootstrapAmount evenly among provided pool IDs and sends the remaining SALT balance to the staking rewards emitter.
- **Upkeep Rewards**: The performUpkeep function is called by the Upkeep contract to distribute rewards based on recent arbitrage profits. It calculates the total profits and determines the proportional rewards for each pool. A fixed percentage of rewards is allocated directly to the SALT/USDS pool, and the remaining rewards are split between staking and liquidity rewards based on the configuration in rewardsConfig.

**Reward Calculation**

- The contract calculates the amount of rewards for each pool based on its share of the total arbitrage profits.
- A fixed percentage of the total SALT rewards is allocated to the SALT/USDS pool, as specified by rewardsConfig.percentRewardsSaltUSDS.
- The remaining rewards are split between staking and liquidity providers according to the rewardsConfig.stakingRewardsPercent.

**Internal Functions**
 - _sendStakingRewards, _sendLiquidityRewards, _sendInitialLiquidityRewards, and _sendInitialStakingRewards are internal functions that handle the actual transfer of rewards to the respective emitters.

### 22 . USDS.sol

The USDS contract is an ERC20 token that represents a stablecoin, which can be borrowed by users who have deposited collateral in the form of WBTC/WETH liquidity. The contract interacts with another contract, ICollateralAndLiquidity, which handles the collateral management. The USDS contract includes functions to mint new tokens and burn existing tokens.


**Minting USDS**
-  The mintTo function allows the CollateralAndLiquidity contract to mint USDS tokens when a user deposits adequate collateral. The minting is restricted to the CollateralAndLiquidity contract, ensuring that only authorized mints occur in response to collateralization.

**Burning USDS**
-  The burnTokensInContract function allows the burning of USDS tokens that are present in the contract itself. This is likely used to burn tokens that are repaid or liquidated, reducing the total supply and ensuring the stability of the stablecoin.

**Ownership and Permissions**
- The owner (deployer) of the contract can set the CollateralAndLiquidity contract address once using setCollateralAndLiquidity. After this, the contract's ownership is renounced by calling renounceOwnership, which is a feature of the Ownable contract from OpenZeppelin. This means that after setting the collateral management contract, no further administrative actions can be taken.


### 23 . StableConfig.sol

The StableConfig contract is a parameter configuration designed to be owned and controlled by a Decentralized Autonomous Organization (DAO). It allows the DAO to adjust various parameters related to liquidation rewards, collateral requirements, and profit sharing within the protocol. 


**rewardPercentForCallingLiquidation**
-  The percentage of the liquidated collateral that is rewarded to the user who initiates a liquidation process. It has a range of 5% to 10% and can be adjusted in increments of 1%.

**maxRewardValueForCallingLiquidation**
-  The maximum USD value of the reward for initiating a liquidation, ranging from 100 to 1000 ether, adjustable in increments of 100 ether.

**minimumCollateralValueForBorrowing**
- The minimum USD value of collateral required to borrow, ranging from 1000 to 5000 ether, adjustable in increments of 500 ether.

**initialCollateralRatioPercent**
- The initial maximum ratio of collateral to borrowed USDS, ranging from 150% to 300%, adjustable in increments of 25%.

**minimumCollateralRatioPercent**
- The minimum collateral-to-borrowed USDS ratio below which positions can be liquidated, ranging from 110% to 120%, adjustable in increments of 1%.

**percentArbitrageProfitsForStablePOL**
- The percentage of arbitrage profits used to form protocol-owned liquidity (POL), ranging from 1% to 10%, adjustable in increments of 1%.

### 24 . CollateralAndLiquidity.sol	


The CollateralAndLiquidity is designed to manage collateralized borrowing and liquidity provision on a decentralized exchange. Users can deposit WBTC (Wrapped Bitcoin) and WETH (Wrapped Ether) as collateral to borrow a stablecoin called USDS. This also allows users to withdraw their collateral and claim rewards, borrow USDS against their collateral, and repay borrowed USDS.


**Collateral Deposit and Withdrawal**
- Users can deposit WBTC and WETH as collateral (depositCollateralAndIncreaseShare) and withdraw their collateral (withdrawCollateralAndClaim), subject to maintaining a sufficient collateralization ratio.
- The contract emits events for collateral deposits and withdrawals, which helps in tracking user activity.

**Borrowing and Repayment**
- Users can borrow USDS (borrowUSDS) up to a maximum amount determined by their collateral value and a predefined collateralization ratio.
- Borrowed amounts are tracked per user, and the contract emits an event when USDS is borrowed.
- Users can repay their borrowed USDS (repayUSDS), and the contract ensures that the repayment amount does not exceed the borrowed amount.

**Liquidation**
- If a user's collateralization ratio falls below the minimum threshold, their position can be liquidated (liquidateUser).
- Liquidators are incentivized with a percentage of the liquidated collateral, and the remaining collateral is sent to a Liquidizer contract for conversion to USDS and subsequent burning.
- The contract emits an event upon liquidation.

**Views and Utility Functions**
- The contract provides view functions to check the value of collateral in USD, the total collateral value, the user's collateral value, the maximum withdrawable collateral, and the maximum borrowable USDS.
- It also includes functions to check if a user can be liquidated and to find all users with liquidatable positions.

### 25 . Liquidizer.sol

The Liquidizer contract is designed to maintain the stability of a collateralized lending system by managing the liquidation process and POL .   Its primary function is to convert various tokens received from liquidated collateral or withdrawn POL into USDS (a stablecoin) and then burn the USDS to maintain the system's collateral balance.



**Token Handling**
-  The contract uses SafeERC20 for safe ERC20 interactions and defines several immutable ERC20 token interfaces for WBTC, WETH, USDS, SALT, and DAI.

**Contract Initialization**
-  The contract sets up references to other system contracts such as IExchangeConfig, IPoolsConfig, ICollateralAndLiquidity, IPools, and IDAO. It also approves the maximum possible amount of WBTC, WETH, and DAI to the IPools contract for future swaps.

**Ownership and Permissions**
- The contract inherits from Ownable, allowing certain functions to be restricted to the owner. After initial setup, ownership is renounced to prevent further changes, indicating a commitment to decentralization.

**Liquidation Handling**
- The incrementBurnableUSDS function is called by the ICollateralAndLiquidity contract when a user's collateral is liquidated. It increases the usdsThatShouldBeBurned counter, which tracks the amount of USDS that needs to be burned to offset the liquidated collateral.

**Burning Mechanism**
- The _burnUSDS internal function transfers USDS to the USDS contract and calls burnTokensInContract to burn them.

**POL Management**
- If there is not enough USDS to burn, the contract will withdraw a small percentage of POL from the IDAO contract and convert the underlying tokens (SALT and DAI) to USDS for burning.

**Token Swapping**
- The performUpkeep function is called by the exchangeConfig.upkeep() contract to swap WBTC, WETH, and DAI for USDS. It uses PoolUtils._placeInternalSwap to perform the swaps within the limits set by maximumInternalSwapPercentTimes1000. Any SALT tokens are burned directly to avoid market impact.

**Event Logging**
- The contract emits an event incrementedBurnableUSDS when the amount of USDS to be burned is incremented.


### 26 . StakingConfig.sol

The StakingConfig contract is designed to manage the configuration parameters for a staking system  with the ability to modify certain parameters related to unstaking periods and percentages, as well as cooldowns for modifications to prevent gaming of the system. 


**minUnstakeWeeks**
- The minimum number of weeks required for an unstake request, after which a minimum percentage (minUnstakePercent) of the staked tokens can be reclaimed.
**maxUnstakeWeeks**
- The maximum number of weeks for an unstake request, after which 100% of the staked tokens can be reclaimed.
**minUnstakePercent**
- The minimum percentage of staked tokens that can be reclaimed after unstaking for the minimum number of weeks (minUnstakeWeeks).
**modificationCooldown**
- The minimum time between modifications to a user's share in the SharedRewards contracts, intended to prevent reward hunting.

Each parameter has a specific range and adjustment granularity:

- `minUnstakeWeeks`: 1 to 12 weeks, adjustable by 1 week.
- `maxUnstakeWeeks`: 20 to 108 weeks, adjustable by 8 weeks.
- `minUnstakePercent`: 10% to 50%, adjustable by 5%.
- `modificationCooldown`: 15 minutes to 6 hours, adjustable by 15 minutes.

The contract provides four functions to adjust these parameters, each emitting an event upon a successful change:

- `changeMinUnstakeWeeks(bool increase)`
- `changeMaxUnstakeWeeks(bool increase)`
- `changeMinUnstakePercent(bool increase)`
- `changeModificationCooldown(bool increase)`


### 27 . Liquidity.sol


The Liquidity  contract is designed to manage liquidity provision and withdrawal . The contract allows users to deposit two types of ERC20 tokens into a liquidity pool, potentially using a zapping mechanism to balance the token amounts, and to withdraw their liquidity later on. Users receive liquidity shares that entitle them to a proportion of future rewards.

**Liquidity Provision (_depositLiquidityAndIncreaseShare)**

- Users can deposit two tokens (tokenA and tokenB) into a liquidity pool.
- The contract supports a zapping feature (_dualZapInLiquidity) that automatically balances the token amounts by swapping excess tokens for the other token to match the pool's required ratio.
- The contract checks if the user has exchange access before allowing a deposit.
- The deposited tokens are approved and then added to the pool via the pools.addLiquidity function.
- The user's liquidity share is increased by calling _increaseUserShare.
Any excess tokens not used in the liquidity provision are returned to the user.
- The contract emits a LiquidityDeposited event.

**Liquidity Withdrawal (_withdrawLiquidityAndClaim)**
- Users can specify the amount of liquidity they wish to withdraw.
- The contract checks if the user has enough liquidity shares to perform the withdrawal.
- The liquidity is removed from the pool, and the corresponding tokens are transferred back to the user.
- The user's liquidity share is decreased by calling _decreaseUserShare, and any pending rewards are claimed.
- The contract emits a LiquidityWithdrawn event.

**Public Wrappers**

- depositLiquidityAndIncreaseShare and withdrawLiquidityAndClaim are public functions that act as wrappers for the internal functions, adding checks for deadlines and preventing interaction with a specific collateral pool (collateralPoolID).


### 28 . StakingRewards.sol

The StakingRewards contract is  designed to handle the distribution of rewards, specifically SALT tokens, to users who stake either SALT tokens or liquidity shares. The contract is abstract, meaning it is intended to be inherited by other contracts that define specific behaviors for different types of staking (e.g., staking SALT tokens or liquidity pool shares).



**Staking and Unstaking**
-  Users can increase or decrease their stake in a particular pool using _increaseUserShare and _decreaseUserShare functions. These functions adjust the user's share and the total shares for the pool, as well as handle the rewards calculation.

**Reward Calculation**
-  The contract calculates rewards based on the user's share of the total staked amount in a pool. When a user stakes or unstakes, virtual rewards are added or removed to keep the reward-to-share ratio consistent. This ensures that users receive a fair proportion of the rewards based on their stake.

**Cooldown Period*8
-  There is a cooldown period enforced for share modifications to prevent immediate unstaking after staking. This cooldown can be bypassed by the DAO.

**Claiming Rewards**
-  Users can claim their rewards without unstaking their shares using the claimAllRewards function. Claimed rewards are tracked as virtual rewards to prevent double claiming.

**Adding Rewards**
-  The addSALTRewards function allows for the addition of SALT rewards to the contract. It requires the pools to be whitelisted and handles the distribution of the added rewards across the pools.


### 29 . Staking.sol

Staking contract for a token called SALT, which allows users to stake SALT tokens in exchange for xSALT tokens at a 1:1 ratio. The contract includes functionality for staking, unstaking with a cooldown period, expedited unstaking with a fee, and recovering staked tokens after the unstaking period. It also allows for the transfer of xSALT from an airdrop contract to a user.


**Staking**
-  Users can stake SALT tokens and receive an equivalent amount of xSALT tokens. The stakeSALT function requires the sender to have exchange access, which is checked against an exchangeConfig contract. Staking increases the user's share in the staking pool, which determines their portion of future SALT rewards.

**Unstaking**
-  The unstake function allows users to initiate an unstake of their xSALT over a specified number of weeks. The amount of SALT they can claim at the end of the unstaking period is calculated based on the duration of the unstake. The longer the unstake period, the higher the percentage of SALT they can reclaim, up to 100% for a full year.

**Expedited Unstaking**
-  Users can choose to unstake their tokens over a minimum period of two weeks, but they will incur an expedited unstake fee, receiving less than the full amount of their staked SALT.

**Recovering Staked Tokens**
-  Once the unstaking period is complete, users can call recoverSALT to claim their SALT tokens. If they unstaked early, a fee is deducted from their original stake, which is then burned.

**Cancel Unstake**
-  Users can cancel a pending unstake with the cancelUnstake function, which allows them to immediately use their xSALT again.

**Airdrop Transfers**
-  The contract has a function transferStakedSaltFromAirdropToUser that allows the airdrop contract to transfer xSALT to users.

### 30 . ManagedWallet.sol


The ManagedWallet contract is designed to manage two wallet addresses: a main wallet and a confirmation wallet. These wallets can be changed through a specific process that involves proposals, confirmations, and a timelock mechanism. The contract is intended to provide a secure way to update wallet addresses, potentially for a system where these wallets have significant permissions or control over assets.


**Proposal Mechanism**
-  The current mainWallet can propose new wallet addresses for both the mainWallet and confirmationWallet by calling the proposeWallets function. The proposed addresses cannot be the zero address, and there cannot be an existing proposal that has not been confirmed or rejected.

**Confirmation Mechanism**
-  The confirmationWallet confirms or rejects the proposal by sending a specific amount of ETH to the contract. If the confirmationWallet sends at least 0.05 ETH, the proposal is considered confirmed, and a timelock is activated. If less than 0.05 ETH is sent, the proposal is considered rejected, and the timelock is set to a maximum value, effectively disabling it.

**Timelock Mechanism**
-  Once a proposal is confirmed, there is a 30-day timelock (TIMELOCK_DURATION) before the change can be finalized. After the timelock expires, the proposedMainWallet can call the changeWallets function to finalize the change of the wallet addresses.

**Events**
-  The contract emits two types of events: WalletProposal when a new proposal is made, and WalletChange when the wallet addresses are successfully changed.


### 31 . AccessManager.sol	

The Acessmanger contract is designed to comply with regional regulations by restricting certain actions such as adding liquidity, collateral, and borrowing a stablecoin (presumably USDS) based on the user's geolocation. The contract allows for the removal of assets by users even if their region becomes restricted after depositing, ensuring that users are not locked out of their funds.

**DAO Integration**
-  The contract is governed by a DAO , which has the authority to update the list of excluded countries and replace the AccessManager contract itself.

**GeoVersioning**
-  The contract uses a geoVersion state variable to represent the current set of geographic restrictions. When the DAO updates the list of excluded countries, geoVersion is incremented, effectively invalidating all previous access grants.

**Access Granting**
-  Users are granted access by calling the grantAccess function with a signature that is verified off-chain. The signature must be generated by an authoritative signer and is tied to the current geoVersion and the user's wallet address.

**Access Verification**
- The _verifyAccess internal function checks if the provided signature is valid for the current geoVersion and the calling wallet address.

**Access Storage**
- Access is stored in a nested mapping _walletsWithAccess, which maps geoVersion to wallet addresses and their access status (true or false).

**Event Logging**
-  The contract emits an AccessGranted event when a user is granted access, which includes the user's wallet address and the geoVersion.

### 32 . Salt.sol


The Salt contract is an ERC20 token with an additional burn feature. The contract inherits from OpenZeppelin's ERC20 standard implementation and implements an interface ISalt. The contract is intended to be used on the Ethereum blockchain, as indicated by the use of ether units in the constants.

**Initial Supply**
-  The contract has a constant INITIAL_SUPPLY set to 100 million SALT tokens (100 * 1,000,000 ether, where ether is a denomination used in Solidity to specify 1e18 wei, the smallest unit of Ether). Upon deployment, all of these tokens are minted and assigned to the deployer's address.

**Token Burning**
- The contract includes a burnTokensInContract function that allows anyone to burn the SALT tokens held by the contract itself. This function retrieves the contract's SALT balance, burns those tokens, and emits a SALTBurned event with the amount burned.

**Total Burned**
- There is a view function totalBurned that calculates the total amount of SALT tokens burned by subtracting the current total supply from the initial supply.

### 33 . SigningTools.sol

The contract SigningTools that contains a single function _verifySignature. This function is designed to verify that a given messageHash has been signed by a specific signer, whose address is hardcoded as EXPECTED_SIGNER. The purpose of this library is to provide a means of verifying the authenticity of messages, ensuring that they have been signed by the expected authority before being processed further.


**The _verifySignature function takes two parameters:**

a messageHash of type bytes32, which represents the hash of the original message, and a signature of type bytes, which is the signature produced by signing the messageHash with the private key corresponding to EXPECTED_SIGNER.

**The function performs the following steps:*

- It checks that the length of the signature is exactly 65 bytes, which is the standard length of an Ethereum signature consisting of the r, s components, and the v recovery id.

- It uses inline assembly to extract the r, s, and v components from the signature. Inline assembly is used for efficiency and direct access to the EVM's instruction set.

- It calls the ecrecover function, which is a built-in Ethereum function that takes a hash and a signature (v, r, s) and returns the address that was used to sign the hash.

- It compares the recovered address with the EXPECTED_SIGNER and returns true if they match, indicating that the signature is valid and was indeed signed by the expected signer.


### 34 . Upkeep.sol

The Upkeep contract is designed to automate various maintenance tasks for a DeFi ecosystem. It interacts with multiple contracts to manage liquidity, rewards, and token emissions. The contract is non-reentrant and uses OpenZeppelin's SafeERC20 library for safe token transfers.



Ukeep.sol performs the following tasks:

**Token Swap and Burn**
-  Swaps tokens for USDS and burns specified amounts of USDS.
**WETH Arbitrage Profits**
-  Withdraws WETH arbitrage profits, rewarding the caller with a percentage.
**Protocol Owned Liquidity (POL) Formation**
-  Converts a percentage of WETH to USDS/DAI POL.
**POL Formation for SALT/USDS**
-  Converts a percentage of WETH to SALT/USDS POL.
**SALT Rewards**
-  Converts remaining WETH to SALT and sends it to the SaltRewards contract.
**SALT Emissions**
- Sends SALT emissions to the SaltRewards contract.
**Distribute SALT Rewards**
- Distributes SALT from SaltRewards to staking and liquidity emitters.
**Rewards Distribution**
- Distributes SALT rewards from the emitters.
**Collect and Distribute DAO Rewards**
- Collects SALT rewards from DAO's POL, sends a portion to the dev team, and burns the rest.
**DAO Vesting**
- Sends SALT from the DAO vesting wallet to the DAO.
**Team Vesting**
- Sends SALT from the team vesting wallet to the team.

### 35 . ExchangeConfig.sol


The ExchangeConfig contract is  a configuration management contract for a decentralized exchange or DeFi protocol. It is designed to hold references to various components of the protocol, such as token contracts (wbtc, weth, dai, usds), a DAO contract, an upkeep contract, initial distribution and airdrop mechanisms, and vesting wallets for the team and DAO. The contract is owned by a DAO, and only the DAO can modify its parameters.

**Immutable References**
- ExchangeConfig several immutable references to external contracts and tokens (salt, wbtc, weth, dai, usds, managedTeamWallet). These cannot be changed after the contract is deployed.

**Contract Initialization**
- The setContracts function is used to set the DAO, upkeep, initial distribution, airdrop, and vesting wallets. This function can only be called once, as enforced by the requirement that the dao address must be zero before it is set.

**Access Management**
- ExchangeConfig allows for an accessManager to be set, which is responsible for determining which wallets have access to certain protocol components. The walletHasAccess function checks if a wallet has access, with the DAO and Airdrop contracts always having access.

**Events**
- ExchangeConfig emits an event (AccessManagerSet) when a new accessManager is set.


## Centralization Risks


 - Only the DAO can finalize ballots, which means that the DAO has control over when a ballot's results are recognized.

 - Enforces a delay (firstPossibleProposalTimestamp) before which no proposals can be made. This could be seen as a centralizing factor as it temporarily prevents stakeholders from participating in governance.

- The voting mechanism relies on off-chain signature verification, which introduces a centralization risk if the entity responsible for generating signatures acts maliciously or is compromised.

- The owner has the power to whitelist and unwhitelist pools, which could be used to favor certain token pairs or exclude others, potentially impacting the protocol's fairness and market dynamics.

- The clearProfitsForPools function introduces a centralization risk as it can only be called by the Upkeep contract. If the Upkeep contract is controlled by a single entity, they have the power to reset profits at will. 


- The CollateralAndLiquidity contract has exclusive rights to call addLiquidity and removeLiquidity, centralizing control over liquidity management to a single contract.

- The addSALTRewards function can be a point of centralization if the entity responsible for calling it has too much control over when and how rewards are distributed.

- The confirmationWallet has significant power as it can unilaterally confirm or reject proposals. If this wallet is controlled by a single entity, it represents a central point of failure.

## systematic Risks


- The quorum is calculated based on the total SALT staked. If a small group accumulates a large amount of SALT, they could potentially manipulate quorums and influence ballot outcomes.

- Voting power is based on the user's share of staked SALT. If a user changes their stake after voting, they must recast their vote. This could lead to discrepancies in vote power if not properly managed.

- If the cost of acquiring enough SALT to influence a vote is lower than the potential gain from manipulating a vote, it could incentivize economic attacks.

- The voting mechanism relies on off-chain signature verification, which introduces a centralization risk if the entity responsible for generating signatures acts maliciously or is compromised.

- All votes are weighted equally, regardless of the number of tokens held by the voter. This could be seen as a risk if the intention was to weight votes by stake size.

- The hardcoded limits on the number of whitelisted pools and the internal swap size could lead to systemic issues if they are not set appropriately in response to changing market conditions.

- The arbitrage mechanism relies on the assumption that the pools are large and liquid enough to allow for profitable arbitrage without causing significant slippage.

- The bit-shifting technique used for normalization can lead to precision loss, especially for very small amounts of tokens. This could result in suboptimal swaps or failure to perform a swap when necessary.

- The Uniswap pool addresses are immutable, meaning that if there is a need to switch to different pools due to liquidity concerns or other reasons, the contract cannot be updated and a new contract must be deployed.

- The cooldown period is a safety feature, but it also means that if a price feed becomes compromised, it cannot be replaced immediately. This could lead to a period where accurate price data is not available.

- If the emissions rate is too high or too low, it could have adverse effects on the token's economy, such as inflation or deflation.

- The performUpkeep function's reliance on the timeSinceLastUpkeep parameter means that if the upkeep contract fails to call this function regularly, the reward distribution could be delayed or disrupted.

- The balance between the rewardPercentForCallingLiquidation and the minimumCollateralRatioPercent is critical. If the reward is too high relative to the collateral ratio, it could incentivize premature or unnecessary liquidations, destabilizing the protocol.

- Adjusting the percentArbitrageProfitsForStablePOL affects how much profit is reinvested into the protocol versus distributed elsewhere. This could impact the protocol's long-term sustainability and growth.


- Liquilizer contract attempts to mitigate market impact by burning SALT directly. However, large swaps of WBTC, WETH, or DAI to USDS could still affect market prices, especially if the internal swap limits are not properly configured.

-  The StakinRewards uses division for calculating rewards, which can lead to precision loss. This is mitigated by favoring the protocol in rounding decisions, but it could still lead to discrepancies over time. 

- The burnTokensInContract function allows anyone to burn tokens that are sent to the contract. If someone accidentally sends tokens to the contract, those tokens can be burned by anyone, leading to potential loss of funds. 

- Ukeep contract approves an unlimited amount of WETH to the pools contract, which could be risky if the pools contract is compromised.




## Codebase Quality Analysis 





- Each contract serves a specific purpose, promoting modularity and encapsulation of functionalities.
Contracts such as ArbitrageSearch.sol, DAO.sol, and Pools.sol have distinct roles, enhancing code readability and maintainability.

- The use of immutable references in contracts like DAO.sol and ArbitrageSearch.sol ensures consistency and minimizes the risk of unintentional changes.

- The DAO.sol contract serves as a governance mechanism, enabling stakeholders to propose changes and vote on them. This promotes a decentralized decision-making process.


- The DAOConfig.sol contract consolidates parameters governing various aspects of DAO operations. This design choice enhances flexibility and allows for easy adjustments.


- The Proposals.sol contract implements a robust voting mechanism with features like proposal creation, quorum requirements, and finalization logic. This organized approach contributes to transparent and democratic governance.


- The Pools.sol contract efficiently manages liquidity pools, handling operations such as adding/removing liquidity, token deposits/withdrawals, swaps, and arbitrage opportunities. This modular approach simplifies the implementation and maintenance of liquidity-related functionalities.


- The PoolStats.sol contract introduces a well-organized method for tracking arbitrage profits, updating indices, and calculating rewards. This design supports efficient profit distribution and contributes to the overall stability of the system.


- Contracts like CoreChainlinkFeed.sol and CoreUniswapFeed.sol expose external interfaces (e.g., IPriceFeed) for seamless integration with other components, promoting interoperability and ease of use.


- Contracts incorporate error handling mechanisms, such as try-catch blocks, to gracefully handle potential issues, enhancing the overall robustness of the system.


- Naming conventions are consistently applied throughout the codebase, improving code readability and maintainability.


## Learning and insights 

- Understanding the architecture of different contracts within Salty.io, such as DAO, Proposals, and Pools, provides insights into how complex systems are structured in decentralized applications. This helps in grasping the principles of modular design and contract interaction.

- The review highlights the implementation of governance mechanisms, including proposal creation, voting, and execution. Learning how DAOs manage decision-making and execute proposals adds knowledge about governance in decentralized ecosystems.

- The codebase review has shed light on various security measures, such as access control, error handling, and checks for potential vulnerabilities. Learning from these security practices is crucial for writing secure and robust smart contracts.
- Examining configurable parameters related to economic incentives, arbitrage profits, and rewards provides insights into the economic models behind decentralized systems. Understanding the economic considerations is essential for designing sustainable and incentivized ecosystems.
- The integration of various contracts, such as Uniswap, Chainlink, and staking contracts, demonstrates how smart contracts interact with external systems. This knowledge is crucial for building interoperable and connected decentralized applications.
- Understanding how TWAP is calculated using Uniswap v3 pools contributes to skills related to obtaining and processing time-sensitive data in decentralized finance applications.
- The use of bit manipulation techniques in the PoolMath contract demonstrates efficiency in dealing with large numbers within the constraints of Solidity. This knowledge is beneficial for optimizing gas usage in smart contracts.

- The adoption of a bisection search algorithm in _bisectionSearch for finding the optimal arbitrage amount reflects an efficient approach in handling unimodal profit functions.

- Evaluating the clarity of the code and the presence of documentation contributes to an understanding of best practices for code readability and the importance of providing clear documentation for others and future maintainers.





## Archicetaure Recommendation




- Consider implementing a mechanism where multiple entities or a decentralized network can collectively finalize ballots to avoid concentration of power within the DAO.

- Instead of enforcing a fixed delay, explore options for a more flexible proposal timing mechanism that adapts to changing circumstances and prevents undue centralization.

- Evaluate the feasibility of moving signature verification on-chain to reduce reliance on off-chain entities, enhancing the security of the voting mechanism.

- Restrict the owner's ability to whitelist and unwhitelist pools, possibly through a decentralized governance process, to ensure fair treatment of all token pairs.

- Explore ways to decentralize the control over profit resets, ensuring that the ability to invoke clearProfitsForPools is not concentrated in the hands of a single entity.

- Consider allowing multiple contracts or entities to participate in liquidity management to avoid concentration of control within the CollateralAndLiquidity contract.

-  Evaluate options for distributing rewards through a decentralized process, reducing the centralization risk associated with the addSALTRewards function.

- Explore a decentralized approach to proposal confirmation to avoid a single point of failure, possibly by involving multiple confirmation wallets.



- Introduce mechanisms to prevent a small group from accumulating a disproportionately large amount of SALT to manipulate quorums, such as imposing limits on individual holdings.

-  Implement a system to automatically recast votes when users change their stake after voting to maintain the integrity of the voting power distribution.

-  Constantly monitor the cost of acquiring influence and adjust the economic parameters to minimize the risk of economic attacks.

-  Consider implementing a voting system that weights votes based on the number of tokens held by the voter to better align with stake size.

-  Instead of hardcoded limits, explore dynamic adjustments for the number of whitelisted pools and internal swap size to adapt to changing market conditions.

-  Regularly assess and adjust the arbitrage mechanism to ensure its effectiveness, especially in response to changes in pool size and market conditions.

-  Evaluate alternative techniques to address precision loss in token amounts, ensuring optimal swaps and preventing failures in critical situations.


- Instead of approving an unlimited amount, restrict the approval of WETH to the pools contract to a specific and reasonable amount, minimizing potential risks if the pools contract is compromised.


## Conclusion

the thorough audit of the Salty.io protocol as part of the Code4Rena Audit Contest has provided valuable insights into the platform's strengths and areas for enhancement. The team's innovative approach to decentralized governance and liquidity management is commendable, showcasing a deep understanding of the evolving landscape of decentralized finance. The identified centralization risks, including concentrated control over certain functions, have been addressed with constructive recommendations aimed at fostering decentralization and minimizing potential vulnerabilities. Additionally, systematic risks related to governance manipulation and voting power discrepancies have been highlighted, prompting recommendations to strengthen these aspects for a fair and secure governance process. The precision and flexibility of various calculations have been examined, and proposed measures seek to enhance the precision of critical operations. The forward-looking recommendations provided are geared towards continuous improvement, emphasizing the importance of adaptability in the dynamic decentralized finance ecosystem. Overall, the Salty.io team's commitment to transparency and security is evident, and we anticipate the implementation of these recommendations will contribute to the platform's long-term success in the decentralized finance space.



##  Message For Salty Team 



Firstly, I would like to express my appreciation for the opportunity to conduct an in-depth audit of the Salty.io protocol as part of the Code4Rena Audit Contest. It has been a rigorous and enlightening process, and your commitment to transparency and security is commendable.

Throughout the audit, we delved deep into the intricacies of your smart contracts, scrutinizing every line of code to ensure the robustness and integrity of the Salty.io protocol. The complexity and innovation embedded in your decentralized governance and liquidity management mechanisms are notable.

Our analysis revealed several strengths in the protocol, such as the well-structured governance model, the implementation of safety features like cooldown periods, and the efforts made to mitigate economic attacks. We acknowledge the thoughtfulness put into the protocol's design and the mechanisms in place to maintain a fair and efficient ecosystem.

However, as with any sophisticated system, we also identified areas where enhancements could be considered to further fortify the protocol. Addressing potential centralization risks, refining voting mechanisms, and ensuring precision in various calculations were among the key recommendations made to optimize the Salty.io protocol.

It is important to recognize that the audit process is not only about identifying weaknesses but also about providing constructive feedback to facilitate continuous improvement. We are confident that with your team's dedication and the community's support, Salty.io has the potential to become a robust and secure platform for decentralized finance.

Thank you once again for entrusting Code4Rena with the audit of Salty.io. We look forward to witnessing the continued evolution of your project and its positive impact on the decentralized finance landscape.



### Time spent:
21 hours