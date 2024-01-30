# Description overview of The SaltyiO Contest
The `SaltyiO` protocol is a Decentralized Exchange deployed on the Ethereum chain that makes use of `Automatic Atomic Arbitrage` to provide zero fees on all `swaps` and also generates yield. 
The arbitrage system replaces the need for fees to be paid during swaps. Whenever a token is bought or sold in a pool, price fluctuates. This is as a result of market imbalances. Usually this imbalances in the prices of assets are usually corrected by arbitrage Bots in order to stabilize said prices. But with AAA, the protocol performs these arbitrage while a swap transaction is ongoing. Performing both the swap and the arbitrage in the same transaction. These arbitrage profits provides zero fees on swaps on all token pairs for the User. One can say these arbitrage profits are same as the swap transaction fees. After the arbitrage Profit is realized it’s shared as such:

72% of Arbitrage profits are sent to Liquidity Providers and SALT stakers 
23% of Arbitrage profits are used to form SALT/USDS and USDS/DAI protocol liquidity. 
5% of Arbitrage profits is sent to any users which takes part in the Decentralized upkeep of the Exchange.


## The key sections the of SaltyiO protocol for this Audit are:

1. `Price Aggregator` - The protocol fetches prices of assets from 3 different price sources(Chainlink, Uniswap v3 and Salty reserves), compares all three Price Sources and uses the average of the closest two as the effective price.
2. `Staking` - This section implements a staking rewards mechanism that handles users receiving rewards. It has the following contracts:
3. `Pools` - This handles liquidity pools, the addition and removal of liquidity, swaps and Token deposits of a User
4. `Arbitrage` - Searches for arbitrage opportunities during an ongoing swap It has the following contracts:
5. `Stable` - Handles USDS borrows, allowing users deposit WBTC/WETH and borrows USDS
6. `DAO` - Handles creating governance proposals, voting, acting on successful proposals and managing POL.
  As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

# System Overview

`ArbitrageSearch.sol`	
`DAO.sol`	
`DAOConfig.sol`
`Proposals.sol`	
`Parameters.sol`		
`Airdrop.sol`	
`BootstrapBallot.sol`	
`InitialDistribution.sol`	
`PoolUtils.sol`		
`PoolStats.sol`		
`Pools.sol`		
`PoolMath.sol`	
`CoreChainlinkFeed.sol`		
`CoreUniswapFeed.sol`	
`PriceAggregator.sol`	
`CoreSaltyFeed.sol`	
`RewardsConfig.sol`	
`Emissions.sol`		
`SaltRewards.sol`		
`USDS.sol`		
`StableConfig.sol`	
`CollateralAndLiquidity.sol`
`Liquidizer.sol`	
`StakingConfig.sol`
`Liquidity.sol`
`StakingRewards.sol`	
`Staking.sol`	
`ManagedWallet.sol`		
`AccessManager.sol`		
`Salt.sol`		
`Upkeep.sol`		

# Privileged Roles:
 The only significant role in Salty is that of the `DAO`. The role of the DAO cuts across all important configurations in the contracts

# Approach Taken-in Evaluating the Salty iO ProtocolAccordingly:

 I analyzed and audited the subject in the following steps;
I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality. The main goal was to take a close look at the important contracts and how they work together in the Salty iO Protocol.
I started with the following contracts, which play crucial roles in the Salty iO  protocol:
   
`ArbitrageSearch.sol`	
`DAO.sol`
`DAOConfig.sol`
`Proposals.sol`	
`Parameters.sol`		
`Airdrop.sol`	
`BootstrapBallot.sol`	
`InitialDistribution.sol`	
`Pools.sol`		
`PoolMath.sol`	
`CoreChainlinkFeed.sol`		
`CoreUniswapFeed.sol`	
`PriceAggregator.sol`	
`CoreSaltyFeed.sol`	
`RewardsConfig.sol`	
`Emissions.sol`		
`SaltRewards.sol`		
`USDS.sol`		
`StableConfig.sol`	
`CollateralAndLiquidity.sol`
`Liquidizer.sol`	
`StakingConfig.sol`
`Liquidity.sol`
`StakingRewards.sol`	
`Staking.sol`	
`ManagedWallet.sol`		
`AccessManager.sol`		
`Salt.sol`		
`Upkeep.sol`		


