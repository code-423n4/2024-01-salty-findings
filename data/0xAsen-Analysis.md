### Description/Project overview - Comprehensive

The Salty protocol is a complex DEX with additional services including liquidity provision, staking, borrowing, and a governance model. Its core purpose is to provide a secure and efficient platform for trading, liquidity pooling, and yield generation while maintaining a stable governance system. 

Here's an overview of its key components:

### **Decentralized Exchange (DEX)**

- **Liquidity Pools:** The protocol features liquidity pools for various token pairs, allowing users to provide liquidity and earn fees from trades.
- **Arbitrage Mechanism:** Includes an arbitrage mechanism to maintain price balance across pools, leveraging the protocol's own liquidity pools and external price feeds for efficiency.
- **Token Swaps:** Users can swap tokens within the ecosystem, with the pools facilitating these exchanges.

### **Staking and Rewards**

- **Staking:** Users can stake the native SALT token or provide liquidity to earn rewards. Staked tokens are represented by xSALT tokens.
- **Reward Distribution:** Rewards are distributed to stakers and liquidity providers, managed by contracts like **`StakingRewards`** and **`SaltRewards`**.
- **Rewards Emission:** **`RewardsEmitter`** contracts handle the gradual distribution of rewards, typically at a rate of 1% per day, ensuring a stable yield.

### **Stablecoin and Collateralized Borrowing**

- **USDS Stablecoin:** The protocol includes a stablecoin (USDS), which users can borrow against collateral - WBTC/WETH.
- **Collateral Management:** **`CollateralAndLiquidity`** manages collateral deposits and withdrawals, maintaining the necessary collateralization ratios.
- **Liquidation Process:** Undercollateralized positions can be liquidated, with rewards provided to liquidators.

### **Governance and DAO**

- **DAO Governance:** The protocol is governed by a Decentralized Autonomous Organization (DAO), allowing token holders to propose and vote on changes.
- **Voting Mechanism:** Proposals cover a wide range of governance aspects, from parameter adjustments to token whitelisting.
- **Contract Upgrades:** The DAO can propose updates to contracts, ensuring the protocol remains adaptable and secure.

### Main contracts and their analysis

**PriceAggregator.sol**

The "PriceAggregator" contract provides robust and reliable BTC and ETH price information by aggregating data from the three distinct price feeds. It ensures reliability and accuracy in price data by comparing and validating the outputs from multiple sources.Functionality:

- **setInitialFeeds:** Configures the initial price feeds (**`priceFeed1`**, **`priceFeed2`**, **`priceFeed3`**). This function is callable only once.
- **setPriceFeed:** Updates one of the three price feeds. This action is governed by a cooldown period to allow thorough evaluation of any new feed before further changes.
- **changeMaximumPriceFeedPercentDifferenceTimes1000:** Adjusts the maximum permissible percent difference between the closest two price feeds, beyond which the aggregated price is considered invalid.
- **changePriceFeedModificationCooldown:** Modifies the cooldown period for updating price feeds, allowing for flexibility in response to changing market conditions or feed performance.
- **_absoluteDifference:** A utility function to calculate the absolute difference between two numbers.
- **_aggregatePrices:** Determines the aggregated price from three inputs, discarding outlier values that significantly deviate from the other two.
- **_getPriceBTC:** Attempts to retrieve the BTC price from a specified price feed, handling any exceptions to ensure stable contract operation.
- **_getPriceETH:** Similar to **`_getPriceBTC`**, but retrieves the ETH price from a specified feed.
- **getPriceBTC:** Public function that aggregates BTC prices from all three feeds and ensures the final price is valid and consistent.
- **getPriceETH:** Corresponds to **`getPriceBTC`**, but for obtaining the ETH price.

**CoreUniswapFeed.sol**

The "CoreUniswapFeed" contract provides time-weighted average prices (TWAPs) for WBTC and WETH using Uniswap V3 pools. The TWAPs are calculated over a 30-minute period to mitigate the risk of price manipulation.

Functionality:

- **_getUniswapTwapWei:** Calculates the TWAP for a given Uniswap V3 pool using Chainlink oracles, returning the price with 18 decimals.
- **getUniswapTwapWei:** Public function wrapping **`_getUniswapTwapWei`**. It uses a try/catch block to handle any errors and returns zero in case of failure.
- **getTwapWBTC:** Calculates the TWAP for WBTC using both the WBTC/WETH and WETH/USDC Uniswap V3 pools. It handles different token orderings in the pools and returns zero if either TWAP is invalid.
- **getTwapWETH:** Similar to **`getTwapWBTC`**, but calculates the TWAP for WETH using the WETH/USDC Uniswap V3 pool.
- **getPriceBTC:** External function that provides the WBTC price as a 30-minute TWAP.
- **getPriceETH:** Corresponds to **`getPriceBTC`**, but returns the WETH price.

**CoreSaltyFeed.sol**

The "CoreSaltyFeed" contract gets BTC and ETH prices using the liquidity pool reserves from Salty.IO. It ensures that price information within the system is derived directly from the market activity on Salty's own pools.

Functionality:

- **getPriceBTC:** Provides the current BTC price in USD. This is calculated based on the reserves of WBTC and USDS in the corresponding pool. It returns zero if the reserves are below a minimal threshold (defined as **`DUST`** in **`PoolUtils`**) to ensure validity.
- **getPriceETH:** Offers the current ETH price in USD, determined by the reserves of WETH and USDS in the pool. Similar to **`getPriceBTC`**, it returns zero if the pool reserves are insufficiently low.

**CoreChainlinkFeed.sol**

The "CoreChainlinkFeed" contract is responsible for fetching the latest BTC and ETH prices using Chainlink oracles. It interfaces with Chainlink's  price feeds.

Functionality:

- **latestChainlinkPrice:** Fetches the latest price from a specified Chainlink oracle. This function converts the price from Chainlink's 8 decimal format to a 18 decimal format. It ensures data freshness by checking if the latest update was within the last 60 minutes, returning zero if this condition is not met or in case of any failure.
- **getPriceBTC:** Provides the latest BTC price in USD. It calls **`latestChainlinkPrice`** using the BTC/USD Chainlink feed.
- **getPriceETH:** Offers the latest ETH price in USD, utilizing the **`latestChainlinkPrice`** function with the ETH/USD Chainlink feed.

