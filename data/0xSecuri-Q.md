### [Non-Critical](#non-critical)

|  Count  | Title                                                     | Context |
| :-----: | :-------------------------------------------------------- | ------- |
| [NC-01] | Misleading comment in `_usersThatProposedBallots` mapping | 1       |
| [NC-02] | Event names do not follow convention                      | 2       |
| [NC-03] | Incorrect visibility of `_getUniswapTwapWei` function     | 1       |

| Total Non-Critical Issues |  3  |
| :-----------------------: | :-: |

## [NC-01] Misleading comment in `_usersThatProposedBallots` mapping

### Description

In the `Proposals.sol`, there's a misleading comment regarding the usage of `_usersWithActiveProposals` instead of `_userHasActiveProposal`.

### Recommendation

The comment should be updated to accurately reflect the variable name `_userHasActiveProposal` instead of `_usersWithActiveProposals`.

[View the code snippet in context](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L62)

## [NC-02] Event names do not follow convention

### Description

In the provided code snippets, event names such as `maximumInternalSwapPercentTimes1000Changed` and `incrementedBurnableUSDS` do not adhere to the convention of starting with an uppercase letter.

### Recommendation

Update the event names to start with an uppercase letter to follow naming conventions. According to the [Solidity documentation](https://docs.soliditylang.org/en/latest/style-guide.html#event-names), event names should be capitalized.

#### Examples

- [View the first code snippet in context](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L16)
- [View the second code snippet in context](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/Liquidizer.sol#L23)

## [NC-03] Incorrect visibility of `_getUniswapTwapWei` function

### Description

The `_getUniswapTwapWei` function in the `CoreUniswapFeed.sol` has public visibility. However, from the underscore in front of its name and the comment above the [`getUniswapTwapWei`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreUniswapFeed.sol#L76) function, we understand that it is intended to be used internally.

### Recommendation

Update the visibility of the `_getUniswapTwapWei` function to internal or private.

[View the code snippet in context](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreUniswapFeed.sol#L50)