I started my analysis by examining the Pool Subsection of the protocol looking at the `Pool.sol` contract, as it serves a pivotal role in the `SaltyiO` protocol. It is dedicated to the storage of the reserves that are used for swaps within the DEX as it handles deposits, arbitrage, and keeps stats for proportional rewards distribution to the liquidity providers.

Then I turned my attention to the Rewards Subsection of the protocol, which is also a crucial component within the `SaltyiO` protocol. Took a deep dive into the `RewardEmitter.sol`, `SaltRewards.sol` and most importantly, `Emissions.sol` which is specifically designed for storing the SALT emissions at launch and then distributing them over time.

Then went over to that Stable Section of the Protocol. Which deals with creation of the protocol Over Collaterized stablecoin and Looked thoroughly at the `Liquidizer.sol` and most importantly the `CollateralandLiquidity.sol` contract as it plays right into the core building block of the protocol. It is a really important contract because all liquidity on the exchange is deposited and withdrawn through it and it also allows users to deposit `WBTC/WETH` liquidity as collateral for borrowing the USDS stablecoin.
 

Then I also check vulnerabilities in Staking Section of the Protocol, the DAO section, Launch section, Arbitrage section and Price sections by reviewing for potential issues such as unchecked external calls, array bounds checking, and reentrancy vulnerabilities. Additionally, ensure that the contract follows best practices for visibility settings, gas usage, and proper handling of edge cases to mitigate potential security risks specific to its implementation.

## Documentation Review:
The documentation was well structured and I read through it thoroughly and asked the sponsor questions in places I didn’t really understand. 

## Compiling code and running provided tests:
 Manuel Code Review In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.
## Line by Line Analysis: 
Pay close attention to the contract's intended functionality and compare it with its actual behaviour on a line-by-line basis.
## Comparison Mode:
 Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

# Architecture Description and Diagram Architecture of the key contracts that are part of the SaltyiO protocol:
## Architecture Description: 
The "SaltyiO" protocol facilitates a decentralized swap system that provides zero fees on swaps due to arbitrage profits. This profits are now shared throughout the protocol to important key players in the system and this ranges from creating Protocol owned liquidity, to rewarding liquidity Providers etc. It provides a stable coin that is backed up my WBTC/WETH as collateral at a collateralization rate of 200%. Also provides liquidation options and a buffer to help users facing sudden harsh market conditions for their collateral.
The protocol provides a voting mecahanism where users makes use of xSALT which is staked SALT token (The protocol token) that launches the exchange right at the start and this goes to show its decentralized nature. 

## Architecture Feedback:
I have but a few recommendations as regarding my study of the protocol. Whenever a User intends on voting I think they should be a minimum amount of xSalt tokens required for casting vote as the protocol did not implement this.  
 
## Formal Verification: 
Consider a professional formal verification of the contract to identify and mitigate any security risks.
Codebase Quality Overall, I consider the quality of the `SaltyiO` protocol codebase to be good. The code appears to be mature and well-developed. I have noticed the implementation of various standards adhere to appropriately.
   Details are explained below:

# Codebase Quality Categories:
  
Code Maintainability and Reliability:
 The `SaltyiO` Protocol contracts demonstrates good maintainability through modular structure, consistent naming. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.

Code Comments: 
During the audit of the `SaltyiO` contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation.
Documentation: 
There Protocols documentation is top tier I must confess, They provide lots of quality external documentation that helps in giving a high level understanding but additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement would create a lot more difference as visualization will give auditors better insight on the project. With considerable enthusiasm. I am confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors.
Testing: 
The audit scope of the contracts to be audited is 80% and it should be aimed to be 100%.
Code Structure and Formatting:
The codebase contracts are well-structured and formatted.

## Systemic & Centralization Risks:
The protocol is widely managed by the DAO leaving little to no room for severe Risks except the DAO is compromised in all forms which is just but a little percentage of probabilty. 

# Conclusion:
Overall the code quality and architecture of the `Saltyio`  protocol contracts are good. The code is well structured, tested and follows solidity best practices. Some opportunities for improvement include adding documentation, more error handling, formal verification. Following best practices in security, decentralization and ongoing maintenance/audits will help ensure the long-term sustainability and success of the protocol. The analysis has identified several systemic and centralization risks present in the `SaltyiO` protocol. These risks span across different contracts, each with its own set of vulnerabilities that could potentially impact the security and decentralization of the overall system. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

## Time spent:
35 hours. 

### Time spent:
35 hours