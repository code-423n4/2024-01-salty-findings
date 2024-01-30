# GAS OPTIMIZATION

# SUMMARY

|        | issue                                                                               | instance |
| ------ | ----------------------------------------------------------------------------------- | -------- |
| [G‑01] | Gas Optimization: Declare Variables Outside the Loop                                | 3        |
| [G‑02] | Gas Optimization: Using Assembly for Efficient Validation of `msg.sender`           | 31       |
| [G‑03] | Gas Optimization: Use Assembly for Writing Address Storage Values                   | 8        |
| [G‑04] | Do not calculate constants                                                          | 1        |
| [G‑05] | Gas Optimization: Change Public State Variable Visibility to Private                | 36       |
| [G‑06] | State variables can be packed to use fewer storage slots                            | 10       |
| [G‑07] | Gas Optimization Issue - Use Constants Instead of type(uintx).max                   | 15       |
| [G‑08] | Gas Optimization: Inlining Internal Functions                                       | 8        |
| [G‑09] | Use assembly to perform efficient back-to-back calls                                | 7        |
| [G‑10] | Gas Optimization: Using Modifiers Instead of Functions                              | 2        |
| [G‑11] | Using delete instead of setting info struct to 0 saves gas                          | 2        |
| [G‑12] | Cache external calls outside of loop to avoid re-calling function on each iteration | 1        |
| [G‑13] | Access mappings directly rather than using accessor functions                       | 2        |
| [G‑14] | Pre-increment and pre-decrement are cheaper than `+1`, `-1`                         | 9        |
| [G‑15] | Using constants instead of enum can save gas                                        | 1        |
| [G‑16] | Consider using OZ EnumerateSet in place of nested mappings                          | 5        |
| [G‑17] | Instead of counting down in for statements, count up                                | 1        |
| [G‑18] | Gas Efficiency through Hardcoded Address Usage                                      | 5        |

## [G-01] Gas Optimization: Declare Variables Outside the Loop

**Explanation of Issue:**
When variables are declared inside a loop in Solidity, a new instance of the variable is created for each iteration of the loop. This can result in unnecessary gas costs, especially when the loop is executed frequently or iterates over a large number of elements. To optimize gas usage, it is advisable to declare the variable outside the loop and initialize it to zero to avoid the creation of multiple instances.

**Examples of Code:**

- _Non-optimized Code:_

```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        for (uint256 i = 0; i < values.length; i++) {
            uint256 total; // This declaration is inside the loop and not initialized
            total += values[i];
        }

        return total; // This line will result in a compilation error; total is not visible here
    }
}
```

- _Optimized Code:_

```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total; // Declare the variable outside the loop without initializing

        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }

        return total;
    }
}
```

**Reason:**
Declaring the variable outside the loop without initializing it inside the declaration is still an improvement, as it avoids the creation of multiple instances of the variable during each iteration. However, initializing the variable to zero inside the declaration further enhances gas optimization. This is crucial, especially in scenarios where the loop is executed frequently or processes a substantial number of elements, contributing to more efficient and cost-effective smart contracts.

There are 3 instances of this issue:

```solidity
File:  src/stable/CollateralAndLiquidity.sol
323   address wallet = _walletsWithBorrowedUSDS.at(i);

326   uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;

329   uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L323

## [G-02] Gas Optimization: Using Assembly for Efficient Validation of `msg.sender`

**Explanation of Issue:**

When validating `msg.sender`, using assembly can significantly reduce the number of opcodes required, leading to a more gas-efficient implementation.

**Examples of Code:**

- _Non-optimized Code:_

  ```solidity
  contract NonOptimizedContract {
      function validateSender() public view {
          // Non-optimized: Standard Solidity validation of msg.sender
          require(msg.sender == address(0x123...), "Unauthorized");
      }
  }
  ```

- _Optimized Code:_
  ```ract OptimizedContract {
      function validateSender() public view {
          // Osolidity
  contptimized: Using assembly for efficient validation of msg.sender
          assembly {
              // Compare msg.sender directly in assembly
              if iszero(eq(sload(msg.sender.slot), 0x123...)) {
                  // Revert if the comparison fails
                  revert(0, 0)
              }
          }
      }
  }
  ```

**Reason:**

The non-optimized code performs a standard Solidity validation of `msg.sender`, which involves multiple opcodes and can contribute to higher gas costs.

In the optimized code, assembly is used to efficiently validate `msg.sender` with fewer opcodes. The `iszero` and `eq` assembly functions directly compare the `msg.sender` value, resulting in a more gas-efficient implementation. This optimization is valuable when frequent and efficient validation of the sender's address is required in smart contracts.

There are 31 instances of this issue:

```solidity
File:  src/AccessManager.sol
43  require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L43

