# Analysis - Salty.IO Contest

![Salty.IO-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FTEwB4jYYdhM.0&w=256&q=75)

## Description overview of The Salty.IO Contest

Salty.IO is designed to be a decentralized exchange that leverages Automatic Atomic Arbitrage to provide zero fees on swaps. It introduces unique features such as the distribution of profits to liquidity providers and stakers, the creation of Protocol Owned Liquidity, the issuance of an overcollateralized stablecoin (USDS), and a governance model that emphasizes decentralization through DAO control. The native token, SALT, has specific attributes aimed at managing its supply dynamics.

**The key contracts of Salty protocol for this Audit are**:

Salty protocol consists of various contracts that collectively implement different functionalities such as arbitrage, governance (DAO), token launch mechanisms, pools management, price feeds, rewards distribution, stablecoin handling, and staking. Security considerations are crucial for the proper functioning and protection of user funds within the protocol.

1. **Access Control:**

   - Contracts like `AccessManager.sol` and `ExchangeConfig.sol` are responsible for access control and geo-region restrictions. Ensuring proper implementation and audit of these mechanisms is essential to prevent unauthorized access and ensure compliance with regulatory requirements.

2. **Configuration Management:**

   - Contracts such as `DAOConfig.sol`, `RewardsConfig.sol`, and `PoolsConfig.sol` allow the DAO to modify various protocol parameters.

3. **Vesting Wallets:**

   - The contracts managing vesting wallets, such as `DAO.sol` and `InitialDistribution.sol`, should implement robust mechanisms to secure the release of tokens over time. Time-lock features and multi-signature requirements can enhance security.

4. **Arbitrage and Swap Functions:**

   - Contracts involved in arbitrage, such as `ArbitrageSearch.sol` and `Pools.sol`, should implement measures to mitigate potential vulnerabilities, such as front-running and sandwich attacks. The use of try/catch blocks in the `Upkeep.sol` contract suggests a defensive approach to handle possible errors during the execution of steps.

5. **Token Handling:**

   - Contracts like `USDS.sol` and `Salt.sol` should ensure secure borrowing, burning, and transfer functions. Critical functions, especially those involving token transfers and burning, should have appropriate access controls and validation checks.

6. **Airdrop and Initial Distribution:**

   - Contracts handling airdrops and initial token distribution, such as `Airdrop.sol` and `InitialDistribution.sol`, should prevent double claims and ensure fair distribution. Proper verification mechanisms should be in place to authenticate participants.

7. **Price Feeds:**

   - Contracts responsible for price feeds, like `CoreChainlinkFeed.sol` and `PriceAggregator.sol`, must securely fetch and aggregate prices. Considerations for oracle manipulation, outlier detection, and emergency mechanisms should be implemented.

8. **Staking and Liquidity Pools:**

   - Contracts handling staking (`Staking.sol`) and liquidity pools (`Liquidity.sol`, `Pools.sol`) should prioritize security in deposit, withdrawal, and reward distribution mechanisms. Measures to prevent vulnerabilities like impermanent loss and unauthorized access should be in place.

These contracts are most important from a security perspective as they relate to key functions like governance, funds handling, access controls and configurable parameters. As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

## System Overview

### Scope

**Arbitrage**

- **ArbitrageSearch.sol**: This contract is an abstract contract that facilitates the search for profitable arbitrage opportunities within a decentralized exchange on the Ethereum. It utilizes the Automatic Atomic Arbitrage mechanism and implements methods for finding a circular arbitrage path involving three tokens, estimating profits at different points in the path, and performing iterative bisection to determine the optimal amount for arbitrage swaps based on token reserves. The contract is part of the Salty.IO protocol, with the core functionality related to arbitrage and profit optimization in decentralized trading.

**DAO**

- **DAO.sol**: DAO allowing users to propose and vote on various governance actions. It handles parameter changes, `token whitelisting`, `sending tokens`, calling other contracts, and more. The contract is an integral part of the `Salty.IO` protocol, incorporating features for managing `proposals`, `voting`, `executing` approved actions, and interacting with other protocol contracts such as `pools`, `rewards`, `stable configurations`, and access management.

