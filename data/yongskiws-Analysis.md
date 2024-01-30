# Salty Audit Analysis
![Saltylogs](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/d4c3faa1-c73d-4dad-b1c8-5180fedac0eb)

Salty.IO is a Decentralized Exchange built on the Ethereum blockchain, employing Automatic Atomic Arbitrage (AAA) to generate yield and offer Zero Fees on all swaps. Through AAA, market inefficiencies are during the swapping process, resulting in profits that are subsequently distributed to liquidity providers and stakers. These earnings are utilized to establish Protocol Owned Liquidity (POL) for the DAO.

Furthermore, Salty.IO introduces USDS, an overcollateralized ERC20 stablecoin native to the protocol, backed by the WBTC/WETH liquidity pool. The exchange is launched with 100% decentralization, where all parameters, regional exclusions, whitelisting, and contracts are under the control of the DAO itself.



# Overview

  

Structuring the code is crucial for conducting a thorough analysis of the code base. This enables auditors to predict the level of contract difficulty, identify critical contracts, and determine security-critical features such as payment functions or assembly usage. Additionally, this approach accurately measures the time required to perform a detailed audit and helps determine the cost of a project audit. To achieve efficiency, clarity, and improvements, it is essential to have a comprehensive view of the entire architecture. This enables the identification of the basic structure, relationships between modules, and key design patterns. By understanding the big picture, we can analyze the complexity of the code and reveal its strengths and potential for improvement with confidence.

  

-  **Lines:** total lines of the source unit

-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)

-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)

-  **Comment Lines:** lines containing single or block comments

-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

  

# Consensys Review

| Type | File                        | Lines | nLines | nSLOC | Comment Lines | Complex. Score | Capabilities |
|------|-----------------------------|-------|--------|-------|---------------|-----------------|--------------|
| 📝   | USDS.sol                    | 61    | 58     | 30    | 10            | 27              |              |
| 📝   | Upkeep.sol                  | 291   | 277    | 155   | 48            | 151             | ♻️           |
| 🎨   | StakingRewards.sol          | 306   | 294    | 157   | 62            | 170             |              |
| 📝   | StakingConfig.sol           | 100   | 96     | 66    | 12            | 37              |              |
| 📝   | Staking.sol                 | 213   | 202    | 106   | 37            | 106             |              |
| 📝   | StableConfig.sol            | 153   | 147    | 98    | 20            | 53              |              |
| 📚   | SigningTools.sol            | 30    | 29     | 19    | 2             | 24              | 🖥🔖         |
| 📝   | SaltRewards.sol             | 149   | 143    | 82    | 25            | 110             |              |
| 📝   | Salt.sol                    | 40    | 38     | 22    | 4             | 19              |              |
| 📝   | RewardsEmitter.sol          | 151   | 148    | 87    | 29            | 89              |              |
| 📝   | RewardsConfig.sol           | 101   | 97     | 66    | 15            | 37              |              |
| 📝   | Proposals.sol               | 446   | 421    | 242   | 63            | 243             | 🧮           |
| 📝   | PriceAggregator.sol         | 197   | 187    | 123   | 23            | 68              | ♻️           |
| 📚   | PoolUtils.sol               | 69    | 66     | 30    | 18            | 35              | 🧮           |
| 🎨   | PoolStats.sol               | 147   | 140    | 82    | 21            | 73              |              |
| 📝   | PoolsConfig.sol             | 156   | 147    | 87    | 20            | 69              |              |
| 📝   | Pools.sol                   | 427   | 410    | 212   | 94            | 147             |              |
| 📚   | PoolMath.sol                | 228   | 224    | 65    | 131           | 29              |              |
| 🎨   | Parameters.sol              | 123   | 122    | 92    | 14            | 53              |              |
| 📝   | ManagedWallet.sol           | 90    | 88     | 46    | 20            | 36              | 💰           |
| 📝   | Liquidizer.sol              | 153   | 148    | 88    | 28            | 73              |              |
| 🎨   | Liquidity.sol               | 163   | 158    | 80    | 37            | 68              |              |
| 📝   | InitialDistribution.sol     | 75    | 74     | 50    | 8             | 34              |              |
| 📝   | ExchangeConfig.sol          | 86    | 83     | 55    | 15            | 38              |              |
| 📝   | Emissions.sol               | 62    | 61     | 34    | 11            | 22              |              |
| 📝   | DAOConfig.sol               | 198   | 190    | 126   | 35            | 69              |              |
| 📝   | DAO.sol                     | 388   | 376    | 239   | 47            | 237             | 🧮           |
| 📝   | CoreUniswapFeed.sol         | 145   | 139    | 81    | 23            | 67              | ♻️           |
| 📝   | CoreSaltyFeed.sol           | 54    | 52     | 32    | 6             | 18              |              |
| 📝   | CoreChainlinkFeed.sol       | 73    | 70     | 44    | 14            | 20              | ♻️           |
| 📝   | CollateralAndLiquidity.sol  | 352   | 338    | 176   | 71            | 183             |              |
| 📝   | BootstrapBallot.sol         | 86    | 84     | 49    | 11            | 36              | 🧮           |
| 🎨   | ArbitrageSearch.sol         | 137   | 134    | 69    | 38            | 44              | Σ            |
| 📝   | Airdrop.sol                 | 101   | 96     | 51    | 17            | 49              |              |
| 📝   | AccessManager.sol           | 79    | 75     | 31    | 21            | 20              | 🧮           |
| 📝📚🎨 | Totals                      | 5630  | 5412   | 3072  | 1050          | 2554            | 🖥💰🧮🔖♻️Σ      |
  

