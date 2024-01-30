### Approach taken when reviewing the protocol

This is a pretty large codebase, so I started with trying to understand the protocol first, before reading the code and getting the general idea of the protocol.

1. Watched the Video Technical Walkthrough to get a sense of the protocol.
2. Looked at the codes with the least amount of lines and started from there. 
3. Started to understand the protocol a little more and grouped the contracts into different sections (Airdrop Section, Config Section, ERC20 Section, DEX Section, Lending Section, Staking Section)
4. Started linking the sections together and seeing how each contract affects another contract (eg how Airdrop section relates to Staking and Staking rewards)
5. Started understanding the Airdrop Section first, InitialDistribution, Airdrop and BootstrapBallot
6. Next, started figuring out the staking section. (SALT staking, xSALT staking)
7. Analyze the DAO and notice how the DAO has the capability of submitting proposals that affects every contract
8. After finishing the DAO, Parameters, Proposals and all the Config, went to look at Pools.
9. Compared the code of Pools to Uniswap V2 (similar idea), to see any difference between how both protocols tackle AMM functionality
10. Analyze the lending section (WBTC-WETH), Collaterization Ratio, Liquidation process (liquidizer)
11. Checked the PriceAggregators (Chainlink, Salt and Uniswap) and cross-compare to the whole lending process
12. Fill in the gaps (Arbitrage, Math)
13. Started end-to-end scenario analysis for hypothetical users (eg a user stakes 100 SALT. What happens? How much votes does he have?)
14. Looked through the test files.
15. Wrote the QA, Analysis and Report whilst going through the protocol

### Summary and Mechanism Review

##### Airdropping tokens

User's POV:

1. The user must have a signature (be whitelisted or something else) and call BootstrapBallot.vote().
2. The user can vote yes or no to start the exchange
3. There is a time limit for the voting duration
4. Once the vote ends, if the vote passes, then users can go to claimAirdrop() and claim their SALT tokens
5. Note that the SALT cannot be claimed immediately, but is staked for the user. To claim the SALT directly, they have to call unstake and wait 52 weeks for the total duration.

- Mechanism is well thought out, users can only vote once and claim once.
- The amount of SALT distributed is distributed equally (5 million), and if there is low number of votes(), then authorized voters can get more SALT.
- It is uncertain how users can obtain a signature in order to be eligible to vote. If getting into the whitelist is easy, then users can create multiple accounts to vote (no) and grief the whole protocol (costs a lot of gas for them though)
- An important thing to note is that the vote can fail, and if the vote does fail, what happens next? Will the contract be redeployed?
- Also, if there are 0 authorized voters (extremely unlikely), then the whole process will fail.
- Contracts used: Airdrop.sol, BootstrapBallot.sol, InitialDistribution.sol, ExchangeConfig.sol, Staking.sol
- Mechanism is dependant on users calling vote and users actually voting yes.


##### Depositing Collateral and Minting USDS

User's POV:

1. The user deposits WBTC and WETH token into the Pools contract through CollateralAndLiquidity.depositCollateralAndIncreaseShare().
2. Users can call `borrowUSDS()` to mint USDS. 
3. Users can repay their USDS debt by calling `repayUSDS()`.

- A common collaterized stablecoin mechanism. Having an initial collateral ratio and minimum collateral ratio is good so that the users position will not get immediately liquidated when withdrawing the max amount of debt
- Initial Collateral ratio is set as 200%, with a 150-300% range. Mincollateral ratio is set at 110%, with a 110-120 range
- Users have to repay their USDS in order to withdraw their liquidity, which is correct.

##### Liquidation

Liquidator's POV:

1. Liquidator calls `liquidateUser()` in CollateralAndLiquidity.sol
2. If the user is primed for liquidation, all his WBTC and WETH tokens will be withdrawn from the pool.
3. The liquidator gets 5% of the WBTC and WETH tokens.
4. The WBTC and WETH tokens will be sent to the Liquidizer contract, get swapped to USDS and burnt
5. The liquidated user gets to keep his USDS debt.

- Interesting mechanism for liquidation. Normally, the liquidator will repay the debt, but in this case the liquidator simply calls a function. 
- Unsure how burning the WBTC and WETH token will help the USDS token. Probably through the rebalancing of the pools?
- Liquidation rewards can range from 5-10%. 

