### Vulnerabilities and Concerns in the Staking System

#### Identified Concern Areas

1. **Virtual Rewards Calculation**:
   - The mechanism for adjusting users' rewards through "virtual rewards" in the `StakingRewards` contract is complex and prone to rounding errors or manipulation. Specifically, the way virtual rewards are added and subtracted during share increases and decreases could lead to imbalances or exploitation opportunities.
   - **Location**: `_increaseUserShare` and `_decreaseUserShare` functions in `StakingRewards`.
   - **Concern**: Precision loss or manipulation in the calculation of virtual rewards could unfairly benefit or penalize users.

2. **Unstaking Flexibility**:
   - The `Staking` contract allows for variable unstaking durations, impacting the amount of claimable SALT. While innovative, this flexibility introduces complexity in the calculation of claimable amounts and potential gaming of the system.
   - **Location**: `unstake` function in `Staking`.
   - **Concern**: Users might find ways to exploit the unstaking terms to their advantage, especially if there are edge cases in the calculation logic.

3. **Reward Distribution Adjustments**:
   - In `Liquidity` and `StakingRewards` contracts, the distribution of rewards upon liquidity provision, withdrawal, or staking could be manipulated by timing the transactions around reward additions.
   - **Location**: Reward calculation logic upon user share changes.
   - **Concern**: Potential for front-running or transaction timing manipulation to maximize personal gains from reward distributions.

4. **Cooldown Mechanisms**:
   - The cooldown periods implemented to prevent rapid staking/unstaking might not be sufficient to deter manipulation, especially if users can find ways to bypass these restrictions.
   - **Location**: Various functions across `Staking` and `StakingRewards` that check for cooldown expirations.
   - **Concern**: Ineffective cooldowns could lead to "reward hunting" behaviors, destabilizing the reward system.

5. **External Contract Dependencies**:
   - The contracts rely on external configurations and pool contracts, which could introduce risks if those external contracts contain vulnerabilities or behave unexpectedly.
   - **Location**: External calls and configurations in all three contracts.
   - **Concern**: Vulnerabilities in external contracts or configurations could compromise the staking system's integrity.

#### Planned Attack to Secure the Staking System

1. **Deep Dive into Virtual Rewards Logic**:
   - Perform a line-by-line review of the virtual rewards calculation and adjustment logic to identify any potential for precision loss, rounding errors, or manipulation.
   - Specifically investigate how virtual rewards are calculated in proportion to real rewards and shares, and ensure that edge cases are handled properly.

2. **Stress Test Unstaking Calculations**:
   - Simulate a wide range of unstaking scenarios with varying amounts and durations to identify any inconsistencies or exploitable patterns in the calculation of claimable SALT.
   - Pay special attention to boundary conditions, such as the minimum and maximum unstake durations.

3. **Analyze Reward Distribution Events**:
   - Review the timing and triggers for reward distributions, especially in response to user actions like staking, unstaking, or liquidity provision. Look for vulnerabilities that could allow users to manipulate the timing of their transactions to gain an unfair advantage.
   - Consider the impact of blockchain-specific factors like block times and transaction ordering.

4. **Evaluate Cooldown Effectiveness**:
   - Assess the implementation and enforcement of cooldown periods to ensure they effectively prevent rapid, exploitative behaviors. Look for loopholes that might allow users to bypass cooldown restrictions.
   - Consider introducing dynamic cooldown periods based on system activity or user behavior patterns.

5. **Audit External Contract Interactions**:
   - Review all interactions with external contracts and configurations for potential vulnerabilities. This includes not only direct interactions but also dependencies on external states or configurations that could affect the staking system.
   - Verify the integrity and security of contracts providing external configurations (like `IExchangeConfig`, `IPoolsConfig`, `IStakingConfig`) and ensure there are fallbacks or safeguards against unexpected changes.

### Summary for the Audit

- **Virtual Rewards**: Focus on the precision and fairness of the virtual rewards system, ensuring that it cannot be gamed or exploited due to rounding or calculation errors.
- **Unstaking Logic**: Thoroughly test the unstaking mechanism for potential exploits, especially around the flexibility of unstaking durations.
- **Reward Distribution**: Investigate the timing and conditions for reward distribution, looking for potential vulnerabilities related to transaction ordering or front-running.
- **Cooldown Mechanisms**: Assess the implementation and effectiveness of cooldown periods, ensuring they serve their intended purpose without introducing additional vulnerabilities.
- **External Dependencies**: Ensure that the system's dependencies on external contracts do not introduce risks, and that there are measures in place to handle unexpected changes in those contracts.

