# Low Findings

## [L-01] Pre-Approval of collateralAndLiquidity on SALT by the DAO puts the DAOs SALT reserves at risk

### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L90

### Description
1. The DAO contract [pre-approves](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L90) the collateralAndLiquidity contract on SALT/USDS and DAI to the maximal amount to save the gas of individual approve calls whenever formPOL is called during an Upkeep.
2. While USDS and DAI are not kept long-term by the DAO, Salt rewards accumulate in the DAO contract from several sources (such as POL rewards, vesting wallet emissions etc.) and form a substantial asset of the DAO.
3. A call to ERC20::approve takes around 45,000 gas units which is less than 2% of the ~2.3M gas units of an Upkeep call (according to Salty's documentation)
4. The pre-approval exposes all of the DAOs SALT reserves to exfiltration through potential exploits of the collateralAndLiquidity contract.
5. Given that SALT is the main token for value accrual in the DAO, the risk is not justified by the marginal gas savings and specific, per-upkeep approvals should be used.

## Recommended Mitigation Steps
Remove the max approvals from the Dao constructor and add an exact amount approval in 


## [L-02] proposeWebsiteUpdate enables creating multiple (contradicting) proposals simultaneously

### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L249C11-L249C31
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L102

### Description
1. The Proposal contract prevents a proposal from being opened if an open proposal exists with the same name. For example, while a ballot to change priceFeed1 is open, there can be no additional ballot to change priceFeed1 (even if the new ballot offers a different feed address). This is ensured bacause the ballot name for setContract is:
```solidity
string memory ballotName = string.concat("setContract:", contractName );
```
where contractName is either priceFeed1,2,3 or accessManager. 

2. In proposeWebsiteUpdate however, the ballot name is defined as
```solidity
string memory ballotName = string.concat("setURL:", newWebsiteURL );
```
Meaning only WebsiteUpdate ballots who propose the same URL are blocked from being opened in parallel. Additional WebsiteUpdate proposals can be opened with different URLs while previous proposals are still open.

3. This is inconsistent with the behaviour of other proposal types (such as setContract) and results in unexpected behaviour for voters (i.e. the last executed proposal URL is set, regardless of its vote-count compared to other suggested URLs that were open in parallel)

### Recommended Mitigation Steps
Change the ballot name of proposeWebsiteUpdate to a fixed string such as "setURL" so that only one updateWebsite proposal can be live simultaneously.


## [L-03] the Upkeep function _formPOL trades equal ammounts of weth to SALT and USDS instead of using the ratio of SALT/USDS in the pool
### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L129
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L166
### Description
1. The [_formPOL](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L129) function in Upkeep swaps WETH to the two tokens to be added as POL in equal amounts. The function is called once for USDS/DAI and once for SALT/DAI
2. While using equal amounts may make sense for USDS/DAI (likely to be traded at close to 1:1), it does not for SALT/DAI whose trading ratio may be very different.
3. When the POL is formed in [DAO.formPOL](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L321C26-L321C58) the depositLiquidityAndIncreaseShare function is called with useZapping set to true. This means that the amounts of SALT/DAI (likely to be extremely imbalanced compared to the pool ratio) will cause a large zap swap to match the pool SALT/DAI ratio.
4. The impact is not only substantial waste of gas, but also a potentially large slippage in the price of SALT/DAI caused by the zap swap, especially if this pool's liquidity is relatively low (Excessive slippage due to zapping swaps is shown with a POC in another finding I submitted).

### Recommended Mitigation Steps
instead of trading equal amounts in _formPOL, get the current reserve ratio from the pool of the relevant tokens and trade the pro-rata amount of WETH for each.

## [L-04] Emissions mechanism creates an unpredictable emissions rate that depends on Upkeep calls rate
### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/Emissions.sol#L55
### Description

1. The current Emissions mechanism is activated from the Upkeep function and calculates the current emission as a percentage of the remaining funds to emit. The percentage is calculated as the pro-rata of a constant (default: 0.5% per week) based on the time since last call. When emissions are calculated this way, the more frequent the Upkeep calls, the slower the emission rate. For example: If Upkeep is called once a week over a month, the total emissions in that month are:  $$\sum_{k=1}^w totalfunds * weeklyrate^k$$ $$w=4$$
If upkeep is called every hour in the same time period, total emissions are: $$\sum_{k=1}^h totalfunds * (weeklyrate/h)^k$$ $$h=4*7*24=672$$ The effect is the reverse of compounded interest where the more frequent the emissions the less funds are emitted in a given period. (The effect is accentuated by the double aplication of emissions rate using this calculation method: once in Emissions.sol and once in RewardsEmitter.sol)  

2. The rate of Upkeep calls depends on market conditions. For example: in high volatility periods the arbitrage profit rate is likely to be higher than normal. This will create a more frequent incentive to call upkeep (the reward for which is 5% of arbitrage profits since the last call).  
3. This creates an undesired uncertainty for both LPs and stakers regarding the rate at which they'll receive emissions, contrary to the common vesting schedules on most DeFi protocols. This in turn creates a disincentive to become a Salty LP/staker.

### Recommended Mitigation Steps
A similar emissions pattern can be achieved without dependency on Upkeep call rate, by using a function of the form 
$$A = \frac{-a}{b} \cdot (e^{-bx2} - e^{-bx1})$$
$$where\;\;a=initialAmount\;\;b=constant\;\;x1=lastUpdateSecondsSinceStart\;\;x2=currentSecondsSinceStart$$ 
(this is the integral between x1,x2 of $f(x) = a \cdot e^{-bx}$ - a function whose form is similar to the one used in the current mechanism)

## [L-05] priceFeedModificationCooldown treats all price feeds (1,2,3) as one, and consequently might delay a feed replacement unnessecarily
### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L50
### Description
1. According to the code comments, the priceFeedModificationCooldown (default of 35 days) is meant to "Allows time to evaluate the performance of the recently updated PriceFeed before further updates are made" however there are three price feeds defined on the aggregator, and the cooldown period treats them as one (meaning if priceFeed1 was recently updated but priceFeed2 was not, pricefeed2 would still be blocked from changing for the duration of the cooldown starting at the latest update of priceFeed1).
2. The implication is that if a price feed that has been active for some time (longer than the cooldown period) proves to be unreliable, replacing it to improve price accuracy might be delayed unnecessarily because of a recent change in another price feed. Since price feeds affect the efficiency of liquidations and collateral management, this in turn might negatively effect the system's financial stability.

### Recommended Mitigation Steps
Either apply the cooldown separately per feed, or remove the cooldown altogether (keeping in mind that price feed changes might be required with urgency for example if a price feed is permanently down or compromised.)

## [L-06] priceFeedModificationCooldown is not applied to the initial feeds that are set in the system by the deployer
### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L37
### Description
1. The cooldown period used to allow for feed evaluation before it is replaced (35 days by default) is only applied when [setPriceFeed](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L47) is called. However, the initial feeds that are set in the system by the deployer and are activated once the boorstrapBallot is finalized, are set through a different function [setInitialFeeds](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L37) and do not activate the cooldown period.
2. Following the same logic that necessitates the cooldown for feeds set by the DAO, initial feeds set by the deployer require the same evaluation time and should have the cooldown period applied to them as well

### Recommended Mitigation Steps
Add a bootstrap function to PriceAggregator, callable by BootstrapBallot only, that will initiate the priceFeedModificationCooldownExpiration variable. BootstrapBallot can call this function during finalizeBallot.

# QA Findings

## [Q-01] - Inaccurate comment in Pools.sol line 292
### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L292
### Description
The comment reads: ``"Deposit the arbitrage profit for the DAO - later to be divided between the DAO, SALT stakers and liquidity providers in DAO.performUpkeep"``
however there is no DAO.performUpkeep function and the distribution happens in other contracts 

### Recommended Mitigation Steps
Change comment to ``"Deposit the arbitrage profit for the DAO - later to be divided between the DAO, SALT stakers and liquidity providers at part of the Upkeep process"``