**USDS.sol**

The "USDS" contract is an ERC20 token contract representing a stablecoin, USDS, which users can borrow against WBTC/WETH liquidity used as collateral. This system ensures that there's a secure backing for each minted USDS, maintaining its stability and value.

Functionality:

- **setCollateralAndLiquidity:** Sets the reference to the CollateralAndLiquidity contract. This function is intended to be called once at deployment time, after which it renounces ownership to prevent further changes.
- **mintTo:** Mints USDS tokens to a specified wallet. This function is only callable by the CollateralAndLiquidity contract, ensuring that USDS is only minted when adequate collateral is provided.
- **burnTokensInContract:** Burns USDS tokens held in the contract. This function is used to remove USDS from circulation, typically in response to loan repayments or liquidations.

**Liquidizer.sol**

The "Liquidizer" contract plays a critical role in managing USDS, the stablecoin in the ecosystem. It is primarily responsible for converting various tokens into USDS and then burning these USDS tokens. This process is vital for maintaining the stability and integrity of the USDS supply, especially in scenarios involving undercollateralized liquidations.

Functionality:

- **setContracts:** Sets references to the CollateralAndLiquidity contract, Pools contract, and DAO. This function is only callable once and renounces ownership afterward.
- **incrementBurnableUSDS:** Called when a user's collateral position is liquidated. It indicates the amount of borrowed USDS that needs to be burned to offset the liquidated collateral.
- **_burnUSDS:** Privately burns a specified amount of USDS within the contract.
- **_possiblyBurnUSDS:** Attempts to burn the USDS amount indicated by **`usdsThatShouldBeBurned`**. If the contract's USDS balance is insufficient, it triggers a withdrawal of Protocol Owned Liquidity (POL) from the DAO, which is then converted to USDS for burning.
- **performUpkeep:** Called by the Upkeep contract to swap WBTC, WETH, and DAI held in the Liquidizer to USDS. It also burns any SALT tokens present in the contract and attempts to burn USDS as needed.

**SaltRewards.sol**

The "SaltRewards" contract is a key component that temporarily holds and distributes SALT rewards from various sources, such as emissions and arbitrage profits. It plays a crucial role in managing the allocation of rewards to both staking and liquidity participants. The contract distributes SALT rewards to two different types of reward emitters: one for xSALT holders (stakingRewardsEmitter) and another for liquidity providers (liquidityRewardsEmitter).

Functionality:

- **_sendStakingRewards:** Sends a specified amount of SALT rewards to the stakingRewardsEmitter for distribution to xSALT holders.
- **_sendLiquidityRewards:** Distributes SALT rewards to various pools within the liquidityRewardsEmitter. The rewards are allocated proportionally based on each pool's contribution to recent arbitrage profits.
- **_sendInitialLiquidityRewards:** Distributes an initial amount of SALT rewards evenly across multiple pools within the liquidityRewardsEmitter.
- **_sendInitialStakingRewards:** Sends an initial amount of SALT rewards to the stakingRewardsEmitter.
- **sendInitialSaltRewards:** Public function to distribute initial SALT rewards to both the liquidity and staking reward emitters. This function is only callable by the InitialDistribution contract.
- **performUpkeep:** Public function to distribute all SALT rewards currently held in the contract. It divides rewards between staking and liquidity providers, with a specific portion directly awarded to the SALT/USDS pool.

**RewardsEmitter.sol**

The "RewardsEmitter" contract is a vital component for distributing SALT rewards over time to participants in the StakingRewards contract. It helps in achieving a more stable yield by gradually emitting rewards, typically at a default rate of 1% per day. There are two versions of this contract: one for xSALT holders (staking rewards) and another for liquidity providers. This mechanism also provides an easy way to monitor the current yield for any pool.

Functionality:

- **addSALTRewards:** Allows adding SALT rewards for later distribution to whitelisted pools. It transfers SALT from the sender to the contract for future distribution.
- **performUpkeep:** Distributes a percentage of the stored SALT rewards to specified StakingRewards pools. The percentage is calculated based on the time elapsed since the last execution of this function.
- **pendingRewardsForPools (View):** Provides the amount of pending SALT rewards for specified pools. It's a view function, meaning it doesn't alter the state of the blockchain.

**Liquidity.sol**

The "Liquidity" contract is an abstract contract extending the "StakingRewards" contract. It allows users to deposit liquidity into pools and increase their share in these pools, which in turn enables them to earn proportional future rewards. The contract facilitates the deposit and withdrawal of liquidity while ensuring that users' shares in the liquidity pool are accurately tracked.

Functionality:

- **_dualZapInLiquidity:** Balances the amounts of two tokens to be added as liquidity to a pool. If one token is in excess, it swaps a portion of it for the other token to match the pool's current ratio of reserves.
- **_depositLiquidityAndIncreaseShare:** Deposits liquidity into a specified pool and increases the user's share in that pool. It supports "zapping" to balance the token amounts and increases the user's share in the liquidity pool.
- **depositLiquidityAndIncreaseShare:** Public function for users to deposit liquidity into a pool. It wraps around "_depositLiquidityAndIncreaseShare" to prevent direct deposits into the collateral pool and requires exchange access for the user.
- **_withdrawLiquidityAndClaim:** Withdraws a specified amount of liquidity from a pool, decreases the user's share in that pool, and claims any pending rewards. It transfers the reclaimed tokens back to the user.
- **withdrawLiquidityAndClaim:** Public function for users to withdraw liquidity from a pool. It wraps around "_withdrawLiquidityAndClaim" to prevent direct withdrawals from the collateral pool.

**StakingRewards.sol**

The "StakingRewards" contract facilitates rewards distribution to users for staking SALT tokens or liquidity shares. It calculates and allocates rewards based on each user's proportional share in the staked pool. This abstract contract is extended by specific contracts for different staking scenarios, such as staking SALT tokens or liquidity shares in various pools.

Functionality:

