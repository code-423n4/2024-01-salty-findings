## [G-01] Optimizing The excludedCountries mapping in DAO.sol

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L67

The excludedCountries mapping uses strings as keys. The set of country codes is limited and known, consider using an enum or a smaller data type like bytes2 for country codes to optimize storage.

To optimize the excludedCountries mapping in your Solidity contract, you can replace the use of strings as keys with a more gas-efficient approach. Since country codes are typically standardized and have a fixed length, using a smaller data type such as bytes2 or an enum can be more efficient. 

**Here's how you can apply this change:**

###### Using bytes2 for Country Codes


**1 - Change the Key Type in Mapping**

Instead of using string for country codes, use bytes2. This type is sufficient to hold standard ISO 3166-1 alpha-2 country codes, which are two characters long.

>mapping(bytes2 => bool) public excludedCountries;

**2 - Modify Access and Assignment**

When accessing or assigning values in the excludedCountries mapping, convert the country codes to bytes2. This can be done using explicit type conversion.

>// Setting a country as excluded
excludedCountries[bytes2('US')] = true;
//...
// Checking if a country is excluded
bool isExcluded = excludedCountries[bytes2(countryCode)];

###### Using an enum for Country Codes:

**An enum can be even more efficient**

1 - Define an enum with all the country codes:

>enum CountryCode { US, CA, GB, CN, IN, PK, RU, AF, CU, IR, KP, SY, VE }

2 - Change the Mapping:

Use the enum as the key type in the mapping.

>mapping(CountryCode => bool) public excludedCountries;

3 - Access and Assignment:

>// Setting a country as excluded
excludedCountries[CountryCode.US] = true;
//...
// Checking if a country is excluded
bool isExcluded = excludedCountries[CountryCode.US];

###### General Considerations

Gas Costs: Using bytes2 or an enum can reduce the gas cost associated with storage and access of the mapping, as they occupy less space than strings.

Maintainability: If the set of countries might change or expand in the future, consider the ease of maintaining these changes with enum versus bytes2.

By implementing these changes, the excludedCountries mapping in your contract should become more gas-efficient, contributing to overall optimization.

## [G-02] Multiplication/Division By Two Should Use Bit Shifting

x * 2 is equal to x << 1 and x / 2 is equal to x >> 1. The MUL and DIV opcodes cost 5 gas, whereas SHL and SHR only cost 3 gas.

>uint256 averagePrice = ( priceA + priceB ) / 2;

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L139

>require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L247

>exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L265

## [G-03] Redundant Array Initialization

Declaring Arrays in Solidity by default assigns all the elements to 0 values therefore any attempt to overwrite them again with 0 is redundant.

>secondsAgo[1] = 0; // to (now)

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L54

## [G-04] Code Optimization By Using Max and Min

The contract was using the expression 2**128 to calculate the maximum storage capacity of the data type. This both lacks readability and costs more gas.

This can be replaced by type(uint).max; or type(uint).min; which is more readable and will also save some gas.

>Example: type(uint128).max

However, keep in mind that type(uint128).max; is equal to 2**128 - 1

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L119-L126

>function _mostSignificantBit(uint256 x) internal pure returns (uint256 msb){
        if (x >= 2**128) { x >>= 128; msb += 128; }
        if (x >= 2**64) { x >>= 64; msb += 64; }
        if (x >= 2**32) { x >>= 32; msb += 32; }
        if (x >= 2**16) { x >>= 16; msb += 16; }
        if (x >= 2**8) { x >>= 8; msb += 8; }
        if (x >= 2**4) { x >>= 4; msb += 4; }
        if (x >= 2**2) { x >>= 2; msb += 2; }
        if (x >= 2**1) { x >>= 1; msb += 1; }
}

## [G-05] Variables Declared But Never Used

The contract PoolUtils has declared a variable STAKED_SALT but it is not used anywhere in the code. This represents dead code or missing logic.

Unused variables increase the contract's size and complexity, potentially leading to higher gas costs and a larger attack surface.

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L16

>bytes32 constant public STAKED_SALT = bytes32(0);

## [G-06] Unused Imports

Having unused code or import statements incurs extra gas usage when deploying the contract.

>import "../staking/interfaces/IStaking.sol";
>import "../Upkeep.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L11
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L18

>import "./rewards/interfaces/IRewardsEmitter.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L6

>import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L6

>import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
>import "openzeppelin-contracts/contracts/utils/math/Math.sol";
>https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L3
>https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L5

>import "../interfaces/ISalt.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L6

>import "openzeppelin-contracts/contracts/utils/math/Math.sol";
>import "./PoolMath.sol";
>https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L6
>https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L13

>import "./interfaces/IDAO.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L14

>import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
>import "./interfaces/IPools.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L3
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L5

>import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L5

## [G-07] Superfluous Event Fields

block.timestamp and block.number are by default added to event information. Adding them manually costs extra gas.

>emit POLWithdrawn(tokenA, tokenB, reclaimedA, reclaimedB);

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L378
