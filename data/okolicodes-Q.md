# [L-01] Stuck eth in the manageWallet contract


In the `manageWallet` contract the logic behind the confirmation of proposed wallets is a receive function that `receives` some `amount` of eth
   ```solidity
 receive() external payable
        {
        require( msg.sender == confirmationWallet, "Invalid sender" );

 // Confirm if .05 or more ether is sent and otherwise reject.
     // Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
        if ( msg.value >= .05 ether )
                activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
``` 
The issue here is that there is no way eth could be transferred out since there is no implementation of a `transfer` function which makes it impossible for an `amount` of Eth which will have accumulated as a result of the usage in the contract to be sent out. Thereby leaving it stuck in the contract forever.

## Recommendation
Implement a transfer function so that the eth inside the contract can be transferred out and be utilized.  

# [L-02] Breaking of protocol AMM invariant



## Recommendation

# [L-03] no check for roundID could affect price
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L32C3-L56C13
In the Price Aggregator Contract some important checks are made to the return values of `thelatestRoundData` function.
```solidity
try chainlinkFeed.latestRoundData()
      returns (
	uint80, // _roundID
	 int256 _price,
	 uint256, // _startedAt
	uint256 _answerTimestamp,
uint80 // _answeredInRound
		)
	{
	// Make sure that the Chainlink price update has occurred within its 60 minute heartbeat
	// https://docs.chain.link/data-feeds#check-the-timestamp-of-the-latest-answer
	uint256 answerDelay = block.timestamp - _answerTimestamp;

		if ( answerDelay <= MAX_ANSWER_DELAY )
				price = _price;
	 else
		price = 0;
			}
	catch (bytes memory) // Catching any failure
	{
// In case of failure, price will remain 0
	}
     if ( price < 0 )
	return 0; 
```
However, the `answeredInround` is not checked like they did with the `price`
```solidity
if ( price < 0 )
	return 0;
```

## Recommendation
```solidity
require(answeredInRound >= roundId, "Price stale");
```
# [L-04] Missing check for the max/min price in the `CoreChainlinkFeed.sol` contract
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L32C3-L56C13
The `CoreChainlinkFeed.sol` The contract uses the aggregator v3 to call the latestRoundData. the function should check for the min and max amount return to prevent this type of edge cases:
https://solodit.xyz/issues/m-16-chainlinkadapteroracle-will-return-the-wrong-price-for-asset-if-underlying-aggregator-hits-minanswer-sherlock-blueberry-blueberry-git
## Recommendation
Tho there are three oracles at work, a very unpleasant scenario might present itself with the others and it is wise to brace Up.
Add a check like this to avoid returning the min price or the max price in case of the price crashes.
```solidity
   require(answer < _maxPrice, "Upper price bound breached");
          require(answer > _minPrice, "Lower price bound breached");
```