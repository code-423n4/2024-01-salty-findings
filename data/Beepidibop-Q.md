## [INFO-1] `CollateralAndLiquidity`: Inconsistent Sources for Immutable Assignment in the Constructor

Some immutable states take assignment from `exchangeConfig` while others take assignment from `_exchangeConfig`.

e.g.

```
        usds = _exchangeConfig.usds();

        wbtc = exchangeConfig.wbtc();

        weth = exchangeConfig.weth();
```

It doesn't really matter In practise since `exchangeConfig` is immutable. But it's more professional to stay consistent.

### Recommendation

Change all immutable assignment sources to `_exchangeConfig`.

### Link To Affected Code

https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/stable/CollateralAndLiquidity.sol#L59-L61