By addressing these areas, the auditing team can help secure the staking system against potential vulnerabilities and ensure its robustness and fairness.

### Step 1: Deep Dive into Virtual Rewards Logic

The virtual rewards mechanism is crucial for maintaining the fairness of the reward distribution as users increase or decrease their stakes. This system ensures that users who join or leave the pool do not unfairly dilute or benefit from the rewards distribution. Here's a detailed breakdown of what the auditing team should focus on:

#### Understanding Virtual Rewards

- **Purpose**: Virtual rewards are used to adjust a user's share of the total rewards in a way that accounts for the timing of their share changes. This prevents users who increase their shares just before a reward distribution from unfairly benefiting at the expense of longer-term participants.

#### Key Functions to Audit

1. **`_increaseUserShare` in `StakingRewards`**:
   - This function increases a user's share and calculates the virtual rewards to be added to their account to maintain the fairness of reward distribution.
   - **Audit Focus**:
     - **Calculation of Virtual Rewards**: Ensure the formula `virtualRewardsToAdd = Math.ceilDiv(totalRewards[poolID] * increaseShareAmount, existingTotalShares)` accurately represents the intended distribution of rewards. Pay attention to the use of `Math.ceilDiv` for rounding up, which could introduce bias in favor of the protocol.
     - **Edge Cases**: Consider scenarios where `existingTotalShares` is very small, which could lead to disproportionately large virtual rewards due to the division.

2. **`_decreaseUserShare` in `StakingRewards`**:
   - This function decreases a user's share and calculates the amount of rewards the user is entitled to, taking into account the virtual rewards.
   - **Audit Focus**:
     - **Proportional Reduction of Virtual Rewards**: The logic for reducing virtual rewards `virtualRewardsToRemove = (user.virtualRewards * decreaseShareAmount) / user.userShare` must be scrutinized to ensure it's proportional and fair.
     - **Handling Zero Shares**: Ensure the function handles cases where a user's share might be reduced to zero correctly, especially in terms of how virtual rewards are adjusted.

#### Potential Vulnerabilities and Checks

- **Rounding Errors**: Given the division and multiplication involved in calculating virtual rewards, there is potential for rounding errors. These errors could accumulate over many transactions, leading to discrepancies in reward distribution.
  - **Check**: Implement tests that simulate many small increases and decreases in shares to observe if rounding errors accumulate in a way that could be exploited.

- **Manipulation Through Rapid Share Changes**: A user might try to manipulate their virtual rewards by rapidly increasing and then decreasing their shares around the time rewards are distributed.
  - **Check**: Analyze the impact of rapid share changes on virtual rewards and consider implementing additional checks or cooldowns to prevent such manipulation.

- **Precision Loss**: Solidity's lack of floating-point arithmetic means that division operations can result in precision loss. This is especially critical in the context of calculating virtual rewards.
  - **Check**: Ensure that the precision loss does not favor either the protocol or the users consistently and that any such loss is minimal and within acceptable bounds.

#### Testing Strategies

- **Fuzz Testing**: Implement fuzz testing to provide random inputs to the share increase and decrease functions, observing the system's behavior under a wide range of conditions.
- **Edge Case Testing**: Specifically test edge cases, such as very small or very large share increases/decreases, and scenarios where the total shares or total rewards are low.
- **Historical Simulation**: Use historical data to simulate the reward and share changes over time, ensuring the virtual rewards system behaves as expected even under unusual market conditions.

By meticulously auditing the virtual rewards logic with a focus on these areas, the team can ensure the system is robust, fair, and resistant to manipulation.

### Step 2: Stress Test Unstaking Calculations

The unstaking mechanism within the `Staking` contract, especially its flexibility regarding unstaking duration and the consequent impact on the claimable SALT amount, introduces a layer of complexity that needs thorough auditing. Here's a structured approach for the auditing team to stress test and validate the unstaking calculations:

#### Understanding the Unstaking Mechanism