- **_increaseUserShare:** This function increases a user's share in a specific whitelisted pool. It ensures that the pool is valid and the share increase is non-zero. If a cooldown is used, it applies to users other than the DAO and ensures that the cooldown period has expired before allowing the share increase. The function also updates the virtual rewards proportionally to maintain the reward/share ratio.
- **_decreaseUserShare:** Decreases a user's share in a pool, sending any pending rewards to the user. It ensures the decrease amount is valid and applies cooldown if necessary. The function calculates the user's share of the total rewards, adjusts the virtual rewards, and sends the claimable rewards to the user.
- **claimAllRewards:** Allows a user to claim all available SALT rewards from multiple pools. It updates the user's virtual rewards balance to reflect the claimed rewards, ensuring they cannot be claimed again.
- **addSALTRewards:** Adds SALT rewards to specified pools. It checks for the validity of the pools and sums the total amount of rewards to be added. The function transfers the SALT from the sender to the contract for the rewards distribution.
- **totalSharesForPools:** Returns the total shares for specified pools.
- **totalRewardsForPools:** Provides the total rewards available in specified pools.
- **userRewardForPool:** Calculates the pending rewards for a user in a specific pool.
- **userRewardsForPools:** Determines the pending rewards for a user in multiple pools.
- **userShareForPool:** Retrieves a user's share in a specific pool.
- **userShareForPools:** Obtains a user's shares in multiple pools.
- **userVirtualRewardsForPool:** Fetches the virtual rewards for a user in a specific pool.
- **userCooldowns:** Returns the cooldown time remaining for a user in specified pools, indicating when they can next modify their stake.

**Staking.sol**

The "Staking" contract, part of the Salty system, facilitates the staking of SALT tokens in exchange for xSALT tokens at a 1:1 ratio. Unstaking xSALT back into SALT involves a duration-based mechanism: a default duration of 52 weeks allows full recovery of staked SALT, while a minimum duration of two weeks provides a portion of it. The contract handles various staking-related operations, manages unstake requests, and integrates with the broader ecosystem for reward distribution and access control.

- **stakeSALT:** Allows users to stake a specified amount of SALT and receive an equal amount of xSALT. Requires the user to have access to the exchange.
- **unstake:** Enables users to initiate the unstaking of a specified amount of xSALT over a chosen duration. The amount of SALT reclaimed varies based on the unstaking period.
- **cancelUnstake:** Allows users to cancel a pending unstake request, enabling them to use their xSALT again.
- **recoverSALT:** Lets users claim the SALT from a completed unstake, with possible deductions based on the unstaking duration.
- **transferStakedSaltFromAirdropToUser:** Facilitates the transfer of xSALT from the Airdrop contract to a user.
- **userXSalt (View):** Provides the amount of xSALT held by a user.
- **unstakesForUser (View):** Retrieves all unstake requests associated with a user within a specific range.
- **unstakesForUser (View):** Retrieves all unstake requests associated with a user.
- **userUnstakeIDs (View):** Returns the unstake IDs associated with a user.
- **unstakeByID (View):** Provides details of an unstake request based on its ID.
- **calculateUnstake (View):** Calculates the reclaimable SALT based on unstaked xSALT and unstaking duration. This function determines the proportion of SALT that can be reclaimed, considering the minimum and maximum unstake periods and percentages.

**RewardsConfig.sol**

The "RewardsConfig" contract, owned by the DAO, is designed for managing various parameters related to rewards within the ecosystem. It's primarily focused on defining the distribution percentages of SALT tokens as rewards and configuring the proportion of these rewards for different purposes.

Functionality:

- **changeRewardsEmitterDailyPercent:** Modifies the daily percentage of rewards distributed by the stakingRewardsEmitter and liquidityRewardsEmitter. This percentage is based on the SALT balance in each emitter contract.
- **changeEmissionsWeeklyPercent:** Alters the weekly percentage of SALT emissions distributed from the Emissions contract to the Liquidity and xSALT Holder Reward Emitters.
- **changeStakingRewardsPercent:** Adjusts the proportion of emissions and arbitrage profits allocated to xSALT holders relative to liquidity providers.
- **changePercentRewardsSaltUSDS:** Updates the percentage of SALT Rewards sent to the SALT/USDS pool, which is crucial as this pair is not involved in the usual arbitrage cycles.

**Emissions.sol**

The "Emissions" contract is designed to manage and distribute SALT token emissions. It controls the gradual release of SALT tokens from its reserves, primarily distributing them to the stakingRewardsEmitter and liquidityRewardsEmitter through the SaltRewards contract. The default distribution rate is set to 0.50% of the remaining SALT balance per week, with the actual amount transferred being adjusted based on the time elapsed since the last execution of the **`performUpkeep`** function.

Functionality:

**performUpkeep:** This function is responsible for the periodic distribution of SALT emissions. It calculates the amount of SALT to distribute based on the time elapsed since its last execution, capping the calculation at a maximum of one week to prevent excessive distribution in a single transaction. The amount of SALT to be distributed is determined by a predefined percentage of the contract's SALT balance. This function is intended to be called only by the Upkeep contract associated with the Salty system.

**StakingConfig.sol**

The "StakingConfig", controlled by the DAO, governs various configurable parameters related to the staking mechanism. These parameters include the minimum and maximum duration for unstaking requests, the minimum percentage of staked SALT reclaimable at the shortest unstaking duration, and the cooldown period for modifying staking shares. Changes to these parameters can only be made by the owner (DAO) of the contract.

Functionality:

- **changeMinUnstakeWeeks:** Alters the minimum number of weeks required for unstaking, where a portion of the staked SALT is reclaimable. The function increases or decreases this duration within a predefined range.
- **changeMaxUnstakeWeeks:** Modifies the maximum number of weeks for unstaking, after which the full amount of staked SALT is reclaimable. The function adjusts this duration within a specified range.
- **changeMinUnstakePercent:** Adjusts the percentage of staked SALT that can be reclaimed when unstaking for the minimum duration. This function allows incremental changes within defined limits.
- **changeModificationCooldown:** Alters the minimum time required between modifications of a user's staking share. This cooldown is adjustable within a certain range to prevent reward hunting.

**Salt.sol**

The **`Salt`** contract is a simple ERC20 token contract for the "SALT" token. It includes mechanisms to mint the initial supply and to burn tokens that are sent to the contract.

Functionality:

- **burnTokensInContract**: This function allows burning of any "SALT" tokens that have been sent to the contract. It checks the balance of "SALT" tokens in the contract and burns them, effectively reducing the total supply. An event **`SALTBurned`** is emitted to log the amount of tokens burned.
- **totalBurned**: A view function that returns the total number of "SALT" tokens burned so far. It calculates this as the difference between the initial supply and the current total supply.