- **DAOConfig.sol**: DAOConfig is owned by the DAO and allows modification of parameters related to the `governance` of the protocol, such as `bootstrapping rewards`, `percentage of rewards burned`, `ballot quorum requirements`, `ballot duration`, required proposal stake, maximum pending tokens for whitelisting, arbitrage profits distribution to Protocol Owned Liquidity, and upkeep reward percentage, with these modifications accessible only by the DAO.

- **Parameters.sol**: Defines a set of parameter types related to various `configurations` and provides a mechanism to execute changes to these parameters based on specified types, allowing for adjustments in the configuration of pools, staking, rewards, stablecoins, DAO, and price aggregators within the system.

- **Proposals.sol**: This contract enables `SALT` stakers to `propose` and `vote` on various types of `ballots`, such as parameter changes, `token whitelisting/unwhitelisting`, `sending tokens`, calling contracts, and updating website URLs. It ensures ballot uniqueness, tracks and validates user voting power, enforces quorums, and provides a mechanism for users to alter votes.

**Launch**

- **Airdrop.sol**: Airdrop contract is manages the distribution of `airdrop rewards`. Users who have retweeted the launch announcement and voted (authorized users) can claim staked SALT after a `BootstrappingBallot` has concluded, with the contract ensuring fair distribution and preventing double claims.

- **BootstrapBallot.sol**: This contract facilitates a voting mechanism for a decentralized exchange `launch`, allowing `airdrop` participants to decide whether to `start up` the exchange, `distribute` SALT, and `establish` initial geo restrictions, with votes verified through `off-chain` whitelisting and retweeting.

- **InitialDistribution.sol**: Handles the distribution of the native token (`SALT`) to various recipients, including `emissions`, `DAO reserve` vesting wallet, initial `development team` vesting wallet, `airdrop participants`, and liquidity and staking bootstrapping pools, upon approval from the associated BootstrapBallot.

**Pools**

- **Pools.sol**: It manages `reserves` used for swaps, handles `deposits` and `withdrawals`, and facilitates arbitrage opportunities. Ut implements various features such as liquidity addition, removal, token deposits, withdrawals, and swaps, along with functionalities related to arbitrage and reward distribution.

- **PoolsConfig.sol**: PoolsConfig contract acts as a configuration manager for `whitelisting` and `managing pools` in a protocol, allowing the `DAO` to control and modify pool-related parameters, such as the maximum number of `whitelisted` pools and the maximum internal `swap` size as a percentage of reserves.

- **PoolStats.sol**: This contract is an abstract contract that keeps `track` of arbitrage profits generated by pools in a protocol, provides functionalities for updating and clearing arbitrage-related statistics, as well as calculating and `distributing` profits to pools based on their contributions to arbitrage.

- **PoolMath.sol**: The PoolMath library includes functions for determining the optimal amount of tokens to be swapped when adding liquidity to a pool, ensuring the resulting reserves maintain a proportional ratio. It employs mathematical computations and quadratic equations to calculate swap amounts based on given reserves and desired liquidity additions.

- **PoolUtils.sol**: The library PoolUtils provides functions for computing `unique pool IDs`, determining whether token orders are `flipped`, and conducting internal swaps within a protocol, with limits based on reserve percentages to mitigate sandwich attacks and encourage reasonable arbitrage opportunities.

**Price Feed**

- **CoreChainlinkFeed.sol**: This contract utilizes `Chainlink` price oracles to retrieve prices for `BTC` and `ETH` with 18 decimals, converting from Chainlink's 8 decimals, and provides functions to fetch the latest prices for `BTC` and `ETH`.

- **CoreSaltyFeed.sol**: CoreSaltyFeed utilizes Salty.IO pools to `retrieve` prices for `BTC` and `ETH` with 18 decimals, converting reserves of wrapped Bitcoin (`WBTC`) and wrapped Ether (`WETH`) against a stablecoin (USDS), and provides functions to fetch the latest prices for BTC and ETH in a decentralized application.