##### Depositing liquidity

User's POV:

1. User can deposit liquidity through `Liquidity.depositLiquidityAndIncreaseShare()`. Take note that users can only deposit into whitelisted pools and that the tokens are whitelisted. 
2. User can withdraw liquidity through `Liquidity.withdrawLiquidityAndClaim()`. Note that there is a buffer time between depositing and withdrawing liquidity to prevent any flashloan rewards exploit.
3. When liquidity is withdrawn, SALT rewards will be distributed to the liquidity provider.

- There is no ERC20 liquidity token issued. Liquidity is just the combination of the amount of token A and token B, which get's pretty confusing when both tokens have different decimals.
- Users can only deposit an equal ratio of token A and token B, which is good to prevent any liquidity imbalance exploit.

##### Staking SALT

User's POV:

1. User calls `stakeSALT()` in the Staking.sol contract. The user's wallet must have access first
2. User shares is increased. When SALT is staked, it becomes xSALT. Users can use xSALT as voting power to ballot in proposals
3. User can unstake xSALT for SALT. The unstaking process is between 1 week to 1 year with a penalty of 80% for faster unstaking.
4. The SALT that is penalized from unstaking prematurely will be burned.
5. Users can cancel their `unstake()` at any point in time.

- Normal Staking mechanism, with penalties. Note that the user's votes still exist even when unstake is called.

##### Swapping Tokens

- Users can call depositSwapWithdraw() to swap tokens from a whitelisted pool (assuming it has enough liquidity)
- Instead of a typical AMM swap, it adjusts the reserves and attempt an arbitrage, which is a novel concept

### Codebase quality analysis

##### [PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol)

| Contract    | Function                                  | Access         | Comments                                                                         |
| ----------- | ----------------------------------------- | -------------- | -------------------------------------------------------------------------------- |
| PoolsConfig | whitelistPool                             | onlyOwner, DAO | From 10-100 max pools, calls updateArbitrageIndicies                             |
| PoolsConfig | unwhitelistPool                           | onlyOwner, DAO | unwhitelist the pools, what if still have liquidity inside? can it be taken out? |
| PoolsConfig | changeMaximumWhitelistedPools             | onlyOwner, DAO | from 20-100 with a default of 50                                                 |
| PoolsConfig | changeMaximumInternalSwapPercentTimes1000 | onlyOwner, DAO | from 0.25% to 2% with a default of 1%                                            |

- The check for whitelist is from the increaseuserShare function. It seems that decreaseUserShare doesn't check for whitelist, which is correct.
- Since the owner is the only whitelisting the pool, issues with whitelisting pools will fall under centralization risks

##### [CoreChainlinkFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol)

| Contract          | Function             | Access        | Comments                                                                                                  |
| ----------------- | -------------------- | ------------- | --------------------------------------------------------------------------------------------------------- |
| CoreChainlinkFeed | latestChainlinkPrice | view function | Called when trying to get a loan, will get WBTC and WETH price. Note that WBTC is returned as 18 decimals |

- **A few Potential Issues**
- The `latestRoundData` is not sufficiently checked, may return stale prices.
- MAX_ANSWER_DELAY is fixed, so make sure that BTC and ETH feeds also have the same delay
- Price returning as 0 instead of reverting can be dangerous. Not sure if it's the best for the protocol since it uses a price aggregator, if another price feed is manipulated then the zero return will be detrimental, however the scenario where 2 price feeds are down simultaneously is pretty unlikely
- Other than that, price return and use of latestRoundData is correct

##### [DAOConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol)

| Contract  | Function                              | Access                  | Comments                               |
| --------- | ------------------------------------- | ----------------------- | -------------------------------------- |
| DAOConfig | changeBootstrappingRewards            | onlyOwner, probably DAO | Comments and code match up, 50k-500k   |
| DAOConfig | changePercentPolRewardsBurned         | onlyOwner, probably DAO | Comments and code match up, 25-75      |
| DAOConfig | changeBaseBallotQuorumPercent         | onlyOwner, probably DAO | Comments and code match up, 5000-20000 |
| DAOConfig | changeBallotDuration                  | onlyOwner, probably DAO | Comments and code match up, 3-14       |
| DAOConfig | changeRequiredProposalPercentStake    | onlyOwner, probably DAO | Comments and code match up, 100-2000   |
| DAOConfig | changeMaxPendingTokensForWhitelisting | onlyOwner, probably DAO | Comments and code match up, 3-12       |
| DAOConfig | changeArbitrageProfitsPercentPOL      | onlyOwner, probably DAO | Comments and code match up, 5-45       |
| DAOConfig | changeUpkeepRewardPercent             | onlyOwner, probably DAO | Comments and code match up, 1-10       |

