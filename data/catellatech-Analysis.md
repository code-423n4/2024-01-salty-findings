<div align="center">
  <h1> Salty.IO: Smart Contracts Analysis </h1>
  <h5>An Ethereum-based DEX with zero swap fees, yield-generating Automatic Arbitrage, and a native WBTC/WETH backed stablecoin.</h5>
  <!-- <img src="https://www.decent.xyz/" width="200" height="200" > -->
</div>

<details> 
<summary> See Index </summary>

## Index

- [Index](#index)
- [1. What is Salty.IO: A Comprehensive Overview](#1-what-is-saltyio-a-comprehensive-overview)
- [2. Diving into all the contracts provided by the protocol](#2-diving-into-all-the-contracts-provided-by-the-protocol)
  - [ArbitrageSearch.sol](#arbitragesearchsol)
  - [DAO.sol](#daosol)
  - [DAOConfig.sol](#daoconfigsol)
  - [Proposals.sol](#proposalssol)
  - [Parameters.sol](#parameterssol)
  - [Airdrop.sol](#airdropsol)
  - [BootstrapBallot.sol](#bootstrapballotsol)
  - [InitialDistribution.sol](#initialdistributionsol)
  - [PoolUtils.sol](#poolutilssol)
  - [PoolsConfig.sol](#poolsconfigsol)
  - [PoolStats.sol](#poolstatssol)
  - [Pools.sol](#poolssol)
  - [PoolMath.sol](#poolmathsol)
  - [CoreChainlinkFeed.sol](#corechainlinkfeedsol)
  - [CoreUniswapFeed.sol](#coreuniswapfeedsol)
  - [PriceAggregator.sol](#priceaggregatorsol)
  - [CoreSaltyFeed.sol](#coresaltyfeedsol)
  - [RewardsConfig.sol](#rewardsconfigsol)
  - [Emissions.sol](#emissionssol)
  - [RewardsEmitter.sol](#rewardsemittersol)
  - [SaltRewards.sol](#saltrewardssol)
  - [USDS.sol](#usdssol)
  - [StableConfig.sol](#stableconfigsol)
  - [CollateralAndLiquidity.sol](#collateralandliquiditysol)
  - [Liquidizer.sol](#liquidizersol)
  - [StakingConfig.sol](#stakingconfigsol)
  - [Liquidity.sol](#liquiditysol)
  - [StakingRewards.sol](#stakingrewardssol)
  - [Staking.sol](#stakingsol)
  - [ManagedWallet.sol](#managedwalletsol)
  - [AccessManager.sol](#accessmanagersol)
  - [Salt.sol](#saltsol)
  - [SigningTools.sol](#signingtoolssol)
  - [Upkeep.sol](#upkeepsol)
  - [ExchangeConfig.sol](#exchangeconfigsol)
  - [Roles and Privileges](#roles-and-privileges)
- [3. Codebase Quality](#3-codebase-quality)
- [4. Security Approach for the Provided Contracts](#4-security-approach-for-the-provided-contracts)
- [5. Addressing Architectural \& Sistemic Risks](#5-addressing-architectural--sistemic-risks)
  - [Architectural recommendations](#architectural-recommendations)
- [6. Integrate the StatefulFuzz Test Suite as an Integral Component of Your Testing Process](#6-integrate-the-statefulfuzz-test-suite-as-an-integral-component-of-your-testing-process)
  - [Implement Your Own Stateful Fuzzing Test Suite](#implement-your-own-stateful-fuzzing-test-suite)
  - [Building Your Invariants](#building-your-invariants)
  - [Write your Handler](#write-your-handler)
- [7. Recommendations beyond the code](#7-recommendations-beyond-the-code)
- [8. Insights, learnings \& common patterns](#8-insights-learnings--common-patterns)

</details>

**The approach and steps we employed to examine the underlying code.**

Our approach to analyzing the source code of the **Salty.IO** Protocol was to simplify the information provided by the protocol, using a variety of diagrams to visually clarify the project's key contracts and break down each important part of these contracts. This enhances understanding for developers, security researchers, and users alike. We identified the fundamental concepts and employed simpler language to explain the functionality and goals of the **Salty.IO** Protocol. Furthermore, we organized the information logically into separate sections, each with identifying titles, to provide a clear overall picture of the subject. Our primary goal was to make the information more accessible and easy to understand.

> NOTE: The provided diagrams are based on our interpretation of the protocol. In the event that any of them is not 100% accurate but aligns with the protocol's initial concept, we invite you to reach out to us via Discord (catellatech). We are willing to share the diagrams for the project team to make any necessary modifications. If the diagrams are deemed accurate, they are fully available for the protocol to use in its documentation.

## 1. What is Salty.IO: A Comprehensive Overview

**Salty.IO** aims to provide a decentralized exchange experience with a focus on automatic atomic arbitrage (AAA) to generate profits, eliminate fees for users, and distribute rewards to liquidity providers and token stakers. The inclusion of a stablecoin **(USDS)** and the use of the **SALT** token for governance and staking add additional layers of functionality to the platform.

**Overview diagram:**
<img src="https://github.com/catellaTech/salty/blob/main/Salty.Overview.drawio.png?raw=true">

## 2. Diving into all the contracts provided by the protocol

### ArbitrageSearch.sol

- The contract is designed to search for circular arbitrage opportunities after a swap has occurred **(e.g., from WETH to WETH)**.
- It uses the bisection search method to find the optimal amount for arbitrage and gain profits.
- The logic is based on calculating profits at the midpoint and to the right of the midpoint, deciding in which direction to continue searching for the optimal amount.

### DAO.sol
- This contract **represents a decentralized autonomous organization responsible for managing various governance actions within Salty.IO ecosystem**. 
- It allows users to **propose** and **vote** on governance actions such as **changing parameters, whitelisting/unwhitelisting tokens, sending tokens, calling other contracts, and updating the website**. 
- The contract handles the creation and tracking of proposals, enforcement of voting requirements, and execution of approved proposals. 
- Its functions range from finalizing parameter **ballots** to forming and managing **protocol-owned liquidity**, **burning rewards, and handling geographical exclusion**. 

### DAOConfig.sol

The **DAOConfig** smart contract in **Salty.IO** is crucial for **configuring** and **governing** a decentralized autonomous organization (DAO). As a DAO-owned contract, it stores essential parameters such as **initial rewards**, **reward burn percentages**, and **voting quorum thresholds**. Its functions enable the **DAO** to dynamically adjust these parameters, ensuring flexibility and adaptability. Integration with the **OpenZeppelin** library guarantees the DAO's exclusive ownership of modifications.

### Proposals.sol

The **Proposals** smart contract allows **SALT token holders** to **propose** and **vote** on various types of ballots, such as parameter changes, token whitelisting/unwhitelisting, token transfers, contract calls, and website URL updates. The contract ensures ballot uniqueness, tracks and validates user voting power, enforces quorums, and provides a mechanism for users to alter votes before finalization. Proposals require a minimum amount of **staked SALT**, and certain restrictions are in place to prevent spam proposals. Users can modify their votes before confirmation, and there is a minimum duration before a proposal can be executed. Specific logic is implemented to handle token-related proposals, such as whitelisting and unwhitelisting, along with other **Salty.IO** project-specific proposal types.

### Parameters.sol

The **Parameters abstract smart contract** defines a set of parameter types, encapsulating configurations for **pools**, **staking**, **rewards**, **stablecoins**, **DAO**, and **price aggregation**. It introduces an enumeration, ParameterTypes, listing these categories. The contract includes an internal function, **\_executeParameterChange**, intended for inheritance. This function enables the modification of parameters based on their types and whether to increase or decrease them, providing a modular and organized approach to govern the protocol's behavior. The actual implementation of parameter change functions is expected in contracts inheriting from Parameters within the **Salty.IO** ecosystem.

### Airdrop.sol

- The **Airdrop contract** is responsible for managing the distribution of **SALT tokens** to users who qualify for the **Airdrop**. Eligible users are those who have retweeted the launch announcement and voted in the **BootstrapBallot**.
- The contract uses the **EnumerableSet** library to maintain a set of authorized user addresses. The amount of **SALT tokens** allocated to each user for the **Airdrop** is determined equitably among all authorized users and is set when **claiming is allowed**.
- The contract implements functions to **authorize wallets**, **allow claiming** once certain conditions are met, and execute the **SALT claim** for authorized users. Additionally, it maintains state to track whether a user has already claimed the Airdrop.
- The ability to claim the Airdrop is activated after the **BootstrapBallot** concludes. This contract also utilizes the exchange configuration interface (**IExchangeConfig**), the staking interface (**IStaking**), and the SALT token interface (**ISalt**), ensuring integration with other components in the **Salty.IO** ecosystem. Furthermore, it inherits from **ReentrancyGuard** to prevent **reentrancy attacks**.

### BootstrapBallot.sol

- The **BootstrapBallot contract** facilitates a **voting mechanism** within the **Salty.IO** project, allowing **airdrop** participants to vote on whether to start up the exchange and determine initial geo restrictions. The contract utilizes the **OpenZeppelin** library, specifically the **ReentrancyGuard** for protection against **reentrancy attacks**. It interacts with the **IExchangeConfig** and **IAirdrop** interfaces, representing configurations for the exchange and the airdrop, respectively. The ballot has a set duration specified during contract deployment.

- Users cast their votes with a **YES** or **NO** choice regarding starting the exchange, distributing **SALT**, and establishing initial geo restrictions. **Each voter can only cast one vote**, and their eligibility is confirmed off-chain through a valid signature. The signature is verified using the **SigningTools** library. Upon voting, users are also authorized to receive the airdrop.

- The contract includes **variables** to track voting tallies for starting the exchange with '**startExchangeYes**' and '**startExchangeNo**'. Once the ballot duration is complete, the **finalizeBallot** function checks whether the '**startExchangeYes**' votes outnumber the '**startExchangeNo**' votes. If the condition is met, it calls certain functions from the exchange configuration and **DAO contracts**, indicating approval to start the exchange. The **ballotFinalized** state is updated, and an **event** (**BallotFinalized**) is emitted, providing information about the outcome of the ballot.
- The contract guards against reentrancy throughout the voting and finalization processes.

### InitialDistribution.sol

- The **InitialDistribution** contract manages the initial distribution of the **SALT token**. Upon approval from the **BootstrapBallot**, it distributes **SALT tokens** to various components of the ecosystem.
- The distribution includes **allocations for emissions**, **DAO reserve**, **development team vesting**, **airdrop participants**, and **rewards for liquidity and staking bootstrapping**.
- The contract ensures a one-time execution of the distribution process by verifying the **SALT balance**. This mechanism ensures that **52 million SALT** goes to **emissions**, **25 million** to the **DAO reserve vesting wallet**, **10 million** to the **development team vesting wallet**, **5 million** to **airdrop participants**, and **8 million** for **liquidity** and **staking** bootstrapping rewards distributed to whitelisted pools through the **ISaltRewards** contract.

### PoolUtils.sol

- This contract is a **library** that simplifies the manipulation of liquidity pools.
- It defines constants like **DUST** to set a **minimum threshold** and **STAKED_SALT** to identify **staked SALT** not associated with a specific pool.
- It also provides **internal functions** to generate unique identifiers for token pairs and perform internal swaps within the protocol, limiting the input amount to a specific percentage of maximum reserves.
- These measures aim to prevent **sandwich attacks**, where attempts are made to manipulate the price by executing fast trades before and after a significant transaction in the liquidity protocol.

### PoolsConfig.sol

- **PoolsConfig**, is owned by the **DAO** and can only be modified by the **DAO**.
- It serves as a **configuration** for the **pools system** and allows the **DAO** to manage the list of pools authorized for token exchange.
- The events **PoolWhitelisted**, **PoolUnwhitelisted**, **MaximumWhitelistedPoolsChanged**, and **maximumInternalSwapPercentTimes1000Changed** are emitted when pools are **added** or **removed** from the **whitelist** and when maximum settings are changed.
- The **whitelist** is managed through an enumerable set of **bytes32** storing the **IDs** of authorized pools.
- Additionally, the contract provides functions to **change** the **maximum number of allowed pools** and the **maximum percentage for internal swaps** within the protocol.
- View functions allow obtaining information about the current state of the pools configuration, such as the **number of authorized pools**, whether a pool is whitelisted, and getting the IDs of authorized pools.

### PoolStats.sol

- **PoolStats**, is abstract and **keeps track of arbitrage profits** generated by pools for proportional reward distribution. The **IPoolStats** interface is implemented within this contract. The storage structure includes **mappings** to record **arbitrage profits** and **arbitrage indices** for each token pair in the pools.

- **The contract has internal functions** to update arbitrage profits and arbitrage indices. It also provides a function to clear accumulated profits for the pools, which is called at the end of the **Upkeep.performUpkeep** function.

- The **main function**, **\_calculateArbitrageProfits**, examines the arbitrage profits generated since the last **Upkeep.performUpkeep** call and proportionally allocates the profits to pools that contributed to them.

- In the** view functions**, calculated profits for all whitelisted pools and arbitrage indices for a specific pool can be obtained.

### Pools.sol

- The **Pools contract** is pivotal in **Salty.IO** system, managing reserves for swaps and providing functions to **add** and **remove liquidity**, perform direct token swaps, and uniquely, **execute atomic arbitrage** following user swaps.
- Additionally, it records statistics for proportional distribution of rewards to liquidity providers.
- Its logic involves **reserve adjustments**, **arbitrage calculations**, and **handling user deposits** and **withdrawals**, thereby contributing to the efficiency and sustainability of the **DEX**.
- Collaboration with contracts like **CollateralAndLiquidity** and **DAO** reinforces its central role in the operation of the ecosystem.

### PoolMath.sol

- The **PoolMath contract** is a library that **provides mathematical functions** used in the logic of the main contract called **IPools**. This contract is designed to be used in the **Salty.IO system** and specifically focuses on** managing liquidity in a token pair** on the platform.

- The **\_mostSignificantBit function** determines the most significant bit of a **non-zero** number, while **\_maximumMSB** finds the maximum most significant bit among several values.
- The **\_zapSwapAmount** function calculates the amount of one token that should be swapped for another so that the added liquidity has the same ratio as the reserves in the pool after the swap.

- The **contract's logic uses the quadratic formula** to solve for the amount of a token that should be exchanged to maintain the correct ratio between reserves after the swap.
- The library also includes utility functions to handle and normalize reduced precision values, as well as safety checks to ensure that token amounts are not too small.

### CoreChainlinkFeed.sol

- The **CoreChainlinkFeed** contract **serves as a price feed leveraging Chainlink oracles** to obtain **real-time prices for Bitcoin (BTC) and Ethereum (ETH) in USD**.
- Adhering to the **IPriceFeed** interface, the contract incorporates **constants**, a **constructor** initializing **Chainlink oracle addresses**, and functions facilitating the retrieval of the latest price data.
- The **latestChainlinkPrice** **internal function** converts **Chainlink's 8-decimal price to 18 decimals**, verifying the timeliness of the data within a specified maximum delay.
- The **external functions** **getPriceBTC** and **getPriceETH** offer a user-friendly interface to access the converted prices, with **error-handling mechanisms** in place to handle potential data retrieval failures, ensuring the reliability of the provided price information.

### CoreUniswapFeed.sol

- The **CoreUniswapFeed** contract **functions as a decentralized price feed**, providing **30-minute Time-Weighted Average Prices (TWAPs)** for **WBTC (Wrapped Bitcoin)** and** WETH (Wrapped Ethereum)** based on the respective **Uniswap v3 pools**. The contract adheres to the **IPriceFeed** interface and is initialized with references to **Uniswap v3 pool addresses** for **WBTC/WETH** (**UNISWAP_V3_WBTC_WETH**) and **WETH/USDC** (**UNISWAP_V3_WETH_USDC**). Additionally, it references **ERC-20 tokens** **(wbtc, weth, and usdc)** and determines the order of token pairs based on their addresses.

- The **core functionality** is encapsulated in the **\_getUniswapTwapWei** **internal function**, which calculates the **TWAP** given a **Uniswap v3 pool** and a specified time interval. The **external function** **getTwapWBTC** combines **TWAPs** from the **WBTC/WETH** and **WETH/USDC** pools, adjusting the values based on token order. Similarly, the **getTwapWETH** function provides the **TWAP** for **WETH**, and both are used to obtain prices for **WBTC** and **WETH** through the **external functions getPriceBTC** and **getPriceETH**.

- To enhance reliability, the contract includes **error-handling mechanisms** with **try/catch blocks**, ensuring that if any failures occur during the **TWAP** calculation, the returned prices default to zero.
- The contract's **modular design** and adherence to the **IPriceFeed** interface make it suitable for integration into decentralized finance (DeFi) applications, allowing them to access reliable and decentralized price information for **WBTC** and **WETH**.

### PriceAggregator.sol

- **PriceAggregator** implements a price aggregator that compares three different price feeds for **BTC (Bitcoin) and ETH (Ethereum)**. This contract is used to mitigate the risk of obtaining incorrect or manipulated data from a single source by discarding outliers. The **contract inherits from the Ownable library** from the **OpenZeppelin Contracts package** and is **designed to be used as a decentralized oracle**.

- The three price feeds are implemented through contracts that comply with the **IPriceFeed interface**. By default, three specific contracts **(CoreUniswapFeed, CoreChainlinkFeed, and CoreSaltyFeed)** are assigned to the variables **priceFeed1, priceFeed2, and priceFeed3**.

- The **contract sets some constraints for updating the price feeds**. A cooldown period (**priceFeedModificationCooldown**) restricts how often a price feed can be updated to allow for performance review before another update is made. The maximum allowed percentage change between two prices is controlled by **maximumPriceFeedPercentDifferenceTimes1000**. Functions are provided to change these settings.

- The **\_aggregatePrices** method is used to determine the aggregated price from the three price feeds, discarding those that are too far apart from the others. This process helps improve the robustness of the oracle.

- The **getPriceBTC** and **getPriceETH** functions return the aggregated prices for **BTC** and **ETH**, respectively, using the three price feeds. These functions employ **error-handling mechanisms** to ensure that, in case one feed fails, the aggregated price remains zero and an exception is thrown if the resulting price is invalid.

- The **contract emits events** **to report changes in the price feeds**, the **maximum allowed change**, and the **cooldown period between updates**.

### CoreSaltyFeed.sol

- **CoreSaltyFeed** **serves as a price feed oracle for BTC (Bitcoin) and ETH (Ethereum) using the Salty.IO pools**. This contract is designed to retrieve and provide prices for these assets with **18 decimals**. The contract **relies on interfaces** such as **IPriceFeed**, **IPools, IExchangeConfig**, and utility functions from **PoolUtils**.

- The **constructor initializes the contract** with instances of the **IPools** interface (pools), as well as **ERC-20 tokens** representing **WBTC (wrapped Bitcoin), WETH (wrapped Ethereum)**, and **USDS (Salty Stablecoin)** obtained from the provided **IExchangeConfig** interface.

- The **getPriceBTC function** returns the price of **BTC** in terms of **USDS (Salty stablecoin)**. It does this by obtaining the reserves of **WBTC and USDS** from the **Salty.IO pools** using the **getPoolReserves** function. If either the **WBTC or USDS reserves** fall below a certain threshold (**PoolUtils.DUST**), the **price is considered invalid**, and the function returns zero. **Otherwise, it calculates the price by dividing the USDS reserves by the WBTC reserves and adjusting for the differing decimals between the two assets.**

- Similarly, the **getPriceETH function** returns the **price of ETH in terms of USDS**. It retrieves the reserves of** WETH and USDS from the pools**, checks for invalid reserves, and calculates the price by dividing the **USDS reserves by the WETH reserves**.

- **In both functions, the returned prices are in units of USDS**, and the prices are calculated based on the relative reserves of the assets in the **Salty.IO pools**. If the reserves are considered too small **(below the DUST threshold)**, the price is set to zero to indicate that it is invalid.

### RewardsConfig.sol

**RewardsConfig contract**, designated for **ownership by a DAO**, facilitates the dynamic adjustment of key parameters governing reward distribution within the protocol. By implementing the **IRewardsConfig** interface and **inheriting from the Ownable contract**, **the DAO gains exclusive control over modifying critical factors**. These include the daily percentage of rewards from emitters, the weekly proportion of **SALT** emissions directed to **liquidity** and **xSALT holders**, the allocation percentage between **xSALT** holders and **liquidity providers**, and the proportion of **SALT rewards** designated for the **SALT/USDS pool**.

- The contract offers functions, exclusively accessible to the **DAO**, to incrementally adjust these parameters within specified ranges, ensuring flexibility in responding to changing dynamics while maintaining transparency through emitted events logging parameter modifications.

### Emissions.sol

- Contract **designed to handle the storage and gradual distribution of SALT emissions over time**. It is closely integrated with other components of the system, including the **ISaltRewards** contract, **IExchangeConfig**, **IRewardsConfig**, and the **ISalt token**.
- The **performUpkeep function**, intended to be called by the designated **Upkeep contract**, orchestrates the emission distribution process. **The emission rate defaults to 0.50% of the remaining SALT balance per week and is dynamically adjusted based on the time elapsed since the last upkeep call**.
- The calculated amount of **SALT** to be distributed is transferred to the **ISaltRewards** contract for subsequent allocation to the **staking** and **liquidity** rewards emitters.
- This contract ensures that emissions are systematically released and utilized within the protocol, contributing to the overall reward distribution mechanism.

### RewardsEmitter.sol

- **Contract responsible for storing SALT rewards for later distribution at a default rate of 1% per day to those holding shares in the specified StakingRewards contract**. The gradual emission rate aims to offset natural rewards fluctuation and create a more stable yield. **This contract also serves as an easy mechanism to see the current yield for any pool, as the emitter acts like an exponential average of the incoming SALT rewards.**

- **The contract allows users to add SALT rewards for later distribution to specified whitelisted pools**. Specified **SALT rewards** are transferred from the sender. The **performUpkeep** function transfers a percentage **(default 1% per day)** of the currently held rewards to the specified **StakingRewards** pools. The percentage to transfer is interpolated based on the time elapsed since the last **performUpkeep** call.

- Additionally, **the contract has views to query pending rewards for specific pools**. The implementation uses the **SafeERC20** library to handle **ERC-20 token transfers safely** and **ReentrancyGuard** to prevent **reentrancy attacks**.

### SaltRewards.sol
- Utility contract that temporarily holds **SALT rewards** from emissions and **arbitrage profits** during the **performUpkeep()** function. The purpose is to send **SALT** rewards to the **stakingRewardsEmitter** and **liquidityRewardsEmitter**, with proportions for the latter based on each pool's share in generating recent **arbitrage profits**.

- The **contract allows sending initial SALT rewards to the liquidityRewardsEmitter** and **stakingRewardsEmitter** **during the initial distribution**. Additionally, it facilitates the **performUpkeep()** function, which distributes **SALT** rewards based on profits generated by different pools. The distribution includes direct rewards to the **SALT/USDS** pool and proportional rewards to **stakers** and **liquidity providers**.

- The implementation uses the **SafeERC20 library to handle ERC-20 token transfers safely**. The contract requires proper authorization from the **InitialDistribution** and **Upkeep** contracts for specific functions.

### USDS.sol
- The **USDS contract is an ERC-20 token contract representing a stablecoin named USDS** **(Salty.IO Stablecoin)**. 
- Users can borrow **USDS** by depositing **WBTC/WETH liquidity** as collateral through the **CollateralAndLiquidity** contract. 
- The **default initial collateralization ratio is set at 200%**, meaning the collateral value must be at least double the borrowed **USDS** value. **There is also a minimum default collateral ratio of 110%**, below **which positions can be liquidated by any user**.

- The **contract includes events for USDS minting and token burning**. The **setCollateralAndLiquidity** function is used to set the **CollateralAndLiquidity** contract address, and it can only be called once, after which ownership is renounced. 
- The **mintTo** function is callable only by the **CollateralAndLiquidity** contract, **allowing the minting of USDS when users deposit sufficient collateral**. The **burnTokensInContract** function is **used to burn USDS tokens that are sent to the contract**, typically during **repayment** or **liquidation** processes.

### StableConfig.sol
- **StableConfig** contract is like the control panel for **Salty** (**DAO**) running a stablecoin protocol. It's owned by the **DAO**, allowing them to tweak important settings that impact how the protocol works. 
- **These settings include things like rewards for liquidation, collateral ratios, and other specific adjustments to how the protocol behaves**. 
- The **contract has defined ranges and methods to make sure these adjustments stay within reasonable limits**. Whenever something gets changed, the contract broadcasts **events**, making sure everyone is on the same page. 

### CollateralAndLiquidity.sol

- **CollateralAndLiquidity** serves as a hub for managing liquidity on **Salty.IO** and **allows users to deposit WBTC and WETH liquidity as collateral for borrowing USDS stablecoin**. **Users can deposit collateral, withdraw collateral, borrow USDS, repay USDS, and even liquidate positions that fall below the required collateral ratio**. 
- The contract interacts with various other contracts, such as the **price aggregator**, **stable configuration**, and a **liquidizer**. **Users are incentivized to deposit collateral by earning rewards, and those who call the liquidation function on undercollateralized positions receive a default percentage of the liquidated collateral**. 
- The **contract also keeps track of users with borrowed USDS**, ensuring they maintain sufficient collateral ratios. 
- It **provides views for calculating the market value of tokens, collateral value, and maximum borrowable amounts**. 

### Liquidizer.sol
- **This facilitates the conversion of tokens from collateral liquidations and protocol-owned liquidity withdrawals into USDS, which is subsequently burned**. To address undercollateralized liquidation scenarios due to a rapid drop in the value of **WBTC/WETH collateral**, the contract withdraws a percentage of protocol-owned liquidity when there is insufficient **USDS** to **burn**. 
- It **interacts with external contracts** such as **ICollateralAndLiquidity, IPools, and IDAO**, and uses **SafeERC20** for secure interactions with **ERC20** tokens. 
- The **incrementBurnableUSDS** function is called when a user's collateral is liquidated, indicating that the borrowed **USDS** should be burned. 
- During **upkeep**, the **performUpkeep** function swaps **tokens** for **USDS** and **burns excess SALT tokens**, withdrawing POL if necessary to reach the amount of **USDS** that needs to be burned. 

### StakingConfig.sol
- **StakingConfig contract serves as a configuration manager for staking-related parameters, exclusively modifiable by the DAO**. 
- It extends the **OpenZeppelin Ownable contract** to ensure that only the **DAO** has the authority to make modifications. 
- The configurable parameters include **minUnstakeWeeks**, representing the minimum number of weeks for an unstake request; **maxUnstakeWeeks**, denoting the maximum number of weeks for an unstake request; **minUnstakePercent**, indicating the percentage of **staked SALT reclaimable** for the minimum unstake weeks; and **modificationCooldown**, setting the minimum time between increasing and decreasing user shares in **SharedRewards** contracts. 
- The **DAO** can dynamically adjust these parameters within specified ranges to fine-tune the staking system's behavior. 
- **Events are emitted** upon successful modifications, providing transparency to stakeholders.

### Liquidity.sol
- This contract, **extends functionality to manage liquidity in external pools**. 
- Users can seamlessly **deposit** and **withdraw** liquidity in token pairs, participating in the associated staking rewards program. 
- The **contract employs a dual zapping mechanism to balance token amounts during liquidity addition**, ensuring the maintenance of token ratios in the pool. 
- **Access control** measures restrict interactions to addresses with exchange access, enhancing security.
- Users can simultaneously increase their **liquidity share** and **claim rewards** when depositing, and similarly, decrease their share and claim rewards during withdrawals. 
- The **contract also handles collateral pools separately**, preventing direct deposits and withdrawals. 
- Through **emitting events**, the contract promotes transparency by broadcasting essential **liquidity-related activities**. 

### StakingRewards.sol
- **Abstract implementation** facilitating reward distribution in **Salty.IO** ecosystem. 
- Users receive rewards, represented by **SALT tokens**, proportionally to their stake or liquidity shares in specific pools. 
- **The contract is designed to be inherited by other contracts**, such as **Staking.sol** and **Liquidity.sol**, each defining the nature of shares in their respective contexts.

- **Key features include** the ability for users to increase and decrease their shares, which are proportional to the rewards they receive. A cooldown mechanism is implemented to prevent rapid modifications to shares, deterring potential exploitation. 
- The **contract also supports the addition of SALT rewards to specific whitelisted pools**, with a cooldown mechanism applied to this process as well.

- **Events are emitted** to provide transparency, notifying users when their shares are increased or decreased, when rewards are claimed, and when **SALT** rewards are added to pools.

- Some **view functions** offer insights into the total shares and rewards for specified pools, as well as details about a user's pending rewards, shares, virtual rewards, and cooldown status for specific pools.

### Staking.sol
- Smart contract **enabling users to stake SALT tokens and receive corresponding xSALT tokens in return**. These **xSALT** tokens can be redeemed after a specified unstaking period. 
- The contract extends the functionality of the "**StakingRewards**" contract and incorporates features specific to staking **SALT** tokens. 
- **Users can stake tokens, receive xSALT, and initiate unstaking processes with varying waiting periods**, where the duration affects the percentage of **SALT** tokens that can be reclaimed. Cancelling pending unstaking processes is also permitted. After the unstaking period elapses, users can claim their SALT tokens. 
- Additionally, the **contract facilitates the transfer of xSALT tokens from the Airdrop contract to users**. 

### ManagedWallet.sol

- **Contract designed to manage two wallet addresses:** a **main wallet** and a **confirmation wallet**. 
- Users can propose changes to these wallets, and the confirmation wallet has the authority to confirm or reject the proposed changes. 
- The confirmation process involves sending a specific amount of **Ether** to the contract, and upon confirmation, there is a **timelock period of 30 days** before the **proposed main wallet can confirm the change**. Once the timelock period has elapsed, the **main wallet can execute the change**, updating the **main** and **confirmation wallets** with the proposed ones. 
- The contract provides an organized mechanism for secure and controlled management of wallet addresses, useful in various decentralized applications.

### AccessManager.sol
- This contract s**erves as a decentralized access control mechanism**, utilizing a **whitelist** to **grant** or **deny** specific functionalities based on users' geographic regions. 
- Managed by a **DAO**, the contract maintains a **geoVersion** that increments when countries are excluded, requiring users to reverify. 
- Access is granted through **off-chain signatures** from authoritative signers, allowing users to request access, which, if verified, adds their wallet address to the whitelist. 
- The **DAO** can update excluded countries, triggering a **geoVersion** increment. 
- The contract provides flexibility for the **DAO** to modify regional restrictions or replace the access mechanism while ensuring existing assets can be withdrawn even if a user's region becomes restricted after depositing assets. 
- **Proposals** and **voting** remain unrestricted to prevent potential issues with universal access blocking.

### Salt.sol
- The **Salt contract is an ERC-20 token named "Salt" with the symbol "SALT"**. 
- It extends the **OpenZeppelin ERC-20 implementation** and has an **initial total supply of 100 million SALT tokens**. 
- The **burnTokensInContract** function allows users to send **SALT** tokens to the contract for burning, reducing the total supply. After receiving the tokens, the contract burns them and **emits a SALTBurned event**. 
- The **totalBurned** **view function** calculates the amount of **SALT** tokens burned by subtracting the current total supply from the initial supply.

### SigningTools.sol
- This library **provides a utility function for verifying signatures**. 
- It defines a constant public address **EXPECTED_SIGNER**, representing the expected signer for verification purposes. 
- The library includes the **_verifySignature** function, which **takes a message hash** and **a signature** as input and verifies whether the signature was produced by the expected signer. 
- The **function uses low-level assembly operations** to extract the **r, s, and v components from the signature** and then employs the **ecrecover** function to **recover the Ethereum address associated with the provided message hash and signature**. 
- The verification succeeds if the recovered address matches the expected signer. 
- This library is designed to be used in conjunction with **BootstrapBallot voting** and the default **AccessManager** for authentication purposes.

### Upkeep.sol
- The **Upkeep** contract **performs a series of maintenance tasks**. 
- Its functions include **token swapping**, **liquidity management**, **reward distribution**, and **handling protocol-owned liquidity (POL)**. 
- It **interacts with various contracts and protocols** within the system, such as **arbitrage profit conversion**, **reward distribution**, and **POL liquidity formation**. Additionally, it takes specific actions related to the native token **SALT**, such as **sending SALT emissions to reward contracts** and **distributing SALT from reward emitters**. 
- It also **manages DAO rewards** and gradually releases **SALT** tokens from vesting wallets over a **10-year period**. All these operations are encapsulated in a function called **performUpkeep()**, where each step is guarded against potential errors that could impact subsequent operations. 

### ExchangeConfig.sol
- This contract **serves as a configuration hub owned by the DAO**, facilitating the management of various parameters and contracts within the decentralized exchange ecosystem. 
- Key components include the **native token contracts (SALT, WBTC, WETH, DAI, USDS)**, a **managed team wallet**, and several** essential contracts** such as **DAO**, **Upkeep**, **InitialDistribution**, **Airdrop**, and **VestingWallets** for the **team and DAO**. 
- The contract ensures that certain parameters and contracts can only be set once, providing a secure and controlled environment. 
- Access to protocol components is determined by an **AccessManager**, allowing the **DAO** to update and include necessary functionalities. 
- The contract aims to **centralize** and **organize** critical configurations and contract references for efficient and secure decentralized exchange operations.

**Overview diagram:**
<img src="https://github.com/catellaTech/salty/blob/main/Salty.s.drawio.png?raw=true">

### Roles and Privileges
- **DAO**:
  - Owner of the **ExchangeConfig** contract.
  - Can set contracts related to the **DAO** **(dao, upkeep, initialDistribution, etc.)**.
  - Can set the **AccessManager** contract.

- **Managed Team Wallet**:
  - Represents a wallet managed by the team.
  - Can receive **SALT** tokens from the team vesting wallet (**teamVestingWallet**).

- **Airdrop**:
  - Guaranteed access to the system to perform actions related to airdrops.

- **Vesting Wallets (teamVestingWallet, daoVestingWallet)**:
  - Can release **SALT** tokens linearly over a period (10 years).
  - Used to gradually distribute tokens to the team and **DAO**.

- **Access Manager**:
  - Controls access to certain components of the protocol.
  - Allows checking if a wallet has access to certain contracts and functionalities.

- **Upkeep**:
  - Represents an automated maintenance task.
  - Can execute multiple maintenance steps, such as **exchanges**, **reward distribution, and liquidity formation.**

- **Salt Rewards**:
  - Can perform maintenance operations to distribute **SALT** rewards to rewards emitters and liquidity.


## 3. Codebase Quality

The code generally demonstrates good quality, with clear and readable coding practices and a **well-defined modular structure**. Descriptive comments are used, and interfaces are employed to promote interoperability. However, **additional attention to security is suggested, especially in contracts like "Upkeep," where potential vulnerabilities and reentrancy risks need to be meticulously addressed**. Additionally, exception handling could be improved. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The **Salty.IO** Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                          |
| **Code Comments**                        | During the audit of the **Salty.IO** contracts codebase, we found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like **NatSpec** could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                   |
| **Documentation**                        | The documentation of the **Salty.IO** project is quite comprehensive and detailed, providing a solid overview of how **Salty.IO** protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be reviewed is **99%**, with the aim of reaching **100%**.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

## 4. Security Approach for the Provided Contracts

- 🔒 **Access Control**: The contracts use an **AccessManager** to decide who gets to do what. It's like having a bouncer for specific actions. Roles such as **mainWallet** and **confirmationWallet** help set the rules in the **ManagedWallet** contract.

- 🔒 **Timelocks**: The **ManagedWallet** contract makes people wait (**timelock**) before certain actions can happen. This helps avoid quick and potentially harmful changes.

- 🔒 **Signature Check**: The **SigningTools** library has a cool trick (**_verifySignature**) to make sure that the right person signed a message. It's used in the **AccessManager** to make sure access requests are legit.

- 🔒 **Reentrancy Protection**: The **Upkeep** contract uses a trick (**ReentrancyGuard**) to stop anyone from trying to sneak in unwanted actions during certain moments.
  
- 🔒 **Token Safety**: The contracts use **SafeERC20** to move tokens around securely. It's like handling tokens with kid gloves to avoid accidents.

- 🔒 **Event Logging**: Important changes are recorded with events. It's like keeping a diary of what's happening, so everyone knows.

- 🔒 **Modularity and Interfaces**: The contracts are designed with **modularity** in mind, utilizing interfaces to define standard behaviors. This enhances **code readability, maintainability, and allows for easier integration with other contracts**.

## 5. Addressing Architectural & Sistemic Risks

- **The Anomalies of ERC-20 Tokens:**

  - ERC-20 Tokens come in all shapes and sizes. Here's a glimpse into some of the variants and potential problems that lurk in the shadows:

    - **Missing return values:** Some tokens don’t return a boolean on ERC-20 methods. For transactions requiring a status check, this can be a potent problem.
    - **Fee on transfer:** Some tokens sneak in a fee on every transfer while others can start doing so in the future.
    - **Upgradable tokens:** These tokens, like USDC, could morph into anything over time.
    - **Rebasing tokens:** These tokens magic away your balance by meddling with different contracts.
    - **Tokens with blocklists:** Some tokens put restrictions on certain transacting parties.
    - **Low/high decimals:** Token numbers can go from unusually low to abnormally high, causing calculation mishaps.
    - **Multiple token addresses:** These tokens exist in more than one places at on
    - **Risk:** Protocol is expected to interact with any erc20 with dex liquidity, as it can be potential payment token for **swapAndExecute** or **bridgeAndExecute**

- **Upkeep Complexity**: The **Upkeep** contract performs multiple steps in its **performUpkeep()** function, and a failure in any of these steps could have severe consequences. The complexity and interdependence of these steps could increase the attack surface and make it challenging to detect potential issues.

- **Dependency on External Contracts**: The **Upkeep** contract relies on interaction with various external contracts, such as **pools**, **exchangeConfig**, **dao**, and others. External dependencies increase complexity and operational risk, especially if these contracts are not thoroughly tested or if they change in the future.

- **Handling of Tokens and Ether**: **Token** and **ether** manipulation in the **Upkeep** contract involves fund handling risks. The use of **try/catch** to handle potential failures may obscure errors and make debugging challenging.

- **Atomic Updates**: Some processes, such as **updating variables** in contracts like **ManagedWallet**, might benefit from more secure and atomic patterns to avoid potential race conditions or state inconsistencies.

- **Access Privileges**: **Role** and **privilege** assignment, as implemented in **AccessManager**, is crucial. Any failure in role or permission management could result in unauthorized access or undue limitations.

- **Signature Security**: The signature validation in **SigningTools** is basic and relies on the signer's address. Enhancing the robustness and security of signature validations could be beneficial.

- **Upgradeability**: The ability to upgrade certain contracts, such as **AccessManager**, could introduce risks if not handled appropriately. The need for updates must be balanced with the security and trust in the new code.

### Architectural recommendations

💡 A robust model is indispensable for systematically evaluating project risks. One highly recommended approach involves employing effective risk modeling techniques. https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

💡 We suggest establishing a foundational architecture with a **System.sol** contract, serving as a central registry where all contracts are registered. The entire system can be organized around a singular contract, such as **SystemRegistry**. This contract acts as a cohesive entity that interconnects all other contracts, allowing for a comprehensive listing of all contracts within the system. Essentially, it functions as a registry, providing a centralized point to access information about all contracts in the system..

💡Evaluate the gas consumption of each function across different scenarios during testing to ensure optimal efficiency and identify potential areas for optimization.

💡Consider decentralized ownership models or implement multi-signature control for critical functions.

💡Ensure proper validation and handling of cross-chain interactions.

## 6. Integrate the StatefulFuzz Test Suite as an Integral Component of Your Testing Process

**Some attack ideas can be tested as invariants, for example:**

- **Fake Signatures**: The signature validation in **SigningTools** uses a basic verification based on the signer's address. An attacker might attempt signature spoofing attacks if they find weaknesses in this implementation.

- **Token Manipulation**: Token and ether manipulation operations in **Upkeep** could be subject to attacks if not handled correctly. Attackers might try to exploit possible flaws in fund handling for their benefit.

- **State Manipulation**: Given the complexity of state updates and interdependent contracts, there's a risk of state manipulation if transitions are not handled securely and atomically.

- **External Dependencies**: Dependency on various external contracts could introduce risks if those contracts are insecure or change unexpectedly. Attacks could aim to exploit these changes or weaknesses.

- **Receive Function Exploitation**: The **receive function** in **ManagedWallet** could be targeted if not handled correctly. Attackers might attempt to leverage this entry point for unwanted actions.

> With an understanding of these attack vectors, the protocol must implement more robust test suites.

### Implement Your Own Stateful Fuzzing Test Suite

- Create a new folder called "**invariant**" in your working environment. In this folder, you will house two **Solidity (.sol)** files named "**invariant.t.sol**" and "**handler.t.sol**" respectively. Once this setup is complete, you can commence writing your tests.

### Building Your Invariants

Start by crafting "**invariant.t.sol**," where you define your tests by laying the foundation for the 'invariant.'

### Write your Handler

Now, let's focus on creating your **handler.t.sol** file. This contract serves as an intermediary to control the contract that captures the state of the Fuzz stateful.

- Through the **handler**, you instruct your Foundry and the **Stateful Fuzzing test suite** to align correctly with the contract capturing the state of the Fuzz stateful. Essentially, you provide guidelines to **Foundry** on when to call specific functions for testing, all with precise instructions to prevent excessive function calls.

- Once your tests are configured, the next step is to write the "**handler.sol**" While you could embed assertions directly into your invariants, a cleaner approach is to compute them in the "**handler.sol**" This ensures that your assertions condense into a single line, maintaining logic validity regardless of variable input parameters. In the development of more intricate software or systems, invariants play a pivotal role in ensuring correctness.

## 7. Recommendations beyond the code

Attackers are becoming increasingly sophisticated, targeting protocols and their users not only through code vulnerabilities but also exploiting their social media presence, such as **X**. In these times more than ever, it is crucial to maintain robust layers of security. To enhance project security, it is imperative to establish control policies mandating **Two-Factor Authentication (2FA)** for all staff accounts and ensuring its widespread utilization whenever possible. It's important to acknowledge that achieving 100% foolproof security solely through smart contract codes is unattainable. To fortify security measures, consider implementing a more comprehensive policy that advocates for the use of **non-SMS 2FA methods**. For additional information, you can explore the recent **Simswap attack** on **Code4rena** detailed in this [Article](https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d).

## 8. Insights, learnings & common patterns

The contracts showcase a complex architecture with diverse functionalities, also offer valuable insights into security and design considerations. The need for a secure implementation from the outset, with clearly defined roles and privileges, is evident in the **AccessManager** contract. Transparency and compatibility between contracts are crucial, as demonstrated in the **ExchangeConfig** contract. Careful management of upgrades, as seen in **Upkeep**, and protection against **reentrancy attacks** are critical aspects. Proper implementation of **signatures** and **authentication**, as exemplified by **SigningTools**, is essential. Secure handling of tokens, as showcased in the **Salt** contract, is vital to prevent losses and ensure integrity. 

### Time spent:
44 hours