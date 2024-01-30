### [L-01] Last few dust amounts in the emission contract will be truncated to zero

This is the calculation for emission:

```
uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );
```

Assume emissionsWeeklyPercentTimes1000 is 500, since timeSinceLastUpkeep is not controllable (max of one week), the `saltBalance` will eventually be a dust amount in the contract. Then, the `saltToSend` will truncate to zero.

```
uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );

// Left 100 wei saltBalance
uint256 saltToSend = 100 * 604800 * 500 / 100 * 1000 * 604800 = 0
```

At the very end, make sure the salt amount can be withdrawn.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/Emissions.sol#L52-L60

### [L-02] In ManagedWallet.sol, the activeTimelock can be set even before proposeWallets is called, so the activeTimelock can be bypassed.

The confirmation wallet can send 0.05 ether even before `proposeWallets()` function is called to skip the TIMELOCK_DURATION.

```
    receive() external payable
        {
        require( msg.sender == confirmationWallet, "Invalid sender" );


                // Confirm if .05 or more ether is sent and otherwise reject.
                // Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
        if ( msg.value >= .05 ether )
                //@audit - Skip this part by depositing 0.05 ether before calling `proposeWallets()`
                activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
        else
                        activeTimelock = type(uint256).max; // effectively never
        }
```

Make sure the `activeTimelock` is only set once the `proposeWallet()` function is called.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L59-L69

### [L-03] In ManagedWallet.sol, the activeTimelock can be nullified at any time

If the confirmation wallet user changes his mind, he can send 1 wei of ether to the contract and reset the `activeTimelock`.

```
        if ( msg.value >= .05 ether )
>               activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
        else
>                       activeTimelock = type(uint256).max; // effectively never
```

This should not be the case, as once the activeTimelock is set, it should not be able to be nullified.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L65-L68

### [L-04] In PriceAggregator.sol, if owner sets the priceFeed1 as the zero address, then he can set the price feeds again, which bypasses the intended restriction.

In setInitialFeeds, the owner can set \_priceFeed1 as the zero address. He can then call `setInitialFeeds()` again

```
        function setInitialFeeds( IPriceFeed _priceFeed1, IPriceFeed _priceFeed2, IPriceFeed _priceFeed3 ) public onlyOwner
                {
                require( address(priceFeed1) == address(0), "setInitialFeeds() can only be called once" );

                //@audit - set _priceFeed1 as address(0)
                priceFeed1 = _priceFeed1;
                priceFeed2 = _priceFeed2;
                priceFeed3 = _priceFeed3;
                }

```

Have some checks in the `setInitialFeeds()` function like `require(_priceFeed1 != address(0))` to prevent the owner from re-setting the initial feeds.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L37-L45

### [L-05] Documentation and Code for Staking.sol does not match up

Document states:

>  Expedited unstaking is possible with a penalty - with the fastest default unstaking period being two weeks with a default penalty of 80%. 

The fastest default unstaking period is actually 1 week, with a default penalty of 80%.

```
        function changeMinUnstakeWeeks(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minUnstakeWeeks < 12)
                minUnstakeWeeks += 1;
            }
        else
            {
>           if (minUnstakeWeeks > 1)
                minUnstakeWeeks -= 1;
            }


                emit MinUnstakeWeeksChanged(minUnstakeWeeks);
        }
```

In my opinion, I think 1 week is better than 2 weeks because the penalty is already very heavy.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingConfig.sol#L34-L48

### [L-06] Users can just vote no in BootstrapBallot because they will be getting the airdrop either way

Users who have a signature (on the whitelist) can vote to start the exchange for the Salty protocol. Honestly, I don't see a reason why a vote is needed to start an exchange because if the vote is a resounding no, then will the whole contract be redeployed?

Also, voters that vote no are not penalized. They will still receive the same amount of airdrop as those who vote yes. So, there is not really a difference between a yes and no vote for the users perspective and the some griefers will vote no for fun because they will get the airdrop either way.

```
        function vote( bool voteStartExchangeYes, bytes calldata signature ) external nonReentrant
                {
                require( ! hasVoted[msg.sender], "User already voted" );


                // Verify the signature to confirm the user is authorized to vote
                bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, msg.sender));
                require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );


                if ( voteStartExchangeYes )
>                       startExchangeYes++;
                else
>                       startExchangeNo++;


                hasVoted[msg.sender] = true;


                // As the whitelisted user has retweeted the launch message and voted, they are authorized to the receive the airdrop.
>               airdrop.authorizeWallet(msg.sender);
                }

Consider implementing a small penalty for those that vote No. The current system is designed to assume the best of people, but I don't know if it's actually the ideal choice.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L48-L65

```

### [L-07] Chainlink's latestRoundData has no check for round completeness and may return stale prices

When collecting data from Chainlink, the data returned is not sufficiently checked. Specifically, roundID is not checked, as well as the min-max threshold.

