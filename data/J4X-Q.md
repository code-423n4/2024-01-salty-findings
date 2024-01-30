# Low

## [L-01] Reserves Above DUST Incorrectly Enforced
[Pools Line 270](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L271)

**Issue Description:**

In the Salty protocol's `_adjustReservesForSwap()` function, the intention, as per the accompanying comment ("// Make sure that the reserves after swap are above DUST"), is to ensure that the reserves always remain above a certain threshold defined by the `DUST` constant. However, the current implementation of this check uses `>=` (greater than or equal to), which leads to an off-by-one error. The actual requirement, as intended, is for the reserves to be strictly greater than `DUST`.

Current implementation:
```solidity
require( (reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves after swap");
```

This condition allows the reserves to be exactly equal to `DUST`, which contradicts the intended logic of keeping them above `DUST`.

**Recommended Mitigation Steps**

To align the implementation with the intended behavior, the `require` statement should be adjusted to enforce that reserves are strictly greater than `DUST`. This change will ensure that the reserves are maintained above the minimal threshold, as originally planned.

Proposed change:
```solidity
require( (reserve0 > PoolUtils.DUST) && (reserve1 > PoolUtils.DUST), "Insufficient reserves after swap");
```

By modifying the condition to `>` (greater than), the function will correctly enforce that the reserves are maintained above the `DUST` level, thus adhering to the protocol's intended safety and operational standards.

## [L-02] Liquidity can still be below DUST after a call to `addLiquidity()`
[Pools Line 146-147](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L146-L147)

**Issue Description:**

n the Salty protocol's `addLiquidity()` function, there's an enforcement to ensure the maximum liquidity added by a user is above the `PoolUtils.DUST` threshold. However, a potential issue arises because the actual liquidity added to the pool may end up being less than `PoolUtils.DUST`. This discrepancy occurs due to the adjustment of the added liquidity based on the current exchange rate within the `_addLiquidity()` call. Consequently, a pool might end up with liquidity below the `DUST` level, conflicting with other contract checks that assume liquidity will always exceed this minimum.

**Recommended Mitigation Steps**

To resolve this issue, the validation in the `addLiquidity` function should be based on the actual amounts of liquidity added (`addedAmountA` and `addedAmountB`), rather than the maximum amounts (`maxAmountA` and `maxAmountB`) initially specified by the user. By checking the `addedAmountA` and `addedAmountB` for adherence to the `DUST` threshold, the protocol can ensure that the liquidity in the pool always remains above this critical level, maintaining consistency with the protocol's rules and intentions.

## [L-03] `proposeCountryInclusion()` does not check for ASCII
[Proposals Line 222](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L222)

**Issue Description:**
In the Salty DAO's `proposeCountryInclusion()` function, there's an existing validation step to ensure the country codes are in the ISO 3166 Alpha-2 format, which consists of two uppercase ASCII characters. While the function includes a `require` statement to check the length of the country code, it doesn't verify if the provided bytes are indeed uppercase ASCII characters:

```solidity
require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );
```

This lack of validation means that any two-byte input, regardless of its content, can be passed as a country code, potentially leading to invalid or misleading entries.

**Recommended Mitigation Steps**

To address this issue, additional validation should be implemented to confirm that each character in the country code falls within the range of uppercase ASCII letters (`0x41` to `0x5A`). This ensures that only valid ISO 3166 Alpha-2 country codes are accepted. The validation can be integrated into the existing function with checks for each character of the country code:

```solidity
require(bytes(country)[0] >= 0x41 && bytes(country)[0] <= 0x5A && bytes(country)[1] >= 0x41 && bytes(country)[1] <= 0x5A, "Invalid country code");
```

By incorporating these character checks, the Salty DAO will accurately enforce the ISO 3166 Alpha-2 standard for country codes, enhancing the integrity of the country inclusion proposal process.

## [L-04] `proposeCountryInclude` does not check if the country is currently excluded`
[Proposals Line 222](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L222)

**Issue Description:**
The `proposeCountryInclude` function in the Salty DAO is designed to allow proposals for removing a country from the blacklist. However, the current implementation does not include a check to verify whether the specified country is already on the blacklist. This oversight means that proposals could be made and potentially approved for countries that are not currently blacklisted, leading to unnecessary and potentially confusing governance actions.

**Recommended Mitigation Steps**

To rectify this, an additional validation step should be incorporated into the `proposeCountryInclude` function to ensure that the country in question is indeed currently excluded (i.e., on the blacklist). This can be done by checking the blacklist status of the country within the DAO:

```solidity
require(dao.isExcluded(country), "country is not excluded");
```

Adding this `require` statement ensures that proposals for including a country are only possible if that country is currently excluded. This check maintains the integrity and relevance of the governance process, ensuring that the DAO's efforts are focused on meaningful and necessary changes.

## [L-05] `proposeCountryExclusion()` does not check for ASCII
[Proposals Line 222](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L222)

**Issue Description:**
The `proposeCountryExclusion()` function in the Salty DAO is designed for proposals to add countries to the blacklist using their ISO 3166 Alpha-2 codes, which consist of two uppercase ASCII characters. While the function checks that two bytes are provided, it does not verify if these bytes represent valid uppercase ASCII characters, potentially allowing for invalid country codes.

```solidity
require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );
```

**Recommended Mitigation Steps**