### Diagrams Risk
![riskdiagrams](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/889a2fad-680b-4b71-a0c5-bc1728d48d1d)


### Diagrams Source Lines (sloc vs. nsloc)
![Sourcelines](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/0bd83427-d9f7-42ec-8211-1b5d6d3d735b)

  

### Exposed Functions

  
| 🌐Public | 💰Payable |
|----------|-----------|
| 174      | 1       |

| External | Internal | Private | Pure | View |
|----------|----------|---------|------|------|
| 131      | 205      | 0     | 11  | 72  |


### StateVariables

  
| Total | 🌐Public |
|-------|----------|
| 217    | 196       |


  

### Capabilities


| Solidity Versions observed | 🧪 Experimental Features | 💰 Can Receive Funds | 🖥 Uses Assembly | 💣 Has Destroyable Contracts |
|----------------------------|--------------------------|-----------------------|--------------------------|-----------------------------|
| =0.8.22 | - | yes | (1 asm blocks) | - | -

| 📤 Transfers ETH | ⚡ Low-Level Calls | 👥 DelegateCall | 🧮 Uses Hash Functions | 🔖 ECRecover | 🌀 New/Create/Create2 |
|------------------|---------------------|-----------------|------------------------|-------------|-----------------------|
| - | - | - | yes | yes | - |

| ♻️ TryCatch | Σ Unchecked |
|------------|-------------|
| yes | yes |



### Dependencies / External Imports
| Dependency / Import Path                                      | Count |
|----------------------------------------------------------------|-------|
| chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol | 1     |
| openzeppelin-contracts/contracts/access/Ownable.sol           | 10    |
| openzeppelin-contracts/contracts/finance/VestingWallet.sol     | 2     |
| openzeppelin-contracts/contracts/security/ReentrancyGuard.sol  | 8     |
| openzeppelin-contracts/contracts/token/ERC20/ERC20.sol         | 7     |
| openzeppelin-contracts/contracts/token/ERC20/IERC20.sol        | 4     |
| openzeppelin-contracts/contracts/token/ERC20/extensions/IERC20Metadata.sol | 1     |
| openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol | 12    |
| openzeppelin-contracts/contracts/utils/Strings.sol             | 1     |
| openzeppelin-contracts/contracts/utils/math/Math.sol           | 4     |
| openzeppelin-contracts/contracts/utils/structs/EnumerableSet.sol | 4     |
| v3-core/interfaces/IUniswapV3Pool.sol                         | 1     |
| v3-core/libraries/FixedPoint96.sol                            | 1     |
| v3-core/libraries/FullMath.sol                                | 1     |
| v3-core/libraries/TickMath.sol                                | 1     |

  
  

### AST Node Statistics
![functionscallsSalty](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/2e6412d6-3f8d-4afd-be4c-10e036b99eb4)
![Aseemblycalls](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/5eca9d6d-35c1-43b8-8042-fe79c60b4c10)
![AStelements](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/02dc1da3-cb02-4dbe-b596-8b39bd8bea1f)

  
  

# Approach to Auditing

  

- We began by thoroughly reviewing and read docs Salty procotol the provided documentations, as well as reviewing the video provided. Here, we made note of important information, noted down unclear parts and overall tried to fully understand how the protocol should function. We also took notes of integrated and inherited protocols and mapped out possible risk areas.

  