```solidity
File:  src/ManagedWallet.sol
44  require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );

61  require( msg.sender == confirmationWallet, "Invalid sender" );

76  require( msg.sender == proposedMainWallet, "Invalid sender" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L44

```solidity
File:  src/Upkeep.sol
103    msg.sender == address(this),
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L103

```solidity
File:  src/dao/DAO.sol
297   require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );

318   require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );

329   require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );

362   require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L362

```solidity
File:  src/dao/Proposals.sol
86  if ( msg.sender != address(exchangeConfig.dao() ) )

124 require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

132 require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L86

```solidity
File:  src/launch/Airdrop.sol
48  require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );

58  require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L48

```solidity
File:  src/launch/InitialDistribution.sol
52  require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L52

```solidity
File:  src/pools/Pools.sol
72  require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );

142  require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );

172  require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L72

```solidity
File:  src/pools/PoolStats.sol
49   require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L49

```solidity
File:  src/rewards/Emissions.sol
42  require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L42

```solidity
File:  src/rewards/RewardsEmitter.sol
84  require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L84

```solidity
File:  src/rewards/SaltRewards.sol
108  require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );

119  require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L108

```solidity
File:  src/stable/Liquidizer.sol
83  require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );

134 require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L83

```solidity
File:  src/stable/USDS.sol
42   require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L42

```solidity
File:  src/staking/Staking.sol
90  require( msg.sender == u.wallet, "Sender is not the original staker" );

106  require( msg.sender == u.wallet, "Sender is not the original staker" );

132  require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L90

```solidity
File:  src/staking/StakingRewards.sol
65  if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown

105 if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L65

## [G-03] Gas Optimization: Use Assembly for Writing Address Storage Values

**Explanation of Issue:**
When writing to address storage values in Solidity, using assembly can provide a gas-efficient alternative. Assembly allows for direct interaction with the Ethereum Virtual Machine (EVM) and enables low-level operations that are not achievable in Solidity. By using assembly to write to address storage values, you can potentially lower the gas cost associated with storage operations.

**Examples of Code:**

- _Non-optimized Code:_

```solidity
contract MyContract {
    address private myAddress;

    function setAddressWithoutAssembly(address newAddress) public {
        myAddress = newAddress;
    }
}
```

- _Optimized Code:_

```solidity
contract MyContract {
    address private myAddress;

    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress);
        }
    }
}
```

**Reason:**  
In the non-optimized code, the address value is assigned directly in Solidity. In the optimized code, assembly is used to write to the storage location. This can result in a potential gas savings as assembly provides direct access to the Ethereum Virtual Machine (EVM) and allows for more efficient storage operations. However, it's essential to use assembly judiciously and consider the trade-offs in terms of readability and security when employing low-level operations. This optimization is particularly beneficial in scenarios where minimizing gas costs is a critical concern.
There are 8 instances of this issue:

```solidity
File:  src/ManagedWallet.sol
33		mainWallet = _mainWallet;
34		confirmationWallet = _confirmationWallet;

51		proposedMainWallet = _proposedMainWallet;
52   	proposedConfirmationWallet = _proposedConfirmationWallet;

80		mainWallet = proposedMainWallet;
81   	confirmationWallet = proposedConfirmationWallet;

87		proposedMainWallet = address(0);
88  	proposedConfirmationWallet = address(0);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L33-L34

## [G-04] Do not calculate constants

**Explanation of Issue:**
Calculating constants in Solidity can lead to inefficient gas usage. Constant variables are replaced at compile-time, which means that expressions assigned to constant variables are recomputed each time the variable is used. This results in unnecessary gas consumption.

**Examples of Code:**

- _Non-optimized Code:_
  ```solidity
  uint256 constant FEE_PERCENTAGE = (100 * 100) / totalSupply;
  ```
- _Optimized Code:_
  ```solidity
  uint256 constant FEE_PERCENTAGE = 1;
  ```