**SigningTools.sol**

The **`SigningTools`** contract serves as a utility for signature verification related to the Salty system.

Functionality:

**`_verifySignature(bytes32 messageHash, bytes memory signature)`:** used to verify digital signatures. It takes a **`messageHash`** and a **`signature`** as inputs. The function decomposes the signature into its components (**`r`**, **`s`**, and **`v`**) and uses the **`ecrecover`** function to recover the address that signed the message. It then compares the recovered address with the **`EXPECTED_SIGNER`** to check if the signature is valid.

**DAO.sol**

The "DAO" contract in the Salty system is a governance contract that facilitates decentralized decision-making. It allows users to propose, vote on, and execute various governance actions, including parameter changes, token whitelisting/unwhitelisting, sending tokens, calling contracts, updating website URLs, and managing geographical exclusions.

Functionality:

- **_finalizeParameterBallot (ballotID)**: Finalizes a vote on parameter change ballots, executing the change if approved.
- **_executeSetContract (ballot)**: Executes contract setting actions based on approved ballots.
- **_executeSetWebsiteURL (ballot)**: Sets a new website URL based on an approved ballot.
- **_executeApproval (ballot)**: Executes various approval actions like unwhitelisting tokens, sending SALT, calling contracts, and updating geographical access based on approved ballots.
- **_finalizeApprovalBallot (ballotID)**: Finalizes and executes approval type ballots.
- **_finalizeTokenWhitelisting (ballotID)**: Finalizes token whitelisting ballots, whitelisting a token if it has the most votes and sufficient SALT for bootstrapping rewards.
- **finalizeBallot (ballotID)**: Finalizes a vote on a specific ballot, checking if it's eligible for finalization.
- **withdrawArbitrageProfits (weth)**: Withdraws WETH arbitrage profits from the Pools contract, transferring them to the caller (Upkeep contract).
- **formPOL (tokenA, tokenB, amountA, amountB)**: Forms protocol-owned liquidity with specified tokens, assuming tokens are already transferred to the contract.
- **processRewardsFromPOL**: Processes SALT rewards from the DAO's protocol-owned liquidity, distributing and burning them.
- **withdrawPOL (tokenA, tokenB, percentToLiquidate)**: Withdraws a portion of protocol-owned liquidity and sends underlying tokens to the Liquidizer for conversion to USDS and burning.
- **countryIsExcluded (country)**: Checks if a specific country is excluded from accessing the DEX.

**DAOConfig.sol**

The "DAOConfig" contract is a configuration contract that holds various parameters governing the DAO's operations. These parameters include rewards, quorum requirements, proposal stakes, and other aspects crucial for the DAO's functioning. The contract's parameters can only be modified by the DAO, ensuring decentralized governance.

- **changeBootstrappingRewards (increase)**: Modifies the bootstrapping rewards for whitelisting new tokens. Increasing or decreasing is based on the **`increase`** boolean flag.
- **changePercentPolRewardsBurned (increase)**: Adjusts the percentage of Protocol Owned Liquidity (POL) rewards that are burned. The function can increase or decrease this percentage.
- **changeBaseBallotQuorumPercent (increase)**: Alters the base quorum percentage required for various types of DAO ballots. The function can increase or decrease the quorum percentage.
- **changeBallotDuration (increase)**: Changes the minimum duration a ballot needs to exist before it can be actioned. The function can either extend or shorten this duration.
- **changeRequiredProposalPercentStake (increase)**: Adjusts the percentage of staked SALT required for a user to make a proposal. This function can either increase or decrease the required stake percentage.
- **changeMaxPendingTokensForWhitelisting (increase)**: Modifies the maximum number of tokens that can be pending for whitelisting at any time. This function can either increase or decrease the limit.
- **changeArbitrageProfitsPercentPOL (increase)**: Alters the percentage of WETH arbitrage profits sent to the DAO for forming SALT/USDS Protocol Owned Liquidity. This function can increase or decrease the percentage.
- **changeUpkeepRewardPercent (increase)**: Adjusts the percentage of WETH arbitrage profits sent to the caller of DAO.performUpkeep(). The function can increase or decrease the reward percentage.

**Parameters.sol**

The "Parameters" contract is an abstract contract within the Salty system that defines various types of parameters used across different configurations like PoolsConfig, StakingConfig, RewardsConfig, StableConfig, DAOConfig, and PriceAggregator. It includes an enumeration of these parameters and a function to execute changes to them. The contract acts as a centralized point for managing and updating key operational parameters of the system.

Functionality:

**_executeParameterChange**: This function modifies a specified parameter within the system based on its type and the direction of change (increase or decrease). It addresses parameters across various configurations like PoolsConfig, StakingConfig, RewardsConfig, StableConfig, DAOConfig, and PriceAggregator. Depending on the parameter type, the corresponding configuration contract's method is called to implement the change.

**Proposals.sol**

The "Proposals" contract is a comprehensive governance contract allowing SALT stakers to propose and vote on various governance actions. This includes parameter changes, token whitelisting/unwhitelisting, sending tokens, calling contracts, and updating website URLs. It ensures the uniqueness of ballots, tracks and validates user voting power, enforces quorums, and allows users to alter their votes.

Functionality:

- **_possiblyCreateProposal**: Internal function to create a proposal if certain conditions are met, including checks for the user's staking status, active proposals, and open ballots with the same name.
- **createConfirmationProposal**: Allows the DAO to create a confirmation proposal, bypassing the typical requirements for proposal creation.
- **markBallotAsFinalized**: Marks a ballot as finalized, updating the status of the ballot and the user who proposed it. Only callable by the DAO.
- **proposeParameterBallot**: Allows users to propose a ballot for changing a system parameter, subject to staking requirements and active proposal limits.
- **proposeTokenWhitelisting**: Proposes a new token for whitelisting, subject to various checks including the number of currently open whitelisting ballots and existing token whitelists.
- **proposeTokenUnwhitelisting**: Proposes the unwhitelisting of a token, ensuring it's currently whitelisted and not a core system token.
- **proposeSendSALT**: Proposes sending a specified amount of SALT to a wallet or contract, limited to a percentage of the DAO's SALT balance.
- **proposeCallContract**: Proposes calling a function on an external contract, specified by the user.
- **proposeCountryInclusion**: Proposes including a country in the system, identified by its ISO 3166 Alpha-2 code.
- **proposeCountryExclusion**: Similar to country inclusion but for excluding a country from the system.
- **proposeSetContractAddress**: Proposes setting a new address for a specified contract within the system.
- **proposeWebsiteUpdate**: Proposes updating the website URL for the system.
- **castVote**: Allows a user to cast a vote on an open ballot, with their voting power determined by their staked SALT.
- **ballotForID**: Returns the details of a ballot given its ID.
- **lastUserVoteForBallot**: Retrieves the last vote cast by a user for a specific ballot.
- **votesCastForBallot**: Returns the total number of votes cast for a specific ballot and vote type.
- **requiredQuorumForBallotType**: Calculates the required quorum for a given ballot type based on the staked SALT.
- **totalVotesCastForBallot**: Returns the total number of votes cast for a given ballot.
- **ballotIsApproved**: Determines if a ballot is approved based on the votes cast.
- **winningParameterVote**: Determines the winning vote type for a parameter change ballot.
- **canFinalizeBallot**: Checks if a ballot meets the criteria to be finalized.
- **openBallots**: Returns a list of all open ballots.
- **openBallotsForTokenWhitelisting**: Returns a list of open ballots specifically for token whitelisting.
- **tokenWhitelistingBallotWithTheMostVotes**: Identifies the whitelisting ballot with the most favorable votes.
- **userHasActiveProposal**: Checks if a user currently has an active proposal in the system.

**Upkeep.sol**

The `Upkeep` contract performs various maintenance tasks in multiple steps to ensure the smooth operation of the platform. It handles the conversion and distribution of tokens, including USDS and SALT, within the ecosystem. Key functions include swapping tokens for USDS and burning them, managing WETH arbitrage profits, distributing rewards, and handling SALT emissions. It also deals with the DAO's protocol-owned liquidity and vesting wallets for the DAO and team.

Functionality:

- **step1**: Executes upkeep on the Liquidizer contract to swap tokens for USDS and burn specific amounts of USDS.
- **step2 (receiver)**: Withdraws WETH arbitrage profits, rewarding the caller of performUpkeep() with a default 5% of the withdrawn amount.
- **step3**: Converts a portion of remaining WETH to USDS/DAI Protocol Owned Liquidity.
- **step4**: Converts a different portion of remaining WETH to SALT/USDS Protocol Owned Liquidity.
- **step5**: Converts the rest of the WETH to SALT and sends it to SaltRewards.
- **step6**: Sends SALT Emissions to the SaltRewards contract, based on the time since the last upkeep.
- **step7**: Distributes SALT from SaltRewards to specific rewards emitters.
- **step8**: Distributes SALT rewards from emitters, calculated based on time since the last upkeep.
- **step9**: Collects SALT rewards from the DAO's Protocol Owned Liquidity, distributing and burning them accordingly.
- **step10**: Sends SALT from the DAO vesting wallet to the DAO, with distribution spread over 10 years.
- **step11**: Sends SALT from the team vesting wallet to the team, again spread over 10 years.
- **performUpkeep**: Executes all the above steps in order, catching and logging any errors.
- **currentRewardsForCallingPerformUpkeep**: Provides a view of the current WETH rewards available for calling performUpkeep(), helping potential callers assess profitability.

**ManagedWallet.sol**

Purpose: the **`ManagedWallet`** contract manages the main and confirmation wallets for the Salty system. It includes mechanisms for proposing and confirming changes to these wallets, with a built-in timelock for security.

Functionality: 

- **proposeWallets**: Allows the current main wallet to propose new main and confirmation wallets.
- **receive**: Special function that processes incoming Ether transactions. Used by the confirmation wallet to confirm or reject wallet proposals based on the amount of Ether sent.
- **changeWallets**: Allows the proposed main wallet to confirm the wallet change after the timelock period has expired.

**ExchangeConfig.sol**

Purpose: The **`ExchangeConfig`** contract is a centralized configuration point for the Salty system, managed by the DAO. It establishes and maintains essential references to various components like the DAO, token contracts (such as **`salt`**, **`wbtc`**, **`weth`**, **`dai`**, **`usds`**), and operational contracts like Upkeep, Initial Distribution, and Airdrop.

Functionality:

- **Set Contracts (`setContracts`)**: Sets the references to DAO, Upkeep, Initial Distribution, Airdrop, and Vesting Wallet contracts. This function can only be called once.
- **Set Access Manager (`setAccessManager`)**: Assigns an Access Manager to the contract, emitting an event upon doing so. The Access Manager controls user access to various functionalities.
- **Wallet Has Access (`walletHasAccess`)**: Checks if a given wallet has access to the protocol components, determined by the Access Manager. Special access is given to the DAO and Airdrop contracts.

**AccessManager.sol**

The **`AccessManager`** contract manages user access based on geolocation, with the capability to update access rights in response to changes in geographic restrictions. It uses offchain data mapping and onchain verification to grant or deny access, ensuring compliance with regional regulations. The DAO has control over this contract, including the ability to update excluded countries, which resets the access permissions (**`geoVersion`**).

Functionality:

- **excludedCountriesUpdated (`function excludedCountriesUpdated() external`):** Increments the **`geoVersion`** to indicate an update in the list of excluded countries, effectively resetting access for all users. This function can only be called by the DAO.
- **_verifyAccess (`function _verifyAccess(address wallet, bytes memory signature) internal view returns (bool)`):** Verifies whether the access request for a wallet is authorized based on a signature. This is a security measure to ensure access is granted correctly.
- **grantAccess (`function grantAccess(bytes calldata signature) external`):** Grants access to the sender for the current **`geoVersion`**. The sender must provide a correct signature from an offchain verifier. This function updates the **`_walletsWithAccess`** mapping to reflect the granted access.
- **walletHasAccess (`function walletHasAccess(address wallet) external view returns (bool)`):** Returns **`true`** if the specified wallet has access at the current **`geoVersion`**. It checks the **`_walletsWithAccess`** mapping for this information.

**InitialDistribution.sol**

The contract **`InitialDistribution`** serves as the distributor for the initial supply of the SALT token in the Salty.IO ecosystem.

Functionality:

- **distributionApproved**: Can only be called by the **`BootstrapBallot`** contract. It distributes the SALT tokens to different parts of the ecosystem, such as Emissions, DAO Vesting Wallet, Initial Development Team Vesting Wallet, Airdrop Participants, and for Liquidity and Staking Bootstrapping.

**Airdrop.sol**

Purpose: The **`Airdrop`** contract in the Salty system is designed to incentivize early supporters and community members through a structured airdrop mechanism.

Functionality:

- **`constructor`**: Sets up the contract with references to **`IExchangeConfig`** and **`IStaking`**, and initializes the SALT token.
- **`authorizeWallet`**: Authorizes a wallet to claim the airdrop based on specific criteria (like voting and retweeting). This can only be done by the **`BootstrapBallot`** contract and before claiming is allowed.
- **`allowClaiming`**: Enables claiming of the airdrop, called by the **`InitialDistribution`** contract post-successful conclusion of the **`BootstrappingBallot`**. It calculates and sets the amount of SALT each user will receive and approves the staking contract to handle the distribution.
- **`claimAirdrop`**: Allows an authorized user to claim their share of the airdrop. It stakes a specific amount of SALT and transfers it to the user as staked SALT (xSALT)
- **`isAuthorized`**: Checks if a wallet is authorized to claim the airdrop.
- **`numberAuthorized`**: Returns the total number of wallets authorized to claim the airdrop.

**StableConfig.sol**

The **`StableConfig`** contract manages various parameters related to the borrowing of USDS against collateral. It controls critical aspects like the rewards for liquidators, maximum reward value, minimum collateral values, and ratios defining how much can be borrowed against the collateral and the threshold for liquidation. These parameters are adjustable by the contract owner (the DAO), ensuring flexibility and adaptability to changing market conditions. The contract's primary purpose is to maintain a balance between incentivizing users for maintaining the ecosystem's stability and ensuring the security and sustainability of the borrowing and liquidation processes.

Functionality: 

- **changeRewardPercentForCallingLiquidation**: Adjusts the percentage reward for users who initiate liquidations, ensuring it remains within a set range and accounts for overall collateral ratio. The change is either an increase or decrease, constrained by specific limits.
- **changeMaxRewardValueForCallingLiquidation**: Modifies the maximum USD value that a user can receive for initiating liquidation. The adjustment can be either an increase or decrease, within predefined limits.
- **changeMinimumCollateralValueForBorrowing**: Alters the minimum collateral value required to borrow USDS. This change can be either an increase or a decrease, with specified boundaries.
- **changeInitialCollateralRatioPercent**: Updates the initial collateral-to-loan ratio. This ratio determines how much USDS can be borrowed against the staked collateral, adjustable both upwards and downwards within a predefined range.
- **changeMinimumCollateralRatioPercent**: Tweaks the minimum collateral-to-loan ratio, below which positions can be liquidated. Adjustments are made with consideration for the ratio after rewards to maintain liquidation viability.
- **changePercentArbitrageProfitsForStablePOL**: Modifies the percentage of arbitrage profits allocated to Protocol Owned Liquidity (POL) in the stablecoin ecosystem. This can be increased or decreased within certain limits.

**BootstrapBallot.sol**

Purpose: The "BootstrapBallot" contract facilitates a voting process to decide whether to launch an exchange and establish initial geographic restrictions. It allows airdrop participants to cast a single vote each, either in favor or against the exchange startup, with their eligibility verified through off-chain checks. The contract tallies votes and, upon reaching the specified completion timestamp, determines if the proposal to start the exchange is approved based on the majority vote. Once finalized, if the vote is affirmative, it triggers the exchange launch and potential initial geographic exclusions through the connected exchange and DAO contracts.

Functionality: 

- **vote**: Allows a whitelisted user to cast a single vote (YES or NO) regarding starting the exchange and distributing SALT. It verifies the user's eligibility through a signature, records the vote, and authorizes the user for the airdrop if they haven't voted already.
- **finalizeBallot**: Checks if the ballot completion time is reached and finalizes the voting process. If the majority voted YES, it triggers the exchange configuration's distribution approval and notifies the DAO to start the exchange. It also emits an event indicating the ballot's outcome and marks the ballot as finalized.

Pool section:

**1. Pools.sol**

Purpose: Manages the liquidity pools within the DEX, handling the addition and removal of liquidity, as well as enabling token swaps. It supports arbitrage opportunities to maintain balance across pools and optimizes gas costs by allowing users to deposit tokens directly. Additionally, it maintains and updates token reserves and user balances, facilitating a structured environment for trading and liquidity provision on the DEX.

Key functionality:

- **setContracts**: This function sets the addresses of the DAO and CollateralAndLiquidity contracts, and can only be called once, as the contract renounces ownership after this call. It's used during the initial setup phase of the contract.
- **startExchangeApproved**: It enables the exchange's operations once it's approved by the bootstrapBallot.
- **addLiquidity**: This function allows the users (through the CollateralAndLiquidity contract) to add liquidity to a pool. It calculates the correct amount of tokens to be added to maintain the pool's balance and issues liquidity tokens in return.
- **removeLiquidity**: Similar to **`addLiquidity`**, but in reverse, this function allows the users(through the CollateralAndLiquidity contract) to remove liquidity from a pool. It calculates and returns the proportional amount of underlying tokens to the liquidity removed.
- **deposit**: Allows users to deposit tokens into the contract, which are not considered as staking but helps in reducing gas costs by minimizing transfers during swaps.
- **withdraw**: Enables users to withdraw their previously deposited tokens
- **_adjustReservesForSwap**: An internal function that adjusts the pool's reserves based on the amount of token being swapped in, using the constant product formula to maintain the pool's balance.
- **_arbitrage**: Executes an arbitrage strategy by trading a token through a series of swaps starting and ending with WETH, aiming to profit from price discrepancies in the pools.
- **_determineSwapAmountInValueInETH**: Calculates the value of the swap amount in terms of WETH. This is used to gauge the impact of a swap on potential arbitrage opportunities.
- **_attemptArbitrage**: After a user swap, this function checks for and executes profitable arbitrage opportunities based on the new state of the pool.
- **_adjustReservesForSwapAndAttemptArbitrage**: Combines a swap operation with an attempt at arbitrage in a single transaction, first executing the swap and then trying to capitalize on any resultant arbitrage opportunities.
- **swap**: Allows users to swap one token for another. It adjusts user deposits, performs the swap, and deposits the output token back to the user's account.
- **depositSwapWithdraw**: Facilitates a swap operation where a user deposits a token, swaps it for another, and then immediately withdraws the swapped token, all in a single transaction.
- **depositDoubleSwapWithdraw**: Similar to **`depositSwapWithdraw`**, but performs two sequential swaps before withdrawing the final token, allowing for more complex swap paths.
- **getPoolReserves**: A view function that provides the current reserves of a specified token pair from the pool.
- **depositedUserBalance**: Another view function that returns the amount of a specific token a user has deposited in the contract.
**2. PoolStats.sol**

