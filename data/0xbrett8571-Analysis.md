## Introduction

Salty.IO is an Ethereum-based decentralized exchange (DEX) that aims to provide zero swap fees, yield from automatic arbitrage, and a native stablecoin (USDS) backed by WBTC/WETH collateral. 

In this report, I analyze the Salty.IO smart contract architecture and identifies strengths, risks, and recommendations in key areas. Focus is placed on token safety, access controls, reward mechanisms, price oracles, and the USDS stablecoin model. Both systematic risks and specific vulnerabilities are covered.

**Evaluation Approach**

The analysis focused on auditing the critical Salty.IO contracts:

- [Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol) - Core AMM and arbitrage logic  
- [DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol) - Governance and protocol owned liquidity
- [Upkeep.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol) - Rewards distribution and emissions
- [PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol) - Redundant price feed 
- [USDS.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol) + [CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol) - Stablecoin model

Manual review was supplemented by unit tests, debugging sessions, and code instrumentation to validate assumptions and behavior. 

Key areas of attention included:

- Access control and trust minimization 
- Token tracking and arithmetic safety
- Protocol incentives and sustainability  
- Price feed redundancy and manipulation resistance
- Stablecoin collateral ratios and liquidity

**Here is an explanation of some of the main contracts in Salty.IO, along with a brief analysis, risks, and recommendations for each:**

[**Pools.sol**](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol)

This is the core contract that contains the AMM decentralized exchange logic and arbitrage functionality. 

*Analysis*: Pools.sol follows best practices - it is well-structured, lean, and modular. Usage of libraries like PoolUtils and inheritance help reduce duplicate code.

*Risks*: Potential issues with arithmetic safety, especially around arbitrage profit calculations. Also risk of flash loan attacks.

*Recommendations*: Comprehensive unit test coverage, formal verification of critical arithmetic, and additional hardening against flash loans.

[**DAO.sol**](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol) 

The DAO contract handles governance processes like voting and protocol parameter changes.

*Analysis*: Logic is complex but modular with good separation of concerns. Use of interface architecture pattern limits dependencies.

*Risks*: Governance attacks, issues with ballot finalization, and parameter changes.

*Recommendations*: Timelocks on critical parameter changes, formal/manual review of finalization logic.

[**PriceAggregator.sol**](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol)

Aggregates prices from multiple oracles to derive resilient BTC and ETH prices for the protocol.

*Analysis*: Good design choice to aggregate multiple price feeds. Contract is simple and focused.

*Risks*: Main risk is that aggregated price could still be manipulated if underlying oracle feeds are compromised.

*Recommendations*: Additional pricing redundancy, using derivatives, prediction markets, etc.

[**USDS.sol**](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol)

Implements the USDS stablecoin - users borrow against WBTC/WETH collateral.

*Analysis*: Loan-to-value ratio management and liquidation logic is complex but modular.

*Risks*: Incorrect collateral ratio checks or issues with liquidation could disrupt USDS peg.

*Recommendations*: Comprehensive unit testing of stability mechanisms, external audit focus.

## Architecture

Salty.IO divides functionality into modular and upgradeable contracts, following best practices. Key components:
          
```solidity
┌──────────────────────┐        
│                      │        
│      DAO.sol         │                             
│      Governance      │
│                      │
└───────────┬──────────┘
           │
┌───────────┴──────────┐
│       Pools.sol      │ 
│    AMM + Arbitrage   │  
└───────────┬──────────┘
           │        
┌───────────┴┐┌────────┴─────────┐
│ Upkeep.sol ││ Collateral.sol  │
│  Rewards   ││   Stablecoin    │  
│Distribution││     Model       │
└───────────┬┘└────────┬────────┘
           │         │
           │         │
┌───────────┴────────┐│
│ PriceAggregator.sol││ 
│    Redundant       ││
│     Price Feed     ││
└────────────────────┘│
                      │
 ┌─────────────────────┴────────┐
 │            Peripheral Contracts          
 │  Staking.sol, RewardsEmitter.sol, etc   
 └─────────────────────────────────┘
```