- **CoreUniswapFeed.sol**: This contract calculates time-weighted average prices (`TWAPs`) for Wrapped Bitcoin (`WBTC`) and Wrapped Ether (`WETH`) using associated `Uniswap v3 pools`, with prices returned in 18 decimals, and provides functions to fetch the latest prices for BTC and ETH in a decentralized application.

- **PriceAggregator.sol**: Compares three different price feeds for Bitcoin (`BTC`) and Ethereum (`ETH`) and aggregates their prices, discarding outliers to ensure reliability, with adjustable parameters for maximum percent difference and `cooldown` periods between price feed modifications.

**Rewards**

- **Emissions.sol**: This Emissions is responsible for storing `SALT emissions` at `launch` and gradually distributing them over `time` to `staking` and `liquidity` rewards emitters, with the default rate being `0.50%` of the remaining SALT balance per `week`, interpolated based on the time elapsed since the last upkeep.

- **RewardsConfig.sol**: RewardsConfig contract is owned by the DAO and allows the DAO to adjust various parameters related to `rewards distribution`, such as the `daily` percentage for rewards emitters, the `weekly` percentage for emissions, the percentage for staking rewards, and the percentage of SALT rewards sent to the SALT/USDS pool.

- **RewardsEmitter.sol**: The contract `RewardsEmitter`, stores SALT rewards for distribution to specified `StakingRewards` contracts, allowing gradual emission at a default rate of `1%` per day to users holding shares in the designated StakingRewards contract, with the added capability of adding SALT rewards and performing upkeep to distribute accumulated rewards based on a specified daily percentage.

- **SaltRewards.sol**: This contract serves as a utility contract that `temporarily` holds SALT rewards from `emissions` and `arbitrage` profits, and during the `performUpkeep`() function, it distributes these rewards to the `stakingRewardsEmitter` and `liquidityRewardsEmitter`, with proportions for the latter based on each pool's share in generating recent arbitrage profits.

**Stable**