- Same as other Config files, used in Parameter, which is inherited from DAO.sol. There are 8 changes here, so there must be 8 changes in the Parameter contract as well.

##### [Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol)

| Contract  | Function                    | Access                             | Comments                                                                                             |
| --------- | --------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Proposals | \_possiblyCreateProposal    | internal (DAO and public can call) | Users must have a percentage of salt stake to call a proposal, one user can propose only one ballot  |
| Proposals | createConfirmationProposal  | called by DAO                      | DAO doesn't need any staked SALT to call a proposal                                                  |
| Proposals | markBallotAsFinalized       | called by DAO                      | This should be internal? If called this accidentally, then proposal will be voided                   |
| Proposals | proposeParameterBallot      | anyone can call                    | Only one proposal per address, to prevent spam                                                       |
| Proposals | proposeTokenWhitelisting    | anyone can call                    | Maximum of 12, default 5                                                                             |
| Proposals | proposeTokenUnwhitelisting  | anyone can call                    | DAO doesn't need any staked SALT to call a proposal                                                  |
| Proposals | proposeSendSALT             | anyone can call                    | Would be quite funny to propose sending salt to yourself                                             |
| Proposals | proposeCallContract         | anyone can call                    | This is quite dangerous as no one knows what the arbitrary contract code holds                       |
| Proposals | proposeCountryInclusion     | anyone can call                    | Interesting requirement of 2 bytes ISO 3166 Alpha-2 Code, can this be bypassed?                      |
| Proposals | proposeCountryExclusion     | anyone can call                    | Excludes a country, is there any country where it must be always excluded? Like tokenunwhitelisting? |
| Proposals | proposeSetContractAddress   | anyone can call                    | Also can be quite dangerous                                                                          |
| Proposals | proposeWebsiteUpdate        | anyone can call                    | website might not even be a proper one                                                               |
| Proposals | castVote                    | anyone with staked salt            | I like how there is recasting of votes. Votes can only be called once                                |
| Proposals | requiredQuorumForBallotType | anyone with staked salt            | whitelist token needs 2x the base quorum, more important things like changing website need 3x        |

- Good that only one user can open one active proposal to prevent spamming
- The salt staking is quite interesting, because before any stake, a user can propose anything without much salt?
- It's a proposal phase so even though anyone can create a proposal, it's still a DAO effort to make sure it passes or not
- All the contracts call the proper channels
- Casting vote checks that the amount of votes can only be used once per ballot
- The required quorum checks for the votes percentage correctly.

##### [Parameters.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol)

| Contract   | Function                 | Access        | Comments          |
| ---------- | ------------------------ | ------------- | ----------------- |
| Parameters | \_executeParameterChange | internal call | Abstract contract |

- Make sure that StakingConfig has 4 types, DAOConfig has 8 types, rewardsConfig has 4 types, StableConfig has 6 types, PoolsConfig has 2 types, PriceAggregator has 2 types
- Function is written properly and all else if calls are correctly placed.
- All intended changes and increments are written correctly

##### [Airdrop.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol)

| Contract | Function         | Access                            | Comments                                                                                                 |
| -------- | ---------------- | --------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Airdrop  | authorizeWallet  | only BootstrapBallot can call     | allowClaiming must not be called first                                                                   |
| Airdrop  | allowClaiming    | only InitialDistribution can call | users who are authorized can claim, but that means that there must be salt balance in the contract first |
| Airdrop  | claimAirdrop     | only authorized wallet            | after allowclaiming and is authorized, only can claim once                                               |
| Airdrop  | isAuthorized     | view function                     | Uses Enumerable Set, added from authorizeWallet function                                                 |
| Airdrop  | numberAuthorized | view function                     | Simply checks the length                                                                                 |

