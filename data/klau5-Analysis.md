# Overview

The Salty project is a k=x * y DEX that generates profit through arbitrage every time a user swaps. Thanks to this, users can swap at zero fee, and the profit from arbitrage is used for protocol operating such as liquidity provider rewards, and staking rewards. You can provide liquidity to the WETH/WBTC pool and use it as collateral to borrow USDS tokens. Liquidity providers and staking rewards are provided in SALT tokens, which are governance tokens, and by staking SALT tokens, you can get voting rights to participate in the DAO.

# Approach taken in evaluating the codebase

| Stage | Detail |
| --- | --- |
| Compile and run test | Clone the repository, install dependencies, set environment and run test. |
| Check scope and sort | List the scopes, sorting them in ascending order. Identify core and peripheral logic. |
| Read docs | Read code4rena README and docs. Research on who’s sponsors and what’s their goal. Review the sponsor's existing projects, if any. |
| Understand core logic | Skim through the code to understand the core logic. |
| Manuel Code Review | Read the code line by line to understand the logic. Find bugs due to mistakes. |
| Organize logic with writing/drawing | Organize business logic and look for logic bugs. |
| Write report, PoC | Write the report and make PoC by using test code. |

# Architecture of Salty

This diagram illustrates how the contracts interact within the Salty ecosystem.