- While we were studying the docs, we had ran static analyzers and linters through the contracts to detect basic vulnerabilities and other non critical errors. The result of our static analysis was then compared to the provided bot reports and those deemed known were removed.

  

- After the documentation review and static analysis, we started the process of manual code inspection. We noted how the contracts were divided into different modules and we inspected the contracts in each module carefully. We stuidied each of the contracts, tested how each function behaves and compared that to how they're expected to behave. We then tried out various attack ideas, including the ones provided by the sponsors. We noted the dependencies, inheritancies, and possible vulnerabilities that can arise from them. We made comparisons to issues found in protocols of the same kind (including older commits) to see if the same mistakes were repeated and how effective the provided fixes were.

  

- After this was done, we made notes on the issues we found and prepared the analysis report.

  
  

# Codebase quality

Overall, I consider the quality of the Salty codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The codebase demonstrates moderate maintainability with well-structured functions and comments, promoting readability. It exhibits reliability through defensive programming practices, parameter validation, and handling external calls safely. The use of internal functions for related operations enhances code modularity, reducing duplication. Libraries improve reliability by minimizing arithmetic errors. Adherence to standard conventions and practices contributes to overall code quality. However, full reliability depends on external contract implementations like openzeppelin & uniswap V3 Core                                                                       |
| **Code Comments**                        | The contracts have comments that are used to explain the purpose and functionality of different parts of the contracts, making it easier for developers to understand and maintain the code. The comments provide descriptions of methods, variables, and the overall structure of the contract.For-Exmaple: The code comments in the "DAO" contract provide essential information. The imported interfaces and contracts are declared, and the contract's title and purpose are described.                                                                                                                                                                         |
| **Testing**                              | The audit scope of the contracts to be audited is 99% and it should be aimed to be 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |


| Codebase Aspect        | Description                              | Explanation                 |
|------------------------|----------------------------------------|----------------------------------------|
| 1. **Folder Structure** | Well-organized folder structure. | - The folder structure facilitates easy navigation. <br> - Each folder has a clear responsibility.  |
| 2. **Modularization**   | Code is divided into logical modules. | - Each section has a primary responsibility. <br> - Modularization facilitates maintenance.   |
| 3. **Documentation**    | Comprehensive code documentation. | - Each function has a clear explanation. <br> - Contracts and critical functions are well-documented. |
| 4. **Security Measures** | Prevention and mitigation of security attacks. | - Ongoing audit for potential vulnerabilities. <br> - Protection against reentrancy and overflows.   |
| 5. **DAO Governance**    | Code related to DAO governance. | - DAO adjustable parameters are stored correctly. <br> - Voting functions and proposal handling are documented. |
| 6. **Smart Contract Interaction** | Clear smart contract interfaces. | - Functions and parameters are easily understandable. <br> - Inter-contract communication is well-explained. |
| 7. **Price Feeds and Oracle Integration** | Use of price aggregators and oracle integration. | - Use of Chainlink and Uniswap TWAP is explained. <br> - Redundancy mechanism and reliability of price aggregator. |
| 8. **Staking and Rewards** | Implementation of staking and reward mechanisms. | - Clear explanation of the use of "userShare". <br> - Management of SALT emissions and reward distribution to liquidity providers. |



### Strengths :

Exemplary unit test coverage: The codebase shines with its meticulous unit testing strategy. The comprehensive suite of unit tests ensures the reliability and resilience of the system's core functionality, providing confidence in its performance under various scenarios.


Artful integration of Natspec: A commendable strength is the thoughtful integration of Natspec. The use of this documentation format not only improves code readability, but also exemplifies a commitment to providing clear and human-friendly documentation that fosters a more collaborative and accessible development environment.

  

### Weaknesses :

Opportunities to improve documentation: While the codebase stands out in many ways, there is an opportunity for refinement in the documentation. Addressing this area of improvement can increase the overall clarity of the codebase, make it more accessible, and facilitate seamless collaboration among developers. In essence, the codebase demonstrates proficiency in testing methodologies and documentation practices. Strengthening the documentation further would strengthen the codebase's comprehensibility and contribute to an even more polished and developer-friendly ecosystem.


# Analysis of the code base and diagram architecture

## ArbitrageSearch
![ArbitrageSearch sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/04dadb55-7b0d-4350-9c6d-afe51eee1026)

