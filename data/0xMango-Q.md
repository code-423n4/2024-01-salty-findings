## It is possible to use invalid signatures to vote in ```BoostrapBallot.sol```

Signature malleability:
Due to the lack of checks in the ```SigningTools.sol``` contract, given signatures can be modified for invalid ```s``` and ```v```. These signatures if passed into ```SigningTools._verifySignature()``` will still generate the desired public key & have the function call pass.

Affected line in question:
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/SigningTools.sol#L11

As proof of this being possible, this modified signature can be used instead of the provided signature in dev/Deployment.sol ln:120.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dev/Deployment.sol#L120

Modified Signature:
```
hex"0x291f777bcf554105b4067f14d2bb3da27f778af49fe2f008e718328a91cae2f8e1314f4b12e29a3ab940f9fc393caa97141250333d7957bb1d3c1093d96480e11b"
```

Original Signature:
```
hex"291f777bcf554105b4067f14d2bb3da27f778af49fe2f008e718328a91cae2f81eceb0b4ed1d65c546bf0603c6c35567a69c8cb371cf4880a2964df8f6d1c0601c";
```

This signature will successfully sign tx's with the dataHash of ```keccak256(abi.encode(block.chainid, msg.sender));```
Where the chainid is the one from Sepolia & msg.sender being Alice's public key.

Solution:
Using OZ ECDSA.sol contract, which has the proper error handling for invalid signatures and ensures for proper lower ```s``` values.

## Uniswapv3's TWAP could potentially be manipulated to cause a DOS or price manipulation

Uniswap's v3 TWAP oracle can be manipulated by validators with high stake.
Due to the nature of Uniswapv3's geometric mean, the price change needs to occur multiple times, to eventually affect the resulting price feed.
Source: 
https://chainsecurity.com/oracle-manipulation-after-merge/
Unfortunately, the amount of times might not have to be that many. To write an observation in UniswapV3 pools, users must pay extra gas, to make the addition to the array holding the historic price data. Because of this, the TWAP is only maintained by users who actively pay this fee. This participation can be very frequent in some pools while non-existent in others. So the feasibility of this attack boils down to the rate of legitimate oracle observations being done against the malicious ones.
Because of this, even the Uniswap team themselves, advises not to rely heavily on their TWAP oracle.(Even v3)

Potential denial of service:

Since the ```PriceAggregator.sol``` contract looks out for an initial 3% difference between prices from the different feeds, else throwing an error.
The UniswapV3 TWAP would only have to be changed this much, to make its participation invalid. If the protocol is in its infancy and little ```usds``` has been borrowed thus far, the ```CoreSaltyFeed.sol``` will also be unreliable.
This could finally lead to functions in ```CollateralAndLiquidity.sol``` to fail, due to the ```PriceAggregator.getPriceETH/BTC()``` failing.

Potential price manipulation leading to wrongful liquidations:

Lets assume the protocol is in a more mature state. A decent amount in ```usds``` has been borrowed & decent dept exists in both ```usds/wtbc``` and ```usds/weth``` pools. Now under these conditions, lets further assume that the potential for discrepancy in ```PriceAggregator.sol``` was changed to 7% by the DAO.
An attack here would involve a UniswapV3 manipulation, slowly changing the price down a 7%. Then in 1 transaction change either the ```usds/weth``` or ```usds/wbtc``` pool, whichever has more dept in ```weth/wbtc```. Now the average price returned by ```PriceAggregator._aggregatePrices()``` will lean towards a price that is 7% off the current spot price, potentially leading to unwanted liquidations. 

Solution:

A solid solution to Oracle-based protocols is the one GMX V2 came up with. Where transactions by users are stored in a queue, to then be relayed by their front-end controlled wallet. In case of GMX this was to prevent sandwich attacks, but for Salty.IO this could present the opportunity to use an off-chain oracle, who is garanteed to have correct price feeds. Also having an js/ts script being executed before executing the desired tx given by the user, could also allow for Salty.IO to check potential UniswapV2/V3 pools for further arbitrage opportunities.