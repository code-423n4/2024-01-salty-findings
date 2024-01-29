#  **Advanced Analysis Report for Salty.IO** 

[Audit approach](#1-audit-approach)

[Brief Overview](#2-brief-overview)

[Scope and Architecture Overview](#3-scope-and-architecture-overview)

[Codebase Overview](#4-codebase-overview)

[Centralization Risks](#5-centralization-risks)

[Systemic Risks](#6-systemic-risks)

[Recommendations](#7-recommendations)

[Conclusions](#8-conclusions)

[Resources](#9-resources)


## **1. Audit approach**

- **Documentation & Video Deep Dive**: A comprehensive analysis of provided docs and video wass conducted to understand protocol functionality, key points were noted, ambiguities were discussed with the dev, and possible risk areas were mapped.

- **Static Analysis & Linter Sweep**: Initial vulnerability and error detection was performed using static analyzers and linters and known issues and false positives were excluded.

- **Code Inspection**: Manual review of each contract within defined sections was conducted, testing function behavior against expectations, and working out potential attack vectors (including sponsored ideas). Vulnerabilities related to dependencies and  inheritances, were assessed. Comparisons with similar protocols (including older commits) was also performed to identify recurring issues and evaluate fix effectiveness.

- **Report Compilation**: Identified issues were generated into a comprehensive audit report.
***

## **2. Brief Overview**

- Salty.IO is a fee-free, yield-generating crypto exchange offering a new take on decentralized finance.
- It uses the concept of "Automatic Atomic Arbitrage" (AAA) in which during swaps, the protocol automatically finds and exploits price discrepancies in the market, turning those profits into rewards for liquidity providers, stakers, and the DAO itself.
- It does this while keeping gas costs low by utilizing a predefined set of arbitrage paths and a bisection search algorithm to keep transactions gas efficient. It aims to make swaps cost as low as 69k when using pre-deposited tokens for lower bound, and 93k gas as upper bound.
- Salty.IO also features its own stablecoin, USDS, backed by WBTC and WETH, and is fully controlled by its DAO, making it a truly decentralized platform. 
***
## **3. Scope and Architecture Overview**

For the audit scope, the contracts are sectioned into nine different sections.

### **[3.1. Core](https://github.com/code-423n4/2024-01-salty/tree/main/src)**
Holds the contracts for setting up protocol addresses and wallets, imposing and lifing restrictions, performing upkeeps for the DEX and also includes the protocol's own SALT token contract. 

#### **[AccessManager.sol](https://github.com/code-423n4/2024-01-decent/blob/main/src/UTB.sol)**

- The contract allows for user whitelisting/blocklisting based on DAO selected locations. Restricted users are not allowed to deposit liquidity/collateral and to borrow USDS. They however are allowed to withdraw existing assets, make proposals and vote.

**Grant access to users** - Upon calling the `grantAccess` function, users are granted access for their geoVersion if they provide the correct message signature from the offchain verifier. User's access status is then updated in the `_walletsWithAccess` mapping. Important to note that users have to call this everytime the new list of excluded countries are updated by the DAO.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/11937b85-61c0-45e4-8f42-70925928488d" alt="AccessManager.sol"> sLOC - 35
</p>

#### **[ExchangeConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol)**

- Here, the DAO can set the DEX parameters, contract addresses and provide information about a user's restriction details.

**Set important contracts** - The contract owner calls the `setContracts` and `setAccessManager` functions to sets the addresses of the DAO, upkeep, initial distribution, airdrop, team vesting wallet, dao vesting wallet and the access manager contracts. The `setContracts` function is required to be called once at deployment time.


<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/1b3360f9-bc43-48ac-b5d8-8afcecea2012" alt="ExchangeConfig.sol"> sLOC - 58
</p>

#### **[ManagedWallet.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol)**

- The protocol provides two wallets for proposals. a main wallet that makes the proposal and the confirmation wallet which can accept or deny the request.

**Wallet proposal and change** - Using the `proposeWallets` function, the main wallet can call for a wallet address change, the confirmation wallet sends >=0,05 ether to the contract to accept the proposal, or less to reject the proposal. The to be main wallet then calls the `changeWallets` function to accept the proposal after the 30 day timelock duration is over.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/e7968d34-b2ea-4e34-a5de-0536c02dd518" alt="ManagedWallet.sol"> sLOC - 48
</p>


#### **[Salt.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol)**

- Is the ERC20 token of the protocol. Initial supply is capped at 100 million ether and can only be burned, i.e deflationary. The tokens are sent to the contract, before burning, so ideally the contract should hold no SALT token.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/5fd8fca6-f7e4-4d42-bad6-01917ea36c54" alt="Salt.sol"> sLOC - 16
</p>

#### **[SigningTools.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol)**
- Holds the expected signature address of the protocol. It verifies that the signature message hash was signed by the access manager.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/6f1a17eb-68d1-49e1-802e-5c8d1e37153a" alt="Signingtools.sol"> sLOC - 20
</p>

#### **[Upkeep.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol)**

- Performs maintenance tasks for the protocol. Users are incentivized to perform this by rewarding them with a 5% share of the WETH arbitrage profits generated since the last time upkeep was done. 

**Perform upkeep** - Users can call the `performupkeep` function to perform the 11 step upkeep functions. These are 

> Swapping liquidizer tokens for USDS and burn USDS

> Withdrawing WETH arbitrage profits from the Pools contract and reward the caller with 5%.

> Converting 5% of the remaining WETH to USDS/DAI POL.

> Converting 20% of the remaining WETH to SALT/USDS POL.

> Converting remaining WETH to SALT and send it to SaltRewards.

> Sending SALT Emissions to SaltRewards.

> Distributing SALT from SaltRewards to stakingRewardsEmitter and liquidityRewardsEmitter.

> Distributing SALT rewards from stakingRewardsEmitter and liquidityRewardsEmitter.

> Collecting SALT rewards from the DAO's POL, send 10% to the dev team, and burn 50%. The remaining SALT stays in the DAO.

> Sending SALT from the DAO vesting wallet to the DAO over 10 years.

> Sending SALT from the team vesting wallet to the team over 10 years.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/ef15ca14-18c9-4aef-966d-a28ea33419d2" alt="Upkeep.sol"> sLOC - 169
</p>



### **[3.2. Launch](https://github.com/code-423n4/2024-01-salty/tree/main/src/launch)**

Initiates the protocol startup. A number of users are whitelisted and can vote for or against starting the DEX. Users that have voted receive an airdrop of xSALT, and are eligible to claim it when once the exchange is launched.


#### **[BootstrapBallot.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol)**
- Allows whitelisted users to vote on starting up the DEX or not.

**Voting action** - By calling the `vote` function and voting true or false for the `voteStartExchangeYes`, whitelisted users, as determined by the signature verification process can vote for or against starting the DEX. After voting, the user's wallet is finally authorized for the airdrop.

**Finalization of ballot** - After the voting period is over, the `finalizeBallot` function can be called to approve the initial distribution, and also start the DEX. 

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/0953ee2a-12dc-4a51-bf28-15f3515b261c" alt="bootstrapballot.sol"> sLOC - 50
</p>


#### **[InitialDistribution.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol)**
- Distributes the 100 million ether SALT tokens to various contracts and allows for claiming.

**Distribution Approval** - On ballot finalizing, the `distributionApproved` function is called and SALT tokens are distributed to various contracts. 

> 52 million ether is sent to the Emissions contract.

> 25 million ether to the DAO Reserve Vesting Wallet.

> 10 million ether to the Initial Development Team Vesting Wallet.

> 5 million ether to the Airdrop contract and allow users to claim their airdrops.

> 8 million ether to the SaltRewards contract.

> 5 million ether from the SaltRewards contract to the whitelisted pools.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/737fd311-98d6-4cb8-980c-8dceee4bda7b" alt="InitialDistribution.sol"> sLOC - 51
</p>


#### **[Airdrop.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol)**
- Authorizes users and allows users to claim airdrops.

**Wallet authorization** - After voting for/against starting the DEX, the `authorizeWallet` function is automatically called to add a user to list of authorized users.

**Allow claiming** - After distribution is approved, upon successful completion of the DEX start-up voting process, the `allowClaiming` function is called. It calculates the amount of salt to stake for each user and allows their claiming.

**Airdrop claiming** - Authroized users can call this to have their SALT tokens staked and receive xSALT in return.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/ed892ef6-203e-452d-9e3a-924b3379d748" alt="Airdrop.sol"> sLOC - 56
</p>


### **[3.3 Staking](https://github.com/code-423n4/2024-01-salty/tree/main/src/staking)**
This section handles the staking/unstaking functionality of the protocol. It also includes the contrct for adding liquidity into liquidity pool, rewarding liquidity provides with SALT for doing so.

#### **[Staking.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol)**
- The contracts allows for staking SALT tokens for xSALT rewards at a 1 to 1 ratio.

**SALT staking** - Users call the `stakeSALT` function, provide a given amount of SALT and receive a given amount of xSALT.

**xSALT unstaking** - Users can stake a given amount of their xSALT over a certain duration, between a default of 2 weeks and 52 weeks. They do this by calling the `unstake` function, which sets their unstake status to pending. Users can call the `cancelUnstake` function to return the xSALT and restake their SALT tokens.

**SALT recovering** - After unstaking, users can recover a certain percentage of their staked SALT tokens, depending on howlong their unstake status is pending. At the minimumm, users can claim 20% of their total staked tokens.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/e14fb892-e46f-4c41-bcc8-801f2b4d6aef" alt="Staking.sol"> sLOC - 117
</p>

#### **[StakingRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol)**
- This contract receives staking rewards for staking SALT and liquidity shares.

**User share update** - The `increaseUserShare` and `decreaseUserShare` are called to update the user's xSALT holdings upon staking and liquidity addition. 

**Rewards claim** - Users call the `claimAllRewards` to claim their available SALT rewards from multiple pools. The rewards are added to the user's virtual rewards balance.

**Salt rewards addition** - Salt rewards can be added to whitlisted pools by calling the `addSALTRewards` function.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/5f4ab008-42e8-425b-b147-a03db71f6203" alt="StakingRewards"> sLOC - 169
</p>


#### **[Liquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol)**
- Allows liquidity providers to add liquidity and increase their liquidity share in the StakingRewards pool so that they can receive proportional future rewards.

**Liquidity addition and removal for shares** - Liquidity providers by calling the `depositLiquidityAndIncreaseShare` and `withdrawLiquidityAndClaim` functions to add/remove liquidity to/from specific pools while increaing/decreasing their reward shares.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/db16d690-3780-48ea-a992-e75942e03bb2" alt="Liquidity.sol"> sLOC - 85
</p>

#### **[StakingConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol)**
- This holds the DAO controlled parameters which can be modified. Unstake weeks, unstake percent and modification cooldown time can be gradually increased or decreased.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/3b106138-c9e8-4dff-9c8e-a3ec4459f483" alt="StakingCongfig.sol"> sLOC - 70
</p>


### **[3.4 Rewards](https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards)**
	
This holds contracts that hold and distribute rewards to the staking and liquidity contracts, at a DAO defined rate.

#### **[Emissions.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol)**
- The contract is responsible for storing and distributing SALT emissions over time. Emissions are distributed gradually to the stakingRewardsEmitter and liquidityRewardsEmitter on performUpkeep at a 0,50% default rate.

**Emissions from perform upkeep** - Upon callins the `performUpkeep` function from the exchangeConfig contract, a portion of the contract salt balance is sent to the `saltRewards` contract.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/c38fed9a-35e4-4c40-b346-0892fffb29ca" alt="Emissions.sol"> sLOC - 35
</p>


#### **[SaltRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol)**
- The contract holds the SALT rewards temporarily from emissions contract and arbitrage profits. 

**Liquidity and staking rewards distribution** - During the perform upkeep and the initial rewards distribution processes, SALT tokens held by this contracts are sent to the pools in the `liquidityRewardsEmitter` and the `stakingRewardsEmitter` contracts through the `_sendLiquidityRewards` and `_sendStakingRewards` functions.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/c4e3969d-7aa2-4be7-859e-1c73999ed1e7" alt="saltrewards.sol"> sLOC - 88
</p>


#### **[RewardsEmitter.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol)**
- The contract stores SALT rewards and distributes them at a default rate of 1% per day to those holding shares in the specified StakingRewards contract. Here, salt rewards can be added to be later distributed to specified whitelisted pools, theough a direct call to the `addSALTRewards` and `perfromUpkeep` functons.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/7288da02-dc17-45d0-9e4c-a7d5efc2383f" alt="RewardsEmitter.sol"> sLOC - 89
</p>


#### **[RewardsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol)**
- Allows the DAO to change rewards distribution parameters. Here, the DAO can change the percentage of staking rewards liquidity providers and xSALT holders get, the percentage of SALT rewards sent to the SALT/USDS pool, the weekly and daily percent of SALT that is distributed from the emissions and rewards emitter contracts.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/c3dd6b44-fae0-4d2c-bced-229f936217f7" alt="rewardsconfig"> sLOC - 70
</p>


### **[3.5 Pools](https://github.com/code-423n4/2024-01-salty/tree/main/src/pools)**
Pools contracts holds the contracts that handle swap, arbitrage, pool calculations and so on.

#### **[Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol)**
- The Pools contract is responsible for managing the liquidity reserves used for swaps within the DEX. It handles deposits of tokens, arbitrage, and keeps track of stats for distributing rewards proportionally to liquidity providers.

**Token deposit and withdrawal** - Users can deposit and withdraw tokens into the pools contract in preparation for swaps. They do these by calling the `deposit` and `withdraw` functions.

**Liquidity addition and removal from pools** - Liquidity can be added to or removed from pools through calls to the `addLiquidity` and `removeLiquidity` functions by the collateralAndLiquidty contract.

**Swap and arbitrage** - Users call the `swap` function to swap one token for another in the pools using their already deposited tokens. This process balances the pool reserves, arbitrages the user's tokens for maximum profit and increases the user's internal token balance which they can later withdraw. 

There're also the function to deposit, swap and withdraw tokens directly without making an initial deposit. 

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/a54a3070-491d-4c3d-8b33-8bce58a94d26" alt="Pools"> sLOC - 229
</p>


#### **[PoolStats.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol)**
- The contract accounts for the arbitrage profits generated by pools, information which is used to distribute rewards proportional to the profits generated per pool. Here, the arbitrage indices of pools can also be updated and stats reset during upkeep.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/7c3a303b-a0d9-4292-837c-39adfc1b4c67" alt="PoolStats.sol"> sLOC - 89
</p>


#### **[PoolUtils.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol)**
- Holds information about pool characteristics, and also conducts internal token swaps within the protocol.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/e4c76c46-52c6-4af3-8c10-48ee5188bbdf" alt="PoolUtils"> sLOC - 33
</p>


#### **[PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol)**
- Holds the pool parameters which can be modified by the DAO. Here pools can be whitelisted and unwhitelisted. Maximum amount of whitelisted pools can be changed, internal swap percetntage can be increase or decreased and so on.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/7f6f41cc-8b7e-4588-9c99-dac794ea77f8" alt="Poolsconfig.sol"> sLOC - 96
</p>

#### **[PoolMath.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol)**
- Utility contract that makes the swap calculations to determin how much tokens are needed to be swapped in order to hold the liquidity reserves liquidity proportions.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/30b320c2-6011-4d59-a6f9-b4714eba9a56" alt="PoolMath.sol"> sLOC - 69
</p>




### **[3.6 Arbitrage](https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage)**

Contains the arbitrage search contract which searches the best possible arbritrage trade path to maximize swap profitability.

#### **[ArbitrageSearch.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol)**
- Provides the best available token arbitrage path in a circular form, to generate maximum profit. There are five major paths.

> WETH->WBTC->SALT->WETH -- When a user swaps WBTC for WETH

> WETH->SALT->WBTC->WETH -- When a user swaps WETH for WBTC

> WETH->WBTC->swapTokenOut->WETH -- When a user swaps WETH for swapTokenOut

> WETH->swapTokenIn->WBTC->WETH -- When a user swaps swapTokenIn for WETH

> WETH->swapTokenOut->swapTokenIn->WETH -- When a user swaps swapTokenIn for swapTokenOut

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/b56d6b18-3d0d-44b2-9b01-100746972553" alt="Arbitragesearch"> sLOC - 72
</p>


### **[3.7 Stable](https://github.com/code-423n4/2024-01-salty/tree/main/src/stable)**

Stable contracts deal with the accounting, borrowing and liquidations of the protocol's overcollateralized stablecoin, USDS.

#### **[CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol)**
-  The contract serves as a hub for managing liquidity on the protocol. It allows users to deposit WBTC/WETH to secure USDS loans, handles stablecoin accounting, and determines when positions need to be liquidated. 

**Collateral deposit and withdrawal** - Users can deposit WETH/WBTC as collateral to increase their collateral shares for rewards and the their ability to borrow. They do this by `depositCollateralAndIncreaseShare` function. Upon repayment of their USDS debt, users can call `withdrawCollateralAndClaim` to withdraw their collateral. 

**USDS borrow and repay** - Using existing collateral, a user can borrow USDS by calling the `borrowUSDS` function. Calling this function mints USDS tokens to the borrower. To repay this debt, the user calls the `repayUSDS` function, which sends the user's USDS to the USDS contract where it is eventually burned.

**Debt liquidation** - When user's collateral level falls below the minimum threshold, the user's debt position can be liquidated by calling the `liquidateUser` function. A certain portion of the collateral value is sent to the liquidator, the rest sent to the liquidator contract for later USDS conversion.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/b678e1e0-79fd-43fa-9931-7774e2c9222d" alt="CollateralAndLiquidity.sol"> sLOC - 190
</p>


#### **[Liquidizer.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol)**
- Here, tokens from the withdrawals in the CollateralAndLiquidity contract are sent to be burned. A certain portion of these tokens are kept to serve as a buffer for the protocol against undercollaterilization. The contracts calculates the amount of USDS that can be burned, and burns them during upkeep performance.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/a2de47bc-d700-4da2-b603-fa7f94724744" alt="Liquidizer.sol"> sLOC - 93
</p>

#### **[USDS.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol)**
- USDS is an overcollateralized stablecoin native to the protocol. It can only be minted by the collateralAndLiquidity contract to users who decide to borrow it by depositing WBTC/WETH liquidity as collateral. If the collateral ratio falls below a DAO defined minimum collateral ratio, the position could be liquidated by any user. There is no cap on the amount that can be minted.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/98d645ed-0ad2-4138-ab0d-0c28f1fba0b4" alt="usds.sol"> sLOC - 33
</p>


#### **[StableConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol)**
- This holds confingurations which can be manipulatied by the DAO. Here, liquidity reward value can be modified. SO can the collateral ratio perecent (minimum and maximum) and the percentage of arbitrage profits that is used to form the protocol owned liquidity.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/14aabfdc-b35e-4faa-b740-420f39e31968" alt="stableconfig.sol"> sLOC - 104
</p>




### **[3.9 Dao](https://github.com/code-423n4/2024-01-salty/tree/main/src/dao)**

The DAO contracts are contracts that are used to adjust protocol configurations, create and vote on proposals and perform various protocol upkeep functionalities. 
#### **[DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol)**
- This is the governance contract that allows community members to propose and vote on changes to the project's parameters, token whitelisting/unwhitelisting, token transfers, interactions with other contracts, and website updates. The contract manages the entire voting process, from proposal submission to execution of approved changes.

**Ballot finalization** - Anyone can call the `finalizeBallot` the votes on a specific ballot. This can only be done after the quorum is met and ballot time has ended.

**Protocol owned liquidity formation and withdrawal** - During the upkeep process, SALT/USDS protocol owned liquidty is formed by calling the `formPOL` function. When the protocol is undercollaterized, the liquidizer can call the `withdrawPOL` function to withdraw a specific amount to cover up the needed amount of collateral to cover up user debts.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/6fb36c86-2c42-4ff0-93ce-03b78ccadac5)" alt="dao.sol"> sLOC - 20
</p>


#### **[Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol)**
- Through this contract, SALT stakers can make proposals and cast their votes on said proposals. These proposals can be made to modify protocol parameters, token management, contract interactions, and website updates. For ballot integrity, voters credentials are validated. For flexibility, users are allowed to alter their votes. Once the votes 

**Proposal creation** - Various proposals exist that the users can call including sending an amount of SALT to a wallet, including and excluding a country, updating websites and so on. Functions for these proposals exist individually.

**Vote casting** - Users call the `castVote` function on an open proposal. As far as the proposal is live, the user can change his position by calling the `castVote` function also.


<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/b1b668b2-a6c3-493b-a661-648e8d9dac98" alt="proposals.sol"> sLOC - 267
</p>


#### **[DAOConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol)**
- This contract allows for changing the DAO contracts parameters. Ballot duration, upkeep reward percent, maximum number of tokens to whitelist, base quorum percent and so on can be increased or decreased as the dao sees fit.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/90ccac37-a175-4f88-876e-4ac502ee7ce0" alt="daoconfig.sol"> sLOC - 134
</p>



#### **[Parameters.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol)**
- This contract holds all the various protocol parameters that can be modified.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/3fcb2a5d-90d0-43d8-ba8a-0bf15d90400b" alt="parameters.sol"> sLOC - 93
</p>


### **[3.9 Price-Feeds](https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed)**

Holds the contracts that return needprotocol prices. The protocol uses chainlink, uniswap and salty reserves to generate and compare token prices. The price feeds can be updated by the DAO once every 35 days at most.

#### **[CoreChainlinkFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol)**
- This contract uses chainlink oracle to retrieve BTC and ETH prices for WBTC and WETH prices. The returned prices are converted to 18 decimals and there's a delay of 60 minutes enforced.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/d8d94d38-6b57-487c-b51e-189ae223d001" alt="CoreChainlinkFeed.sol"> sLOC - 47
</p>


#### **[CoreUniswapFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol)**
- This provides prices based on Uniswap v3 pools for WBTC and WETH. There's a 30 minutes TWAP period to resist price manipulation.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/343347c3-44d1-406d-a437-fe213d353161" alt="CoreUniswapFeed.sol"> sLOC - 87
</p>


#### **[CoreSaltyFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol)**
- This provides BTC and ETH prices using salty pools.

<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/6c705204-81dc-4ee1-b8fd-4442e6b38d17" alt="CoreSaltyFeed.sol"> sLOC - 34
</p>


#### **[PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol)**
- This compares prices returned by salty pools, uniswap pools and chainlink feed to provide the most likely accurate prices. A comparison is made of the three price feeds to account for downtime or manipulations in any of the feeds. The pricefeeds can can also be changed by the DAO for a specified cooldown period
<p align="center">
    <img width= auto src="https://gist.github.com/assets/112232336/6db086d2-7d90-4124-a502-49fddb417119" alt="PriceAggregator.sol"> sLOC - 133
</p>


***
## **3. Codebase Overview**

- **Audit Information** - For the purpose of the security review, Salty.IO comprises thirty-five smart contracts in scope totaling over 3000 SLoC. Its core design principle is composition, enabling efficient and flexible integration. It is scheduled to be deployed only on mainnet, hence contracts are compiled with solidity version 0.8.22. Chainlink oracle, uniswap and salty pool are used to generate token prices. 

- **Documentation and NatSpec** - The codebase is divided into nine major sections - core, arbitrage, dao, launch, pool, price-feed, reward, stable and staking sections. There provided documentation provided a brief overview of what each section is about withut too much details about each individual contracts. The video explanation also did a good job breaking down the sections and their functions. The contracts are well commented (not necessarily to NatSpec), and explanations where given as to expected functionalities of each function. These made the audit process a bit easier.

- **Gas Optimization and readability** - Gas optimization seems to be a core part of the codebase's development, as evident from the goal of meeting between 69k - 93k gas costs for swaps. Ultimately, a number of tradeoffs had to be made to balance out gas costs and also made the code more readable. In this respect, the abscence of Yul assembly, which is very gas efficient albeit hard to audit and prone to errors is understandable. However, the use of require/revert error and strings over custom errors seems to go against the narrative, since custom errors are arguably more readable and also gas efficient.

- **Token Support** - The protocol mainly works with ERC20 tokens. WBTC, WETH, SALT, xSALT, DAI and USDS are the main tokens supported by the protocol. Other tokens including non-standard erc20 tokens can be whitelisted by the dao and be swapped by users. Fee on transfer tokens however are not supported. 

- **Testing** - The overall test coverage is about 99% and each section implements various test ideas. Consequently, this improved the modularity of the codebase and helped eliminate most of the basic bugs. There are no invariant or fuzz tests implemented.

- **Blocklist Implementations** - The protocol implements restictions on certain users based on geographical locations. Accounts from countries like Russia, North Korea, Syria, United Kingdom and so on are resrtricted from accessing the protocol. The DAO can add/remove these restrictions.  

- **Attack Vectors** - Various points of attack exists for a protocol of this size. Token pricing manipulations, Issues from non standard ERC20 tokens like Link, logical errors that could affect rewards distribution, dao hijacks and so on.    

***

## **4. Centralization Risks** 
Although the protocol aims to be fully decentralzied and to a great effect, steps to ensure this were taken. A bit of centralization still exist in certain core contracts e.g ExchangeConfig.sol. This however doesn't seem to pose a lot of risk to the protocol as these roles can be transferred to the DAO after deployment. Ultimately, the only source of centralization can come from the DAO itself, in cases of malicious DAO takeover.

***
## **5. Systemic Risks** 

- Voting influencing through bribery/tokeninzing attacks, sybil attacks.
- DAO engagement issues in which proposals do not meet quorum as very few people are participating.
- External dependencies - Openzeppelin contract, uniswap, chainlink and so on.
- Smart contract bugs which at best might be contained to erring contract and at worst could affect entire protocol.
- Non standard erc20 tokens are sources of protocol manipulation. WBTC, one of the protocol's core token is pausable, which can cause issues with usds debt repayment and borrower liquidation. Important to also note that SALT token has deflautionary mechanics which can pose issues to protocol accounting.

***
## **6. Recommendations**

- The codebase should be sanitized, and the contracts without comments should be commented to make it easier to understand, audit and use. Those with comments should also be brought up to date to conform with NatSpec. 

- Use of `approve` methods are deprecated, they can be replaced with `safeIncreaseAllowance` and `safeDecreaseAllowance` to protect from tokens that have issues with approve.

- A sweep function should be put in place in the `ManagedWallet` contract to avoid losing the ether sent to the contract of wallet approvals. Also, resetting the `proposedMainWallet` and `proposedConfirmationWallet` to zero upon rejection of the proposal can be a great help. That way, a new wallet can be proposed without having to first allow the previous wallet. 

- Two step variable and address updates should be implemented. Most of the setter functions implement the changes almost immediately which can be jarring for the contracts/users. Adding these fixes can help protect from mistakes and unexepected behaviour.

- Using BTC prices for WBTC leaves the protocol vulnerable to risks from WBTC depegging, important to keep that in mind. Also, for situations arising from the pausable nature of WBTC and other black swan events, a pause functionality can be a great help for the protocol.

- Quorum votes should be cached in the ballot struct to prevent issues arising from changes in quorum due to changes in token supply.

- A veto function should be considered. Although this increases centralization, it's needed to protect the protocol from malicious DAO hijack or 51% attack.

- A deauthorize wallet function can also be considered for the initial airdrop distirbution.

- The one user per proposal rule can be reconsidered for the DAO as a proposal's inability to meet quorum can put cause that newer, possibly more important proposals cannot be passed by the DAO during this period.
***
## **7. Conclusions**

-  In conclusion, for a one-man job, the codebase is pretty well designed which is commendable, but a number of risks were identified and they need to be fixed. Recommended measures should be implemented to protect the protocol from potential attacks. Timely audits and codebase cleanup should be conducted to keep the codebase fresh and up to date with evolving security times.
***
## **8. Resources**

- [C4 ReadMe](https://code4rena.com/audits/2024-01-saltyio#top)
- [Protocol overview documentation](https://docs.salty.io/)
- [Protocol technical documentaion](https://tech.salty.io/)
- [Contract video analysis](https://www.youtube.com/watch?v=bmAjm8J3q3Y)


### Time spent:
48 hours