- The first thing that happens is that the bootstrap Ballot must authorize all the wallet it wants to authorize
- Once allow claiming is called (can only be called once), then there is no more authorized wallet.
- This means even before the salt is sent to the initial distribution contract, some wallets must already be authorized.
- Users must vote in order to be authorized in BootstrapBallot.sol

- For users, they have to vote first, then wait until initial distribution is called, then call claimAirdrop(). They can only claim once. Their tokens will pass through the staking contract and then be transferred to the user (must check this interaction)
- Is this authorization independent from the entire protocol? In other words, does it affect anything else?
- All functions are checked to be written correctly. Users that are not authorized cannot claim. Also, once initial distribution starts, no one can be authorized anymore.

##### [BootstrapBallot.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol)

| Contract        | Function        | Access                        | Comments                                                                |
| --------------- | --------------- | ----------------------------- | ----------------------------------------------------------------------- |
| BootstrapBallot | constructor     | -                             | Take note of the ballotDuration, most important part of this contract   |
| BootstrapBallot | vote            | only users with the signature | signature check done offchain. Message hash includes the msg.sender     |
| BootstrapBallot | finalizeBallot  | public                        | If votes are equal, also assume fail. Function can only be called once. |
| BootstrapBallot | authorizeWallet | only BootstrapBallot can call | allowClaiming must not be called first                                  |

- Only can vote once, but must call vote() to be authorized
- **Potential Issue** What if the majority votes to not start the exchange? what happens? The whole contract is redeployed? Also, is getting whitelisted easy? Can users just keep voting no with multiple accounts? (because they will still get airdrop anyways if the votes succeed)
- If succeed, will call initialdistribution.distributionApproved, and then airdrop.allowClaiming will be called. startExchangeApproved() will also be called.
- Voting and finalizing is done correctly. Pools and initial distribution does receive the approval

##### [InitialDistribution.sol]

| Contract            | Function             | Access                        | Comments                                            |
| ------------------- | -------------------- | ----------------------------- | --------------------------------------------------- |
| InitialDistribution | distributionApproved | Only BootstrapBallot can call | The 100M salt token must already be in the contract |

- 52M to emissions, 25M to DAO vesting wallet, 10M to Team vesting wallet, 5M to airdrop participants, 5M to liquidity and 3M to staking, total to 100M.
- No salt should be left in this contract after distributionApproved is called.
- Once distributionApproved is called, no more wallets can be authorized. So these wallets may be on the whitelist but if they don't vote, they don't get the airdrop.
- This function assumes that the Ballot succeeds, and the WhitelistedPools is correct.
- No potential issues in this contract alone. MILLION_ETHER is written correctly, every calculation is multiplied properly. Check cannot be griefed because no other SALT can be minted other than the 100M given to the msg.sender of the SALT contract.

##### [PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol)

| Contract        | Function                                         | Access                  | Comments                                                                          |
| --------------- | ------------------------------------------------ | ----------------------- | --------------------------------------------------------------------------------- |
| PriceAggregator | setInitialFeeds                                  | onlyOwner, probably DAO | The require statement is a little wrong, more comments below                      |
| PriceAggregator | setPriceFeed                                     | onlyOwner, probably DAO | Call once every 35 days, priceFeedModificationCooldown can be changed, 30-45 days |
| PriceAggregator | changeMaximumPriceFeedPercentDifferenceTimes1000 | onlyOwner, probably DAO | Same as other Config files, comments and code match up, from 1000 - 7000          |
| PriceAggregator | changePriceFeedModificationCooldown              | onlyOwner, probably DAO | comments and code match up, from 30-45 days                                       |
| PriceAggregator | \_aggregatePrices                                | internal                | gets the average of the two closest price                                         |

- **Potential Issue:** in setInitialFeeds, if owner sets the first price feed1 as 0 address, then he can call setInitialFeeds again
- Not sure about the purpose of a `priceFeedModificationCooldown` since it's only changed if something serious is going to happen / has happened. It should be able to be changed immediately.
- Config files are set properly, with lower bound and upper bound tallying up with the comments
- Aggregate prices is written correctly. Even if 1 price feed returns 0, hopefully the other two price feeds returns the correct price, and gets the average sum
- The percentageDifference tolerance is about 3%, and be changed from 1% - 7%

##### [CoreSaltyFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol)