**Reason:**  
Calculating constants each time they are used wastes gas due to redundant computations. By assigning a constant value directly, we avoid recomputing the expression, leading to more efficient gas usage and better contract performance.
[Reffrence](https://code4rena.com/reports/2022-07-golom#g-24-do-not-calculate-constants)

There are 1 instances of this issue:

```solidity
File:  src/Salt.sol
13   uint256 public constant INITIAL_SUPPLY = 100 * MILLION_ETHER ;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol#L13

## [G-05] Gas Optimization: Change Public State Variable Visibility to Private

**Explanation of Issue:**

Public state variables expose sensitive data and operations to the entire blockchain, potentially compromising security and increasing gas costs unnecessarily. It's essential to restrict access to state variables to only what is absolutely necessary to maintain a secure and efficient smart contract.

**Examples of Code:**

- _Non-optimized Code:_

  ```solidity
  contract MyContract {
      uint256 public sensitiveData;

      function updateSensitiveData(uint256 _newValue) public {
          sensitiveData = _newValue;
      }
  }
  ```

- _Optimized Code:_

  ```solidity
  contract MyContract {
      uint256 private sensitiveData;

      function updateSensitiveData(uint256 _newValue) public {
          sensitiveData = _newValue;
      }
  }
  ```

**Reason:**

1. **Security:** Public state variables are susceptible to manipulation by any user on the blockchain, posing a significant security risk. By making state variables private, access is restricted, mitigating the potential for malicious attacks targeting sensitive data.

2. **Encapsulation:** Private state variables enhance encapsulation, encapsulating the internal logic of the contract and minimizing external interference. This improves contract maintainability and reduces the likelihood of bugs resulting from unintended external interactions.

3. **Gas Costs:** Public state variables incur higher gas costs due to the automatic generation of getter and setter functions by Solidity. By using private state variables, unnecessary gas consumption associated with these additional functions is eliminated, enhancing contract efficiency and reducing transaction costs.

There are 36 instances of this issue:

```solidity
File:  src/AccessManager.sol
29    uint256 public geoVersion;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L29

```solidity
File:  src/ManagedWallet.sol
    address public mainWallet;
    address public confirmationWallet;

	// Proposed wallets
    address public proposedMainWallet;
    address public proposedConfirmationWallet;

	// Active timelock
    uint256 public activeTimelock;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L20-L28

```solidity
File:  src/dao/DAO.sol
63  string public websiteURL;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L63

```solidity
File:  src/dao/DAOConfig.sol
24   uint256 public bootstrappingRewards = 200000 ether;

28   uint256 public percentPolRewardsBurned = 50;

40   uint256 public baseBallotQuorumPercentTimes1000 = 10 * 1000;

45   uint256 public ballotMinimumDuration = 10 days;

49   uint256 public requiredProposalPercentStakeTimes1000 = 500;

53   uint256 public maxPendingTokensForWhitelisting = 5;

57   uint256 public arbitrageProfitsPercentPOL = 20;

61   uint256 public upkeepRewardPercent = 5;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L24

```solidity
File:  src/dao/Proposals.sol
42   uint256 public nextBallotID = 1;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L42

```solidity
File:  src/launch/Airdrop.sol
32   uint256 public saltAmountForEachUser;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L32

```solidity
File:  src/launch/BootstrapBallot.sol
	uint256 public startExchangeYes;
	uint256 public startExchangeNo;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L29-L30

```solidity
File:  src/pools/PoolsConfig.sol
37   uint256 public maximumWhitelistedPools = 50;

41   uint256 public maximumInternalSwapPercentTimes1000 = 1000;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L37

```solidity
File:  src/rewards/RewardsConfig.sol
	uint256 public rewardsEmitterDailyPercentTimes1000 = 1000;  // Defaults to 1.0% with a 1000x multiplier

	// The weekly percent of SALT emissions that will be distributed from Emissions.sol to the Liquidity and xSALT Holder Reward Emitters.
	// Range: 0.25% to 1.0% with an adjustment of 0.25%
	uint256 public emissionsWeeklyPercentTimes1000 = 500;  // Defaults to 0.50% with a 1000x multiplier

	// By default, xSALT holders get 50% and liquidity providers get 50% of emissions and arbitrage profits
	// Range: 25% to 75% with an adjustment of 5%
    uint256 public stakingRewardsPercent = 50;

	// The percent of SALT Rewards that will be sent to the SALT/USDS pool.
	// This is done as SALT/USDS while an important pair for the exchange isn't involved in any arbitrage swap cycles (which would yield arbitrage profit for it as well as all pools which take part in arbitrage take a share of the profit).
	// This is because it isn't part of the usual token/WBTC and token/WETH structure - which allows other pools to be part of arbitrage swap cycles when other pools are traded.
	// Range: 5% to 25% with an adjustment of 5%
    uint256 public percentRewardsSaltUSDS = 10;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L19-L33

```solidity
File:  src/stable/StableConfig.sol
    uint256 public rewardPercentForCallingLiquidation = 5;

	// The maximum reward value (in USD) that a user receives for instigating the liquidation process.
	// Range: 100 to 1000 with an adjustment of 100 ether
    uint256 public maxRewardValueForCallingLiquidation = 500 ether;

	// The minimum USD value of collateral - to borrow an initial amount of USDS.
	// This is to help prevent saturation of the contract with small amounts of positions and to ensure that liquidating the position yields non-trivial rewards
	// Range: 1000 to 5000 with an adjustment of 500 ether
    uint256 public minimumCollateralValueForBorrowing = 2500 ether;

    // the initial maximum collateral / borrowed USDS ratio.
    // Defaults to 2.0x so that staking $1000 worth of BTC/ETH LP would allow you to borrow $500 of USDS
    // Range: 150 to 300 with an adjustment of 25
    uint256 public initialCollateralRatioPercent = 200;

	// The minimum ratio of collateral / borrowed USDS below which positions can be liquidated
	// and the user losing their collateral (and keeping the borrowed USDS)
	// Range: 110 to 120 with an adjustment of 1
	uint256 public minimumCollateralRatioPercent = 110;

	// The percent of arbitrage profits that are used to form USDS/DAI Protocol Owned Liquidity.
	// The liquidity generates yield for the DAO and can be liquidated in the event of undercollateralized liquidations.
	// Range: 1 to 10 with an adjustment of 1
	uint256 public percentArbitrageProfitsForStablePOL = 5;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L20-L44

```solidity
File:  src/staking/Staking.sol
30  uint256 public nextUnstakeID;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L30

```solidity
File:  src/staking/StakingConfig.sol
	uint256 public minUnstakeWeeks = 2;  // minUnstakePercent returned for unstaking this number of weeks

	// The maximum number of weeks for an unstake request at which point 100% of the original staked SALT is reclaimable.
	// Range: 20 to 108 with an adjustment of 8
	uint256 public maxUnstakeWeeks = 52;

	// The percentage of the original staked SALT that is reclaimable when unstaking the minimum number of weeks.
	// Range: 10 to 50 with an adjustment of 5
	uint256 public minUnstakePercent = 20;

	// Minimum time between increasing and decreasing user share in SharedRewards contracts.
	// Prevents reward hunting where users could frontrun reward distributions and then immediately withdraw.
	// Range: 15 minutes to 6 hours with an adjustment of 15 minutes
	uint256 public modificationCooldown = 1 hours;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L18-L31

## [G-06] State variables can be packed to use fewer storage slots

**Explanation of Issue:**
Timestamps and block numbers in storage do not necessarily need to be of type `uint256`. Utilizing a smaller data type can significantly optimize gas usage without sacrificing functionality. For instance, a timestamp represented by `uint48` can effectively cover millions of years into the future, while a block number only increments approximately once every 12 seconds. Understanding the magnitude of these numbers provides insight into selecting more efficient data types.

**Examples of Code:**

- _Non-optimized Code:_

  ```solidity
  contract Example {
      uint256 lastBlockNumber;
      uint256 lastTimestamp;

      function updateData() public {
          lastBlockNumber = block.number;
          lastTimestamp = block.timestamp;
      }
  }
  ```

- _Optimized Code:_

  ```solidity
  contract OptimizedExample {
      uint48 lastBlockNumber;
      uint48 lastTimestamp;

      function updateData() public {
          lastBlockNumber = uint48(block.number);
          lastTimestamp = uint48(block.timestamp);
      }
  }
  ```

**Reason:**  
Using smaller data types such as `uint48` for timestamps and block numbers drastically reduces storage consumption and gas costs. This optimization is feasible due to the vast ranges covered by these smaller types, which are more than sufficient for the foreseeable future.

There are 10 instances of this issue:

```solidity
File:  src/ManagedWallet.sol
28   uint256 public activeTimelock;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L28

```solidity
File:   src/Upkeep.sol
63	uint256 public lastUpkeepTimeEmissions;
64	uint256 public lastUpkeepTimeRewardsEmitters;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L63-L64

```solidity
File:  src/dao/DAOConfig.sol
40   uint256 public baseBallotQuorumPercentTimes1000 = 10 * 1000;

45   uint256 public ballotMinimumDuration = 10 days;

49   uint256 public requiredProposalPercentStakeTimes1000 = 500;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L40

```solidity
File:  src/rewards/RewardsConfig.sol
19   uint256 public rewardsEmitterDailyPercentTimes1000 = 1000;

23   uint256 public emissionsWeeklyPercentTimes1000 = 500;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L19

```solidity
File:  src/staking/StakingConfig.sol
18   uint256 public minUnstakeWeeks = 2;

22   uint256 public maxUnstakeWeeks = 52;

26   uint256 public minUnstakePercent = 20;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L18

## [G-07] Gas Optimization Issue - Use Constants Instead of type(uintx).max

**Explanation of Issue:**
Utilizing constants instead of `type(uintx).max` is essential for gas optimization in Ethereum smart contracts. Constants, being precomputed at compile-time, offer a more efficient approach compared to the runtime computations required by `type(uintx).max`, resulting in significant gas savings.

**Examples of Code:**

- **Non-Optimized:**

```solidity
function transferFundsNonOptimized(address _to, uint256 _amount) public {
    require(balanceOf(msg.sender) >= _amount, "Insufficient balance");
    require(_amount < type(uint256).max, "Amount exceeds maximum value");

    // Transfer logic
    // ...
}
```

- **Optimized:**

```solidity
uint256 constant MAX_UINT256 = type(uint256).max;

function transferFundsOptimized(address _to, uint256 _amount) public {
    require(balanceOf(msg.sender) >= _amount, "Insufficient balance");
    require(_amount < MAX_UINT256, "Amount exceeds maximum value");

    // Transfer logic
    // ...
}
```

**Reason:**
In the non-optimized example, using `type(uint256).max` within the function introduces unnecessary runtime computations, leading to higher gas consumption. The optimized version, using a constant (`MAX_UINT256`), ensures that the maximum value is precomputed at compile-time, resulting in a more gas-efficient smart contract. This optimization becomes crucial for contracts that involve frequent arithmetic operations with the maximum possible value.

There are 15 instances of this issue:

```solidity
File:  src/ManagedWallet.sol
37   activeTimelock = type(uint256).max;

68   activeTimelock = type(uint256).max; // effectively never

86   activeTimelock = type(uint256).max;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L37

```solidity
File:  src/Upkeep.sol
91  weth.approve( address(pools), type(uint256).max );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L91

```solidity
File:  src/dao/DAO.sol
90		salt.approve(address(collateralAndLiquidity), type(uint256).max);
91		usds.approve(address(collateralAndLiquidity), type(uint256).max);
92		dai.approve(address(collateralAndLiquidity), type(uint256).max);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L90-L92

```solidity
File:  src/dao/Proposals.sol
165   require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" ); // 5 quadrillion max supply with 18 decimals of precision
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L165

```solidity
File:  src/pools/PoolStats.sol
14   uint64 constant INVALID_POOL_ID = type(uint64).max;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L14

```solidity
File:  src/rewards/RewardsEmitter.sol
50  salt.approve(address(stakingRewards), type(uint256).max);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L50

```solidity
File:  src/rewards/SaltRewards.sol
41		salt.approve( address(stakingRewardsEmitter), type(uint256).max );
42		salt.approve( address(liquidityRewardsEmitter), type(uint256).max );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L41-L42

```solidity
File:  src/stable/Liquidizer.sol
71		wbtc.approve( address(pools), type(uint256).max );
72		weth.approve( address(pools), type(uint256).max );
73		dai.approve( address(pools), type(uint256).max );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L71-L73

## [G‑08] Gas Optimization: Inlining Internal Functions

**Explanation of Issue:**
When internal functions are only called once within a contract, they can be inlined to save gas. Failure to do so can result in additional gas costs, typically ranging from 20 to 40 gas, due to the overhead of extra JUMP instructions and additional stack operations incurred by function calls.

**Examples of Code:**

- _Non-optimized Code:_

  ```solidity
  contract Example {
      uint256 public value;

      function setValue(uint256 _newValue) internal {
          value = _newValue;
      }

      function updateValue() public {
          setValue(100); // Only called once
      }
  }
  ```

- _Optimized Code:_

  ```solidity
  contract Example {
      uint256 public value;

      function updateValue() public {
          value = 100; // Inlined the function call
      }
  }
  ```

**Reason:**  
Inlining internal functions that are only called once eliminates the overhead associated with function calls, reducing gas costs and improving the efficiency of the smart contract.

There are 8 instances of this issue:

```solidity
File:  src/AccessManager.sol
51   function _verifyAccess(address wallet, bytes memory signature ) internal view returns (bool)
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L51

```solidity
File:  src/arbitrage/ArbitrageSearch.sol
63  function _rightMoreProfitable( uint256 midpoint, uint256 reservesA0, uint256 reservesA1, uint256 reservesB0, uint256 reservesB1, uint256 reservesC0, uint256 reservesC1 ) internal pure returns (bool rightMoreProfitable)
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L63

```solidity
File:  src/dao/DAO.sol
113  function _finalizeParameterBallot( uint256 ballotID ) internal

131  function _executeSetContract( Ballot memory ballot ) internal

148  function _executeSetWebsiteURL( Ballot memory ballot ) internal

155  function _executeApproval( Ballot memory ballot ) internal

219  function _finalizeApprovalBallot( uint256 ballotID ) internal

235  function _finalizeTokenWhitelisting( uint256 ballotID ) internal
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L113

## [G-09] Use assembly to perform efficient back-to-back calls

**Explanation of Issue:** If a similar external call is performed back-to-back, using assembly can optimize gas consumption by reusing function signatures, function parameters, and memory space. This optimization can potentially avoid memory expansion costs.

**Examples of Code:**
For example, assuming Contract A has two functions B and C, and inside Contract D, you make external calls to B and C:

- _Non-optimized Code:_

```solidity
contract A {
    function B() external {
        // Function B implementation
    }

    function C() external {
        // Function C implementation
    }
}

contract D {
    A public contractA;

    function callFunctions() external {
        contractA.B();
        contractA.C();
    }
}
```

- _Optimized Code:_

```solidity
contract A {
    function B() external {
        // Function B implementation
    }

    function C() external {
        // Function C implementation
    }
}

contract D {
    A public contractA;

    function callFunctions() external {
        assembly {
            let successB := call(gas(), contractA, 0, 0, 0, 0, 0)
            let successC := call(gas(), contractA, 0, 0, 0, 0, 0)

            // Handle success or failure of the calls
        }
    }
}
```

**Reason:**

The optimization involves using assembly to perform efficient back-to-back calls. Here's the reasoning behind this optimization:

1. **Gas Cost:** By using assembly to make back-to-back calls, we can reuse function signatures and function parameters that stay the same. This eliminates the need to recreate the function call stack for each call, resulting in gas savings.

2. **Memory Optimization:** With assembly, we can reuse the same memory space for each function call, including scratch space, the free memory pointer, and the zero slot. This enables us to avoid memory expansion costs, further optimizing gas consumption.

As a smart contract Solidity auditor and gas optimizer, it is important to ensure that the optimized code maintains its intended functionality and adheres to best practices. Care should be taken to handle success or failure of the calls appropriately within the assembly block. Gas optimization should be done carefully, considering the trade-offs between gas cost and code efficiency.

There are 7 instances of this issue:

```solidity
File:  src/dao/DAO.sol
160			poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.wbtc() );
161			poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.weth() );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L160-L161