- **CollateralAndLiquidity.sol**: This contract deployed contract through which all liquidity on the exchange is de`posited and `withdrawn`. It also allows users to deposit `WBTC/WETH`liquidity as`collateral`for borrowing the`USDS` stablecoin, enforcing default initial collateralization ratios and minimum collateral ratios, and providing functionality for borrowing, repaying, and liquidating positions.

- **Liquidizer.sol**: Liquidizer facilitates the swapping of tokens (`WBTC, WETH, DAI`) received from `collateral` liquidation and Protocol Owned Liquidity withdrawal to `USDS`, which is subsequently burned. It includes mechanisms to handle `undercollateralized` liquidation scenarios, where insufficient USDS is available for burning. In such cases, a small percentage of Protocol Owned Liquidity (`POL`) is withdrawn from the DAO, converted to USDS, and added to the burnable buffer. The contract is also responsible for `incrementing` the amount of USDS that should be burned when a user's collateral position is liquidated.

- **StakingConfig.sol**: This contract is a configuration contract owned by the `DAO`, allowing modification of parameters related to `staking`, `unstaking`, and modification `cooldown` periods within protocol.

- **USDS.sol**: The contract facilitates the `borrowing` and `burning` of a stablecoin (`USDS`) by users who deposit `WBTC/WETH` liquidity as `collateral`, with the `collateralization` ratio set at `200%` by default and the ability for any user to liquidate positions falling below `110%`.

**Staking**

- **Liquidity.sol**: Enables users to add `liquidity` to a Dex pool, adjust their liquidity `share`, and claim `rewards`, with functionality for dual token zapping to optimize liquidity addition, and restrictions preventing direct interaction with a collateral pool.

- **Staking.sol**: Staking of `SALT` tokens to receive `xSALT` at a `1:1` ratio, allowing users to unstake `xSALT` after a specified duration to reclaim a percentage of the staked SALT, with expedited unstaking options and functionality for cancelling pending unstakes.

- **StakingConfig.sol**: The "StakingConfig" contract, owned by a DAO, allows modification of `staking` parameters, including minimum and maximum `unstake weeks`, minimum `unstake percentage`, and modification `cooldown periods`, with adjustments only permitted by the DAO.

- **StakingRewards.sol**: This is an abstract base contract, allowing users to receive `SALT` token rewards for `staking` SALT or liquidity shares in whitelisted pools, with rewards proportional to their share of the stake and based on the share at the time rewards are added.

**1. AccessManager (`AccessManager.sol`):** Manages access control for users based on their geo region. It uses off-chain verification to map user IPs to geolocations and allows the DAO to update the list of excluded countries.

**2. ExchangeConfig (`ExchangeConfig.sol`):** Configures and manages access to various protocol components. It controls access to liquidity provision, collateral addition, and borrowing of the protocol. It can be updated by the DAO to incorporate different mechanics, such as open access or decentralized ID services.

**3. ManagedWallet (`ManagedWallet.sol`):** Manages two wallet addresses (main and confirmation wallets). Changes to these wallets follow a two-step process, requiring proposals from the main wallet and confirmation from the confirmation wallet with a time lock of 30 days.

**4. Salt (`Salt.sol`):** Implements the ERC-20 token standard for the protocol's native token, SALT. It has functionality to burn tokens, and it provides views of the total burned amount.

**5. SigningTools (`SigningTools.sol`):** Contains a utility function for verifying the signature of a message. It is used for verifying access requests.

**6. Upkeep (`Upkeep.sol`):** Performs various tasks for the protocol. The key tasks include:

- Swapping tokens for USDS and burning USDS.
- Withdrawing WETH arbitrage profits, rewarding the caller, and converting a percentage to different Protocol Owned Liquidity (POL).
- Converting WETH to SALT and distributing it to the SaltRewards contract.
- Handling SALT emissions, distributing rewards, and processing rewards from DAO's POL.
- Distributing SALT from the DAO vesting wallet and team vesting wallet linearly over 10 years.
- The `performUpkeep()` function orchestrates the execution of these tasks, and each step is wrapped in a try/catch block to prevent a failure in one step from affecting subsequent steps.

### Roles

I identify the following roles in the Salty protocol:

1. **DAO (Decentralized Autonomous Organization):**

   - Responsible for modifying sensitive parameters in the Protocol.

2. **Pool Management:**

   - The `poolsConfig` contract manages whitelisted pools.

3. **Exchange Configuration:**

   - The `exchangeConfig` contract is responsible for configuring exchange-related parameters.

4. **Staking Rewards Configuration:**

   - The `stakingConfig` contract handles configuration related to staking rewards (e.g., cooldown periods, unstake durations).

5. **Staking:**

   - Users who stake SALT and receive xSALT in return.

6. **Liquidity Providers:**

   - Users who provide liquidity shares to specific pools.

7. **Rewards Emitter:**

   - A mechanism (not explicitly defined in the provided code) that emits rewards for staking and liquidity.

8. **Airdrop Contract:**
   - A contract (referenced in the `transferStakedSaltFromAirdropToUser` function) that manages the transfer of xSALT from an airdrop to users.

## Approach Taken-in Evaluating The Salty Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Salty Protocol.

    I start with the following contracts, which play crucial roles in the Salty protocol:

    **Main Contracts I Looked At**

    I start with the following contracts, which play crucial roles in the Salty protocol:

                AccessManager.sol
                DAO.sol
                Proposals.sol
                Pools.sol
                PoolMath.sol
                SaltRewards.sol
                CollateralAndLiquidity.sol
                Liquidizer.sol
                Upkeep.sol
                Staking.sol
                Airdrop.sol

    I started my analysis by examining the intricate structure and functionalities of the `Salty.IO` protocol, delving into key components such as the `ArbitrageSearch` contract, which leverages an `Automatic Atomic Arbitrage` mechanism for profitable opportunities within decentralized exchanges. The DAO contracts, including DAO.sol and DAOConfig.sol, play a crucial role in `governance`, `proposing` and `voting` on various actions, and `adjusting parameters`. Further, exploration encompassed Launch contracts managing `airdrop rewards`, decentralized exchange launches, and `initial token distribution`, as well as Pools contracts orchestrating reserve management, `arbitrage`, and `reward distribution`.

    Diving into the `Price Feed` section, `CoreChainlinkFeed.sol`, `CoreSaltyFeed.sol`, `CoreUniswapFeed.sol`, and `PriceAggregator.sol` were scrutinized for their roles in fetching and `aggregating` prices from different sources. `Rewards-related` contracts, such as `Emissions.sol`, `RewardsConfig.sol`, `RewardsEmitter.sol`, and `SaltRewards.sol`, were explored to understand the emission and distribution mechanisms. The `Stable` section covered contracts like `CollateralAndLiquidity.sol`, Liquidizer.sol, StakingConfig.sol, and USDS.sol, managing collateral, liquidity, and stablecoin operations.

    The Staking section involved detailed analysis of Liquidity.sol, Staking.sol, StakingConfig.sol, and StakingRewards.sol, focusing on SALT token staking, `liquidity provision`, and `reward distribution`. The protocol's essential utility contracts, including AccessManager.sol, ExchangeConfig.sol, ManagedWallet.sol, Salt.sol, SigningTools.sol, and Upkeep.sol, were also examined for their roles in managing access control, configuring components, handling wallets, implementing the `native token`, verifying signatures, and performing various protocol `upkeep` tasks. The analysis aimed to provide a comprehensive understanding of the `Salty.IO` protocol, considering both individual contract functionalities and their interdependencies.

2.  **Documentation Review**:

    Then went to Review [This Docs](https://docs.salty.io/) and [this](https://tech.salty.io/) for a more detailed and technical explanation of salty protocol.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Codebase Quality

Overall, I consider the quality of the Salty protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Salty Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                                     |
| **Code Comments**                        | During the audit of the Salty contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                               |
| **Documentation**                        | The documentation and Video of the Salty project is quite comprehensive and detailed, providing a solid overview of how Salty protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as `diagrams`, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 99% this is Good.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. but some order of functions does not follow the Solidity Style Guide According to the Solidity Style Guide, functions should be grouped according to their visibility and ordered: constructor, receive, fallback, external, public, internal, private. Within a grouping, place the view and pure functions last.                                                                                                                                                                                                                                                                                                                 |
| **Error**                                | Use custom errors, custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in try-catch blocks, and are easier to re-use and maintain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Imports**                              | In Salty protocol the contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Salty protocol. These risks encompass concentration risk in ArbitrageSearch, Proposals.sol risk and more, third-party dependency risk, and centralization risks arising.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. **ArbitrageSearch.sol**

   - **Mathematical Precision**: The contract relies on mathematical calculations for arbitrage opportunity identification. Precision errors or rounding issues in these calculations could lead to inaccurate results, affecting the profitability assessment.

   - **Assumption of Unimodal Profit Function**: The `_bisectionSearch` function assumes a unimodal profit function (having only one peak). This assumption may not hold true in all market conditions, leading to potential miscalculation of optimal arbitrage amounts.

   - **Token Supply and Decimals**: The contract mentions that certain assumptions have been made regarding token supply and decimals (e.g., uint112.max). These assumptions may not be valid for all tokens, and changes in token characteristics could impact the accuracy of the calculations.

2. **DAO.sol**

   - **Liquidity Bootstrapping Risks**: The process of forming Protocol Owned Liquidity (`POL`) involves depositing tokens and creating liquidity pools. The success of this process depends on the availability of sufficient bootstrapping rewards (bootstrappingRewards) and assumes that the tokens have already been transferred to the contract. Any issues with the availability of rewards or token transfers can impact liquidity formation.

3. **Proposals.sol**

   - **Quorum Calculation**: The method `requiredQuorumForBallotType` calculates the required quorum based on the total amount of staked SALT. The method should ensure that the quorum is not susceptible to manipulation.

4. **Airdrop.sol**

   - **Staking Approval for All Users**: The Airdrop contract approves the entire SALT balance for staking, and this approval applies to all authorized users. If there are concerns about potential misuse of approvals, a more granular approach could be considered to approve only what is needed for each user.

5. **PoolUtils.sol**

   - **Dust Value as Constant**: The constant DUST is set to 100, representing a threshold for treating token reserves less than dust as if they don't exist. Depending on the token's decimal precision, this value might not be universally applicable. The assumption of 18 decimals is mentioned, but it's important to ensure this constant is appropriate for all tokens.

   - **Arbitrary Precision Arithmetic**: The use of arithmetic operations involves fixed constants like 100, 1000, etc. It's crucial to consider the potential impacts of these fixed values and whether they accommodate various token sizes and decimal precisions. Unintended behavior could occur if tokens with different precision or values are used.

6. **PoolsConfig.sol**

   - **Limited Pool Diversity**:The contract limits the maximum number of whitelisted pools. A potential systemic risk arises if the diversity of available pools is insufficient to accommodate various token pairs. A limited pool ecosystem may result in reduced liquidity and increased vulnerability to market fluctuations.

   - **Adjustable Parameters**:The contract includes adjustable parameters (maximumWhitelistedPools and maximumInternalSwapPercentTimes1000). The risk lies in the potential misuse or unintentional changes to these parameters. Improper adjustments might lead to unexpected behavior, impacting the functionality and security of the protocol.

7. **PoolStats.sol**

   - **Potential Arithmetic Overflow**: The contract tracks arbitrage profits for each pool using the `_arbitrageProfits` mapping. In the `_updateProfitsFromArbitrage` function, when updating profits, there is no check for potential arithmetic overflow. If the sum of profits becomes very large, it could lead to unexpected behavior, potentially affecting the accuracy of reward distributions.

   - **Fixed Arbitrage Distribution Model**: The contract divides the arbitrage profits equally among the three pools involved in the arbitrage path (`WETH -> arbToken2 -> arbToken3 -> WETH`). This fixed distribution model assumes an equal contribution from each pool, which may not accurately reflect the actual contribution of each pool to the arbitrage. A more dynamic and proportional distribution model based on individual contributions could be considered.

8. **PoolMath.sol**

   - **Complex Mathematical Operations**: The contract involves `complex` mathematical operations, particularly `quadratic` equations and square root calculations. While these operations are necessary for the intended functionality, they introduce a risk of precision errors, especially in the context of Ethereum's limited computational capabilities.

9. **RewardsConfig.sol**

   - **Arbitrary Precision Math**: The contract involves arithmetic operations with percentages represented as integers (e.g., `rewardsEmitterDailyPercentTimes1000`). While the contract attempts to handle precision using `1000x` multipliers, there's a risk of precision loss in `arithmetic` operations, especially when performing calculations over extended periods.

   - **Fixed Adjustment Steps**: The contract uses fixed step sizes for adjustments (e.g., `±250 for percentage adjustments`). In a more decentralized system, it might be preferable to allow token holders or a broader governance mechanism to vote on adjustment sizes, providing a more inclusive decision-making process.

10. **SaltRewards.sol**

    - **Gas Limit Consideration**: The contract `approves` an unlimited amount of `SALT` for transfers to the `stakingRewardsEmitter` and `liquidityRewardsEmitter`. While this is done for efficiency, it may pose a risk if these emitters are compromised, as they can potentially drain the entire SALT balance.

11. **USDS.sol**

    - **No Cap on Minting**: The USDS contract does not enforce any cap on the minting of `USDS` tokens. Without a `maximum limit`, there is a risk of minting an `excessive` amount of tokens, which could lead to unexpected consequences, such as dilution of value.

12. **CollateralAndLiquidity.sol**

    - **Arithmetic Operations**: The contract uses arithmetic operations in several places, such as calculating token values and ratios. Ensure that these calculations cannot result in overflow or underflow. For instance, check the potential overflow/underflow risks in the `maxWithdrawableCollateral` and `maxBorrowableUSDS` functions.

13. **Upkeep.sol**

    - **Reward Mechanism**: The distribution of rewards (`step2`) depends on the manual calculation of a reward percentage. Ensure that the reward distribution mechanism is transparent, fair, and resistant to manipulation. Consider using well-established reward distribution patterns or decentralized mechanisms.

    - **Timed Operations**: The contract relies on time-based triggers for certain operations (e.g., `emissions and rewards distribution`). Ensure that time-dependent functions are secure and resistant to timestamp manipulation. Consider using block numbers or other secure timestamp verification mechanisms.

    - **Vesting Mechanism**: The linear distribution of tokens from vesting wallets (`step10 and step11`) spans over ten years. The long duration introduces potential risks associated with changing circumstances or evolving project conditions. Consider providing flexibility in the vesting mechanism to adapt to unforeseen changes.

### Centralization Risks:

1. **ArbitrageSearch.sol**

   - **Governance of Constants**: Some constants, such as `MIDPOINT_PRECISION`, are hardcoded in the contract. If these values need adjustment due to changing market conditions, the contract might require updates. Governance mechanisms for such changes are not explicitly defined, and a centralized update mechanism could introduce centralization risks.

2. **DAO.sol**

   - **Initial Exclusions**: Thecontract has default `exclusions` for certain countries, and the DAO itself can remove these exclusions. The initial exclusions and subsequent changes depend on DAO governance, which may have `centralization` implications.

3. **Proposals.sol**

   - **Active Proposal Tracking**: The Proposals contract maintains a mapping to track which users have `active` proposals. If not managed correctly, this could centralize the proposal creation process or result in inaccurate data.

4. **Airdrop.sol**

   - **Equal Distribution**: The contract `distributes` an equal share of the `airdrop` to all authorized users. While this may be intentional, it could centralize the distribution, and there might be scenarios where a more dynamic `distribution` based on user `participation` or other factors could be beneficial.

5. **BootstrapBallot.sol**

   - **Exclusive Decision Making**: The decision to start the exchange and the associated actions are `exclusive` to the outcome of the `vote`. This concentration of decision-making power in a single `vote` could be seen as a centralization risk, especially if there's limited participation or `diversity` in the voter base.

6. **InitialDistribution.sol**

   - **Fixed Token Distribution**: The distribution of tokens is `hard-coded` within the contract (`52 million Emissions, 25 million DAO Reserve, 10 million Initial Development Team, 5 million Airdrop Participants, 8 million Liquidity and Staking Bootstrapping`). This fixed distribution may not be adaptable to changing circumstances, and any misalignment with the evolving needs of the ecosystem could be a centralization risk.

7. **PoolUtils.sol**

   - **Limited Pool Diversity Consideration**: The logic assumes that limiting the internal `swap` amount as a percentage of reserves is sufficient to prevent certain attacks. However, the effectiveness of this strategy depends on the `diversity` of pools and their respective reserves. If the pool ecosystem is limited, attackers might exploit the constraints more easily.

   - **Atomic Arbitrage Considerations**: The comment mentions that the `limitation` on the internal swap amount is intended to make sandwich attacks less profitable due to `atomic arbitrage`. While this concept might deter some attacks, its effectiveness relies on the assumption that arbitrage opportunities are consistently available and profitable. A lack of such opportunities could weaken the defense mechanism.

   - **PerformUpkeep Reward Mechanism**:The protocol awards 5% of pending arbitrage profits to users calling `Upkeep.performUpkeep()`. Depending on the frequency and effectiveness of this mechanism, it might introduce centralization risks if a small group of users consistently benefits from arbitrage opportunities. The distribution of rewards should be carefully evaluated.

   - **Hardcoded Values**: The use of hardcoded values like `100`, `1000`, and `100,000` may limit the flexibility of the system. If these values are meant to be configurable, their hardcoded nature could hinder adaptability to changing market conditions or ecosystem dynamics.

8. **PoolStats.sol**

   - **Fixed Arbitrage Path**: The PoolStats assumes a fixed arbitrage path (WETH -> arbToken2 -> arbToken3 -> WETH) when calculating profits. If the protocol evolves, and additional or different arbitrage paths are introduced, the contract might need adjustments. A more adaptive and configurable approach to accommodate changing arbitrage paths could enhance decentralization.

9. **Pools.sol**

   - **Arbitrage Functionality**: The `_arbitrage` function allows users to exploit arbitrage opportunities. While arbitrage can be profitable, it also introduces the risk of potential front-running and manipulation. Depending on market conditions, this could impact the fairness and stability of the system.

10. **PoolMath.sol**

    - **Assumption of Token0 Excess**: The comment in the code assumes that token0 is in excess in terms of current reserve ratio. This assumption may introduce a `centralization` risk if it does not hold true for all scenarios or if it is based on centralized decision-making.

    - **Precision Reduction Consideration**: The code performs precision reduction by shifting numbers to normalize them to 80 bits. While this may be necessary for computational reasons, it introduces a `centralization` risk if the decision to use 80 bits is based on centralized considerations that may not be transparent to external stakeholders.

11. **RewardsConfig.sol**

    - **Fixed Adjustment Steps**: The contract uses fixed step sizes for adjustments (e.g., `±250 for percentage adjustments`). In a more decentralized system, it might be preferable to allow token holders or a broader governance mechanism to vote on adjustment sizes, providing a more inclusive decision-making process.

12. **RewardsEmitter.sol**

    - **Lack of Governance**: There is no `governance` mechanism implemented in the contract for making decisions related to emission rates or changing configurations. A lack of governance introduces `centralization` concerns.

13. **CollateralAndLiquidity.sol**

    - **Borrowing Limits**: The maxBorrowableUSDS function determines the maximum amount of USDS a user can borrow. This centralized control may lead to concerns about transparency and fairness. Consider a decentralized governance mechanism to determine borrowing limits or at least allow the community to participate in the decision-making process.

    - **User Wallet Tracking**: The contract maintains a list of wallets with borrowed USDS. While this functionality is necessary for liquidation, centralizing this information raises privacy concerns. Consider implementing privacy-preserving techniques or using decentralized identifiers (DIDs) to manage user identities more securely.

14. **Liquidity.sol**

    - **Exclusive Control Over Liquidity Management**: The contract has exclusive control over the deposit and withdrawal of liquidity from the Pools contract. Consider implementing decentralized governance mechanisms to involve the community in decision-making processes related to liquidity management.

15. **ManagedWallet.sol**

    - **Confirmation with Ether**: Relying on the sending of Ether for confirmation introduces an additional layer of complexity and may not be suitable for all use cases. The contract currently checks if at least 0.05 ether is sent, and this approach might be limiting, especially if used with custodial wallets that do not support Ether transfers. Consider alternative confirmation mechanisms that are more widely supported.

    - **Fixed Timelock Duration**: The timelock duration for confirming wallet changes is fixed at 30 days. This fixed duration might not be suitable for all scenarios. Consider making the timelock duration configurable or implementing a dynamic timelock mechanism based on the specific requirements of the application.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Salty protocol.**

## Conclusion

In general, The Salty.IO protocol presents a well-designed architecture. The Salty protocol exhibits strengths comprehensive testing, and detailed documentation, we believe the team has done a good job regarding the code. However, there are areas for improvement, including governance features, profit distribution optimization, and code documentation. Systemic risks include initial governance role assignment and dynamic proposal generation, while centralization risks involve certain contracts and functions. Recommendations include enhancing governance features, optimizing profit distribution, improving documentation, addressing systemic risks, and mitigating centralization risks. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
30 hours