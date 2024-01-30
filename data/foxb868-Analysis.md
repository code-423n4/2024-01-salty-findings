 Based on reviewing the Salty.IO codebase and architecture, the main objectives of the project appear to be:

1. To build a decentralized exchange (DEX) with zero swap fees and automatic yield generation. 

The core Pools.sol contract implements an automated market maker (AMM) model and arbitrage functionality to generate yield from swap transactions. This aligns incentives by pairing user swaps with arbitrage profits.

2. To offer a native stablecoin (USDS) collateralized by WBTC/WETH.

The USDS.sol stablecoin allows borrowing against WBTC/WETH collateral. The collateral model and incentive structure around USDS aims to maintain its 1:1 peg with USD.

3. To decentralize control of the platform as much as possible.

Salty.IO is governed by a DAO.sol contract where SALT holders vote on protocol changes. No special admin roles or privileges exist. This ensures the platform lacks centralized points of failure.

4. To bootstrap sustainable liquidity and usage of the DEX.

Incentive mechanisms focus on rewarding liquidity providers with emissions and protocol revenue share. Adoption incentives are structured to make Salty.IO self-sustaining. So in summary - zero fee swaps, yield from arbitrage, USDS stablecoin, control decentralization via the DAO, and bootstrapping sustainable platform usage appear to be the main goals behind this project.

## Architecture

Salty.IO divides functionality into contracts reflecting the separation of concerns

```diff
┌─────────────────────────────────┐
│                                 │  
│            DAO.sol              │
│       Governance Logic          │
│                                 │ 
└─────────────────┬───────────────┘
                  │
┌─────────────────┴───────────────┐
│            Pools.sol            │
│         AMM + Arbitrage         │ 
└─────────────────┬───────────────┘
                  │ 
┌─────────────────┴───┬────────────┐
│       Rewards.sol      │         │
│    Incentives and      │         │
│      Distribution       │         │
└─────────────────┬──────┘         │
                  │               │ 
┌─────────────────┴───────────────┤
│       Stablecoin.sol           │
│    USDS Issuance and           │  
│     Collateralization          │
└───────────────────────────────┘
```

**Trust Minimization**

- DAO contract is the only privileged role in Salty - avoiding single points of compromise.
- USDS design depends on collateral health without system ability to liquidate assets in black swan events.

```solidity
function liquidateUndercollateralized
        checkCurrentCollateralRatio() < MIN_COLLATERAL_RATIO 
        liquidateCollateral()
```

- Failure to attract liquidity makes architecture unsustainable.

**Sustainability Incentives**

Salty rewards liquidity with protocol fees: 

```
arbitrageRewards = swapFees + lpRewards 
```

However, this risks circularity where lacking liquidity prevents revenue generation needed to subsidize rewards required to attract capital.

Recommendations:

- Vest protocol revenues to treasury as collateral for USDS until revenue sustainably exceeds costs. 
- Governance control of subsidy ratios.

This aligns sustainability with value captured by the protocol rather than available capital or speculative timing.

**Incremental Decentralization**

- Salty Ambitious innovation deserves methodical, incremental roll out to validate assumptions, incentivizes, and parameters before irreversible decentralization.
- Sandbox environments on testnets, gradually elevating privileges, and feature tiering based on validation are advisable given combination of novel arbitrage, AMM, stablecoin, and governance components.

The **PoolMath.sol** contract performs some key calculations related to token swaps, liquidity pools, and balancing ratios when adding liquidity. I look out for

**Overflow Risks**

- The `_zapSwapAmount` function performs some multiplication and division calculations on token reserve balances and liquidity amounts. These intermediate values could potentially overflow `uint256`.

- The 80-bit normalization via bitshifts helps mitigate this but should be reviewed carefully. Values could still overflow if balances/liquidity grows large enough.

**Precision Loss**

- There is precision loss inherent in the 80-bit normalization. This could cause some loss of precision for very small reserve balances.

- Should validate that 80 bits of precision is sufficient for the scaled down values used in the calculation.

- The reserve balances and liquidity amounts later get stored as 128-bit uints, so no permanent loss of precision at least. 

**Logical Correctness**

- Review the quadratic formula derivation and implementation for correctness. 

- Validate the logical conditions around determining which token amount is in excess and needs to be swapped.

Overall the use of normalization, capping precision loss, and 128-bit reserves helps mitigate most issues. **But overflow and precision impacts should be thoroughly reviewed and tested with extremes. The logical conditions around order and swap amounts also need to be fully validated.**

**PriceAggregator Contract**

- Fetches prices from three price feeds: CoreUniswapFeed, CoreChainlinkFeed, CoreSaltyFeed
- Each feed returns 0 on failure
- Aggregates the two closest price feed values
- Discards aggregate if feed values differ by >3% 

**Price Feed Analysis**

- CoreUniswapFeed uses TWAP from high liquidity Uniswap pools
- CoreChainlinkFeed uses latest RoundData with heartbeat validation 
- CoreSaltyFeed uses internal Salty protocol reserve prices

**So invalid data would result in 0 price for that source. Two valid prices isolates faulty feed. And 3% deviation guardrail ensures prices are aligned.**

**Potential Improvements**

- Modify to check all 3 prices instead of closest two
- Reduce max deviation between sources to 2% 

This would further minimize risk from any single oracle issue.

### Time spent:
17 hours