```solidity
File:  src/dao/DAO.sol
254			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
255			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );


350		salt.safeTransfer( address(salt), saltToBurn );
351		salt.burnTokensInContract();
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L254-L255

```solidity
File:  src/launch/Airdrop.sol
81		staking.stakeSALT( saltAmountForEachUser );
82		staking.transferStakedSaltFromAirdropToUser( msg.sender, saltAmountForEachUser );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L81-L82

```solidity
File:  src/launch/BootstrapBallot.sol
76			exchangeConfig.initialDistribution().distributionApproved();
77			exchangeConfig.dao().pools().startExchangeApproved();
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L76-L77

```solidity
File:  src/launch/InitialDistribution.sol
		salt.safeTransfer( address(emissions), 52 * MILLION_ETHER );

	    // 25 million		DAO Reserve Vesting Wallet
		salt.safeTransfer( address(daoVestingWallet), 25 * MILLION_ETHER );

	    // 10 million		Initial Development Team Vesting Wallet
		salt.safeTransfer( address(teamVestingWallet), 10 * MILLION_ETHER );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L56-L62

```solidity
File:  src/Upkeep.sol
209		saltRewards.stakingRewardsEmitter().performUpkeep(timeSinceLastUpkeep);
210		saltRewards.liquidityRewardsEmitter().performUpkeep(timeSinceLastUpkeep);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L209-L210

## [G-10] Gas Optimization: Using Modifiers Instead of Functions

**Explanation of Issue:**
Using modifiers instead of functions can save gas in certain scenarios. Modifiers provide a way to encapsulate common validation logic and apply it to multiple functions within a contract, reducing gas costs compared to separate function calls.

**Examples of Code:**

- _Non-optimized Code:_

  ```solidity
  // SPDX-License-Identifier: MIT
  pragma solidity 0.8.9;

  contract Inlined {
      function isNotExpired(bool _true) internal view {
          require(_true == true, "Exchange: EXPIRED");
      }

      function foo(bool _test) public returns(uint) {
          isNotExpired(_test);
          return 1;
      }
  }
  ```

- _Optimized Code:_

  ```solidity
  // SPDX-License-Identifier: MIT
  pragma solidity 0.8.9;

  contract Modifier {
      modifier isNotExpired(bool _true) {
          require(_true == true, "Exchange: EXPIRED");
          _;
      }

      function foo(bool _test) public isNotExpired(_test) returns(uint) {
          return 1;
      }
  }
  ```

**Reason:**  
Using modifiers reduces gas costs by encapsulating common validation logic and applying it directly to functions, eliminating the need for separate function calls. In the provided example, deploying the contract with a modifier is cheaper and saves around 30 gas compared to deploying a contract with separate validation functions. However, it's important to consider that modifiers may increase the code size of the contract, which could have implications depending on the specific use case and contract requirements.

There are 2 instances of this issue:

```solidity
File:  src/launch/Airdrop.sol
90   function isAuthorized(address wallet) public view returns (bool)
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L90

