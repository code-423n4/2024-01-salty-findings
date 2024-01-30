## \[G‑01\] Save gas by preventing zero amount in  burn()

```
_burn( address(this), balance );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/USDS.sol#L56

```
_burn( address(this), balance );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Salt.sol#L28

## \[G-02\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

```
if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
```

&nbsp;https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L170

```
uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L246

```
withdrawnAmount = weth.balanceOf( address(this) );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L307

```
uint256 saltBalance = salt.balanceOf(address(this));
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L63

```
tokenA.safeTransferFrom(msg.sender, address(this), addedAmountA );
                tokenB.safeTransferFrom(msg.sender, address(this), addedAmountB );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L161C3-L163C1

&nbsp;

```
swapTokenIn.safeTransferFrom(msg.sender, address(this), swapAmountIn );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L385

```
tokenA.safeTransferFrom(msg.sender, address(this), maxAmountA );
                tokenB.safeTransferFrom(msg.sender, address(this), maxAmountB );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L88C3-L89C67

## \[G-03\] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L302

```
require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L297

```
require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L124

```
require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L58

```
require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L49

```
72: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );

142: require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L72

```
require(msg.sender == address(this), "Only callable from within the same contract");
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L97

## <ins>\[G-04\] Events are not indexed</ins>

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

```
event BootstrappingRewardsChanged(uint256 newBootstrappingRewards);
    event PercentPolRewardsBurnedChanged(uint256 newPercentPolRewardsBurned);
    event BaseBallotQuorumPercentChanged(uint256 newBaseBallotQuorumPercentTimes1000);
    event BallotDurationChanged(uint256 newBallotDuration);
    event RequiredProposalPercentStakeChanged(uint256 newRequiredProposalPercentStakeTimes1000);
    event MaxPendingTokensForWhitelistingChanged(uint256 newMaxPendingTokensForWhitelisting);
    event ArbitrageProfitsPercentPOLChanged(uint256 newArbitrageProfitsPercentPOL);
    event UpkeepRewardPercentChanged(uint256 newUpkeepRewardPercent);
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L11C5-L18C70

&nbsp;https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L25C2-L38C110

## \[G-05\] Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.

&nbsp;

```
string public websiteURL;

    // Countries that have been excluded from access to the DEX (used by AccessManager.sol)
    // Keys as ISO 3166 Alpha-2 Codes
    mapping(string=>bool) public excludedCountries;
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L63C2-L67C49

```
event ProposalCreated(uint256 indexed ballotID, BallotType ballotType, string ballotName);
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L23C1-L24C1

&nbsp;

## \[G-06\] Duplicated require()/if() checks should be refactored to a modifier or function

```
File:  src/dao/DAO.sol

297: 		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );


318: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );

329: 		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );
```

&nbsp;https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L297

## \[G-07\] Make 3 event parameters indexed when possible

it’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows for more efficient searching of events.

[Reference](https://code4rena.com/reports/2023-01-drips#g-05-make-3-event-parameters-indexed-when-possible)

```
event LiquidityDeposited(address indexed user, address indexed tokenA, address indexed tokenB, uint256 amountA, uint256 amountB, uint256 addedLiquidity);
    event LiquidityWithdrawn(address indexed user, address indexed tokenA, address indexed tokenB, uint256 amountA, uint256 amountB, uint256 withdrawnLiquidity);
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L20C3-L22C1

&nbsp; https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L25C2-L38C110

## \[G-08\] Use constants instead of type(uintx).max

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```

In the above example, we have a contract with a constant MAX\_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX\_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```
salt.approve(address(collateralAndLiquidity), type(uint256).max);
                usds.approve(address(collateralAndLiquidity), type(uint256).max);
                dai.approve(address(collateralAndLiquidity), type(uint256).max);
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L90C3-L92C67

&nbsp;

```
require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L165

```
uint64 constant INVALID_POOL_ID = type(uint64).max;
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L14C1-L15C1

```
salt.approve(address(stakingRewards), type(uint256).max);
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L50

```
salt.approve( address(stakingRewardsEmitter), type(uint256).max );
        salt.approve( address(liquidityRewardsEmitter), type(uint256).max );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/SaltRewards.sol#L41C3-L42C71

```
wbtc.approve( address(pools), type(uint256).max );
                weth.approve( address(pools), type(uint256).max );
                dai.approve( address(pools), type(uint256).max );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/Liquidizer.sol#L71C3-L73C52

```
File: src/ManagedWallet.sol

37: activeTimelock = type(uint256).max;

68: activeTimelock = type(uint256).max;

