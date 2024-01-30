## 01] Summary Of Codebase
Salty.io is a decentralized exchange that is community owned and governed by a Decentralized autonomous Orgamization(DAO). It generates yield by Automatic Atomic Arbitrage which forms protocol owned liquidity that incentivices liquidity providers and stakers.
The DEX is designed for a seamless and extensive decentralized trading experience which comprises diverse modules. The arbitrage functionality not only identifies arbitrage opportunities during user swaps but also executes the arbitrage swaps within the Pools.sol contract. Governance responsibilities are handled by the dao module, enabling proposal creation, voting, successful proposal execution, and managing Protocol Owned Liquidity (POL) through adjustable parameters stored in Config.sol contracts. The launch feature oversees the initial airdrop, token distribution, and a decentralized vote by airdrop recipients to kickstart the DEX and distribute SALT. At the core of trading infrastructure is the pools module, managing liquidity pools, swaps, arbitrage, and user token deposits to optimize gas costs. The redundant price aggregator in the "/price_feed" module utilizes various sources to provide accurate BTC and ETH prices for the overcollateralized stablecoin while rewards handles global SALT emissions, rewards distribution to liquidity providers and stakers, and incorporates a mechanism to mitigate rewards volatility over time. The protocol also integrates features like stablecoin management , staking rewards , and essential contracts such as the SALT token, default AccessManager, and the Upkeep contract within the root. This inclusive design ensures a resilient and multifaceted decentralized exchange ecosystem.

#### 1.1] Description
**Staking** - Wallets which have access can deposit liquidity into the protocol which increases their user shares in the pool and get their rewards on withdrawal or claimimng. `StakersRewardsEmitter` emits 1% daily to `liquidityRewardsEmitter` to prevent volatility. 

**LIquidity**_ - Liquidity providers provide liquidity into the WETH/WBTC pools at a rate of 200% and a liquidation threshold of 110% of their collateral. The Liquidity providers get rewards in return.

***PriceFeeds***- The DEX uses chainlink, Uniswap and CoreSalty pricefeeds to determine price. The prices are aggregated by finding the average of two of the most close prices from the three prices.

**DAO** - Salty.io is managed by a DAO that allows users to propose and vote on various governance actions such as handling creation of governance proposals, voting, acting on successful proposals, managing POL changing parameters, whitelisting/unwhitelisting tokens, sending tokens, calling other contracts, and updating the website


## 02] Architecture Improvements

#### 2.1] Readability 
Reduce the length of constructors, functions and comments to improve readability. Having long lines makes it hard to understand what the code does and having it with a maximum of 80 characters will make it not only readable but a good software engineering practice.

#### 2.2] Natspec
Adding Natspec on core functions will make it easier for developers,users,contributors and security researchers to understand the code and the intended functionality of the exchange. Avoid using fixed values on Natspec on functions that are subjected to change eg if the DAO can update the rate via the config, let the comment not use the already set rate.

   
    

## 03] Centralization Risk
The system is fully decentralized and there is no instance of a decentralization risk.

## 04] Time Spent
 Day 1 - Day 4 Reading the documentation and watching salty.io walkthrough video. 

 Day 5 - Day 7 Skimming through the code, checking best practices and ensuring it is doing whats intende to do. 

 Day 8 - Day 10 Finding bugs, testing and referring back to the documentation 

 Day 11 - Day 13 Finding bugs, writing reports and Proof of Concepts.

#### Total hours Spent
123 hours






### Time spent:
123 hours