```solidity
File:  src/pools/PoolsConfig.sol
119  function isWhitelisted( bytes32 poolID ) public view returns (bool)
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L119

## [G-11] Using delete instead of setting info struct to 0 saves gas

There are 2 instances of this issue:

```solidity
File:  src/staking/StakingRewards.sol
301  cooldowns[i] = 0;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L301

```solidity
File:  src/price_feed/CoreUniswapFeed.sol
54  secondsAgo[1] = 0; // to (now)
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L54

## [G-12] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

There are 1 instances of this issue:
`stableConfig.minimumCollateralRatioPercent()` is external function call in side loop.

```solidity
File:  src/stable/CollateralAndLiquidity.sol
326  uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L326

## [G-13] Access mappings directly rather than using accessor functions

**Explanation of Issue:**
The issue here is that accessor functions are being used to access mappings in the code, which can result in unnecessary gas costs. Accessing mappings directly can help save gas by avoiding the need for additional JUMP instructions and stack setup.

**Examples of Code:**

- _Non-optimized Code:_

```solidity
mapping(uint256 => address) public myMapping;

// Access the mapping indirectly using an accessor function
function getValue(uint256 key) external view returns(address) {
    return getMappingValue(key);
}

function getMappingValue(uint256 key) internal view returns(address) {
    return myMapping[key];
}
```