```
                try chainlinkFeed.latestRoundData()
                returns (
                        uint80, // _roundID
                        int256 _price,
                        uint256, // _startedAt
                        uint256 _answerTimestamp,
                        uint80 // _answeredInRound
                )
```

Implement the roundId check as well

```
require(answeredInRound >= roundID, "round not complete");
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L32-L39

### [L-08] CoreChainlinkFeed uses the BTC_USD address when it should be using the WBTC-BTC one as well

CoreChainlinkFeed assumes that the price of BTC is equal to the price of WBTC.

This is the BTC/USD feed:

https://data.chain.link/feeds/ethereum/mainnet/btc-usd

This is the WBTC/BTC feed:

https://data.chain.link/feeds/ethereum/mainnet/wbtc-btc

Note that the WBTC/BTC is not 1:1. Users may not get the actual WBTC price (since protocol is using WBTC instead of BTC) which will affect collaterization ratio and future liquidations.

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L32-L39

### [L-09] The 3 Oracles used in the protocol uses a different denominator for their price, which results in inconsistent price displayed

The protocol uses a price aggregator that aggregates prices from Chainlink, Salty and Uniswap.

The protocol intends to get the price of WBTC and WETH in terms of USD.

In CoreChainlinkFeed, the protocol gets the price of BTC in terms of USD.

```
        // https://docs.chain.link/data-feeds/price-feeds/addresses
        AggregatorV3Interface immutable public CHAINLINK_BTC_USD;
        AggregatorV3Interface immutable public CHAINLINK_ETH_USD;
```


In CoreSaltyFeed, the protocol gets the price of WBTC in terms of USDS.

```
        function getPriceBTC() external view returns (uint256)
                {
        (uint256 reservesWBTC, uint256 reservesUSDS) = pools.getPoolReserves(wbtc, usds);


                if ( ( reservesWBTC < PoolUtils.DUST ) || ( reservesUSDS < PoolUtils.DUST ) )
                        return 0;


                // reservesWBTC has 8 decimals, keep the 18 decimals of reservesUSDS
                return ( reservesUSDS * 10**8 ) / reservesWBTC;
                }
```

In CoreUniswapFeed, the protocol gets the price of WBTC in terms of USDC. (WBTC-WETH-USDC)

```
        constructor( IERC20 _wbtc, IERC20 _weth, IERC20 _usdc, address _UNISWAP_V3_WBTC_WETH, address _UNISWAP_V3_WETH_USDC )
                {
                UNISWAP_V3_WBTC_WETH = IUniswapV3Pool(_UNISWAP_V3_WBTC_WETH);
                UNISWAP_V3_WETH_USDC = IUniswapV3Pool(_UNISWAP_V3_WETH_USDC);

```

By doing so, the protocol assumes that USD,USDC and USDS all have the same price, and WBTC and BTC have the same price.

This is slightly inaccurate since they don't have the same price. WBTC has a different price from BTC.

Also, USDC has a different price from USD, that is why there is a Chainlink oracle for that: 

https://data.chain.link/feeds/ethereum/mainnet/usdc-usd

Lastly, and most importantly, USDS may not be at $1 all the time. There are instances of a collateralized stablecoin depegging (USDR, DOLA, even DAI).

The most accurate feed would be the WBTC-USDC price from Uniswap TWAP. If the protocol wants to be more accurate in the pricing, consider adding a WBTC/BTC price for Chainlink and a USDC/USDS pool for SALT.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L14-L17
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol

### [L-10] Protocol assumes that USDS will always be at $1.

When minting USDS and checking for liquidation, USDS will always be assumed to be $1. 

This is my guess for the assumption: When the price of USDS is more than $1, users will deposit more collateral, mint USDS and swap them for USDC because they will earn money. This swap will eventually bring the price back down to a dollar.

When the price of USDS is less than $1, users will buy more USDS because it is cheaper to pay back their debt now, which will bring the price back up to a dollar.

While this assumption is true, it does not count for flash crashes. If WBTC and WETH happens to drop until most positions becomes undercollaterized, then the protocol will incur a lot of bad debt. In this sense, bad debt doesn't actually refer to a monetary amount, but it refers to USDS being depegged. When positions becomes undercollaterized and before liquidation, users will start selling their USDS instead of repaying their USDS. This will create a downward pressure and since the position is undercollaterized, even if WBTC and WETH is swapped for USDS and burned, it will not help to bring the price of USDS up much. Eventually, the price of USDS will fall and it's extremely hard to bring it back up to its peg.

The last option is to burn the Protocol owned liquidity in order to bring the price of USDS back up, but that might still not be sufficient.

Not sure about the actual recommendation, but one thing the protocol could do is use the USDC value for USDS instead of its own value. Also, it makes sense since the value of WETH and WBTC is calculated in USD/USDC and not USDS.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L68-L77