## DAO
![DAO sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/4a9e3bfe-4f01-4304-9203-dae08f1a865b)

## DAOConfig
![DAOCONFIG sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/4c5a3c34-8462-4238-ad6a-6b3c86d03ac2)

## Proposals
![proposal sol](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/1f819144-0a9a-4108-9084-af51ba636b47)

## Airdrop
![airdrop](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/10606573-edb6-4e79-ba9e-9db79e86114d)

## BootstrapBallot
![BootstrapBallot](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/e2a1dac3-0aeb-4945-9703-723f20c56f36)

## InitialDistribution
![InitialDistribution](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/11f8b3f5-d5a9-4ddb-965a-68b9be2cafd6)

## PoolUtils
![PoolUtils](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/49f928d9-baad-4d83-bb66-75026c11f5b5)

## PoolsConfig
![PoolsConfig](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/728d9784-8393-47d6-97bb-9af1ba86db61)

## PoolStats
![PoolStats](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/8fbc6f2b-c049-44d5-8d26-2e8befbafcb6)

## Pools
![Pools](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/facfeaa0-49ed-4ec5-8352-f58907d05ca9)

## PoolMath
![PoolMath](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/7d47ca73-34b4-4a4b-b191-8965de49c228)

## RewardsConfig
![RewardsConfig](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/c7439136-9bdf-47f0-8365-49351dbc75ab)

## PriceAggregator
![PriceAggregator](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/a098a660-eb86-44b8-903e-4bb6b01f2b6b)

## RewardsEmitter
![RewardsEmitter](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/8d48d062-435c-4209-bc4a-eda5e4039d87)

## SaltRewards
![SaltRewards](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/c33970e8-96d4-417b-9c09-1d115c9f226b)

## StableConfig
![StableConfig](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/39aca442-3c3c-4325-aff9-3ffd406893e2)

## CollateralAndLiquidity
![CollateralAndLiquidity](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/a0fdec07-787c-47d1-a316-13beaa52cbe9)

## Liquidizer
![Liquidizer](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/4bb3366f-e10d-4b51-84b6-9c49fe3ac5de)

## StakingConfig
![StakingConfig](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/cb383a90-be93-40b7-a617-a3ed7ad3d8bf)

## Liquidity
![Liquidity](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/25e41dca-6fdd-4c1f-bbdc-1982e9f6723a)

## StakingRewards
![StakingRewards](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/2512a3a5-5a4e-4c1e-919d-e638edc6eaa8)

## Staking
![Staking](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/03e953c9-6afb-4475-b6ec-415a61c904f3)

## Upkeep
![Upkeep](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/f266affb-54d1-4976-85e2-0ee37c96c073)

# Approach we followed when reviewing the code


`Upkeep` a complex set of maintenance steps for various contracts and operations within them, including token swaps, arbitrage, reward distribution, and liquidity management.

`StakingRewards` StakingRewards abstract contract that provides staking functionality for SALT and issuance of rewards in the form of SALT tokens. These contracts allow for increases and decreases in user stakes, claim of rewards, as well as addition of SALT rewards into a specific pool.
  
`Airdrop `maintains a list of users eligible to claim Salty airdrops. Authorized users can claim a certain amount of SALT after a successful BootstrappingBallot contract. Claims can be made after authorization and when airdrop claims are permitted.

`CollateralAndLiquidity` functions as a liquidity and collateral manager on an exchange. Users can deposit collateral (WBTC and WETH), borrow USDS, withdraw collateral, and liquidate user positions that do not meet the minimum collateral ratio. The contract also monitors the amount of USDS borrowed by each user and provides various display functions for value information in USD.

`DAO` manages different aspects of the system and provides flexibility to change parameters and perform other management actions through a decentralized voice and decision-making system.

`Proposals` to manage voting and proposals in the SALT DAO ecosystem


# Parameters Code Base Decentralization

The DAO has exclusive control over an extensive list of exchange parameters. For safety, parameters are adjusted incrementally or decrementally within a specified range.

### DAO:

- `bootstrappingRewards`: The amount of SALT provided as a bootstrapping reward for a new pool when a new token is whitelisted. Default: 200k SALT

- `percentPolRewardsBurned`: For rewards distributed to the DAO, the percentage of SALT that is burned with the remaining staying in the DAO for later use. Default: 50%

- `baseBallotQuorumPercent`: The minimum amount of xSALT required for ballot quorum. Default: 10%

- `ballotMinimumDuration`: How many days minimum a ballot has to exist before it can be taken action on. Default: 10 days