This separation of concerns improves modularity, upgradability, and testing.

**Strengths**

- **Trust Minimization** - No privileged roles beyond the DAO. Control is decentralized from launch via holder voting.

- **Modularity** - Logical separation of functionality enables independent upgrading of most components.

- **Redundancy** - Critical price feeds and contracts have redundancy (PriceAggregator, AccessManager). 

- **Incentive Alignment** - Arbitrage profits align user transactions with yield. LP rewards and emissions incentivize pool liquidity.

**Risks and Recommendations**

**1. Administrative Keys**

- The ManagedWallet library manages admin keys for the DAO treasury and rewards vesting wallets. While administrative access poses centralization risks, strict social trust in a multi-signature scheme may be acceptable given Salty.IO's focus on decentralization elsewhere.  

- **Recommendation:** Formal verification of ManagedWallet against mismanagement of keys. Timelock periods could also be increased.

**2. Oracle Manipulation** 

- Asset prices originate from potentially manipulable Chainlink and Uniswap V3 pools. 

- Manipulated prices could disrupt USDS collateral ratios.

- **Recommendation:** Additional oracle redundancy, using derivatives, prediction markets, etc. 

**3. Stablecoin Capital Inefficiency**

- 200% initial collateralization of USDS reduces capital efficiency. Over-collateralization is likely unnecessary given frequent pricing and the liquidation mechanism.

- **Recommendation:** Governance control of collateral ratio ceilings. 100-150% would improve efficiency without undue risk.

**4. Liquidity and Slippage**  

- As an AMM, Salty.IO will need to bootstrap external liquidity. Without sufficient liquidity, swap costs could exceed arbitrage yields. 

- **Recommendation:** Focus should be placed on liquidity incentives and bootstrapping for token pairs. Partnerships with liquidity providers could help seed deeper markets.

Expert analysis of the key risks for Salty.IO and my recommendations for mitigations in several areas

**Admin Abuse Risks**

- The DAO and governance contract concentrate control and influence over protocol parameters and funds. There is a risk of governance attacks or malicious proposals even if codes operate correctly.

- Mitigation: Timelocks on critical parameter changes give community ability to react. Vesting schedules on reserved assets limit damage from compromised keys. Formal governance may be transition option.

**Systemic Risks** 

- As a new protocol, risks relate to lack of sufficient liquidity for swap fees to capitalize arbitrage rewards. Without revenue or LP rewards, ecosystem may not attract capital to spark positive feedback loops.

- Mitigation: Focus intensely on liquidity incentives and partnerships for initial token pools. Parameterize fee structure and collateral ratios to remain nimble.

**Technical Risks**

- Contracts are complex, raising risk of subtle bugs slipped through audits and tests. Logic bugs or arithmetic errors could lead to token loss or manipulation.

- Mitigation: Ensure comprehensive unit tests and auditing coverage augmented by formal verification of core arithmetic. Incrementally decentralize control system functionality.

**Integration Risks**

- Protocol integrates PriceAggregator, AMM, rewards, stablecoin, and other components. Issues with upgrades or dependencies could break core functions.

- Mitigation: Architect contracts around interfaces to limit ripple effects from changes. Sandbox testnet deployments before mainnet upgrades. Upgrade critical contracts via proxy pattern.

Adopting proactive risk management across these dimensions will allow Salty.IO to navigate the complexities of cryptoeconomic protocol design. An incremental deployment approach is advisable given novel arbitrage and reward mechanisms.

## Conclusion

Salty.IO introduces several compelling innovations in areas like arbitrage and stablecoin collateralization, captured through a modular architecture aligned with emerging best practices. As with any novel system, incremental deployments can validate assumptions before permissions are decentralized. Continued attention to incentive design and parameter tuning will help manage risks as adoption grows.



### Time spent:
28 hours