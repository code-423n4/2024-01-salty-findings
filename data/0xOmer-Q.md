## Missing check for wallets to not be the same addresses
`ManagedWallet` contract introduces 2 wallets `mainWallet` and `confirmationWallet`. It is possible to pick them as same addresses. Since they have different roles they must not be same. Consider adding relevant checks both in `constructor` and `proposeWallets()` to block the logic fail.


## ProposedConfirmationWallet can be overwritten
According to the [comment](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L48), overwriting a proposal should not allowed. Though the mentioned check exist for [proposedMainWallet](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L49), there isn't for `proposedConfirmationWallet`.

## `approve()` should be replaced with `safeIncreaseAllowance()` / `safeDecreaseAllowance()`
`approve` is subject to known front-running attacks and does not handle non-standard ERC20 behaviors. Consider using `safeIncreaseAllowance` and `safeDecreaseAllowance` instead:

```ruby
src/staking/Liquidity.sol:
   86          if (swapAmountA > 0) {
   87:             tokenA.approve(address(pools), swapAmountA);
   88  

  100          else if (swapAmountB > 0) {
  101:             tokenB.approve(address(pools), swapAmountB);
  102  

  154          // Approve the liquidity to add
  155:         tokenA.approve(address(pools), maxAmountA);
  156:         tokenB.approve(address(pools), maxAmountB);
  157  
```
Keep in mind that safeApprove() is deprecated by OZ and recommended using their `safeIncreaseAllowance()` and `safeDecreaseAllowance()`

See this discussion: [SafeERC20.safeApprove() Has unnecessary and unsecure added behavior](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2219)

**NOTE:** In bot report this issue is mentioned but it recommends to use `safeApprove()` which is deprecated and not safe to use.


## Missing zero address check in [`PriceAggregator.setPriceFeed()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L47)
It should exist to prevent any undesired outcome. Consider adding: 
`require(
            address(newPriceFeed) != address(0),
            "newPriceFeed address can not be zero"
        );
`