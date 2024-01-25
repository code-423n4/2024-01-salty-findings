# Salty.IO Protocol Analysis

# Table of Contents
- [1. Overview of the Salty.IO Protocol](#1-overview-of-the-salty-io-protocol)
    - [1.1. Key Features of the DEX](#1-1-key-features-of-the-dex)
    - [1.2. The Core Modules and Main Actors of the Salty.IO Protocol](#1-2-the-core-modules-and-main-actors-of-the-salty-io-protocol)
    
- [2. Analysis of the Protocol Components and Smart Contracts](#2-analysis-of-the-protocol-components-and-smart-contracts)
    - [2.1. The DAO](#2-1-the-dao)
      - [2.1.1. DAO.sol](#2-1-1-dao-sol)
      - [2.1.2. Proposals.sol](#2-1-2-proposals-sol)
      - [2.1.3. DAOConfig.sol](#2-1-3-daoconfig-sol)
      - [2.1.4. Parameters.sol](#2-1-4-parameters-sol)
      - [2.1.5. Principal Actors of the DAO](#2-1-5-principal-actors-of-the-dao)

    - [2.2. Pools](#2-2-pools)
      - [2.2.1. Pools.sol](#2-2-1-pools-sol)
      - [2.2.2. ArbitrageSearch.sol](#2-2-2-arbitragesearch-sol)
      - [2.2.3. PoolsConfig.sol](#2-2-3-poolsconfig-sol)
      - [2.2.4. PoolStats.sol](#2-2-4-poolstats-sol)
      - [2.2.5. PoolUtils.sol and PoolMath.sol](#2-2-5-poolutils-sol-and-poolmath-sol)
      - [2.2.6. Principal Actors of the Pool](#2-2-6-principal-actors-of-the-pool)
    
    - [2.3. Price Feeds](#2-3-price-feeds)
      - [2.3.1. PriceAggregator.sol](#2-3-1-priceaggregator-sol)
      - [2.3.2. CoreChainlinkFeed.sol](#2-3-2-corechainlinkfeed-sol)
      - [2.3.3. CoreUniswapFeed.sol](#2-3-3-coreuniswapfeed-sol)
      - [2.3.4. CoreSaltyFeed.sol ](#2-3-4-coresaltyfeed-sol-)
    
    - [2.4. Stablecoin - Lending/Borrowing](#2-4-stablecoin-lending-borrowing)
      - [2.4.1. USDS.sol](#2-4-1-usds-sol)
      - [2.4.2. StakingRewards.sol](#2-4-2-stakingrewards-sol)
      - [2.4.3. Liquidity.sol](#2-4-3-liquidity-sol)
      - [2.4.4. CollateralAndLiquidity.sol](#2-4-4-collateralandliquidity-sol)
      - [2.4.5. Liquidizer.sol](#2-4-5-liquidizer-sol)
      - [2.4.6. StableConfig.sol](#example)
      - [2.4.7. Principal Actors of the Stable Module](#2-4-7-principal-actors-of-the-stable-module)
    
    - [2.5. Staking](#2-5-staking)
      - [2.5.1. Staking.sol](#2-5-1-staking-sol)
      - [2.5.2. StakingConfig.sol](#2-5-2-stakingconfig-sol)
      - [2.5.3. Principal Actors of the Staking Module](#2-5-3-principal-actors-of-the-staking-module)
    
    - [2.6. Rewards](#2-6-rewards)
      - [2.6.1. RewardsEmitter.sol](#2-6-1-rewardsemitter-sol)
      - [2.6.2. Emissions.sol](#2-6-2-emissions-sol)
      - [2.6.3. SaltRewards.sol](#2-6-3-saltrewards-sol)
      - [2.6.4. RewardsConfig.sol](#2-6-4-rewardsconfig-sol)
      - [2.6.5. Principal Actors of the Rewards Module](#2-6-5-principal-actors-of-the-rewards-module)
    
    - [2.7. Exchange Launch](#2-7-exchange-launch)
      - [2.7.1. BootstrapBallot.sol](#2-7-1-bootstrapballot-sol)
      - [2.7.2. InitialDistribution.sol](#2-7-2-initialdistribution-sol)
      - [2.7.3. Airdrop.sol](#2-7-3-airdrop-sol)
      - [2.7.4. ManagedWallet.sol](#2-7-4-managedwallet-sol)
      - [2.7.5. AccessManager.sol](#2-7-5-accessmanager-sol)
      - [2.7.6. SigningTools.sol](#2-7-6-signingtools-sol)
      - [2.7.7. ExchangeConfig.sol](#2-7-7-exchangeconfig-sol)
      - [2.7.8. Upkeep.sol](#2-7-8-upkeep-sol)
      - [2.7.9. Principal Actors of the Launch Module](#2-7-9-principal-actors-of-the-launch-module)

- [3. Systemic and Technical Risks](#3-systemic-and-technical-risks)

    - [3.1. Use a Secondary Upkeep During Times of High Network Congestion](#use-a-secondary-upkeep-during-times-of-high-network-congestion)
    - [3.2. Monitor the LINK Balance of your Upkeep(s)](#monitor-the-link-balance-of-your-upkeep)
    - [3.3. Deployment to another, More Cost Effective Second-Layer Chain](#deployment-to-another-more-cost-effective-second-layer-chain)
    - [3.4. Make Key Features of the Protocol Pausable or Add Rate Limits](#make-key-features-of-the-protocol-pausable-or-add-rate-limits)
    - [3.5. Upgradeable Contracts](#upgradeable-contracts)
    - [3.6. Use of Third-Party Libraries - Ecrecover](#use-of-third-party-libraries-ecrecover)
    - [3.7. Testing Functions with Complex Logic](#testing-functions-with-complex-logic)
    - [3.8.Monitor Critical Contract Transactions](#monitor-critical-contract-transactions)

- [4. Centralization Risks](#4-centralization-risks)

- [5. Analysis of Protocol Testing Strategies](#5-analysis-of-protocol-testing-strategies)



<a id="1-overview-of-the-salty-io-protocol"></a>
# 1. Overview of the Salty.IO Protocol

Salty.IO is a decentralized exchange (DEX) that provides feeless swaps. The protocol is entirely governed by a DAO and provides a native stablecoin (USDS) that is backed by an overcollateralized pool of WBTC/WETH tokens. 

**There are a few key features that are particularly interesting about this exchange:**

**Feeless swaps and arbitrage profits:**

In order to allow feeless swaps, the protocol needs a sustainable way to remunerate liquidity providers without relying entirely on the SALT protocol token. Salty.IO provides a mechanism they call "Automatic Atomic Arbitrage" or AAA. This generates arbitrage profits from in-transaction swaps on all liquidity pools that belong to the protocol.

By performing those arbitrage trades within the transaction of the actual user swap, the protocol does not need to compete with highly sophisticated arbitrage bots. Furthermore, those trades are highly optimized in terms of gas fees and profits.

To achieve the maximum profit, the protocol uses an iterative bisection search to evaluate the most profitable amounts for the arbitrage trades.

Those profits are then distributed to SALT (governance/protocol token) stakers, to the protocol owned liquidity pools that help to stabilize the peg of the USDS stablecoin and the remaining part goes to use who participate in the upkeep of the exchange.


**Decentralization through DAO governance:**

It is also refreshing sign to see that the team strives to achieve a maximum amount of decentralization by handing over the governance of the protocol to a DAO.

Users can stake protocol/governance SALT tokens in order to participate in the governance process and to be eligible for SALT staking rewards.


**Overcollateralized Stablecoin USDS:**

Not many DEXs provide their own stablecoin, so, this is definitely an interesting approach... The peg is assured by using various measures, like having a high initial overcollateralization rate in the form of WBTC/WETH tokens for users who want to borrow USDS. Also, the protocol owned liquidity pools can provide additional USDS tokens to be burned in the case of undercollateralized liquidations of USDS borrowers.


**Protection against oracle price failure:**

It is also great to see that the protocol does not rely on only one single price feed, but uses three different feeds to provide most accurate prices for BTC and ETH and to protect the protocol against a fatal failure from one of those price feeds.

<a id="1-1-key-features-of-the-dex"></a>
## 1.1. Key Features of the DEX

**Here is a very short summary of the main features of the DEX. A more in-depth explanation of those features is provided in the sections below:** 

* Feeless swaps on all protocol pools
* Automatic Atomic Arbitrage to provide those feeless swaps, to reward SALT stakers and to inject liquidity into the protocol owned pools
* USDS native stablecoin
* Users can borrow USDS by providing WBTC/WETH collateral
* The protocol is entirely governed by a DAO to achieve maximum decentralization.
* SALT rewards for liquidity providers and SALT stakers
* Protocol owned liquidity pools (SALT/USDS and USDS/DAI) to protect the USDS peg in the case of potential losses from undercollateralized liquidations
* Three different price feeds for BTC and ETH from Chainlink, Uniswap v3 and Salty.IO pools to obtain accurate prices
* Airdrop at protocol launch


<a id="1-2-the-core-modules-and-main-actors-of-the-salty-io-protocol"></a>
## 1.2. The Core Modules and Main Actors of the Salty.IO Protocol

The Salty.IO protocol consists of 7 key modules or components that provide the various features of the DEX. Those modules correspond more or less with the main folders in the code repository. There are just a few minor modifications that are detailed in the sections below that provide detailed information about each component and their associated contracts.

**Here is a list of all protocol modules:**

* DAO
* Pools
* Price Feeds
* Stablecoin
* Staking
* Rewards
* Exchange & Launch


**And, those are the different exchange actors. Some of them are exchange contracts, others are exchange users (externally owned accounts):**

**Contracts:**

* DAO
* Upkeep contract
* Liquidizer contract
* BootstrapBallot contract
* CollateralAndLiquidity contract
* InitialDistribution contract
* Airdrop contract


**Users:**

* SALT Staker
* Swap User
* USDS Borrower
* Liquidity Provider
* Airdrop Participant
* Protocol Team


**The image below provides a high-level overview of all the protocol components and the actors involved:**

![Architecture](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/Architecture.png)


<a id="2-analysis-of-the-protocol-components-and-smart-contracts"></a>
# 2. Analysis of the Protocol Components and Smart Contracts


<a id="2-1-the-dao"></a>
## 2.1. The DAO 

**Source:** https://github.com/code-423n4/2024-01-salty/tree/main/src/dao

To achieve a maximum amount of decentralization a DAO has been implemented, which has complete ownership and full control over the Salty.IO protocol.

DAO members (stakers of SALT tokens) can propose various changes related to the Salty.IO protocol and vote on those proposed changes. Proposals include the change of various protocol parameters, modifications to the tokens whitelist, sending SALT tokens, updating the Salty.IO website URL, calling other contracts... and more. The DAO contract handles typical DAO related functionality, such as the creation of proposals, vote tracking and the execution of proposals if the configured requirements (such as the quorum) are met.  

**The DAO consists of the following smart contracts:**

* DAO
* Proposals
* DAOConfig
* Parameters

The image below provides a high-level overview of the DAO with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report. 
     

![DAO](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/DAO.png)


<a id="2-1-1-dao-sol"></a>
### 2.1.1. DAO.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol

This is the main contract of the protocol DAO and it allows to finalize ballots, withdraw arbitrage profits to the Upkeep contract, add liquidity to the protocol owned liquidity (POL), process team rewards and burn SALT tokens... Those features are explained in more detail in the "Functions" section below.

#### Imports:

OpenZeppelin ReentrancyGuard and contracts/interfaces from all other protocol components expect the Launch component.

#### State Variables: 

References to various key components of the protocol, such as: liquidity pools, price aggregator, rewards emitter, liquidizer, configurations for the exchange, pools, staking, rewards and the DAO, as well as the tokens required for the protocol owned liquidity (PO): SALT, DAI and USDS. All of those variables are immutable and internal and they are set in the constructor.

On top of those variables, we also have the IPFS URL of the protocol website and a mapping for all excluded countries both are public.

**Immutable:** pools, proposals, exchangeConfig, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator, liquidityRewardsEmitter, collateralAndLiquidity, liquidizer, salt, usds, dai

**Others:** websiteURL, excludedCountries

#### Functions:

**The external state-changing functions allow to:**

* finalize specific ballots that are in a finalizable state  
* withdraw WETH arbitrage profits from the pools to the Upkeep contract 
* create protocol owned liquidity (POL) with the specified tokens and amounts for the SALT/USDS or USDS/DAI pools. This can only be called by the Upkeep contract. Also, a zapping algorithm is used to use all specified tokens  
* process team rewards from the POL and burn the remaining SALT tokens. Again, this can only be called by the Upkeep contract 
* withdraw a specified amount of tokens from the POL, convert them into USDS and burn them in case the recovered collateral amount from liquidations is not sufficient to burn the required amount of USDS. This function can only be called by the Liquidizer contract  


**Here is a list of the corresponding functions:**

* finalizeBallot(uint256 ballotID)
* withdrawArbitrageProfits(IERC20 weth)
* formPOL(IERC20 tokenA, IERC20 tokenB, uint256 amountA, uint256 amountB)
* processRewardsFromPOL()
* withdrawPOL(IERC20 tokenA, IERC20 tokenB, uint256 percentToLiquidate)


**There is 1 external view function that allows to verify if a specific country is excluded from the whitelist:**

* countryIsExcluded(string country) returns (bool)


All core functions emit corresponding events.


<a id="2-1-2-proposals-sol"></a>
### 2.1.2. Proposals.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol

The Proposals contract allows all SALT stakers to create different types of proposals and to vote for those proposals. DAO members can make proposals to change specific protocol parameters, to add or remove tokens from a whitelist, to send SALT to a specified address, to update the price fee contracts, to call external contracts... and more.


#### Imports:

Various OpenZeppelin contracts as well as interfaces for pool, staking and configuration management.


#### State Variables: 

State is stored for exchange, DAO and pool configuration parameters and for various ballot related data, such as: ballots and open ballots, the next ballot Id, votes for a specific ballot, last user vote for a specific ballot, if a specific user has an active proposal, the last user vote for a specific ballot... and others.

Most of the ballot related variables are mappings, such as: mapping(address=>bool) private _userHasActiveProposal; which verifies if a specific user has an active proposal.

The names of the different state variables are self-explanatory, so no further detail is provided here:

**Public immutable:** staking, exchangeConfig, poolsConfig, daoConfig, salt

**Public ballot related:** openBallotsByName, ballots, nextBallotID, 

**Private ballot related:** _allOpenBallots, _openBallotsForTokenWhitelisting, _votesCastForBallot, _lastUserVoteForBallot, _userHasActiveProposal, _usersThatProposedBallots, firstPossibleProposalTimestamp


#### Functions:  

The external state changing functions allow to vote for a specific proposal and to make different types of proposals, such as proposals for country inclusion and exclusion, proposal to send SALT tokens, to call a specific contract, to change DAO or protocol related parameters... and more. The DAO can also finalize a specific ballot.

The names of the different functions are self-explanatory, so no further detail is provided here.

**External state changing functions:**

* castVote(uint256 ballotID, Vote vote )
* createConfirmationProposal(string ballotName, BallotType ballotType, address address1, string string1, string description) 
* markBallotAsFinalized(uint256 ballotID)	
* proposeParameterBallot(uint256 parameterType, string description)
* proposeTokenWhitelisting(IERC20 token, string tokenIconURL, string description)
* proposeTokenUnwhitelisting(IERC20 token, string tokenIconURL, string description) 
* proposeSendSALT(address wallet, uint256 amount, string description) 
* proposeCallContract(address contractAddress, uint256 number, string description) 
* proposeCountryInclusion(string country, string description)
* proposeCountryExclusion(string country, string description)
* proposeSetContractAddress(string contractName, address newAddress, string description)
* proposeWebsiteUpdate(string newWebsiteURL, string description )
   

**External view functions:**                                                 

* ballotForID(uint256 ballotID) returns (Ballot)
* lastUserVoteForBallot(uint256 ballotID, address user) returns (UserVote)
* votesCastForBallot(uint256 ballotID, Vote vote) returns (uint256)
* requiredQuorumForBallotType(BallotType ballotType) public view returns (uint256 requiredQuorum)
* totalVotesCastForBallot(uint256 ballotID) public view returns (uint256)
* ballotIsApproved(uint256 ballotID) returns (bool)
* winningParameterVote(uint256 ballotID) returns (Vote)
* canFinalizeBallot(uint256 ballotID) returns (bool)
* openBallots() returns (uint256[] memory)
* openBallotsForTokenWhitelisting() returns (uint256[])
* tokenWhitelistingBallotWithTheMostVotes() returns (uint256)
* userHasActiveProposal(address user) returns (bool)
 

**The following events are emitted when a new proposal is created, a ballot is finalized or a vote is cast:**  

* ProposalCreated;
* BallotFinalized;
* VoteCast;


<a id="2-1-3-daoconfig-sol"></a>
### 2.1.3. DAOConfig.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol

This contract allows to configure various DAO specific parameters, such as: initial bootstrapping rewards for the launch of the protocol, the reward in % for the caller of the upkeep, the duration of a ballot, the percentage of POL rewards that are burned... and more.


#### Imports:

The ownable contract from OpenZeppelin and the interface for the DAOConfig.


#### State Variables: 

All state variables are public and they are initialized with default values for various DAO specific configurations.

**The names of the different state variables are self-explanatory, so no further detail is provided here:**
 
* bootstrappingRewards = 200000 ether  
* percentPolRewardsBurned = 50
* baseBallotQuorumPercentTimes1000 = 10 * 1000 
* ballotMinimumDuration = 10 days
* requiredProposalPercentStakeTimes1000 = 500
* maxPendingTokensForWhitelisting = 5 
* arbitrageProfitsPercentPOL = 20 
* upkeepRewardPercent = 5


#### Functions:  

The functions listed below allow to change various DAO specific parameters. Their names are self-explanatory, so, no further details are provided. 

**External state changing functions:**

* changeArbitrageProfitsPercentPOL(bool increase) 
* changeBallotDuration(bool increase) 
* changeBaseBallotQuorumPercent(bool increase) 
* changeBootstrappingRewards(bool increase) 
* changeMaxPendingTokensForWhitelisting(bool increase) 
* changePercentPolRewardsBurned(bool increase) 
* changeRequiredProposalPercentStake(bool increase) 
* changeUpkeepRewardPercent(bool increase)  


**Corresponding events are emitted by all functions:**

* BootstrappingRewardsChanged
* PercentPolRewardsBurnedChanged
* BaseBallotQuorumPercentChanged
* BallotDurationChanged
* RequiredProposalPercentStakeChanged
* MaxPendingTokensForWhitelistingChanged
* ArbitrageProfitsPercentPOLChanged
* UpkeepRewardPercentChanged


<a id="2-1-4-parameters-sol"></a>
### 2.1.4. Parameters.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol

This is an abstract contract from which the DAO contract inherits. It contains only one function: _executeParameterChange(), which allows to change a specific parameter, like the bootstrapping rewards, the minimum collateral ratio, the ballot duration, the staking rewards... and many others. 

The different parameters are defined in the ParameterTypes enum and there are 6 different categories:

* Pool settings: maximumWhitelistedPools, maximumInternalSwapPercentTimes1000 
* Staking configuration: minUnstakeWeeks, maxUnstakeWeeks...
* Reward settings: stakingRewardsPercent, percentRewardsSaltUSDS...
* USDS borrowing; initialCollateralRatioPercent, rewardPercentForCallingLiquidation...
* DAO settings: bootstrappingRewards, ballotDuration...
* Price aggregator: maximumPriceFeedPercentDifferenceTimes1000...


#### Functions:  

As already mentioned, there is only 1 internal function, that receives a specific parameter type and that calls the corresponding function on the specified instance (PoolsConfig, StakingConfig, RewardsConfig...) in order to increase or decrease the selected parameter.  
 
* _executeParameterChange(ParameterTypes parameterType, bool increase, IPoolsConfig poolsConfig, IStakingConfig stakingConfig, IRewardsConfig rewardsConfig, IStableConfig stableConfig, IDAOConfig daoConfig, IPriceAggregator priceAggregator )


<a id="2-1-5-principal-actors-of-the-dao"></a>
### 2.1.5. Principal Actors of the DAO

#### DAO

The DAO is the owner of the DAOConfig contract and it is also the only actor that is allowed to modify DAO specific configurations, like:

* the bootstrapping rewards
* the percentage of POL rewards to be burned
* the duration of a ballot, the base ballot quorum percentage
* the required percentage for a proposal
* the maximum pending tokens to be whitelisted
* the POL arbitrage percentage and the percentage for the upkeep rewards

The DAO is also the only actor that is allowed to perform the following action in the Proposals contract:

* creation of a confirmation proposal
* mark ballots as finalized

All other functions in the Proposals contract (that call the _possiblyCreateProposal internal function) can either be called by the DAO itself or by holders of SALT tokens.

The corresponding functions are listed above in the "2.1.3 DAOConfig.sol" and "2.1.2 Proposals.sol" sections.


#### DAO members - holders of SALT tokens

As mentioned above, holders of SALT tokens are allowed to execute all functions in the Proposals contract that call the "_possiblyCreateProposal" internal function. They are allowed to make the following propositions:

* parameter ballot
* token whitelisting and unwhitelisting
* send SALT tokens
* call a specific contract
* country inclusion and exclusion
* website update
* cast a vote 

The corresponding functions are listed above in the "2.1.2 Proposals.sol" section.


#### Upkeep

The following functions in the DAO contract can only be called by the Upkeep contract:

* withdraw arbitrage profits => withdrawArbitrageProfits(...)
* form a POL => formPOL(...)
* process rewards from a POL => processRewardsFromPOL(...)

#### Liquidizer

The following function in the DAO contract can only be called by the Liquidizer contract:

* withdraw POL => withdrawPOL(...) 


<a id="2-2-pools"></a>
## 2.2. Pools 

**Source:** https://github.com/code-423n4/2024-01-salty/tree/main/src/pools

The provided liquidity pools are another key component of the DEX. Users can provide liquidity and earn rewards, which in turn enables other users to swap tokens. The unique feature of this DEX are zero fee-swaps and automated atomic arbitrage (AAA) trades that make those zero-fee swaps possible. When a user swaps a token, the DEX searches for arbitrage opportunities within all the pools that are maintained by the Salty.IO protocol. Then, various trades are performed and the user receives the target tokens, without having to pay a fee. All of this happens within a single transaction and the entire process is highly optimized and requires only about 70k - 90k of gas. 

The gains from the arbitrage trade(s) are distributed amongst different parties. The largest part goes toward SALT stakers and liquidity providers, another part is used to contribute towards DAO owned liquidity pools (POL): SALT/USDS and USDS/DAI. The utility of those pools will be discussed in more detail in the "Stable Component" section of the protocol. And, finally, the remaining part (currently set at 5%) goes towards upkeep participants. 

**The Pools module consists of the following smart contracts:**

* Pools
* PoolsConfig
* PoolStats
* PoolUtils
* PoolMath

The image below provides a high-level overview of the Pools module with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report.      


![Pools](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/Pools.png)


<a id="2-2-1-pools-sol"></a>
### 2.2.1. Pools.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol

This is the main contract of the Pools component and it allows to add and remove liquidity, to swap tokens and to deposit and withdraw soure tokens for the swap in order to save on gas fees. Those features are explained in more detail in the "Functions" section below.

#### Imports:

Various OpenZeppelin contracts, amongst them, the ReentrancyGuard and a math library. Another important import is the abstract ArbitrageSearch contract that searches for arbitrage opportunities in all protocol pools. And, finally, all the other contracts from the Pools module (listed above) are imported as well. 


#### State Variables: 

The contract uses state variables for the DAO and the collateralAndLiquidity contract and mappings for the user deposits and pool reserves.
 
**Public:** dao, collateralAndLiquidity, exchangeIsLive 

**Private:** _poolReserves, _userDeposits


#### Functions:

**The external state-changing functions allow to:**

* initialize the dao and collateralAndLiquidity state variables - this function is called only once during deployment
* launch the exchange. This updates the arbitrage indices for all whitelisted pools - can only be called by the BootstrapBallot contract
* add and remove liquidity to one of the whitelisted pools. This can only be called by the CollateralAndLiquidity contract to ensure a correct value for the total liquidity is provided.
* deposit and withdraw tokens that can be used as the source token for a swap in order to save on gas fees
* perform a swap from the specified source to a target token. There are 3 different functions that allow to perform a swap when no source token has been deposited (higher gas costs), when a source token has been deposited and the user can also swap from TokanA to TokenB to TokenC

**Here is a list of the corresponding functions:**
 
* setContracts(IDAO _dao, ICollateralAndLiquidity _collateralAndLiquidity) 
* startExchangeApproved()   
* addLiquidity(IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 totalLiquidity) 
* removeLiquidity(IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity)  
* deposit(IERC20 token, uint256 amount) 
* withdraw(IERC20 token, uint256 amount)  
* swap(IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline) 
* depositSwapWithdraw(IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline)  
* depositDoubleSwapWithdraw(IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline) 


**There are also 2 external view function that allow to retrieve the pool reserves and the deposited balance for a specific user:**

* getPoolReserves(IERC20 tokenA, IERC20 tokenB) returns (uint256 reserveA, uint256 reserveB)
* depositedUserBalance(address user, IERC20 token) public view returns (uint256)


**All core functions emit corresponding events:**

* LiquidityAdded
* LiquidityRemoved
* TokenDeposit
* TokenWithdrawal
* SwapAndArbitrage



<a id="2-2-2-arbitragesearch-sol"></a>
### 2.2.2. ArbitrageSearch.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol

This is an abstract contract that is used by the Pools contract in order to find profitable arbitrage opportunities within the different pools of the Salty.IO protocol. 

#### Functions:

**The contract provides 3 pure/view functions that allow to:**

* determine a potentially profitable swap path for the provided source and target tokens. The resulting path will always have the following format: WETH->arbToken2->arbToken3->WETH
* determine, if for given token reserves and a provided amount-in swap value, the most profitable arbitrage swap is either larger (to the right) or smaller (to the left) than the provided value (midpoint)
* perform an iterative bisection search for the value range of 1/128th to 125% of the provided amount-in swap value to determine the most profitable amount-in swap value for the given token reserves of the 3 selected pools                                       


**Here is a list of the corresponding functions:**
 
* _arbitragePath(swapTokenIn, swapTokenOut) returns (arbToken2, arbToken3)
* _rightMoreProfitable(midpoint, reservesA0, reservesA1, reservesB0, reservesB1, reservesC0, reservesC1) internal returns (bool rightMoreProfitable)
* _bisectionSearch(swapAmountInValueInETH, reservesA0, reservesA1, reservesB0, reservesB1, reservesC0, reservesC1) returns (uint256 bestArbAmountIn)
  

<a id="2-2-3-poolsconfig-sol"></a>
### 2.2.3. PoolsConfig.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol

This contract is inherited by the Pools contract and allows to whitelist/unwhitelist pools and to change the maximum number of whitelisted pools and the percentage amount of protocol internal swaps in respect to the reserves held in the corresponding pool.

 
#### Imports:

The contract uses OpenZeppelin's EnumerableSet library to manage the whitelist of pools as bytes32 types. The ownable contract is also used and the ownership is held by the DAO.


#### State Variables: 

The contract uses state variables for the pool whitelist, the underlying pool tokens, the maximum number of whitelisted pools and the maximum internal swap percentage.

**Public:** underlyingPoolTokens, maximumWhitelistedPools, maximumInternalSwapPercentTimes1000  

**Private:** _whitelist (EnumerableSet.Bytes32Set)


#### Functions:

**The external state-changing functions allow to:**

* whitelist and unwhitelist pools
* change the maximum number of whitelisted pools
* change the maximum internal swap percentage


**Here is a list of the corresponding functions:**

* whitelistPool(IPools pools, IERC20 tokenA, IERC20 tokenB) 
* unwhitelistPool(IPools pools, IERC20 tokenA, IERC20 tokenB)
* changeMaximumWhitelistedPools(bool increase)
* changeMaximumInternalSwapPercentTimes1000(bool increase)


**There are also several external view function that allow to obtain information regarding the whitelisting of pools and tokens:**

* numberOfWhitelistedPools() returns (uint256)
* isWhitelisted(bytes32 poolID) returns (bool)
* whitelistedPools() returns (bytes32[])
* tokenHasBeenWhitelisted(IERC20 token, IERC20 wbtc, IERC20 weth) returns (bool)


**All core functions emit corresponding events:**

* PoolWhitelisted
* PoolUnwhitelisted
* MaximumWhitelistedPoolsChanged
* maximumInternalSwapPercentTimes1000Changed


<a id="2-2-4-poolstats-sol"></a>
### 2.2.4. PoolStats.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol

This is an abstract contract that is inherited by the main Pools contract and it allows to keep track of the generated profits (from arbitrage swaps) of each individual pool.

#### State Variables: 

The contract uses state variables for the exchangeConfig, the poolsConfig and the weth token contract. There are also mappings for the arbitrage profits by pool and the arbitrage indices that are involved in a given swap path.
 
**Public:** exchangeConfig, poolsConfig, _weth, _arbitrageProfits, _arbitrageIndicies  


#### Functions:

**The external state-changing functions allow to:**

* reset the arbitrage statistics for each pool - this is called at the end of a performUpkeep call
* update arbitrage indices for all whitelisted pools with the corresponding pool indices that could potentially participate in an arbitrage trade


**Here is a list of the corresponding functions:**
 
* clearProfitsForPools()
* updateArbitrageIndicies()


**There are also 2 external view function that allow to retrieve the generated profit for each whitelisted pool and the arbitrage indices for a specific pool:**

* profitsForWhitelistedPools() returns (uint256[] _calculatedProfits)
* arbitrageIndicies(bytes32 poolID) returns (ArbitrageIndicies)



<a id="2-2-5-poolutils-sol-and-poolmath-sol"></a>
### 2.2.5. PoolUtils.sol and PoolMath.sol

**Sources:** 

* https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol
* https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol

Those are 2 helper libraries that are used by various contracts of the protocol.


#### Functions:

**The functions provided by the PoolUtils library allow to:**

* return the unique poolID for the given two tokens
* return the unique poolID and whether the provided token order is flipped
* perform an internal token swap, whereby amountIn is limited to a certain percentage of the reserves in the specified pool


**Here is a list of the corresponding functions:**
 
* _poolID(IERC20 tokenA, IERC20 tokenB ) returns (bytes32 poolID)
* _poolIDAndFlipped(IERC20 tokenA, IERC20 tokenB ) returns (bytes32 poolID, bool flipped)
* _placeInternalSwap(IPools pools, IERC20 tokenIn, IERC20 tokenOut, uint256 amountIn, uint256 maximumInternalSwapPercentTimes1000 ) returns (uint256 swapAmountIn, uint256 swapAmountOut)


**The functions provided by the PoolMath library allow to:**

* determine the most significant bit of a non-zero number
* determine the maximum most significant bit of the provided values
* determine how much of token0 needs to be swapped to token1 such that the liquidity added has the same proportion as the reserves in the pool after that swap


**Here is a list of the corresponding functions:**
 
*  _mostSignificantBit(uint256 x) returns (uint256 msb)
*  _maximumMSB(uint256 r0, uint256 r1, uint256 z0, uint256 z1 ) returns (uint256 msb)
*  _zapSwapAmount(uint256 reserve0, uint256 reserve1, uint256 zapAmount0, uint256 zapAmount1 ) returns (uint256 swapAmount)



<a id="2-2-6-principal-actors-of-the-pool"></a>
### 2.2.6. Principal Actors of the Pool

#### DAO

The DAO is the only contract that can call the setContracts function in the Pools contract as well as all functions in the PoolsConfig contract:

* Pools::setContracts

* PoolsConfig::whitelistPool 
* PoolsConfig::unwhitelistPool
* PoolsConfig::changeMaximumWhitelistedPools
* PoolsConfig::changeMaximumInternalSwapPercentTimes1000


#### BootstrapBallot

The BootstrapBallot contract calls the startExchangeApproved function on the Pools contract.


#### CollateralAndLiquidity

The CollateralAndLiquidity contract calls the addLiquidity and removeLiquidity functions on the Pools contract.


#### Upkeep

The Upkeep contract calls the clearProfitsForPools function on the PoolStats contract.


#### Swap User

Any user who wants to swap tokens can use the swap related functions in the Pools contract:

* deposit
* withdraw
* swap
* depositSwapWithdraw
* depositDoubleSwapWithdraw


<a id="2-3-price-feeds"></a>
## 2.3. Price Feeds 

**Source:** https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed

The protocol uses 3 different price aggregators to provide correct prices for WBTC and WETH. After fetching prices from all 3 sources, the average price from the closest two is calculated and returned. So, in case any of those 3 aggregators is down or provides incorrect prices, the protocol is still able to return correct token prices.

**The following price aggregators are currently used:**

* Chainlink price feeds for BTC and ETH
* Uniswap v3 30 minute TWAP's from the WBTC/WETH and WETH/USDC pools
* The Salty.IO WBTC/USDS and WETH/USDS pools


**The Price-Feeds component consists of the following smart contracts:**

* PriceAggregator
* CoreChainlinkFeed
* CoreUniswapFeed
* CoreSaltyFeed


The image below provides a high-level overview of the Price-Feeds module with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report.      


![Price-Feed](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/PriceFeeds.png)


<a id="2-3-1-priceaggregator-sol"></a>
### 2.3.1. PriceAggregator.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol

This is the main contract of the Price-Feeds component. It compares the returned prices from 3 different feeds and it returns the average price that is calculated from the 2 closest feeds.


#### Imports:

The Ownable contract from OpenZeppelin is used and the DAO is set as the owner of this contract. All external state changing functions of this contract can only be executed by the DAO.


#### State Variables: 

The contract uses state variables for the 3 price feeds, for the maximum allowed price difference (in percent) between 2 prices, for the minimum delay between 2 price feed changes, and for the expiration timestamp at which a new price feed change can again be performed 
 
**Public:** priceFeed1, priceFeed2, priceFeed3, maximumPriceFeedPercentDifferenceTimes1000, priceFeedModificationCooldown, priceFeedModificationCooldownExpiration


#### Functions:

The external state changing functions allow to set all 3 price feeds at the initial setup, set a specific price feed, set the maximum allow price difference of two feeds and modify delay between price feed changes.

**Here is a list of the corresponding functions:**
 
* setInitialFeeds(IPriceFeed _priceFeed1, IPriceFeed _priceFeed2, IPriceFeed _priceFeed3)
* setPriceFeed(uint256 priceFeedNum, IPriceFeed newPriceFeed) 
* changeMaximumPriceFeedPercentDifferenceTimes1000(bool increase)
* changePriceFeedModificationCooldown(bool increase)

   
**There are also 2 external view function that return the prices for BTC and ETH:**

* getPriceBTC() returns (uint256 price)
* getPriceETH() returns (uint256 price)


**All core functions emit corresponding events:**

* PriceFeedSet
* MaximumPriceFeedPercentDifferenceChanged
* SetPriceFeedCooldownChanged


<a id="2-3-2-corechainlinkfeed-sol"></a>
### 2.3.2. CoreChainlinkFeed.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol

This contract uses Chainlink price oracles to retrieve prices for BTC and ETH.


#### Imports:

The Chainlink AggregatorV3Interface is used for the BTC and ETH price feeds.


#### State Variables: 

The contract uses state variables for the maximum allowed response time from the Chainlink oracles and for the two Chainlink price feeds. 
 
**Public constant:** MAX_ANSWER_DELAY

**Public immutable:** CHAINLINK_BTC_USD, CHAINLINK_ETH_USD


#### Functions:

**The contract provides 2 external view functions that return the BTC and ETH prices:**

* getPriceBTC() returns (uint256)
* getPriceETH() returns (uint256)


<a id="2-3-3-coreuniswapfeed-sol"></a>
### 2.3.3. CoreUniswapFeed.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol

This contract uses Chainlink price oracles to retrieve prices for BTC and ETH.


#### Imports:

This contract uses various interfaces and libraries from the Uniswap v3-core contracts, like:

* IUniswapV3Pool
* FixedPoint96
* TickMath
* FullMath


#### State Variables: 

The contract uses state variables for the Uniswap time weighted average price period (TWAP), which is set to 30 minutes. And, for the two required Uniswap pools, for the 3 required tokens and whether the pool tokens are specified in flipped order.  
 
**Public constant:** TWAP_PERIOD

**Public immutable:** UNISWAP_V3_WBTC_WETH, UNISWAP_V3_WETH_USDC, wbtc, weth, usdc, wbtc_wethFlipped, weth_usdcFlipped


#### Functions:

**The contract provides 2 external view functions that return the BTC and ETH prices:**

* getPriceBTC() returns (uint256)
* getPriceETH() returns (uint256)


<a id="2-3-4-coresaltyfeed-sol-"></a>
### 2.3.4. CoreSaltyFeed.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol

This contract uses the Salty.IO pools to retrieve prices for BTC and ETH.


#### State Variables: 

The contract uses state variables for the Salty.IO pools and for the 3 required tokens. 

**Public immutable:** pools, wbtc, weth, usdc


#### Functions:

**The contract provides 2 external view functions that return the BTC and ETH prices:**

* getPriceBTC() returns (uint256)
* getPriceETH() returns (uint256)


<a id="2-4-stablecoin-lending-borrowing"></a>
## 2.4. Stablecoin - Lending/Borrowing   

**Source:** https://github.com/code-423n4/2024-01-salty/tree/main/src/stable

The Stablecoin module provides lending and borrowing capabilities via an overcollateralized stablecoin (USDS). Users who want to borrow USDS need to provide WBTC/WETH liquidity at an initial collateralization rate of 200%. If the value of the provided collateral drops below 110% of the borrowed USDS amount, the position of the borrower is liquidated.   


**The Stablecoin module consists of the following smart contracts:**

* USDS
* StakingRewards (actually: StakingAndLiquidityRewards)
* Liquidity
* CollateralAndLiquidity
* Liquidizer
* StableConfig


**Remark:** The StakingRewards contract handles rewards for SALT stakers as well as for pool liquidity providers. This is an abstract contract from which the Liquidity contract inherits, which is also abstract. The liquidity contract handles the deposit/withdrawal of pool liquidity (thematically associated with the Pools module). And, finally, the CollateralAndLiquidity contract inherits from the Liquidity contract. This contract allows users to borrow USDS by depositing WBTC/WETH collateral.

Those 3 contracts provide functionality for the Pools, the Staking, the Rewards and the Stablecoin modules. But, because of their connection (inheritance chain) I added all three of them here to the Stablecoin module.

The image below provides a high-level overview of the Stablecoin module with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report.      


![Stable](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/Stable.png)


<a id="2-4-1-usds-sol"></a>
### 2.4.1. USDS.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol

As already mentioned, this is an overcollateralized stablecoin that is pegged to the US dollar. The peg is maintained by a pool of WBTC/WETH tokens with an initial collateralization rate of 200%. If the provided collateral of a borrower falls below 110% of the borrowed USDS amount, the position is liquidated, the liquidated assets are converted into USDS and an equal amount to the borrowed USDS amount is burned to maintain the peg. 

In case of an undercollateralized liquidation (sharp price drop of WBTC and WETH token), DAO owned liquidity from the SALT/USDS and USDS/DAI pools can be burned to make up for the missing amount of USDS tokens from the liquidation.


#### Functions:

**The external state-changing functions allow to:** 

* set the instance of the CollateralAndLiquidity contract - this function is called only once during deployment
* mint USDS tokens for users who provide WBTC/WETH collateral - this can only be called by the CollateralAndLiquidity contract
* burn all USDS tokens that are held by this contract 

**Here is a list of the corresponding functions:**
 
* setCollateralAndLiquidity(ICollateralAndLiquidity _collateralAndLiquidity)
* mintTo(address wallet, uint256 amount)
* burnTokensInContract()


**The following events are emitted, when USDS is either minted or burned:**

* USDSMinted
* USDSTokensBurned


<a id="2-4-2-stakingrewards-sol"></a>
### 2.4.2. StakingRewards.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol

This is an abstract contract that is inherited by the Liquidity contract and it handles the rewards (in SALT tokens) for users who either stake SALT tokens or provide liquidity for one of the pools of the Dalty.IO protocol. There are 2 contracts that derive from this abstract contract:

* Staking (see Staking module below for more details): in this case, reward shares represent the amount of staked SALT tokens.
* Liquidity (see below for more details): in this case, reward shares represent the amount of provided liquidity for a specific pool.


#### Imports:

This contract imports amongst others, the ReentrancyGuard and a math library from OpenZeppelin.


#### State Variables: 

References to the SALT token, the exchange-, staking- and pools configuration. Also, a nested mapping that stores the UserShareInfo data for each user and each poolID, a mapping that stores the total SALT rewards for each pool and finally, a mapping that stores the total shares for each poolID.

**Public:** salt, exchangeConfig, stakingConfig, poolsConfig, totalRewards ( mapping(bytes32=>uint256) ), totalShares ( mapping(bytes32=>uint256) )

**Private:** _userShareInfo ( mapping(address=>mapping(bytes32=>UserShareInfo)) ) 


#### Functions:

**There is one external state-changing function that allows to add SALT rewards for the specified whitelisted pools:** 

* addSALTRewards(AddedReward[] calldata addedRewards) 

**also, two internal state-changing functions that allow to increase and decrease user shares:**
 
* _increaseUserShare(address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown) 
* _decreaseUserShare(address wallet, bytes32 poolID, uint256 decreaseShareAmount, bool useCooldown)


**and several external view function that allow to retrieve the number of shares and rewards per pool and user:**

* totalSharesForPools(bytes32[] poolIDs) returns (uint256[] shares)
* totalRewardsForPools(bytes32[] poolIDs) returns (uint256[] rewards
* userRewardForPool(address wallet, bytes32 poolID)  returns (uint256)
* userRewardsForPools(address wallet, bytes32[] calldata poolIDs) returns (uint256[] rewards)
* userShareForPool(address wallet, bytes32 poolID) returns (uint256)
* userShareForPools(address wallet, bytes32[] calldata poolIDs) returns (uint256[] shares
* userVirtualRewardsForPool(address wallet, bytes32 poolID) returns (uint256)
* userCooldowns(address wallet, bytes32[] calldata poolIDs) returns (uint256[] cooldowns)
 

**All core functions emit corresponding events:**

* UserShareIncreased
* UserShareDecreased
* RewardsClaimed
* SaltRewardsAdded


<a id="2-4-3-liquidity-sol"></a>
### 2.4.3. Liquidity.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol

This abstract contract allows users to add/withdraw liquidity to/from specific pools in the Salty.IO protocol. The contract also inherits from the StakingRewards contract, which manages the SALT rewards for providing pool liquidity.


#### State Variables: 

A reference to the pools contract and the poolID of the WBTC/WETH collateral pool. 
 
**Public:** pools, collateralPoolID 


#### Functions:

**There are two external state-changing functions that allow to deposit and withdraw liquidity:**
 
* depositLiquidityAndIncreaseShare(IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 deadline, bool useZapping) returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity)
* withdrawLiquidityAndClaim(IERC20 tokenA, IERC20 tokenB, uint256 liquidityToWithdraw, uint256 minReclaimedA, uint256 minReclaimedB, uint256 deadline) returns (uint256 reclaimedA, uint256 reclaimedB)
 

**The following events are emitted, when liquidity is either deposited or withdrawn:**

* LiquidityDeposited
* LiquidityWithdrawn


<a id="2-4-4-collateralandliquidity-sol"></a>
### 2.4.4. CollateralAndLiquidity.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol

This contract inherits from the abstract Liquidity contract we just discussed and it allows to deposit/withdraw WBTC/WETH collateral and to borrow USDS stablecoin against that collateral. 

 
#### Imports:

The contract uses OpenZeppelin's EnumerableSet library to keep track of wallets that have borrowed USDS.


#### State Variables: 

Various token and contract references. An EnumerableSet to manage all addresses with borrowed USDS and a mapping that stores the borrowed USDS amount for each user.


**Immutable public and public:** usds, wbtc, weth, stableConfig, priceAggregator, liquidizer, usdsBorrowedByUsers ( mapping(address=>uint256) )    

**Private:** _walletsWithBorrowedUSDS (EnumerableSet.Bytes32Set)


#### Functions:

**The external state-changing functions allow to:**

* deposit and withdraw WBTC/WETH collateral
* borrow USDS against existing collateral
* liquidate undercollateralized users

**Here is a list of the corresponding functions:**

* depositCollateralAndIncreaseShare(uint256 maxAmountWBTC, uint256 maxAmountWETH, uint256 minLiquidityReceived, uint256 deadline, bool useZapping) 
* withdrawCollateralAndClaim(uint256 collateralToWithdraw, uint256 minReclaimedWBTC, uint256 minReclaimedWETH, uint256 deadline) 
* borrowUSDS(uint256 amountBorrowed)
* liquidateUser(address wallet) 


**There are also several external view function that allow to obtain information regarding the user collateral value in USD, the value of the total collateral, the amount of USDS that can be borrowed by a specific user, if a user can be liquidated...:**

* underlyingTokenValueInUSD(uint256 amountBTC, uint256 amountETH) returns (uint256)
* totalCollateralValueInUSD() returns (uint256)
* userCollateralValueInUSD(address wallet) returns (uint256)
* maxWithdrawableCollateral(address wallet) returns (uint256)  
* maxBorrowableUSDS(address wallet) returns (uint256)
* numberOfUsersWithBorrowedUSDS() returns (uint256)
* canUserBeLiquidated(address wallet) returns (bool)
* findLiquidatableUsers(uint256 startIndex, uint256 endIndex) returns (address[] memory)
* findLiquidatableUsers() returns (address[] memory) 


**All core functions emit corresponding events:**

* CollateralDeposited
* CollateralWithdrawn
* BorrowedUSDS
* RepaidUSDS
* Liquidation



<a id="2-4-5-liquidizer-sol"></a>
### 2.4.5. Liquidizer.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol

This contract swaps tokens from collateral liquidations and Protocol Owned Liquidity withdrawals to USDS, which is then burned.


#### State Variables: 

Various references to token and configuration contracts and a variable that holds the amount of USDS that should be burned.

**Public:** wbtc, weth, usds, salt,dai, exchangeConfig, poolsConfig, collateralAndLiquidity, pools, dao, usdsThatShouldBeBurned  


#### Functions:

**The external state-changing functions allow to increment the amount of USDS that should be burned and to perform an upkeep. During an upkeep, WBTC, WETH and DAI are swapped to USDS, which is then burned:**

* incrementBurnableUSDS( uint256 usdsToBurn )
* performUpkeep()


**The incrementBurnableUSDS functions emit the following event:**

* incrementedBurnableUSDS


<a id="2.4.6. StableConfig.sol"></a>
### 2.4.6. StableConfig.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol

This contract allows to modify various parameters that are relevant for the USDS Lending/Borrowing module, like the minimum collateralization ratio, maximum reward percentage for liquidating undercollateralized users... and more. The DAO is the owner of this contract and the only one allowed to change those parameters.


#### State Variables: 

Each parameter is stored in a corresponding uint256 public state variable and initialized with a default value. Here is a list of those variables with their initial values:

* rewardPercentForCallingLiquidation = 5%
* maxRewardValueForCallingLiquidation = 500 USD 
* minimumCollateralValueForBorrowing = 2500 USD
* initialCollateralRatioPercent = 200%
* minimumCollateralRatioPercent = 110%
* percentArbitrageProfitsForStablePOL = 5%


#### Functions:

**Here is a list of the corresponding state changing functions:**

* changeRewardPercentForCallingLiquidation(bool increase)
* changeMaxRewardValueForCallingLiquidation(bool increase)
* changeMinimumCollateralValueForBorrowing(bool increase)
* changeInitialCollateralRatioPercent(bool increase)
* changeMinimumCollateralRatioPercent(bool increase)
* changePercentArbitrageProfitsForStablePOL(bool increase) 
 

**The following events are emitted whenever a parameter is changed:**

* RewardPercentForCallingLiquidationChanged
* MaxRewardValueForCallingLiquidationChanged
* MinimumCollateralValueForBorrowingChanged
* InitialCollateralRatioPercentChanged
* MinimumCollateralRatioPercentChanged
* PercentArbitrageProfitsForStablePOLChanged


<a id="2-4-7-principal-actors-of-the-stable-module"></a>
### 2.4.7. Principal Actors of the Stable Module

#### DAO

The DAO is the only contract that can call state changing functions in the StakingConfig contract.

 
#### CollateralAndLiquidity

This is the only contract allowed to call the mintTo function on the USDS contract and the incrementBurnableUSDS function in the Liquidizer contract. 


#### Liquidity Providers & Borrowers

User that have access to the exchange can call the following functions on the Liquidity contract:

* depositLiquidityAndIncreaseShare
* withdrawLiquidityAndClaim
 
And, the following functions on the CollateralAndLiquidity contract:

* depositCollateralAndIncreaseShare
* withdrawCollateralAndClaim
* borrowUSDS


#### Upkeep

The only contract allowed to call the performUpkeep function in the Liquidizer contract.


#### Anyone

Anyone can call the liquidateUser function in the CollateralAndLiquidity contract and earn a liquidation fee. 

<a id="2-5-staking"></a>
## 2.5. Staking   

**Source:** https://github.com/code-423n4/2024-01-salty/tree/main/src/staking

The Staking component allows users to stake SALT tokens and to receive rewards from SALT emissions, from arbitrage profits and from the initial bootstrapping at the launch of the Salty.IO protocol. There is an unstaking period of currently 52 weeks in order to obtain 100% of the staked SALT. If the user wants to unstake prior to that 52 week period, a fee needs to be paid and the user will only obtain a percentage of the staked SALT tokens. If the user unstakes after 2 weeks (the current minimum), he will receive only 20% of the staked SALT tokens.  

**The Staking module consists of the following smart contracts:**

* Staking
* StakingConfig


**Remark:** In the previous section, we already discussed the StakingRewards and the Liquidity contracts, which also provide functions for the Pools component respectively for the Stablecoin component, so, those contracts won't be discussed here again. 

The image below provides a high-level overview of the Staking module with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report.      


![Staking](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/Staking.png)


<a id="2-5-1-staking-sol"></a>
### 2.5.1. Staking.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol

This is the main contract of the Staking component and it allows to stake and unstake SALT tokens. For each staked SALT token the user receives 1 xSALT token and earns rewards in the form of SALT tokens, which are proportional to the amount of staked tokens.


#### State Variables: 

Allow to manage a list of unstakeIds for each user, the unstake data for a specific unstakeId and a counter that holds the value of the next unstakeId.
 
**Public:** nextUnstakeID

**Private:** _userUnstakeIDs ( mapping(address => uint256[]) ), _unstakesByID ( mapping(uint256=>Unstake) )


#### Functions:

**The external state-changing functions allow to:**

* stake SALT tokens and earn proportional staking rewards.
* initiate an unstake request. Currently, the minimum delay to recover staked tokens is 2 weeks, which enables the user to recover 20% of staked SALT tokens. To recover 100% of the stake, the current unstake delay is 52 weeks. Once, this request has been initiated, no more staking rewards will be earned.
* cancel a pending unstake request and resume staking rewards.
* recover claimable SALT tokens from a completed unstake request.
* send xSALT from the Airdrop contract to an eligible user


**Here is a list of the corresponding functions:**
 
* stakeSALT(uint256 amountToStake)
* unstake(uint256 amountUnstaked, uint256 numWeeks)
* cancelUnstake(uint256 unstakeID)
* recoverSALT(uint256 unstakeID) 
* transferStakedSaltFromAirdropToUser(address wallet, uint256 amountToTransfer)


**There are also several external view function that allow to retrieve information about the amount of xSALT a user holds, pending unstakes... and more:**

* userXSalt(address wallet) returns (uint256)
* unstakesForUser(address user, uint256 start, uint256 end) returns (Unstake[])
* unstakesForUser(address user) returns (Unstake[])
* userUnstakeIDs(address user) returns (uint256[])
* unstakeByID(uint256 id) returns (Unstake)
* calculateUnstake(uint256 unstakedXSALT, uint256 numWeeks) returns (uint256)
 

**All core functions emit corresponding events:**

* SALTStaked
* UnstakeInitiated
* UnstakeCancelled
* SALTRecovered
* XSALTTransferredFromAirdrop


<a id="2-5-2-stakingconfig-sol"></a>
### 2.5.2. StakingConfig.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol

This contract allows to modify staking specific parameters, like the minimum and maximum number of unstake weeks, the minimum percentage of staked tokens a user receives, if he unstakes after the minimum unstake period, and the cooldown period between increasing and decreasing user share to avoid frontrunning of reward distributions.

#### State Variables: 

**The contract uses the following initial values for those public state variables:**
 
* minUnstakeWeeks = 2
* maxUnstakeWeeks = 52
* minUnstakePercent = 20
* modificationCooldown = 1 hours


#### Functions:

**The external state-changing functions allow to modify the state variables listed above:**

* changeMinUnstakeWeeks(bool increase)
* changeMaxUnstakeWeeks(bool increase)
* changeMinUnstakePercent(bool increase)
* changeModificationCooldown(bool increase)


**All functions emit corresponding events:**

* MinUnstakeWeeksChanged
* MaxUnstakeWeeksChanged
* MinUnstakePercentChanged
* ModificationCooldownChanged


<a id="2-5-3-principal-actors-of-the-staking-module"></a>
### 2.5.3. Principal Actors of the Staking Module

#### DAO

The DAO is the only contract allowed call the functions in the StakingConfig contract:

* Pools::setContracts

* PoolsConfig::whitelistPool 

 
#### SALT Staker

Can call: stakeSALT, unstake, cancelUnstake and recoverSALT on the Staking contract.


#### Airdrop Contract

Can call transferStakedSaltFromAirdropToUser on the Staking contract.


<a id="2-6-rewards"></a>
## 2.6. Rewards   

**Source:** https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards

The Rewards component handles the emission of SALT rewards to liquidity providers and SALT stakers (xSALT holders). To make emissions more stable the rewards are emitted at a current rate of 1% per day of the current balance. 

**The Rewards module consists of the following smart contracts:**

* RewardsEmitter
* Emissions
* SaltRewards
* RewardsConfig


**Remark:** In a previous section, we already discussed the StakingRewards contracts, so, this contract won't be discussed here again. 

The image below provides a high-level overview of the Rewards module with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report.      


![Rewards](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/Rewards.png)


<a id="2-6-1-rewardsemitter-sol"></a>
### 2.6.1. RewardsEmitter.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol

This contract stores SALT rewards, which are distributed to eligible users at a daily rate of 1%. All users that hold staking reward shares (SALT stakers and liquidity providers) are eligible for SALT rewards. Two versions of this contract will be deployed:

* The first one handles the rewards distribution for SALT stakers - accumulates rewards for the staked SALT pool 
* The second one handles the rewards distribution for liquidity providers - accumulates rewards for all whitelisted pools       

Both contracts call the addSALTRewards function on the abstract StakingRewards contract that is inherited by the Staking contract.
 

#### State Variables: 

References to various configuration contracts, a boolean that indicates whether the distribution is for liquidity providers or xSALT holders, the max time since the last upkeep call and a mapping of the pending rewards for each poolId.
 
**Immutable/Constant Public:** stakingRewards, exchangeConfig, poolsConfig, rewardsConfig, salt, isForCollateralAndLiquidity, MAX_TIME_SINCE_LAST_UPKEEP = 1 day

**Public:** pendingRewards ( mapping(bytes32=>uint256) )


#### Functions:

**The external state-changing functions allow to add SALT rewards for later distribution and do an upkeep that transfers 1% of the currently held SALT tokens to eligible pools:**

* addSALTRewards(AddedReward[] addedRewards) 
* performUpkeep(uint256 timeSinceLastUpkeep)
 

**There is one external view function that allows to retrieve the pending rewards for eligible pools:**

* pendingRewardsForPools(bytes32[] pools) returns (uint256[])


<a id="2-6-2-emissions-sol"></a>
### 2.6.2. Emissions.sol
                                                                                                       
**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol

This contract stores the SALT emissions from the launch of theSalty.IO protocol and distributes them at a current rate of 0.5% per week to the corresponding RewardsEmitter contract. 


#### State Variables: 

References to configuration contracts, the SaltRewards contract and a constant that holds the value for the max time since the last upkeep call.

**Constant Public:** MAX_TIME_SINCE_LAST_UPKEEP = 1 weeks

**Immutable Public:** saltRewards, exchangeConfig, rewardsConfig, salt


#### Functions:

**There is only one external state-changing functions that transfers 0.5% (per week) of the SALT tokens from the project launch (initial emission) to the  corresponding RewardsEmitter contract (for stake holders respectively liquidity providers) via the SaltRewards contract:**

* performUpkeep(uint256 timeSinceLastUpkeep)
 

<a id="2-6-3-saltrewards-sol"></a>
### 2.6.3. SaltRewards.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol

This contract temporarily holds SALT tokens from the initial emission at launch and from arbitrage profits. Those tokens are then periodically sent to the corresponding RewardsEmitter contract (for xSALT holders respectively liquidity providers). 


#### State Variables: 

References to configuration and RewardsEmitter contracts and the Id for the SALT/USDS pool
 
**Immutable Public:** stakingRewardsEmitter, liquidityRewardsEmitter, exchangeConfig, rewardsConfig, salt, saltUSDSPoolID 


#### Functions:

**There are two external state-changing functions that allow to:**

* send the initial SALT rewards (expected 8 million tokens) to the liquidityRewardsEmitter and the stakingRewardsEmitter.
* perform an upkeep to send rewards to stakers and liquidity providers.  

**Here are the corresponding functions:**

* sendInitialSaltRewards(uint256 liquidityBootstrapAmount, bytes32[] poolIDs) 
* performUpkeep(bytes32[] poolIDs, uint256[] profitsForPools)
 


<a id="2-6-4-rewardsconfig-sol"></a>
### 2.6.4. RewardsConfig.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol

This contract allows to modify rewards parameters, like the percentage of daily rewards distribution, the percentage of weekly SALT emissions, the rewards percentage for xSALT holders and liquidity providers and the percentage of SALT rewards that will be sent to the SALT/USDS pool.


#### State Variables: 

**Store the values for the various reward parameters:**

* rewardsEmitterDailyPercentTimes1000 = 1000 : 1% per day
* emissionsWeeklyPercentTimes1000 = 500 : 0.5% per week
* stakingRewardsPercent = 50%
* percentRewardsSaltUSDS = 10/


#### Functions:

**The external state-changing functions allow to modify the state variables listed above:**

* changeRewardsEmitterDailyPercent(bool increase) 
* changeEmissionsWeeklyPercent(bool increase) 
* changeStakingRewardsPercent(bool increase) 
* changePercentRewardsSaltUSDS(bool increase) 


**For each parameter change, a corresponding event is emitted:**

* RewardsEmitterDailyPercentChanged
* EmissionsWeeklyPercentChanged
* StakingRewardsPercentChanged
* PercentRewardsSaltUSDSChanged


<a id="2-6-5-principal-actors-of-the-rewards-module"></a>
### 2.6.5. Principal Actors of the Rewards Module

#### DAO

The DAO is the only contract allowed call the functions in the RewardsConfig contract:

 
#### Upkeep

This is the only actor allowed to call the performUpkeep functions on the RewardsEmitter, Emissions and SaltRewards contracts.


#### InitialDistribution

Calls sendInitialSaltRewards on the SaltRewards contract

<a id="2-7-exchange-launch"></a>
## 2.7. Exchange Launch 

**Source:** https://github.com/code-423n4/2024-01-salty/tree/main/src/launch

The Exchange/Launch module uses a decentralized ballot to start up the exchange. Eligible airdrop recipients can participate in the vote and if the vote passes, the exchange is launched and SALT tokens are distributed to various contracts and vesting wallets (for the DAO and the Salty.IO team). This module also contains the base contracts in the root folder that allow to manage user access based on their geolocation, to manage the team wallet and to perform the main upkeep.

**The Launch module consists of the following smart contracts:**

* BootstrapBallot
* InitialDistribution
* Airdrop
* ManagedWallet
* AccessMAnager
* SigningTool
* ExchangeConfig
* Upkeep

The image below provides a high-level overview of the Launch module with all smart contracts involved, the core features provided by each individual contract and all involved actors. A detailed description of each smart contract is provided in the following sections of the report.      


![Exchange-Launch](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/Exchange-Launch.png)


<a id="2-7-1-bootstrapballot-sol"></a>
### 2.7.1. BootstrapBallot.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol

This is the main contract of the Launch component and it allows to start up the exchange if a majority of authorized voters cast a "YES" vote. To participate in the ballot, a valid signature is required.


#### State Variables: 

The contract uses state variables for the exchangeConfig and the airdrop contract. For the completion timestamp of the vote, booleans that indicate if the ballot is finalized and if the start of the exchange is approved, uints that hold the number of "YES" and "NO" votes and a mapping that allows to verify if a specific user has already voted. 
 
**Immutable Public:** exchangeConfig, airdrop, completionTimestamp 

**Public:** ballotFinalized, startExchangeApproved, startExchangeYes, startExchangeNo, hasVoted ( mapping(address=>bool) )


#### Functions:

**The external state-changing functions allow to cast a vote to start up the exchange and to finalize the ballot once the voting period has finished:**

* vote( bool voteStartExchangeYes, bytes signature )
* finalizeBallot()


<a id="2-7-2-initialdistribution-sol"></a>
### 2.7.2. InitialDistribution.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol

This is the main contract of the Launch component and it allows to ...


#### Imports:

The contract uses the VestingWallet contract from OpenZeppelin that handles the vesting of the SALT tokens for the DAO (25 million) and the team (10 million).  


#### State Variables: 

The contract uses state variables for the DAO and the team vesting wallets and various references to required contracts.

 
**Immutable Public:** dao, salt, poolsConfig, emissions, bootstrapBallot, daoVestingWallet, teamVestingWallet, airdrop, saltRewards, collateralAndLiquidity


#### Functions:

**There is only one external state-changing functions that is called by the bootstrapBallot contract and performs the initial token distribution:**

* distributionApproved()


<a id="2-7-3-airdrop-sol"></a>
### 2.7.3. Airdrop.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol

This is the main contract of the Launch component and it allows to ...


#### Imports:

The contract uses the EnumerableSet library from OpenZeppelin to manage the list of users that are authorized for the airdrop.


#### State Variables: 

The contract uses state variables for various contracts, an EnumerableSet to manage authorized users, a boolean that indicates is claiming is allowed, the SALT amount for each user and a mapping that allows to verify if a specific user already claimed airdrop tokens.
 
**Immutable Public:** exchangeConfig, staking, salt 

**Public:** claimingAllowed, saltAmountForEachUser, claimed ( mapping(address=>bool) )

**Private:** _authorizedUsers ( EnumerableSet.AddressSet )


#### Functions:

**The external state-changing functions allow to:**

* authorize a specific wallet for the airdrop
* claim airdrop tokens for all eligible users
* claim airdrop tokens for a specific authorized user


**Here is a list of the corresponding functions:**
 
* authorizeWallet( address wallet )
* allowClaiming()
* claimAirdrop()


**There are also 2 external view function that allow to verify if a specific address is eligible for the airdrop and how many addresses are currently authorized for the airdrop:**

* isAuthorized(address wallet) returns (bool)
* numberAuthorized() returns (uint256)
 

<a id="2-7-4-managedwallet-sol"></a>
### 2.7.4. ManagedWallet.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol

This contract allows to modify the address of the team wallet and it provides a main wallet and a confirmation wallet. To change the team wallet address, the main wallet needs to propose a new main wallet and confirmation wallet. The confirmation wallet then needs to send 0.05 ETH to this contract to confirm the change and to activate the timelock (30 days). Once the timelock has expired, the proposed main wallet can call the changeWallets function to finalize the address change.


#### State Variables: 

Addresses for the main and confirmation wallets and for the two proposed wallets. Also, a constant value for the timelock (30 days) and a boolean that indicates if the timelock is active.
 
**Constant Public:** TIMELOCK_DURATION

**Public:** mainWallet, confirmationWallet, proposedMainWallet, proposedConfirmationWallet, activeTimelock


#### Functions:

**The external state-changing functions allow to:**

* propose new wallet addresses - called by the current main wallet
* confirm the wallet proposals and activate the timelock (receive function) - called by the confirmation wallet, which needs to send a minimum of 0.05 ETH
* finalize the change of the wallet addresses


**Here is a list of the corresponding functions:**
 
* proposeWallets(address _proposedMainWallet, address _proposedConfirmationWallet)
* receive() external payable
* changeWallets() 


<a id="2-7-5-accessmanager-sol"></a>
### 2.7.5. AccessManager.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol

This contract grants users access to the exchange depending on their geolocation. Whenever the DAO updates the excluded countries list, the geoVersion of the AccessManager is incremented and users are required to request access to the exchange once again. The contract restricts users from adding liquidity, adding collateral and borrowing USDS, but always allows existing assets to be removed.


#### Imports:

The SigningTools library to verify the validity of the provided signature 


#### State Variables: 

A reference to the DAO, the current geoVersion and a mapping that allows to verify if a specific user has access to the current geoVersion.
  
**Immutable Public:** dao 

**Public:** geoVersion

**Private:** _walletsWithAccess ( mapping(uint256 => mapping(address => bool)) )


#### Functions:

**The external state-changing functions allow to update the current geoVersion whenever new countries are added to the excluded list and grant access to a user:**

* excludedCountriesUpdated()
* grantAccess(bytes signature)


**There is also one external view function that allow to verify if a specific address has access to the exchange:**

* walletHasAccess(address wallet) returns (bool)
  

<a id="2-7-6-signingtools-sol"></a>
### 2.7.6. SigningTools.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol

This is a simple library that allows to verify if the signature that was provided by the user is valid for a given messageHash.


#### Functions:

_verifySignature(bytes32 messageHash, bytes memory signature ) returns (bool)
 

<a id="2-7-7-exchangeconfig-sol"></a>
### 2.7.7. ExchangeConfig.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol

This contract allows to modify the AccessManager contract and to verify if a specific address has access to the protocol components. The DAO and the Airdrop contracts always have access.


#### Imports:

The contract uses the VestingWallet contract from OpenZeppelin that handles the vesting of the SALT tokens for the DAO (25 million) and the team (10 million).  


#### State Variables: 

References to various token and protocol contracts and two VestingWallets for the team wallet and the DAO wallet.

 
**Immutable Public:** salt, wbtc, weth, dai, usds, managedTeamWallet 

**Public:** dao, upkeep, initialDistribution, airdrop, accessManager, teamVestingWallet, daoVestingWallet


#### Functions:

**One external state-changing functions that allows to modify the AccessManager contract:**

* setAccessManager( IAccessManager _accessManager )
  

**There is also one external view function that allows to verify if a specific address has access to protocol components:**

* walletHasAccess(address wallet) returns (bool)


<a id="2-7-8-upkeep-sol"></a>
### 2.7.8. Upkeep.sol

**Source:** https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol

This contract performs an 11-step upkeep that uses up to 2.3 million units of gas. A detailed description of all the actions taken can be found in the contract documentation (see link above). The contract swaps liquidated tokens to USDS and burns them, arbitrage profits are sent to various POL pools and SALT rewards are distributed.


#### State Variables: 

References to various token and protocol contracts and timestamps for previous upkeep calls.
 
**Immutable Public:** pools, exchangeConfig, poolsConfig, daoConfig, stableConfig, priceAggregator, saltRewards, collateralAndLiquidity, emissions, dao, weth, salt, usds, dai  

**Public:** lastUpkeepTimeEmissions, lastUpkeepTimeRewardsEmitters


#### Functions:

**One external state-changing functions that performs the upkeep:**

* performUpkeep() 


**There is also one external view function that allows to verify the amount of WETH that will currently be rewarded for calling performUpkeep:**

* currentRewardsForCallingPerformUpkeep() returns (uint256)


<a id="2-7-9-principal-actors-of-the-launch-module"></a> 
### 2.7.9. Principal Actors of the Launch Module

#### BootstrapBallot 

The only contract allowed to call the distributionApproved function on the InitialDistribution contract and the authorizeWallet function on the Airdrop contract.

 
#### InitialDistribution

The only contract allowed to call the allowClaiming function on the Airdrop contract.


#### User / Airdrop Participant

Can call the vote function on the BootstrapBallotcontract, the claimAirdrop function on the Airdrop contract and grant access on the AccessManager contract.


#### DAO

The DAO is the owner of the ExchangeConfig and the only contract allowed to execute the excludedCountriesUpdated function on the AccessManager contract.           



<a id="3-systemic-and-technical-risks"></a>
# 3. Systemic and Technical Risks

Due to the measures taken by the protocol team, most of the typical risk factors for decentralized blockchain projects have already been addressed and in terms of the protocol architecture there are not really any serious issues that still need to be handled or corrected.

**Here are just a few ideas that could provide some additional minor improvements for the protocol:**  


<a id="use-a-secondary-upkeep-during-times-of-high-network-congestion"></a>
## 3.1. Use a Secondary Upkeep During Times of High Network Congestion

The performUpkeep function on the Upkeep contract can consume up to 2.3 million units of gas if there are 100 whitelisted pools. During times of high network congestion, gas prices can spike quite significantly. 

Those are also times, where certain more critical steps in the upkeep process may need to be called more frequently. High gas prices oftentimes coincide with market crashes and this means, more positions than usual will become undercollateralized and will need to be liquidated...

If gas prices rise to 200 gwei per unit of gas, calling the performUpkeep function would cost ~$1000 if those 2.3 million units of gas need to be paid. If gas prices rise to 400 gwei or more (which already happened several times in the past), the cost would be close to $2000.

Potentially, under those extreme circumstances, calling the performUpkeep function will become unprofitable at times where an even higher frequency of calling that function will be required. This means, that there will be a risk that the protocol will lose some funds due to undercollateralized positions that won't be liquidated in time.

**Possible solutions:**

To mitigate those issues, the team could think about adding secondary upkeep functions to the Upkeep contract that deal only with the bare essentials. A custom logic upkeep could then be registered with Chainlink that regularly checks a "checkUpkeep" function and that executes the secondary performUpkeep function only during those times of high stress and when certain conditions are met (like for example, the gas price rises above a certain limit). 

An additional custom logic upkeep should also be registered with Chainlink for the liquidateUser function in the CollateralAndLiquidity contract. Again, if certain conditions are met (like sharp price drops, rising gas prices...) this upkeep should be called more frequently in order to prevent fund losses from undercollateralized positions that are not liquidated in time.

This function has a much lower gas cost than the performUpkeep function in the Upkeep contract. However, callers of that function are eligible for a maximum payout of $500, which may not be enough, if gas prices rise sharply.

    
<a id="monitor-the-link-balance-of-your-upkeep"></a>
## 3.2. Monitor the LINK Balance of your Upkeep(s)

If the team uses Chainlink Upkeep(s), don't forget to closely monitor the corresponding LINK balances in order to avoid service disruption especially during times of high market volatility and when certain upkeeps may need to be called more frequently.


<a id="deployment-to-another-more-cost-effective-second-layer-chain"></a>
## 3.3. Deployment to another, More Cost Effective Second-Layer Chain

In order to gain access to a larger audience, the team should also consider deploying the protocol to another EVM compatible second layer blockchain or sidechain with lower gas fees.

**Potential candidates are:** Arbitrum, Optimism, Polygon and Avalanche

Again, this would be especially beneficial during times of high network congestion and high gas fees on the Ethereum mainnet.

An interaction/communication between those 2 protocols could even be established by using a service like the Chainlink cross chain interoperability protocol (CCIP). Not all protocol features would need to be provided on the second layer chain. For example, there is no need to deploy the DAO on the second layer chain, but services like token swaps and USDS lending could be provided.

<a id="make-key-features-of-the-protocol-pausable-or-add-rate-limits"></a>
## 3.4. Make Key Features of the Protocol Pausable or Add Rate Limits

Currently, there is no possibility to pause crucial functionalities, like fund withdrawals in case of a severe, unforeseen protocol failure, caused by an attacker or any other means. 

It is possible to either give certain trusted parties the ability to trigger such a circuit breaker or to add programmatic rules that automatically trigger one or more specific circuit breakers for critical protocol features when certain conditions are met.

A slightly softer approach would be for example to limit the amount a user can withdraw within a specific period of time if the protocol is under attack or other extreme circumstances are occurring.

If there is a need to pause crucial protocol features, the team needs to act immediately, which means, this can't be implemented via the DAO governance process. Core team members who are entrusted with the key to a multisig wallet would need to enforce such an action.

This, of course goes against the idea of protocol decentralization and potential clients may not feel comfortable in using such a protocol.

So, there are advantages and disadvantages to both approaches. There is no perfect or single, right decision and the team carefully needs to look at both options and decide if a higher level of potential "disaster" prevention is more favorable than a most optimal level of system decentralization.


<a id="upgradeable-contracts"></a>
## 3.5. Upgradeable Contracts

The contracts of the protocol are currently not upgradeable. This means, if at a later stage bugs need to be fixed are additional features need to be rolled out, things will be more complicated than they would be if the protocol would have been designed with upgradeability in mind:

* More effort is required to deploy a new version
* The rollout needs to be planned very carefully
* Current users need to be informed about the protocol change
* Websites, script... that use the old contracts will be outdated and all contract references need to be modified
* There could be some downtime where the protocol is not accessible


On the other hand, rolling out "upgradeable" smart contracts goes somewhat against the idea and mantra of building decentralized web3 solutions that are "truly" different from traditional web2 applications. If a team can easily upgrade a protocol and change whatever they want and whenever they see fit, this vision is no longer fulfilled.

I personally prefer protocols that are not upgradeable, and I think they hold more credibility. However, the team needs to weigh up all advantages and disadvantages of both approaches.


<a id="use-of-third-party-libraries-ecrecover"></a>
## 3.6. Use of Third-Party Libraries - Ecrecover

OpenZeppelin contracts and libraries are heavily used throughout the protocol. However, in the SigningTools library, the Solidity ecrecover method is used in the _verifySignature function.

This function is known to be vulnerable to signature malleability and OpenZeppelin's ECDSA helper library should be used instead:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol

Currently, ecrecover is used in the protocol to verify signatures in order to prevent unauthorized users from participating in the vote for the launch of the exchange and/or to prevent users from excluded geolocations to use the exchange. 

So, using this function does not pose any risk for the exchange, however it is always better to use trusted and tested libraries that provide additional security to the protocol.  
 

<a id="testing-functions-with-complex-logic"></a>
## 3.7. Testing Functions with Complex Logic 

A few functions in the protocol use more complex logic than most other functions. Special precaution should be taken in the testing approach of those kinds of functions.

There may be some strange edge cases that can only be determined with extensive fuzz testing.

For example, I didn't see any dedicated tests for the functions in the PoolMath library. Those functions are tested indirectly via the functions in different Pool contracts that make use of this library. However, it would be better to provide dedicated unit tests and fuzz tests for the functions in the PoolMath library in order to uncover those otherwise difficult to find potential edge cases.

The Arbitrage abstract contract also holds fairly complex logic. However, this contract has been tested extensively using simple unit tests, integration tests as well as sophisticated fuzz tests.


<a id="monitor-critical-contract-transactions"></a>
## 3.8. Monitor Critical Contract Transactions

It is recommended to monitor critical transactions in order to notice suspicious operations immediately and to give enough time to respond to potential contract exploits.

**Good candidates for transaction monitoring are:**
 
* Spikes in deposits and/or withdrawals of funds
* Unstaking a large number of SALT tokens...

It may also be a good idea to automate the monitoring of critical transactions using an automated monitoring tool, like "Hal Notify", which efficiently tracks on-chain activity and provides real-time notifications about specific configurable events.

Transaction monitoring can also be combined with a security tool like "OpenZeppelin Defender", which allows to automatically trigger protective measures, like pausing a contract if suspicious patterns are detected.

* Hal Notify: https://www.hal.xyz/products/hal-notify 
* OpenZeppelin Defender: https://www.openzeppelin.com/defender 


<a id="4-centralization-risks"></a>
# 4. Centralization Risks

As the protocol uses a DAO governance model it already reaches a fairly high level of decentralization. The only parameter that can be changed outside the DAO governance process is the team wallet, which of course makes totally sense. 

Also, the process of changing the team wallet requires two distinct steps and enforces a 30 day timelock and requires 0.05 ETH to be deposited by the confirmation wallet to the ManagedWallet contract, which mitigates the risk of accidently assigning the wallet to a non-existent address and loosing access.

**However, not even a DAO can provide full decentralization! In my opinion, the problem lies in the following code section:**

```
uint256 userVotingPower = staking.userShareForPool(msg.sender, PoolUtils.STAKED_SALT);
...
_votesCastForBallot[ballotID][vote] += userVotingPower;
```

This means, that one or a small number of powerful stake holders can easily overrule the preferred decision of the majority.

**Let's take a look at the following scenario:**

* 2 protocol owners stake together 100000 SALT tokens 
* 1000 protocol users stake together 50000 SALT 
* An unpopular decision needs to be made...
* The two protocol owners vote "YES"
* 1000 protocol users vote "NO"
* The two protocol owners easily overrule the vast majority of voters

In normal day-to-day operations, this is not an issue, however, during critical times, when "things break", this can become a big problem and "decentralized" suddenly is no longer "decentralized".

Ideally, there should be an additional rule that takes precedence over staking power in the evaluation of the voting result. For example, if the absolute number of "NO" votes is 80% higher than the absolute number of "YES" votes, then the final result should be "NO" independent of the staking power behind the "YES" vote.
 

<a id="5-analysis-of-protocol-testing-strategies"></a>
# 5. Analysis of Protocol Testing Strategies

The protocol has been tested extensively using a variety of unit test, integration tests and fuzz tests. Complex functions, like the ones in the Arbitrage contract have been tested by all three test strategies and different invariants have been evaluated that probably allowed to uncover some of the more exotic edge cases.

As already mentioned, I could not find dedicated tests for the PoolMath library, so those kinds of tests should probably be added to the protocol.

**The current test coverage is at 97.7% in terms of lines of code tested. The image below provides a detailed overview of the test coverage for each smart contract file:**

![TestCoverage1](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/tests1.png)

![TestCoverage2](https://raw.githubusercontent.com/rspadinger/C4-Salty/master/code/images/tests2.png)



### Time spent:
40 hours