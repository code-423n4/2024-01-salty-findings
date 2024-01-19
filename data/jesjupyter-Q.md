## Redundant Function `countryIsExcluded`

The function `countryIsExcluded` only returns the `excludedCountries[country]`, however, since this info could be retrieved directly from `excludedCountries(country)` as excludedCountries is public. The function defined here is redundant.

