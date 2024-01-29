## Redundant Function `countryIsExcluded`

The function `countryIsExcluded` only returns the `excludedCountries[country]`, however, since this info could be retrieved directly from `excludedCountries(country)` as excludedCountries is public. The function defined here is redundant. The same works for `Proposals::ballotForID`, `Proposals::lastUserVoteForBallot` and `Proposals::votesCastForBallot`.

Linked Code: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L384-L388