| Contract      | Function    | Access                                                       | Comments                                                                                 |
| ------------- | ----------- | ------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| CoreSaltyFeed | getPriceBTC | view function, called by PriceAggregator, no change of state | Check that reservesWBTC is in 8 decimals when pools.getPoolReserves is called            |
| CoreSaltyFeed | getPriceETH | view function, called by PriceAggregator, no change of state | Check that pools.getPoolReserves returns the proper reservesWETH and reservesUSDS amount |

- The reserve price cannot be less than DUST, which is 100.
- This contract interacts with pools.getPoolReserves, so must check whether that function returns the correct amount

##### [RewardsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol)

| Contract      | Function                         | Access                  | Comments                                                    |
| ------------- | -------------------------------- | ----------------------- | ----------------------------------------------------------- |
| RewardsConfig | changeRewardsEmitterDailyPercent | OnlyOwner, probably DAO | Comments and Code match up, from 250 - 2500 (0.25% to 2.5%) |
| RewardsConfig | changeEmissionsWeeklyPercent     | OnlyOwner, probably DAO | Comments and Code match up, from 250 - 1000 (0.25% to 1%)   |
| RewardsConfig | changeStakingRewardsPercent      | OnlyOwner, probably DAO | Comments and Code match up, from 25 - 75 (25% to 75%)       |
| RewardsConfig | changePercentRewardsSaltUSDS     | OnlyOwner, probably DAO | Comments and Code match up, from 5 - 25 (5% to 25%)         |

- Probably same as StakingConfig and the other Configs file, all change is called in Parameter.sol which is an abstract contract inherited by DAO.sol

##### [Emissions.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol)

| Contract  | Function      | Access                        | Comments                                              |
| --------- | ------------- | ----------------------------- | ----------------------------------------------------- |
| Emissions | performUpkeep | Only Upkeep contract can call | Calls rewardsConfig.emissionsWeeklyPercentTimes1000() |

- upkeep contract is set in the ExchangeConfig contract
- checked the calculation of saltToSend, seems correct

```

uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );

Assume 100e18 as saltBalance, 0.5% should be 5e17.

(1 second)
100e18 * 1 * 500 / (100 * 1000 * 604800) = 826719576720

(1 week)
100e18 * 604800 * 500 / (100 * 1000 * 604800) = 5e17 (0.5%)
```

- This is called in the Upkeep contract, at step 6. It is assumed that the emission contract already has some salt balance? I think it's from initial distribution of 52 million salt.
- **Potential Issue:** Must check last emission, whether it will truncate to zero, apparently dust values would truncate.
- Emission calculation is checked, timeSinceLastUpkeep is checked, max of 1 week.

##### [USDS.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol)

| Contract | Function                  | Access                             | Comments                                                                                       |
| -------- | ------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------- |
| USDS     | setCollateralAndLiquidity | onlyOwner, inherited Ownable       | renounceOwnership() can be dangerous is the collateralAndLiquidity contract address is changed |
| USDS     | mintTo                    | only collateralAndLiquidityCanCall | check how collateralAndLiquidity calls `mintTo`, check whether `amount` can be manipulated     |
| USDS     | burnTokensInContract      | public                             | will only be an error if DAO mistakenly deposits USDS into contract                            |

- Inherits ERC20, Ownable
- Inheritance is correct, the minting from collateral contract is also correct.

##### [StakingConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol)

| Contract      | Function                   | Access            | Comments                                           |
| ------------- | -------------------------- | ----------------- | -------------------------------------------------- |
| StakingConfig | changeMinUnstakeWeeks      | onlyOwner, is DAO | Comments and code match up, range only from 1-12   |
| StakingConfig | changeMaxUnstakeWeeks      | onlyOwner, is DAO | Comments and code match up, range only from 20-108 |
| StakingConfig | changeMinUnstakePercent    | onlyOwner, is DAO | Comments and code match up, range only from 10-50  |
| StakingConfig | changeModificationCooldown | onlyOwner, is DAO | Comments and code match up, range only from 15-600 |

- All the change is called in Parameter.sol, which is an abstract contract inherited from the DAO
- Only DAO can call these changes, which is changed through a balloting process
- The increase and decrease can be quite inconvenient (If minUnstakeWeeks is currently at 1, then have to call 11 times to become 12), but it's probably a design decision.
- **Potential Issue**: changeminunstake weeks ranges from 1-12, but docs mention 2-12.
- All other comments are checked to be consistent with the code. 