- **Mechanism Overview**: Users can choose to unstake their xSALT over a variable duration, affecting the amount of claimable SALT. Shorter unstaking periods result in receiving less than the full staked amount, introducing a trade-off between liquidity and reward.

#### Key Functions to Audit

1. **`unstake` in `Staking`**:
   - Initiates the unstaking process, setting the amount of xSALT to unstake and the duration over which the unstake will occur.
   - **Audit Focus**:
     - **Calculation of Claimable SALT**: The formula used to determine the claimable SALT based on the unstaked amount and the duration needs careful examination. Ensure that `calculateUnstake` accurately reflects the intended policy for partial unstaking.
     - **Duration Validation**: Verify that the unstaking duration (`numWeeks`) adheres to the configured minimum and maximum bounds, and assess how these bounds impact the claimable amount.

2. **`calculateUnstake` in `Staking`**:
   - Calculates the amount of SALT claimable at the end of the unstaking period based on the unstaked xSALT and the duration.
   - **Audit Focus**:
     - **Proportional Claim Calculation**: Ensure the logic for determining the proportion of the claimable SALT relative to the unstaked amount over the chosen duration is correct and free from vulnerabilities.
     - **Handling of Boundary Conditions**: Pay special attention to the handling of minimum and maximum unstaking durations, especially their impact on the claimable SALT amount.

#### Potential Vulnerabilities and Checks

- **Inconsistent Claim Amounts**: Variability in unstaking durations could lead to scenarios where users receive inconsistent claim amounts due to rounding errors or logical flaws in the calculation.
  - **Check**: Conduct tests with varying unstaking amounts and durations to identify any inconsistencies or anomalies in the claimable SALT amounts.

- **Duration Manipulation**: Users might attempt to manipulate the unstaking duration to maximize their claimable SALT, especially near the boundary conditions.
  - **Check**: Evaluate the system's response to unstaking requests made at the edges of the minimum and maximum duration limits to ensure it behaves as expected without allowing for exploitation.

- **Unstaking Under Extreme Conditions**: Consider how the system handles unstaking during periods of high volatility or when the protocol is under stress (e.g., rapid price movements, high congestion).
  - **Check**: Simulate unstaking under extreme market conditions to assess if there are scenarios where the unstaking mechanism could be gamed or would fail to operate as intended.

#### Testing Strategies

- **Scenario Analysis**: Develop a comprehensive set of unstaking scenarios that cover a wide range of unstaking amounts and durations, including edge cases at the minimum and maximum duration limits.
- **Automated Testing**: Implement automated tests that simulate a large number of unstaking operations under various conditions to identify any potential issues in the calculation logic or its implementation.
- **Contract Interactions**: Test how the unstaking mechanism interacts with other contract functions, particularly those related to staking, reward distribution, and share adjustments, to ensure there are no unintended side effects.

By focusing on these aspects, the auditing team can ensure the unstaking mechanism is robust, equitable, and aligns with the protocol's intended liquidity and reward policies, thereby securing the system against potential vulnerabilities related to unstaking calculations.

### Step 3: Analyze Reward Distribution Events

The timing and conditions under which rewards are distributed, especially in response to user actions like staking, unstaking, or liquidity provision and withdrawal, are critical aspects that require thorough auditing to prevent manipulation and ensure fairness. Here's a detailed approach for the auditing team to analyze and secure the reward distribution events:

#### Understanding Reward Distribution Dynamics

- **Dynamics Overview**: The reward distribution mechanism must ensure that rewards are fairly allocated based on the proportion of a user's stake or liquidity share in the pool. Any changes in stake or liquidity shares should be accurately reflected in the reward allocation without opening avenues for manipulation.

#### Key Functions and Areas to Audit

1. **Reward Addition (`addSALTRewards` in `StakingRewards`)**:
   - Adds rewards to the pool, which are then distributed to users based on their share.
   - **Audit Focus**:
     - **Fairness in Reward Allocation**: Ensure that the addition of rewards does not disproportionately benefit users who recently changed their stake or liquidity share.
     - **Timing Vulnerabilities**: Investigate if users could manipulate the timing of their stake or liquidity share changes to unfairly increase their reward allocation.