- `requiredProposalPercentStake`: The percent of staked SALT that a user has to have to make a proposal. Default: 0.50%

- `maxPendingTokensForWhitelisting`: The maximum number of tokens that can be pending for whitelisting at any time. Default: 5

- `arbitrageProfitsPercentPOL`: The share of the arbitrage profits that are sent to the DAO to form SALT/USDS Protocol Owned Liquidity. Default: 20%

- `upkeepRewardPercent`: The share of the WETH arbitrage profits sent to the DAO that are sent to the caller of DAO.performUpkeep(). Default: 5%

### POOLS:

- `maximumWhitelistedPools`: The maximum number of pools that can be whitelisted at any one time. Default: 50

- `maximumInternalSwapPercent`: For swaps internal to the protocol, the maximum swap size as a percent of the reserves. Default: 1%

### REWARDS:

- `rewardsEmitterDailyPercentTimes1000`: The target daily percent of rewards distributed by the stakingRewardsEmitter and liquidityRewardsEmitter. Default: 1%

- `emissionsWeeklyPercentTimes1000`: The weekly percent of SALT emissions that will be distributed from the Emissions contract. Default: 0.5%

- `stakingRewardsPercent`: The percentage of rewards received by SALT stakers (compared to liquidity providers). Default: 50%

- `percentRewardsSaltUSDS`: The percent of SALT Rewards that will be sent directly to the SALT/USDS pool. Default: 10%

### STAKING:

- `minUnstakeWeeks`: The minimum number of weeks for an unstake request at which point minUnstakePercent of the original staked SALT is reclaimable. Default: 2 weeks

- `maxUnstakeWeeks`: The maximum number of weeks for an unstake request at which point 100% of the original staked SALT is reclaimable. Default: 52 weeks

- `minUnstakePercent`: The percentage of the original staked SALT that is reclaimable when unstaking the minimum number of weeks. Default: 20%

- `modificationCooldown`: The minimum time between staking and unstaking or between depositing and withdrawing liquidity (to avoid rewards sniping). Default: 1 hour

### STABLECOIN:

- `rewardPercentForCallingLiquidation`: The collateral percent that a user receives for instigating the liquidation process. Default: 5%

- `maxRewardValueForCallingLiquidation`: The maximum reward value (in USD) that a user receives for instigating the liquidation process. Default: $500 USD

- `minimumCollateralValueForBorrowing`: The minimum USD value of collateral - to borrow an initial amount of USDS. Default: $2500 USD

- `initialCollateralRatioPercent`: The initial maximum collateral / borrowed USDS ratio. Default: 200%

- `minimumCollateralRatioPercent`: The minimum ratio of collateral / borrowed USDS below which positions can be liquidated. Default: 110%

- `percentArbitrageProfitsForStablePOL`: The percent of arbitrage profits that are used to form USDS/DAI Protocol Owned Liquidity. Default: 5%

### PRICE FEED:

- `maximumPriceFeedPercentDifference`: The maximum percent difference between two non-zero PriceFeed prices when aggregating price. Default: 3%

- `setPriceFeedCooldown`: The required cooldown between calls to setPriceFeed. Default: 35 days


# Risk Assessment:


## Gas Risks

which could introduce additional risks related to price volatility, liquidity, and slippage. Ensuring adequate liquidity and monitoring gas currency prices is essential to maintain the contract's functionality.

Review gas usage in each function and ensure there are no significant gas leaks.

Evaluate whether the Bisection Search and algebra implementations have been optimized efficiently.
  
## Solidity Version

The contract interacts with various external contracts, which could contain bugs or vulnerabilities. Regularly updating the contract and its dependencies can help minimize this risk.

## Dependency Risks

external libraries and interfaces, which could introduce additional risks related to version compatibility, security, and maintenance. Ensuring that the dependencies are up-to-date, well-maintained, and secure is crucial for the contract's long-term viability.

## Governance Mechanism

Review governance mechanisms, especially the role of DAOs. Ensure decision-making and asset management policies are well defined.

## Feed Price

Review and evaluate oracle security against various types of attacks, including price manipulation attacks or fake data intrusions.
Consider using multiple oracles to reduce the risk of a single point of failure.
Implement a continuous monitoring system to detect suspicious or unusual price changes.
Set early warnings and emergency actions if there are indications of price manipulation or oracle failure.

# Conclusion

In general, the Salty project exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.



### Time spent:
36 hours