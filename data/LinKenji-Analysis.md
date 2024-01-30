## Project Summary

Salty.IO is building a decentralized exchange that aims to offer zero fee swaps, yield generation from automatic arbitrage, and a native stablecoin called USDS collateralized by WBTC/WETH.

The system is governed by a DAO where SALT token holders vote on parameters and upgrades. No special admin access exists. Pools.sol implements the AMM DEX logic with arbitrage done atomically at swap time to yield profits. USDS.sol manages stablecoin collateral ratios, liquidations, and redemptions.

Incentives focus on distributing arbitrage yields to liquidity providers and SALT stakers to drive adoption. The goal is ultimately a self-sustaining platform lacking centralized points of failure.

**Key Risks**

**Liquidity Bootstrapping:** As an AMM, Salty.IO requires external liquidity to function efficiently. Without incentives attracting liquidity in a circular way, high swap costs could prevent profitability.

**Technical:** The smart contract system is complex. Subtle logical errors or bugs could disrupt core functionality around swaps, yields, or the USDS peg. Rigorous testing is critical.

**Price Manipulation:** Asset valuations originate from external Chainlink and Uniswap price oracles. Market volatility could be amplified by manipulated oracles. 

**Governance:** The DAO introduces risks around proposal attacks, improper parameter changes, and funds mismanagement. Timelocks and process controls are important.

**Summary**

Salty.IO brings innovative architecture but has complexity risks. Prioritizing liquidity incentives and strong technical audit practices will help mitigate issues in core areas like AMM swaps, price oracles, and governance.

The core price feed contracts are `PriceAggregator.sol` and its associated `IPriceFeed` implementations (`CoreUniswapFeed`, `CoreChainlinkFeed` etc.) The main vulnerabilities stem from:

**Centralized Price Feeds**

If an attacker compromises the private keys of the admin or DAO governing the price feed contracts, they could switch the underlying oracles to a malicious one. For example:

```solidity
function setPriceFeed(uint id, IPriceFeed feed) onlyOwner external {
  // Set manipulated feed 
}
```

This would allow manipulating BTC and ETH prices.

**Oracle Manipulation**

Even without keys, manipulating the existing Uniswap TWAP or Chainlink feeds via flash loans could provide false arbitrage and liquidation opportunities. For example spamming the DEX pools prior to snapshot timestamps.

**Asset Correlation** 

The Uniswap and Chainlink oracles rely on proper USD valuation of their quote assets like WBTC, WETH etc. If stable or synthetic assets like USDT/USDC are manipulated, so would be the BTC/ETH prices.


**Mitigations**

- Use a decentralized oracle aggregator like DIA instead of a single one
- Add protections against sudden price deviations  
- Monitor mempool for transaction spam around timestamps
- Ensure diversity of price quote assets 

### The automated arbitrage mechanism in Salty.IO could be vulnerable to frontrunning exploits if privileged actors can predict upcoming opportunities. Let me analyze the arbitrage code flow:

The key arbitrage calculations occur in `ArbitrageSearch.sol` which is called from `Pools.sol` on every swap event. 

When a user swap between two tokens occurs, `_attemptArbitrage()` is triggered:

```solidity
function swap(
  IERC20 tokenIn, 
  IERC20 tokenOut,
  uint amountIn
) external {

  // Place user swap

  uint arbitrageProfit = _attemptArbitrage(
    tokenIn, 
    tokenOut,
    amountIn 
  );

}

function _attemptArbitrage(
  IERC20 tokenIn,
  IERC20 tokenOut,
  uint amountIn  
) internal returns (uint profit) {

  // Calculate and perform arbitrage
}
```

The issue lies in the sequential order - the user swap occurs first, then arbitrage is checked.

This means miners or mempool frontrunners aware of pending swaps could take the following actions:

1. Anticipate user swap about to occur
2. Frontrun - take similar swap first
3. Gain priority in arbitrage profits

Even if the user swap reverts, the frontrunner keeps any profits.

**Mitigations** involve:

- Flipping order - arbitrage first, then user swap 
- Pool partitioning amongst swappers
- Mempool manipulation protections

## Liquidation Vulnerabilities

The key liquidation entry point is the `liquidateUser()` function. An attacker could potentially manipulate state before calling this in several ways:

**Frontrunning Legitimate Liquidations**

If an attacker sees a pending TX to liquidate an undercollateralized position, they could attempt to frontrun it and liquidate first. This would steal some profits and prevent the proper liquidation.

**Manipulating Oracle Prices** 

If an attacker temporarily pushes collateral asset prices much lower, solvent positions would falsely appear undercollateralized. This could enable unwarranted liquidations.

**Boosting Rewards Before Liquidating**

An attacker could temporarily inflate a user's collateral and rewards share, then rapidly borrow USDS to make them eligible for liquidation. This could allow stealing their pending rewards and share.

**Other Transaction Order Front-Running**

General frontrunning of transactions around liquidations could allow attackers to profit from side effects.

## Preventative Checks

However, `CollateralAndLiquidity` contains some key mitigating checks:

- `canUserBeLiquidated()` prevents unwarranted liquidation
- Usage of `nonReentrant` modifier prevents reentrancy issues 
- Only allowing increasing collateral, not withdrawing

Proper monitoring for transaction reordering and validating price feed sanity is prudent. But the core logic appears sound.

## Third Party Dependencies

The main external code dependencies are:

- **OpenZeppelin libraries** - Used extensively for common DeFi constructs like SafeMath, ReentrancyGuard etc. As an established audited library this poses minimal risk.

- **Chainlink Price Feeds** - Used as one of the potential price oracle sources. As decentralized infrastructure, reasonable reliance. But availability issues would cause problems.

- **Uniswap V3 TWAP** - Also used as a potential price feed source. As on-chain DEX fairly manipulate resistant but potential for liquidity issues.

## Operational Dependencies  

Operationally, main external risks include:

- **Public Ethereum infrastructure** - Outages to core blockchain networks/tooling could break system utility

- **Frontend hosting** - Attacks disrupting the Salty.IO site/interface would limit usability

- **Maker governance actions** - As DAI is a key aspect, risks to its peg stability from Maker proposals is a factor

## Conclusion

Reasonably limited external code dependencies help security. But production infrastructure issues still pose risks. Likely best mitigated at the operations level through redundancy.


Diagram showing the core flow of assets between the main Salty.IO protocol contracts:

```js
              SALT Tokens 
                │
           Emissions
                │ 
           ┌────────┴────────┐
           │              │
         Staking      Liquidity 
           │              │
           │              │
xSALT Holders     Liquidity Providers   
           │              │
     StakingRewards  CollateralAndLiquidity
                     │
                     │
              ┌─────┴──────┐
              │           │
           WBTC/WETH     USDS
              │           │
              │           │
              │       Borrow & 
              │      Repay USDS
              │           │
         Pools ┤┬┬┬┬┬┬┬¤┬┬┬┬┬          
              ││││││││ │││││
      Price Feeds  Swaps   │  
                         Arbitrage
                           │
                    ┌────────┴────┐
                DAO │            │ Upkeep
                    │            │
                 Protocol       Rewards
               Owned Liquidity    │
                                    │
                               Callers
```

- Emissions distributes newly minted SALT to Staking and Liquidity rewards contracts
- StakingRewards and CollateralAndLiquidity let users earn rewards proportionally
- CollateralAndLiquidity allows borrowing USDS against WBTC/WETH collateral
- Pools handles swaps between assets, runs arbitrage trades
- Arbitrage profits go to DAO and Upkeep for POL formation and rewards
- USDS and collateral assets flow between Pools and Collateral contracts

### Time spent:
50 hours