2. **User Share Adjustments (`_increaseUserShare` and `_decreaseUserShare` in `StakingRewards`)**:
   - Adjusts a user's share in response to staking, unstaking, or liquidity provision/withdrawal.
   - **Audit Focus**:
     - **Impact on Reward Distribution**: Analyze how increases or decreases in user shares immediately before or after reward additions affect the reward distribution.
     - **Virtual Rewards Adjustment**: Examine the adjustments to virtual rewards during share changes for potential manipulation points.

#### Potential Vulnerabilities and Checks

- **Front-Running Reward Additions**: There's a potential risk that users could front-run the addition of rewards by rapidly increasing their stake or liquidity share just before rewards are added.
  - **Check**: Assess the system's resilience to front-running by simulating transactions that rapidly increase user shares immediately before reward additions.

- **Transaction Ordering Manipulation**: Users might attempt to exploit the transaction ordering within a block to maximize their reward allocation.
  - **Check**: Evaluate the contract's logic in the context of transaction ordering within a block, particularly how pending transactions might impact reward calculations.

- **Rapid Share Changes Around Reward Events**: Investigate scenarios where users frequently increase and decrease their shares around reward distribution events to game the system.
  - **Check**: Simulate scenarios with rapid share changes around the time of reward additions to identify potential vulnerabilities.

#### Testing Strategies

- **Historical Transaction Analysis**: Use historical data to identify patterns where users might have attempted to manipulate the reward distribution. This analysis could reveal potential vulnerabilities that need addressing.

- **Comprehensive Scenario Testing**: Develop tests that cover a wide range of user behaviors, including rapid share changes and strategic timing of transactions around reward events, to ensure the system can handle such cases without being exploited.

- **Smart Contract Simulations**: Utilize smart contract simulation tools to mimic the contract's behavior under various conditions, focusing on the interaction between share adjustments and reward distributions. This can help identify unforeseen issues in the contract logic.

- **Stress Testing**: Conduct stress tests that simulate high volumes of user interactions, particularly around the times rewards are added or when significant share adjustments occur, to evaluate the system's performance and fairness under stress.

By meticulously analyzing the reward distribution events and focusing on the identified areas, the auditing team can ensure that the reward mechanism is secure, fair, and resistant to manipulation, thereby safeguarding the integrity of the staking system.

### Step 4: Evaluate Cooldown Effectiveness

Cooldown mechanisms are implemented to prevent rapid, potentially exploitative behaviors like quick staking/unstaking or reward hunting. These mechanisms need to be effective to maintain system stability and fairness. Here’s how the auditing team can approach evaluating and ensuring the effectiveness of cooldown periods:

#### Understanding Cooldown Mechanisms

- **Purpose**: Cooldown periods are designed to introduce a mandatory wait time between certain actions, such as staking, unstaking, or claiming rewards, to deter users from exploiting the system for unfair advantage.

#### Key Functions and Areas to Audit

1. **Cooldown Implementation (`_increaseUserShare` and `_decreaseUserShare` in `StakingRewards` and relevant functions in `Staking`)**:
   - Implements cooldown periods after users stake, unstake, or make other significant changes to their positions.
   - **Audit Focus**:
     - **Consistency and Enforcement**: Ensure that cooldown periods are consistently enforced after relevant actions and cannot be bypassed through any contract functions.
     - **Cooldown Duration**: Assess whether the duration of cooldown periods is appropriate for their intended purpose and is effective in deterring rapid, exploitative actions.

2. **Cooldown Expiration Handling**:
   - Manages actions post-cooldown, allowing users to proceed with operations like unstaking or reward claims.
   - **Audit Focus**:
     - **Expiration Tracking**: Verify that cooldown expirations are accurately tracked and updated, especially in edge cases like multiple rapid transactions or blockchain reorgs.
     - **User Actions Post-Cooldown**: Ensure that once a cooldown expires, user actions are processed as intended without unintended side effects or vulnerabilities.

#### Potential Vulnerabilities and Checks

- **Cooldown Bypass Techniques**: Users might discover ways to bypass cooldown periods, for example, by interacting with the contract in unexpected ways or exploiting loopholes in the logic.
  - **Check**: Review all contract functions to identify any that might inadvertently allow users to bypass cooldown restrictions, including indirect interactions through other contracts.

- **Manipulation Around Cooldown Expirations**: There could be strategies to game the system by timing actions around the expiration of cooldown periods, particularly in relation to reward distribution events.
  - **Check**: Simulate user actions timed around cooldown expirations to identify potential exploitation scenarios, especially those that could lead to unfair reward claims.