The `PoolStats` contract, which is inherited by the `Pools.sol` contract, primarily functions to track and manage arbitrage profits generated across various liquidity pools. It allocates these profits proportionally for rewards distribution based on each pool's contribution to overall arbitrage profits. The contract achieves this by recording and resetting arbitrage profits, updating arbitrage indices, and calculating the individual contributions of whitelisted pools to the arbitrage profits.

Functionality: 

- **_updateProfitsFromArbitrage**: Records the profits made from arbitrage involving a specific path defined by two tokens. It updates the total arbitrage profits for the pool identified by these tokens.
- **clearProfitsForPools**: Resets the arbitrage profit stats for all pools. It's intended to be called only by the Upkeep contract to refresh the stats after upkeep operations.
- **_poolIndex**: Retrieves the index of a specific pool within the list of whitelisted pools. If the pool isn't found, it returns a maximum invalid value.
- **updateArbitrageIndicies**: Updates the indices of pools in the whitelisted pool list that contribute to a specific arbitrage path. It ensures that the indices reflect the current set of whitelisted pools.
- **_calculateArbitrageProfits**: Calculates the amount of arbitrage profit generated by each whitelisted pool. It divides the total profit by the number of pools contributing to each arbitrage operation.
- **profitsForWhitelistedPools**: Provides the calculated arbitrage profits for all whitelisted pools, showing how much each pool contributed to the total arbitrage profits.
- **arbitrageIndicies**: Returns the indices of pools involved in a specific arbitrage path, identified by a pool ID.

**3. ArbitrageSearch.sol**

Purpose: The **`ArbitrageSearch`** contract, which is inherited by the **`Pools.sol`** contract, focuses on identifying profitable arbitrage opportunities in a decentralized exchange setting. Its primary function is to find a circular trading path starting and ending with WETH (Wrapped Ethereum) that can generate profit through arbitrage. The contract includes methods to determine the optimal amount of WETH to be used in arbitrage trades, considering the reserves of the tokens involved in the arbitrage cycle. These methods are employed to ensure that the trading activities are profitable and efficient, contributing to the overall liquidity and stability of the exchange's ecosystem.

Functionality: 

- **_arbitragePath**: This function determines the middle two tokens in a potential arbitrage path that starts and ends with WETH. It's used to identify a circular trading route for arbitrage opportunities, such as WETH → Token2 → Token3 → WETH.
- **_rightMoreProfitable**: This function estimates whether moving to the right of a given midpoint in an arbitrage search will be more profitable than the midpoint itself. It's a part of the search algorithm to find the most advantageous arbitrage point, comparing profits at two closely positioned points.
- **_bisectionSearch**: This function uses a bisection algorithm to find the optimal amount of WETH to start an arbitrage trade, aiming for the highest profit. It iteratively narrows down a range to pinpoint the best input amount, considering various pool reserves and assuming a single peak in the profit function.

**4. PoolsConfig.sol**

This contract, owned by the DAO, manages the whitelisting of token pairs for pools, controls pool-related parameters, and maintains the infrastructure for internal swap limits and pool pairings.

Functionality:

- **whitelistPool**: Adds a pair of tokens to the whitelist, allowing them to be used in the associated pools. It ensures the number of whitelisted pools does not exceed the maximum limit and emits an event upon successful whitelisting.
- **unwhitelistPool**: Removes a token pair from the whitelist, essentially disallowing their use in the pools. It also updates the internal structures to reflect this change and triggers an event for the unwhitelisting.
- **changeMaximumWhitelistedPools**: Adjusts the maximum number of pools that can be whitelisted. It allows increasing or decreasing the limit within a specified range and emits an event to indicate the change.
- **changeMaximumInternalSwapPercentTimes1000**: Modifies the maximum percentage limit for internal swaps within the protocol, providing control over the swap size relative to the pool reserves. An event is emitted to log this change.
- **numberOfWhitelistedPools**: A view function that returns the current number of whitelisted pools, providing insights into the pool's configuration.
- **isWhitelisted**: Aview function that checks if a given poolID is whitelisted, indicating whether it's available for use in the pools.
- **whitelistedPools**: It returns an array of all currently whitelisted poolIDs, offering a comprehensive view of the active pools.
- **underlyingTokenPair**: Provides the token pair associated with a given poolID, useful for identifying the tokens involved in a specific pool.
- **tokenHasBeenWhitelisted**: Checks if a token has been whitelisted, specifically in pairing with WBTC and WETH, to determine its eligibility for use in the pools.

**5. PoolUtils.sol**

**`PoolUtils`** is a utility library providing essential functions for managing the liquidity pools. It includes functions for generating unique pool identifiers, handling token order, and executing controlled internal swaps. The library is focused on maintaining consistent liquidity pool identification and ensuring secure, limited internal token swaps to protect against market manipulation. It is used in conjunction with the main pools contract (**`Pools.sol`**) to facilitate effective and secure operations of the liquidity pools within the decentralized exchange.

Functionality:

- **_poolID**: Computes a unique identifier for a liquidity pool consisting of two tokens. The identifier is the same regardless of the order of the tokens.
- **_poolIDAndFlipped**: Similar to **`_poolID`**, but additionally returns a boolean indicating if the order of the input tokens was flipped to maintain a consistent identifier.
- **_placeInternalSwap**: Conducts a token swap within the protocol, limiting the input amount to a specified percentage of the token reserves. The function is designed to mitigate potential sandwich attacks and integrates with the protocol's arbitrage mechanism.

**6. PoolMath.sol**