- _Optimized Code:_

```solidity
mapping(uint256 => address) public myMapping;

// Access the mapping directly
function getValue(uint256 key) external view returns(address) {
    return myMapping[key];
}
```

**Reason:**
In the non-optimized code, an accessor function `getMappingValue` is used to retrieve the value from the mapping `myMapping`. However, this introduces unnecessary gas costs because it requires an additional function call, resulting in extra JUMP instructions and stack setup. By accessing the mapping directly in the `getValue` function, we eliminate the need for the accessor function and reduce gas consumption, leading to improved gas optimization in the smart contract.

There are 2 instances of this issue:

```solidity
File:  src/dao/DAO.sol
386  return excludedCountries[country];
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L386

```solidity
File:  src/pools/PoolStats.sol
145   return _arbitrageIndicies[poolID];
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L145

## [G-14] Pre-increment and pre-decrement are cheaper than `+1`, `-1`

**Explanation of Issue:**
When incrementing or decrementing a variable in Solidity, using pre-increment (`++`) and pre-decrement (`--`) operators is more gas-efficient than using the `+1` and `-1` operations. Pre-increment and pre-decrement do not require an additional arithmetic operation, resulting in lower gas costs compared to the alternative.

**Examples of Code:**

- _Non-optimized Code:_

```solidity
contract Counter {
    uint256 public counter;

    function incrementCounter() public {
        counter = counter + 1;
    }

    function decrementCounter() public {
        counter = counter - 1;
    }
}
```