#### Testing Strategies

- **Comprehensive Function Testing**: Test each contract function that should enforce a cooldown, ensuring that the cooldown is correctly started, enforced, and expired without exceptions.

- **Edge Case Analysis**: Specifically target edge cases in testing, such as actions taken milliseconds before or after a cooldown expires, or actions taken during blockchain reorganizations that might affect block timestamps.

- **Behavioral Simulation**: Simulate user behaviors that might be used to exploit cooldown mechanisms, including rapid sequences of actions and actions timed around cooldown expirations, to ensure the system remains robust against such strategies.

- **Contract Interaction Testing**: Consider the contract's interactions with other contracts and external calls that might impact cooldown logic, ensuring that these interactions do not introduce vulnerabilities.

By focusing on these areas, the auditing team can validate the effectiveness of cooldown mechanisms, ensuring they serve their intended purpose without introducing additional vulnerabilities or being susceptible to manipulation. This step is crucial for maintaining the integrity and fairness of the staking system.

### Step 5: Audit External Contract Interactions

The staking system's reliance on external contracts and configurations introduces a layer of dependency that could potentially bring vulnerabilities or unexpected behaviors. Ensuring the security and integrity of these interactions is crucial for the overall security of the staking system. Here's a structured approach for auditing external contract interactions:

#### Understanding External Dependencies

- **External Contracts**: These include contracts for token handling (`ISalt`, `SafeERC20`), exchange configurations (`IExchangeConfig`), pools configurations (`IPoolsConfig`), and staking configurations (`IStakingConfig`).
- **Purpose of Dependencies**: These external contracts provide essential functionalities, such as token transfers, access control, and parameter configurations, which are integral to the staking system's operations.

#### Key Areas and Functions to Audit

1. **Token Handling and Transfers (`SafeERC20` interactions in `Staking` and `StakingRewards`)**:
   - Ensures safe interaction with ERC20 tokens, mitigating common issues like reentrancy and transfer failures.
   - **Audit Focus**:
     - **Token Transfer Security**: Verify that token transfers, approvals, and other interactions are securely handled, particularly looking for reentrancy vulnerabilities and proper handling of transfer failures.
     - **Contract Trust Assumptions**: Assess any assumptions made about the behavior of the token contracts, such as compliance with the ERC20 standard and non-malicious behavior.

2. **Configuration and Access Control (`IExchangeConfig`, `IPoolsConfig`, `IStakingConfig`)**:
   - Provides parameters and access controls critical for the operation of the staking system.
   - **Audit Focus**:
     - **Configuration Integrity**: Ensure that the configuration values obtained from these contracts are validated and handled securely, protecting against invalid or malicious configurations.
     - **Access Control Enforcement**: Confirm that access controls, such as those determining who can stake or unstake, are robustly enforced and cannot be bypassed.

#### Potential Vulnerabilities and Checks

- **External Contract Manipulation**: If any external contract behaves unexpectedly or maliciously, it could compromise the staking system.
  - **Check**: Review the trust assumptions made about each external contract, considering scenarios where these contracts behave in unexpected ways, and implement safeguards where necessary.

- **Configuration Changes**: Unexpected changes in configurations provided by external contracts could disrupt the staking system's operations.
  - **Check**: Implement mechanisms to handle configuration changes gracefully, including possible failsafes or alert systems for significant changes.

#### Testing Strategies

- **Interface Mocking**: Create mock versions of external contracts to simulate various behaviors and responses, including non-standard behaviors and failure modes, to test the staking system's resilience.

- **Integration Testing**: Conduct integration tests with the actual external contracts (or their testnet versions) to ensure that the interactions work as expected in a real-world environment.

- **Parameter Variation**: Systematically vary the configurations and parameters provided by external contracts to test the staking system's response to a wide range of conditions.

- **Monitoring and Alerts**: Develop monitoring tools to track real-time interactions with external contracts and alert on unexpected behaviors or configuration changes that could indicate potential vulnerabilities.

By thoroughly auditing external contract interactions, the auditing team can identify and mitigate potential vulnerabilities introduced through these dependencies, ensuring the staking system's security and reliability. This step is vital for protecting the system against external risks and ensuring its robust operation.

### Time spent:
60 hours