1. In the Doc, it was said that 45% of remaining salt after 10 % transferred to the Team will be burnt. This is seen in the test suites (DAO.t.sol)

However, It appears that in the Dao.sol:processRewardsFromPOL(), 50% burn was implemented after the 10% transfer to team

This signal some form of discrepancies

References:

Test: 
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/tests/DAO.t.sol#L872

DAO contract: 
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L348


2. The CoreSaltyFeed.sol contract uses a uniswapv2 spot price in the getPriceBTC() and getPriceETH() respectively. It is commonly known that this is susceptible to a flashloan manipulation attacks. Since there is an alternative with the TWAP of uniswapv3, this should be removed or not used at all

References;

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreSaltyFeed.sol#L32

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreSaltyFeed.sol#L45

3. The CoreUniswapFeed.sol contract implementing the TWAP price is also susceptible to manipulation. Read about it here: https://github.com/euler-xyz/uni-v3-twap-manipulation/blob/master/cost-of-attack.pdf

It is recommended to use some sort of decentralized price oracle like Tellor alongside chainlink as a safenet