![Architecture of Salty](https://gist.github.com/assets/70058709/5f7335c6-7b3c-409b-8383-22fe9153090b)

# Centralization risks

All operations, except for initial deployment settings, are determined through the DAO. The DAO can change most of the protocol parameters, and the parameters can only be increased/decreased/unchanged, not set to arbitrary numbers. There are also upper and lower limits for parameters, so there is a limit to manipulating parameters with a 51% attack.

Proposals to change important contract addresses or website addresses need confirm votes to enhance security. Proposals to transfer SALT tokens owned by the DAO are limited to a maximum of 5% of the balance to reduce potential attack damage. The total vote must reach the minimum quorum, and a differential is applied to the quorum depending on the importance of the proposal. It is well designed to minimize damage even in the event of a 51% attack.

# Mechanism review

## Swap and Arbitrage

It is a k=x * y DEX. The unique point is that arbitrage is performed every time a user makes a swap. If the swap is from token1 to token2, it attempts to arbitrage in the order of WETH → token2 → token1 → WETH. If there is no profit from arbitrage, it does not perform arbitrage and ends. The arbitrage profit is delivered to the DAO and used for protocol operation and rewards.

Except for the special pool registered at deployment, it can only be registered in the whitelist through voting, so only pools such as token1/WETH, token1/WBTC can be registered in the whitelist. There is no arbitrary token1/token2 pool other than SALT/USDS, DAI/USDS.

In the special case of swapping from WETH to WBTC or WBTC to WETH, it uses the SALT pool to arbitrage in the order of WETH → SALT→ WBTC → WETH or WETH → WBTC → SALT → WETH.

If arbitrage profit occurs, the profit from the pool where the swap was made is recorded. When distributing liquidity provider rewards, the pools can receive rewards in proportion to the profits generated in each pool.

## Providing Liquidity

When you provide liquidity to a pool, you can receive SALT tokens as a reward. Only users who have registered a signature can provide liquidity. If the user requests the zapping option and the ratio of the two tokens to be added is not the same as the pool, the tokens can be swapped to match the ratio.

The cooldown option is used to prevent sudden frequent changes by not being able to make changes again for a certain period of time after making changes to the liquidity.

## USDS Tokens and Collateral Tokens

You can borrow (mint) USDS with WETH and WBTC as collateral. The collateral tokens supply liquidity to the WETH/WBTC pool. Only users who have registered a signature can borrow USDS.

By default config, you must deposit at least twice the loan amount as collateral. Three price feeds are used to calculate the value of the collateral. The USD price of WETH, WBTC is obtained from three price feeds. Among them, the average price of the two feeds with the smallest difference is used as the token price. If the price difference between the two feeds is greater than n% of the calculated average price, it is determined that there is a problem with the price.

When borrowing USDS, it is minted, and when repaying the loan, it is burned. If the value of the collateral falls or the minimum required collateral ratio is changed, the collateral for the existing loan may be insufficient. At this time, anyone can liquidate the debt of a user with insufficient collateral.

The collateral for the liquidated debt is delivered to the Liquidizer, and part of the collateral is provided as a reward to the user who called the liquidation function. The Liquidizer swaps the received collateral tokens for USDS and burns them. If there is not enough collateral tokens, it receives 1% of the POL and uses it for liquidation.

## SALT Tokens

Initially mint 100 million and distribute it at a set ratio at launch. The SALT tokens are used for liquidity provider rewards, staking rewards, etc. By staking SALT tokens, you can participate in the DAO. You can get as many voting rights as the number of tokens you staked. Staked SALT tokens are called xSALT, and users who staked are called xSALT holders.

Tokens are burned according to some rules, causing deflation. The DAO deposits some of the profits from arbitrage into the SALT/USDS, USDS/DAI pool. This is called POL (Protocol Owned Liquidity), and the liquidity supply rewards (SALT) generated from the POL are burned at a certain rate. Also, when unstaking, if the period is set short and not all SALT is returned, the tokens that are not returned are burned.

| Receiver | Distribution Period | Amount |
| --- | --- | --- |
| Emissions | It is paid as a reward to liquidity providers and xSALT holders. It delivers about n% of the holdings as a reward per week. As the held tokens decrease, the amount of rewards decreases, but the value of the token is increased by token burn. | 52 million |
| DAO Reserve | Send to VestingWallet and distribute linearly over 10 years. Give tokens to the DAO contract. | 25 million |
| Initial Development Team | Send to VestingWallet and distribute linearly over 10 years. It is transfered to the development team's account. | 10 million |
| Airdrop Participants | At the time of launch, it airdrops to users who participated in the Bootstrap vote. The airdrop is delivered in the form of xSALT. | 5 million |
| Liquidity Bootstrapping Rewards | Initial rewards to be distributed to liquidity providers. After that, liquidity provider rewards are distributed from the Emissions contract and arbitrage profits. | 5 million |
| Staking Bootstrapping Rewards | Initial rewards to be distributed to xSALT holders. After that, xSALT holder rewards are distributed from the Emissions contract and arbitrage profits. | 3 million |
|  |  | 100 million total |

## Staking

Stake SALT tokens and receive SALT tokens as rewards. Only users who have registered a signature can stake.

When unstaking, user set the unstaking period. You can withdraw the staked SALT tokens after this period has passed. The shorter the period set, the less SALT you can get back. You can get back all the SALT you staked only if you set the unstaking period to the maximum. SALT that is not returned by setting the period short is burned.

If you cancel the unstaking request, you can re-stake the SALT tokens that were previously unstaked. If you want to unstake again, you need to request it again, and the previous unstaking period is reset and you have to wait again.

## DAO

The proposals that can be made to the DAO are as follows:

1. Change the protocol's parameters
2. Change important settings of the protocol
3. Add/Remove whitelist
4. Transfer SALT owned by the DAO
5. Call the `callFromDAO` function of any contract

Only users who own a certain percentage of xSALT can create new proposals. xSALT holders can vote as many votes as they own xSALT. The types of votes that can be cast vary depending on the type of proposal. For parameter change proposals, you must choose one of three options: increase/decrease/no change. For other votes, you have to choose one of two options: Yes/No.

After a certain minimum period has passed and received more than quorum votes, the vote can be finalized. The number of quorums required varies depending on the type of proposal. Basically, it is based on n% of the current total xSALT supply. Proposals to change the protocol parameters require n%, proposals to add to the whitelist require 2 * n%, and other general proposals require 3 * n% votes.

When you finalize the vote, if it is a proposal to set the protocol's parameters, one of increase/decrease/no change is executed, and for others, the proposal is executed if the approval vote is more than half.

### Changing Protocol Parameters

You can customize various parameters such as reward ratios and delays. Parameters cannot be set to specific values, but must be voted to increase/decrease/no change. When increasing or decreasing, only a certain amount can be increased or decreased at a time, and there are upper and lower limits for each parameter.

### Changing Important Protocol Settings

You can change important contracts such as the AccessManager contract or change the price feed. You can also change the URL of this protocol's website. If a proposal to change important settings is successful in voting, it does not change immediately, but opens another confirm vote. Only after the confirm vote is completed can these be changed. There is a cooldown for changing the price feed, so you can't change the feed often.

### Add/Remove Whitelist

Register a new whitelist token. Register this token's WETH, WBTC pool as a whitelist, and use SALT tokens owned by the DAO as initial liquidity provider rewards. If removed from the whitelist, this token's WETH, WBTC pools are removed from the whitelist. No more liquidity provider rewards will be additionally distributed to this pool.

## Upkeep

Anyone can call the `performUpkeep` function, and when this function is called, various profits and rewards are settled. The caller of `performUpkeep` is given a reward to encourage them to call.

It performs settlement in a total of 11 steps.

1. It makes the Liquidizer burn the USDS that was repaid or liquidated.
2. It withdraws the DAO's arbitrage reward (WETH) and gives part of it as a reward to the caller.
3. It swaps part of the arbitrage reward received in step2 to USDS, DAI and deposits POL in USDS/DAI.
4. It swaps part of the arbitrage reward received in step2 to USDS, SALT and deposits POL in USDS/SALT.
5. It swaps all the remaining arbitrage reward WETH in step2 to SALT and sends it to the SaltRewards contract. This will be used for liquidity provider rewards and staking rewards.
6. It requests the Emissions to send SALT for rewards to SaltRewards. It sends tokens pre-deposited in Emissions at a set rate.
7. It requests SaltRewards to distribute SALT tokens to RewardsEmitter. It sends to the RewardsEmitter for liquidity providers and the RewardsEmitter for staking at their respective set ratios.
8. It requests to send a certain proportion of the rewards deposited in RewardsEmitter to StakingRewards so that users can claim.
9. It settles the liquidity provider rewards for POL. 10% is sent to the team wallet, part of the remaining is burned, and the remaining is owned by the DAO.
10. Request the DAO vesting wallet to release SALT tokens. They will be released linearly over 10 years.
11. Request the team vesting wallet to release SALT tokens. They will be released linearly over 10 years.

### Time spent:
55 hours