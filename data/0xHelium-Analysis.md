# Salty.io Analysis reports
![Imgur](https://i.imgur.com/0kcgGjN.png)
## Note for sponsor to contextualize the analysis
The following analysis should be taken with the following in mind:
- Most of the recommendation are after the codebase overview, so there is where you should look at.
- The following analysis reports of this codebase mainly adds lots of personal recommandation to improve said codebase.
- Sponsor can use the visual representation to improve their docs and use the recommandations to improve the codebase.

### Codebase Overview
Salty.IO is a Decentralized Exchange on Ethereum which uses Automatic Atomic Arbitrage (AAA) to generate yield and provide Zero Fees on all swaps.
![Imgur](https://i.imgur.com/4YNcJZ9.png)

With AAA, market inefficiencies are arbitraged at swap time to create profits - which are then distributed to liquidity providers and stakers and used to form Protocol Owned Liquidity (POL) for the DAO.

Additionally, Salty.IO provides USDS, an overcollateralized ERC20 stablecoin native to the protocol which uses WBTC/WETH LP as collateral.
![Imgur](https://i.imgur.com/VJIPyeC.png)

### Technical overview
The Salty.IO codebase is divided up into the following folders:
/arbitrage - handles searching for arbitrage opportunities at user swap time with the actual arbitrage swaps being done within Pools.sol itself. 
![Imgur](https://i.imgur.com/gAgV19D.png)
As seen in the above image , we can conclude that swaping token1 -> token2 following the inverse route as swapping token2 -> token1 This mechanism is gas efficient in theory, but in practice it's prone to sandwich attacks by MEVs. Consider the scenario below:Arbitrage is following WETH-> WBTC-> SALT-> WETH path, A mev can just buy some WBTC to increase the price prior to swapping WETH-> WBTC and sell after the swapping to make profits.
To remediate consider adding a slippage protection to the arbitrage

/dao - handles creating governance proposals, voting, acting on successful proposals and managing POL (Protocol Owned Liquidity).
DAO adjustable parameters are stored in ~Config.sol contracts and are stored on a per folder basis.

/launch - handles the initial airdrop, initial distribution, and bootstrapping ballot (a decentralized vote by the airdrop recipients to start up the DEX and distribute SALT).

/pools - a core part of the exchange which handles liquidity pools, swaps, arbitrage, and user token deposits (which reduces gas costs for multiple trades) and pools contribution to recent arbitrage trades (for proportional rewards distribution).

/price_feed - implements a redundant price aggregator (initially using Chainlink, Uniswap v3 TWAP and the Salty.IO reserves) to provide the BTC and ETH prices used by the overcollateralized stablecoin framework.

/rewards - handles global SALT emissions, SALT rewards (which are sent to liquidity providers and stakers), and includes a rewards emitter mechanism (which emits a percentage of rewards over time to reduce rewards volatility).

/stable - includes the USDS contract and collateral functionality which allows users to deposit WBTC/WETH LP as collateral, borrow USDS (which mints it when borrowed), repay USDS (which burns it) and allow users to liquidate undercollateralized positions.

/staking - implements a staking rewards mechanism which handles users receiving rewards proportional to some "userShare".What the userShare actually represents is dependent on the contract that derives from StakingRewards.sol (namely Staking.sol which handles users staking SALT, and CollateralAndLiquidity.sol which handles users depositing collateral and liquidity).

/ - includes the SALT token, the default AccessManager (which allows for DAO controlled geo-restriction) and the Upkeep contract (which contains a user callable performUpkeep() function that ensures proper functionality of ecosystem rewards, emissions, POL formation, etc).

### High level schema
At the highest level Salty.io can be summarized to a marketplace for lending, staking, decentralized exchange, and DAO as seen
in the following schema.
![Imgur](https://i.imgur.com/ELHBEwH.png)

### Codebase weaknesses
![Imgur](https://i.imgur.com/gAgV19D.png)
- As seen in the above image , we can conclude that swaping token1 -> token2 following the inverse route as swapping token2 -> token1
This mechanism is gas efficient in theory, but in practice it's prone to sandwich attacks by MEVs. Consider the scenario below:
Arbitrage is following WETH-> WBTC-> SALT-> WETH path, A mev can just buy some WBTC to increase the price prior to swapping WETH-> WBTC
and sell after the swapping to make profits. To remediate consider adding a slippage protection to the arbitrage
- Codebase is not fully decentralized as it leverage the DAO to make some decisions in the protocol, this can be remediated by making 
the protocol fully decentralized as it will bring more trust among users; The DAO model can still be manipulated through heavy voting power. Consider implementing a strong mechanism to prevent a single person from owning all the votes proposals.
- Codebase unwhitelist some markets for some reasons, consider verifying that users cannot still borrow from the depreceated market before shutting them down, because this can incur bad debt for the protocol.

## Codebase strenghs
- Codebase is inovating on available AMM to create a new type that incur 0 fees for swapping, especially by implementing an arbitrage into
the dex.
- Codebase is using a price aggregator with multiples price source to ensure that at least 2 feeds returns the same price, this design is
secure but in some edge cases it can be bypassed when chainlink return the min price while asset price dropped below the min price
```solidity
	function _aggregatePrices( uint256 price1, uint256 price2, uint256 price3 ) internal view returns (uint256)
		{
		uint256 numNonZero;

		if (price1 > 0)
			numNonZero++;

		if (price2 > 0)
			numNonZero++;

		if (price3 > 0)
			numNonZero++;

		// If less than two price sources then return zero to indicate failure
		if ( numNonZero < 2 )
			return 0;

		uint256 diff12 = _absoluteDifference(price1, price2);
		uint256 diff13 = _absoluteDifference(price1, price3);
		uint256 diff23 = _absoluteDifference(price2, price3);

		uint256 priceA;
		uint256 priceB;

		if ( ( diff12 <= diff13 ) && ( diff12 <= diff23 ) )
			(priceA, priceB) = (price1, price2);
		else if ( ( diff13 <= diff12 ) && ( diff13 <= diff23 ) )
			(priceA, priceB) = (price1, price3);
		else if ( ( diff23 <= diff12 ) && ( diff23 <= diff13 ) )
			(priceA, priceB) = (price2, price3);

		uint256 averagePrice = ( priceA + priceB ) / 2;

		// If price sources are too far apart then return zero to indicate failure
		if (  (_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000 )
			return 0;

		return averagePrice;
		}
```
- Codebase is well structured and natspec and comments were very detailled and helpfull.

## Systemic risk
- The DAO approach is a backdoor that can be exploited by a single entity by simply owning a large amount of voting power over other users. Consider a stric check for votes power to not exceed a threshold in order to avoid manipulation.
- The system relies on users calling a keeper function to keep things healthy, this have to be considered however: what if no user call that function especially if gas fees become so high on ethereum and calling it becomes unprofitable ? what will happen ? Protocol should consider calling performUpkeep themselves in certain scenario.
```solidity
function performUpkeep() public nonReentrant
		{
		// Perform the multiple steps of performUpkeep()
 		try this.step1() {}
		catch (bytes memory error) { emit UpkeepError("Step 1", error); }

 		try this.step2(msg.sender) {}
		catch (bytes memory error) { emit UpkeepError("Step 2", error); }

 		try this.step3() {}
		catch (bytes memory error) { emit UpkeepError("Step 3", error); }

 		try this.step4() {}
		catch (bytes memory error) { emit UpkeepError("Step 4", error); }

 		try this.step5() {}
		catch (bytes memory error) { emit UpkeepError("Step 5", error); }

 		try this.step6() {}
		catch (bytes memory error) { emit UpkeepError("Step 6", error); }

 		try this.step7() {}
		catch (bytes memory error) { emit UpkeepError("Step 7", error); }

 		try this.step8() {}
		catch (bytes memory error) { emit UpkeepError("Step 8", error); }

 		try this.step9() {}
		catch (bytes memory error) { emit UpkeepError("Step 9", error); }

 		try this.step10() {}
		catch (bytes memory error) { emit UpkeepError("Step 10", error); }

 		try this.step11() {}
		catch (bytes memory error) { emit UpkeepError("Step 11", error); }
		}
```

## Audit approach
- Day1: Read the docs and understand the protocol, i did some research to understand AMM in deep with all the technical concepts such as arbitrage lending borrowing, price aggregator etc...
- Day2: Delve deep into the codebase to get a general code architecture understanding. Here i try to understand the code architecture to get a good grasp of the codebase, in other words i made myself familiar with the codebase.
- Day3: Delve more deeper into the smarts contracts to understand the code in more deepness. Here i start finding some issues and setting them aside to write the report after i have some time.
- Day4: Compile the findings and write the reports







### Time spent:
22 hours