- _Optimized Code:_

```solidity
contract Counter {
    uint256 public counter;

    function incrementCounter() public {
        ++counter;
    }

    function decrementCounter() public {
        --counter;
    }
}
```

**Reason:**
Using pre-increment (`++`) and pre-decrement (`--`) operators instead of `+1` and `-1` operations offers gas savings. Pre-increment and pre-decrement do not require an additional arithmetic operation to increment or decrement a variable, resulting in lower gas costs. In Solidity, gas costs are associated with computational operations, and by minimizing the number of operations, we can achieve better gas optimization. In the provided example, the non-optimized code uses `+1` and `-1` operations to increment and decrement the `counter` variable, while the optimized code utilizes pre-increment and pre-decrement operators, resulting in improved gas efficiency. As a smart contract auditor and gas optimizer in Solidity, it is important to use pre-increment and pre-decrement operators when appropriate to achieve optimal gas optimization.

There are 9 instances of this issue:

```solidity
File:  src/stable/CollateralAndLiquidity.sol
350  return findLiquidatableUsers( 0, numberOfUsersWithBorrowedUSDS() - 1 );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L350

```solidity
File:  src/stable/StableConfig.sol
52  uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - rewardPercentForCallingLiquidation - 1;

60   rewardPercentForCallingLiquidation -= 1;