86: activeTimelock = type(uint256).max;
```

&nbsp;https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L37

## \[G-09\] Pre-calculation Boosts Efficiency

```
uint256 public baseBallotQuorumPercentTimes1000 = 10 * 1000;
```

If you pre-calculate the value of `10 * 1000` and assign it directly to the variable, it will save gas during contract deployment and execution because the computation is done at compile-time rather than runtime.

&nbsp;https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L40

```
if (baseBallotQuorumPercentTimes1000 < 20 * 1000) <@
                baseBallotQuorumPercentTimes1000 += 1000;
            }
        else
            {
            if (baseBallotQuorumPercentTimes1000 > 5 * 1000 ) <@
```

&nbsp;https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L102C1-L107C53

&nbsp;

```
uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L90

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L324C3-L329C104

```
return uint256(price) * 10**10;
```

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price\\\_feed/CoreChainlinkFeed.sol#L59](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price%5C_feed/CoreChainlinkFeed.sol#L59)

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price\\\_feed/CoreUniswapFeed.sol#L70C4-L72C50](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price%5C_feed/CoreUniswapFeed.sol#L70C4-L72C50)

## **\[G-10\]** Compute known value `off-chain`

If You know what data to hash, there is no need to consume more computational power to hash it using keccak256 , you’ll end up consuming 2x amount of gas.

```
bytes32 nameHash = keccak256(bytes( ballot.ballotName ) );


                if ( nameHash == keccak256(bytes( "setContract:priceFeed1_confirm" )) )
                        priceAggregator.setPriceFeed( 1, IPriceFeed(ballot.address1) );
                else if ( nameHash == keccak256(bytes( "setContract:priceFeed2_confirm" )) )
                        priceAggregator.setPriceFeed( 2, IPriceFeed(ballot.address1) );
                else if ( nameHash == keccak256(bytes( "setContract:priceFeed3_confirm" )) )
                        priceAggregator.setPriceFeed( 3, IPriceFeed(ballot.address1) );
                else if ( nameHash == keccak256(bytes( "setContract:accessManager_confirm" )) )
                        exchangeConfig.setAccessManager( IAccessManager(ballot.address1) );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L133C3-L142C71

&nbsp;

## \[G-11\] Amounts should be checked for 0 before calling a transfer

It is considered a best practice to verify for zero values before executing any transfers within smart contract functions. This approach helps prevent unnecessary external calls and can effectively reduce gas costs.

This practice is particularly crucial when dealing with the transfer of tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can determine whether a value is zero by utilizing the == operator. Here's an example demonstrating how to check for a zero value before performing a transfer:

```
Copy code
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

In the provided example, we ensure that the amount parameter is greater than zero before executing the transfer to the designated recipient address. If the amount is zero or negative, the function will revert, preventing the transfer from taking place.

&nbsp;

&nbsp;

```
tokenA.safeTransferFrom(msg.sender, address(this), addedAmountA );
                tokenB.safeTransferFrom(msg.sender, address(this), addedAmountB );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L161C3-L163C1

```
_userDeposits[msg.sender][token] += amount;
```

&nbsp;https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L209

&nbsp;

```
swapTokenIn.safeTransferFrom(msg.sender, address(this), swapAmountIn );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L385

## \[G-12\] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case, we are also able to efficiently store the function signatures together in memory as one word, saving multiple MLOADs in the process.

```
tokenA.safeTransfer( address(liquidizer), reclaimedA ); 
tokenB.safeTransfer( address(liquidizer), reclaimedB );
```

&nbsp;https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L375

```
poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.wbtc() );
                        poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.weth() );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L160C1-L161C89

```
poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
            poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

            bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
            bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L254C4-L258C88

&nbsp;

```
tokenA.safeTransferFrom(msg.sender, address(this), addedAmountA );
                tokenB.safeTransferFrom(msg.sender, address(this), addedAmountB );
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L161C3-L163C1

```
saltRewards.stakingRewardsEmitter().performUpkeep(timeSinceLastUpkeep);
                saltRewards.liquidityRewardsEmitter().performUpkeep(timeSinceLastUpkeep);
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L209C3-L210C76

&nbsp;

## \[G-13\] OR in if-condition can be rewritten to two single if conditions

Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

Reference : <ins>https://code4rena.com/reports/2023-08-shell#g-05-or-in-if-condition-can-be-rewritten-to-two-single-if-conditions</ins>

```
107: if ( reservesA0 <= PoolUtils.DUST || reservesA1 <= PoolUtils.DUST || reservesB0 <= PoolUtils.DUST || reservesB1 <= PoolUtils.DUST || reservesC0 <= PoolUtils.DUST || reservesC1 <= PoolUtils.DUST )
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L107

&nbsp;

## \[G-14\] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```
Ballot memory ballot = ballots[ballotID];
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L261

### \[G-15\] Optimize gas usage by avoiding variable declarations inside loops

The variables are all declared inside the loop. This means that a new instance of each variable will be created for each iteration of the loop. Saves `200-300 Gas` per iteration.

```
uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
                        uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L426C4-L427C61

&nbsp;