To ensure that only valid ISO 3166 Alpha-2 country codes are proposed for blacklisting, the function should include additional checks to confirm that each character is within the ASCII range for uppercase letters (`0x41` to `0x5A`). This validation ensures that the input conforms to the correct format for country codes:

```solidity
require(bytes(country)[0] >= 0x41 && bytes(country)[0] <= 0x5A && bytes(country)[1] >= 0x41 && bytes(country)[1] <= 0x5A, "Invalid country code");
```

Implementing these character checks will ensure the accuracy and validity of country codes proposed for exclusion, thereby upholding the integrity of the DAO's blacklist management process.

## [L-06] `proposeWebsiteUpdate()` does not check for a valid URL
[Proposals Line 249](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L249)

**Issue Description:**

In the Salty DAO, the `proposeWebsiteUpdate()` function facilitates the voting process for changes to the website. However, the function currently lacks validation for the input URL, meaning it does not verify whether the provided string is a valid URL starting with "http://" or "https://". Additionally, it does not check if the rest of the string consists of valid ASCII characters. This oversight could lead to the acceptance of invalid or improperly formatted URLs.

**Recommended Mitigation Steps**

To ensure that only valid URLs are proposed for website updates, the function should include checks to confirm:

1. The URL begins with "http://" or "https://".
2. All subsequent characters in the URL string fall within the valid ASCII range (`0x2D` to `0x7A`).

The implementation could first verify the URL's prefix and then loop through the rest of the string to check each character:

```solidity
string memory httpPrefix = "http://";
string memory httpsPrefix = "https://";
require(hasPrefix(url, httpPrefix) || hasPrefix(url, httpsPrefix), "URL must start with 'http://' or 'https://'");

for (uint i = 0; i < bytes(url).length; i++) {
    require(bytes(url)[i] >= 0x2D && bytes(url)[i] <= 0x7A, "Invalid character in URL");
}

// 'hasPrefix' is a helper function to check if the URL starts with the specified prefix
```

By adding these checks, the Salty DAO will ensure that only properly formatted and valid URLs are considered for website updates, enhancing the protocol's governance and operational integrity.

##  [L-07] `requiredXSalt` can be zero
[Proposals Line 92](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L92)

**Issue Description:**

In the `_possiblyCreateProposal()` function of the Salty DAO, which is called when a user proposes a proposal, there's a calculation to determine the required amount of xSalt (staked SALT) needed:

```solidity
// Make sure that the sender has the minimum amount of xSALT required to make the proposal
uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

require( requiredXSalt > 0, "requiredXSalt cannot be zero" );
```

However, an edge case exists where `requiredXSalt` can be zero. This situation occurs when the total staked SALT is below 50, and `requiredProposalPercentStakeTimes1000` is set to 2% (2000). The resulting calculation would be `49 * 2000 / 100000`, which equals zero due to roundign down, thereby preventing any proposals from being created even though SALT is staked.

**Recommended Mitigation Steps**

To address this issue, an additional check should be added to handle cases where `requiredXSalt` computes to zero while SALT is staked. In such scenarios, `requiredXSalt` should default to a minimum value (e.g., 1), enabling proposals as long as the user has staked at least 1 SALT. This change ensures that the ability to create proposals is not inadvertently blocked due to the specificities of the calculation in edge cases:

```solidity
if (totalStaked > 0 && requiredXSalt == 0) { requiredXSalt = 1; }
```

Incorporating this logic will maintain the functionality of the proposal system, particularly in low liquidity conditions, ensuring that the governance process remains accessible and fair.

## [L-08] First staker can claim all rewards from period before
[StakingRewards Linhe 78](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L78)

**Issue Description:**

In the Salty protocol's `StakingRewards` contract, there is an issue related to the distribution of rewards when no stakers are present. When rewards are distributed during a period with no stakers and a user subsequently stakes SALT, the implementation of `_increaseUserShare()` results in the first staker claiming all the rewards accumulated during the staker-less period:`

```solidity
// Determine the amount of virtualRewards to add based on the current ratio of rewards/shares.
// The ratio of virtualRewards/increaseShareAmount is the same as totalRewards/totalShares for the pool.
// The virtual rewards will be deducted later when calculating the user's owed rewards.
if ( existingTotalShares != 0 ) // prevent / 0
	{
	// Round up in favor of the protocol.
	uint256 virtualRewardsToAdd = Math.ceilDiv( totalRewards[poolID] * increaseShareAmount, existingTotalShares );

	user.virtualRewards += uint128(virtualRewardsToAdd);
	totalRewards[poolID] += uint128(virtualRewardsToAdd);
	}

// Update the deposit balances
user.userShare += uint128(increaseShareAmount);
totalShares[poolID] = existingTotalShares + increaseShareAmount;

emit UserShareIncreased(wallet, poolID, increaseShareAmount);
}
```

The current logic is designed to distribute virtual rewards proportionally among stakers. However, to prevent division by zero, the function inadvertently allocates all accumulated rewards to the first staker in cases where they are the first to stake after a period with no stakers.

**Recommended Mitigation Steps**

To address this issue, the reward distribution mechanism should be adjusted to withhold rewards during periods with no stakers. The rewards accumulated during these periods should be retained by the rewards distributor and only begin to be distributed again once stakers are present. This change ensures that rewards are fairly allocated among stakers and not disproportionately claimed by a single user due to a timing advantage.