131  minimumCollateralRatioPercent -= 1;

148  percentArbitrageProfitsForStablePOL -= 1;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L52

```solidity
File:  src/staking/Staking.sol
179  return unstakesForUser( user, 0, unstakeIDs.length - 1 );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L179

```solidity
File:  src/dao/DAOConfig.sol
151  maxPendingTokensForWhitelisting -= 1;

193  upkeepRewardPercent -= 1;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L151

```solidity
File:  src/staking/StakingConfig.sol
44   minUnstakeWeeks -= 1;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L44

## [G‑15] Using constants instead of enum can save gas

Enum is expensive and it is [more efficient to use constants](https://www.codehawks.com/finding/clm84992q02j9w9ruebun36d9) instead. An illustrative example of this approach can be found in the [ReentrancyGuard.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/181d518609a9f006fcb97af63e6952e603cf100e/contracts/utils/ReentrancyGuard.sol#L34-L35).

There are 1 instances of this issue:

```solidity
File:  src/dao/Parameters.sol
14  enum ParameterTypes {
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol#L14

## [G-16] Consider using OZ EnumerateSet in place of nested mappings

Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).
A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.
OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

There are 5 instance(s) of this issue:

```solidity
File:  src/AccessManager.sol
26  mapping(uint256 => mapping(address => bool)) private _walletsWithAccess;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L26

```solidity
File:  src/dao/Proposals.sol
51  mapping(uint256=>mapping(Vote=>uint256)) private _votesCastForBallot;

55  mapping(uint256=>mapping(address=>UserVote)) private _lastUserVoteForBallot;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L51

```solidity
File:  src/pools/Pools.sol
49  mapping(address=>mapping(IERC20=>uint256)) private _userDeposits;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L49

```solidity
File:  src/staking/StakingRewards.sol
36  mapping(address=>mapping(bytes32=>UserShareInfo)) private _userShareInfo;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L36

## [G-17] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 1 instances of this issue:

```solidity
File: src/stable/CollateralAndLiquidity.sol
338  for ( uint256 i = 0; i < count; i++ )
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L338

## [G-18] Gas Efficiency through Hardcoded Address Usage

note : these are insteance missid from bots.
**Explanation of Issue:**
Using `address(this)` in your contract can lead to additional gas costs, as it requires an EXTCODESIZE operation to retrieve the contract's address from its bytecode. This operation, when used frequently, can contribute to increased gas consumption. To enhance gas efficiency, it is recommended to replace instances of `address(this)` with a hardcoded address, especially when the same address is used multiple times in the contract.

**Examples of Code:**

- _Non-optimized Code:_

  ```solidity
  contract MyContract {
      function doSomething() public {
          // Using address(this) to retrieve the contract's address
          require(msg.sender == address(this), "Caller is not authorized");

          // Do something
      }
  }
  ```

- _Optimized Code:_

  ```solidity
  contract MyContract {
      address public immutable myAddress = 0x1234567890123456789012345678901234567890;

      function doSomething() public {
          // Using the pre-calculated and hardcoded address
          require(msg.sender == myAddress, "Caller is not authorized");

          // Do something
      }
  }
  ```

**Reason:**  
The optimization involves replacing `address(this)` with a pre-calculated and hardcoded address (e.g., `myAddress`). This adjustment eliminates the need for the EXTCODESIZE operation, reducing the gas cost associated with retrieving the contract's address from its bytecode. By employing this optimization technique, gas efficiency is improved, especially when the same address is referenced multiple times in the contract. This practice enhances the overall performance and cost-effectiveness of the smart contract.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

There are 5 instances of this issue:

```solidity
File:  src/dao/DAO.sol
170  if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )

246  uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );

300  uint256 depositedWETH = pools.depositedUserBalance(address(this), weth );

307  withdrawnAmount = weth.balanceOf( address(this) );

365  uint256 liquidityHeld = collateralAndLiquidity.userShareForPool( address(this), poolID );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L170