##### [ManagedWallet.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol)

| Contract      | Function       | Access                       | Comments                                                                                             |
| ------------- | -------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------- |
| ManagedWallet | proposeWallets | called by mainWallet         | 2-step transfer to change the mainwallet and confirmation wallet                                     |
| ManagedWallet | receive()      | called by confirmationWallet | Ether is stuck in contract, and the confirmationWallet can alter the activeTimelock duration anytime |
| ManagedWallet | changeWallets  | called by proposedMainWallet | Must wait for activeTimelock to set and reset the variables                                          |

- Interesting to see how the mainWallet only can call proposeWallet and the confirmation wallet only can set the activeTimelock.
- **3 potential issues:**
- 1. The activeTimelock can be set even before proposeWallets is called, so the activeTimelock can be bypassed
- 2. The contract cannot draw out the ether, ether stuck in contract
- 3. Confirmation wallet can reject the whole proposal by either not calling the contract or just sending in 1 wei.

##### [AccessManager.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol)

| Contract      | Function                 | Access                                | Comments                                                                                             |
| ------------- | ------------------------ | ------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| AccessManager | excludedCountriesUpdated | called by DAO                         | can be inconvenient because all verified wallets have to be verified again, does it affect anything? |
| AccessManager | grantAccess              | public, users must have the signature | geoVersion can be changed                                                                            |
| AccessManager | \_verifyAccess           | internal, called by grantAccess       | Check whether the \_verifySignature function is written correctly                                    |

- grantAccess / walletHasAccess is used by which contracts?

##### [Salt.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol)

| Contract | Function             | Access   | Comments                                                                     |
| -------- | -------------------- | -------- | ---------------------------------------------------------------------------- |
| Salt     | constructor          | one time | mints 100M salt to msg.sender, and doesn't have any mint, consider hard cap? |
| Salt     | burnTokensInContract | public   | same as USDS, error will be on the user side                                 |

- Inherits ERC20
- Mint is correct, mints to a msg.sender, which is supposed to be sent to the InitialDistribution.

### Architecture Review

##### `Design Patterns and Best Practices:`

- Common design patterns are used, like onlyOwner, or checking for DAO access, or reentrancy guards
- Abstract contracts and Inheritance is used well.
- Nuances like checking for DUST amounts, having min-collateral and initial-collateral, having a time delay for staking and unstaking shows that the protocol is written well.

##### `Code Readability and Maintainability:`

- Code is quite difficult to read due to the sheer size of the codebase and the amount of Math involved in AMM.
- Code also has a lot of external calls to other contracts which is quite confusing, but there is a common pattern of using Config and Parameters 
- There is also a lot of internal calls, but that is to be expected from a large codebase with many different function

##### `Error Handling and Input Validation:`

- Events and Error messages are easy to understand. 
- Input is validated well in every config file, and the code matches the comments.

##### `Interoperability and Standards Compliance:`

- Good knowledge of the ERC20 standard when creating USDS and SALT tokens. 

##### `Testing and Documentation:`

- Extensive tests (unit, integration tests) done, reaching an overall coverage of almost 100%. 
- Documentation (whitepaper, github) is plentiful and includes quality reasoning at every juncture of the protocol. 
- **One improvement could be having more diagrams and a summarized version of how the protocol works from the top down, with a end-to-end scenario of how a user can interact with the protocol  (what function should the user call first, what can a user accomplish, why is the user incentivized to use the protocol)**

##### `Upgradeability:`

- Protocol is not intended to be upgradeable, but contracts can be redeployed and rerouted easily through the changeManagers and changeRegistries function.

##### `Dependency Management:`

Protocol relies on external libraries like OpenZeppelin. Protocol should keep an eye on vulnerabilities that affects those external integrations, and make changes where necessary.

Overall, great architecture from the protocol, slight changes would be to the written code itself (using modifiers for repeated code, checking zero values, checking overflows etc) and more real life scenarios in the documentation.

##### `Centralization Risks`

Not much centralization risks as the protocol is almost run fully on the DAO and votes. The only thing the DAO can do is control which proposals to be passed by finalizing the Ballot. Otherwise, most of the protocol functions like clockwork.

### Time spent:
40 hours