The **`PoolMath`** library in Solidity is designed to facilitate specific mathematical operations related to the liquidity pools. It includes functions for bit-level calculations, aligning token ratios for liquidity addition, and optimizing token swaps for maintaining desired reserve ratios in liquidity pools. This library is essential for ensuring efficient and balanced liquidity management.

Functionality:

- **_mostSignificantBit**: Finds the most significant bit (the highest order bit that is set to 1) in a given number. It's used for calculations involving bit manipulation.
- **_maximumMSB**: Determines the highest most significant bit across multiple values. It's useful for comparing and aligning the bit lengths of different numbers.
- **_zapSwapAmount**: Calculates how much of one token (assumed to be in excess) should be swapped for another token before adding liquidity to a pool. This is to ensure the token ratio matches the pool's reserves ratio after the swap, optimizing liquidity addition.
- **_determineZapSwapAmount**: Decides the amount of tokens that need to be swapped to achieve a desired ratio between two tokens. It calls **`_zapSwapAmount`** and adjusts token amounts for adding liquidity to a pool.

**`CollateralAndLiquidity`** 

The **`CollateralAndLiquidity`** contract manages the deposit and withdrawal of WBTC/WETH liquidity, allows users to use this liquidity as collateral for borrowing USDS stablecoin, and handles the liquidation of undercollateralized positions. It sets initial and minimum collateralization ratios, rewards liquidators, and uses a price aggregator for valuing collateral in USD. 

Functionality:

- **`depositCollateralAndIncreaseShare`**: Allows a user to deposit WBTC/WETH as collateral and increase their share for future rewards.
- **`withdrawCollateralAndClaim`**: Enables users to withdraw their WBTC/WETH collateral and claim any pending rewards.
- **`borrowUSDS`**: Users can borrow USDS stablecoin using their existing collateral, ensuring the borrowed amount does not exceed the maximum allowed.
- **`repayUSDS`**: Users repay borrowed USDS, which adjusts their borrowed amount and sends USDS to be burned later.
- **`liquidateUser`**: Allows liquidation of a user's position if it falls under the minimum collateral ratio. The liquidator gets a reward, and the remainder is converted to USDS and burned.
- **`underlyingTokenValueInUSD`**: Calculates the current market value in USD for a specified amount of BTC and ETH.
- **`totalCollateralValueInUSD`**: Determines the total market value in USD of all WBTC/WETH collateral deposited.
- **`userCollateralValueInUSD`**: Computes the market value in USD of a specific user's collateral.
- **`maxWithdrawableCollateral`**: Calculates the maximum amount of collateral a user can withdraw while maintaining the required collateral ratio.
- **`maxBorrowableUSDS`**: Computes the maximum amount of USDS a user can borrow based on their collateral.
- **`numberOfUsersWithBorrowedUSDS`**: Returns the number of users who have borrowed USDS.
- **`canUserBeLiquidated`**: Checks if a user can be liquidated based on their collateral value and borrowed USDS ratio.
- **`findLiquidatableUsers (with two variants)`**: Identifies users who are eligible for liquidation within a specified range or in total.

### Architecture

### **System Workflow**

1. **Liquidity Provision and Trading**
    - Users add liquidity to pools in **`Pools.sol`**, receiving liquidity tokens.
    - Swaps are executed within these pools, with prices adjusted via arbitrage mechanisms.
2. **Staking and Reward Accumulation**
    - Users stake SALT tokens in **`Staking.sol`**, receiving xSALT.
    - **`StakingRewards.sol`** and **`RewardsEmitter.sol`** calculate and distribute rewards based on staked amounts.
3. **Borrowing and Stability Management**
    - Users borrow USDS against their WBTC/WETH collateral via **`CollateralAndLiquidity.sol`**.
    - **`Liquidizer.sol`** intervenes during undercollateralized positions, converting assets to USDS and burning them.
4. **Governance Participation**
    - SALT holders participate in governance decisions via **`DAO.sol`** and **`Proposals.sol`**, influencing the ecosystem’s development.
5. **Token Distribution and Burning**
    - **`Emissions.sol`** distributes SALT tokens periodically.
    - **`Salt.sol`** and **`USDS.sol`** have mechanisms to burn tokens to control supply.

Here is a non-exhaustive diagram of the interactions between the contracts that I created and used for reference throughout the contest:
![SaltyArchitecture](https://img001.prntscr.com/file/img001/Hcz8WFiCRImoNgesSm53PQ.png)


### Centralization/Systemic risks

The system is sufficiently decentralized where the main governance mechanisms are controlled by the DAO. The systemic risks of oracles failing and providing wrong prices are mitigated by getting prices from 3 different feeds. 

### Approach taken

Day 1-3: Initial Familiarization and High-Level Overview
- Gain a comprehensive understanding of the Salty protocol's purpose and structure.
- Review all smart contracts briefly to develop a high-level mental model of their interactions and the overall architecture.

Day 4-6: In-Depth Contract Review
- Start a detailed review of each contract, focusing on core components like Emissions, RewardsEmitter, and Liquidizer.
- Examine user roles, permissions, and typical interactions within the system.

Day 6-8: Contract Interaction Flows
- Follow user flows through the system, including staking, liquidity provision, reward claiming, and governance participation.
- Verify the flows against the intended functionality outlined in the documentation.

Day 9-10: Test Case Development and Execution
 Develop and run custom test cases, particularly for edge cases and potential security vulnerabilities.
- Test the integration and interaction of different contracts within the ecosystem.

Day 11-12: Governance and Upgradeability Review
-Analyze the governance model, including DAO functionalities, proposal mechanisms, and voting processes.
-Assess the upgradeability approach and its implications for the protocol’s future adaptability and security.

Day 13-14: Final Review and Issue Compilation
- Conduct a final comprehensive review of all contracts and user flows.
- Compile and categorize identified issues based on severity and impact.
- Prepare a detailed analysis report, including findings, recommendations, and suggestions for improvements.
### Conclusion
The Salty protocol presents a comprehensive DeFi ecosystem with a focus on stablecoin borrowing, liquidity provision, and reward distribution. Its well-structured smart contracts manage collateralized borrowing, liquidations, and reward emissions. 

The integration of various price feeds and the unique reward distribution system enhance the protocol's robustness.

However, there are still some weak spots left that should be fixed after the current audit.

### Time spent:
80 hours