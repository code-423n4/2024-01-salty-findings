# Analysis

## Table of Contents

- [Approach](#approach)
- [Architecture Overview](#architecture-overview)
- [Testability](#testability)
- [Centralization Risks](#centralization-risks)
- [Systemic Risks](#systemic-risks)
- [Other Recommendations](#other-recommendations)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)

## Approach

The auditing process commenced with a high level but brief overview of the entire Salty.io ecosystem. This wasn't restricted merely to the recent updates in scope, i.e after the audits but even original implementations where considered. Subsequent to this, several hours were dedicated to examining findings from previous audits and drawing insights from them. This was followed by a meticulous, line-by-line security review of the sLOC in scope. The concluding phase of this process entailed interactive sessions with the developers on Discord. This fostered an understanding of unique implementations and facilitated discussions about potential vulnerabilities and observations that I deemed significant, requiring validation from the team.

## Architecture Overview

Salty.IO is a DeFi protocol that includes new features, including atomic arbitrages and counterswaps. The novelty of the protocol introduces intrinsic risks, since these new patterns have not been stress tested over time.

### Deeper Technical Overview

Arbitrage - This section focuses on detecting arbitrage opportunities during user swap activities. The execution of arbitrage transactions takes place in the Pools.sol component, demonstrating a seamless integration of opportunity identification and action within the system.

DAO - This area is dedicated to the governance aspect of the platform. It encompasses the creation of governance proposals, facilitating voting processes, implementing the outcomes of successful proposals, and managing Protocol Owned Liquidity (POL). The configurable parameters by the DAO are strategically stored in Config.sol contracts, organized distinctly for each area of operation.

Launch - Responsible for overseeing the initial stages of the platform, including the execution of the initial airdrop, managing the distribution process, and conducting the bootstrapping ballot. This vital process involves a decentralized voting mechanism by the airdrop recipients, crucial for initiating the DEX and allocating SALT tokens.

Pools - Serving as a core component of the exchange, this section handles various critical functions such as managing liquidity pools, executing token swaps, arbitrage, and facilitating user token deposits. The latter is instrumental in reducing gas costs associated with multiple trades. Additionally, it plays a significant role in arbitrage trades by contributing to them and enabling proportional distribution of rewards.

Price Feed - Implements a robust and redundant price aggregator framework, initially integrating sources like Chainlink, Uniswap v3 TWAP, and Salty.IO's own reserves. This setup is essential in providing accurate BTC and ETH price data, which is pivotal for the functioning of the overcollateralized stablecoin framework.

Rewards - This segment is key to the distribution mechanics of the platform, managing the global emission of SALT tokens. It ensures that rewards are appropriately distributed to liquidity providers and stakers. The inclusion of a reward emission mechanism in this part is noteworthy, as it steadily releases a portion of rewards over time, thereby reducing the volatility in reward distribution.

Stable - This section is integral to the platform's stability, featuring the USDS contract and collateral functionalities. It enables users to engage in various financial activities such as depositing WBTC/WETH LP as collateral, borrowing and repaying USDS (thus minting and burning it in the process), and liquidating positions that are undercollateralized.

Staking - This area is designed to administer a staking rewards system. It allows users to receive rewards proportional to their "userShare," a dynamic metric dependent on the specific contract derived from StakingRewards.sol. This includes contracts like Staking.sol, which manages SALT staking, and CollateralAndLiquidity.sol, which oversees user deposits in terms of collateral and liquidity.

Root Directory - The root directory is a crucial part of the system, encompassing the SALT token, the default AccessManager, and the Upkeep contract. The AccessManager plays a vital role in allowing DAO-controlled geographical restrictions, while the Upkeep contract houses the performUpkeep() function, a user-operable feature that is essential for the consistent and efficient functioning of the ecosystem's rewards, emissions, POL formation, and more.

TLDR in regards to the arithmetic used in the protocol, it's fairly documented and implemented with an emphasis on. However, some arithmetic formulas do not have specifications, and some have specifications that do not match the code. We did not identify an explicit testing strategy to increase confidence in the system’s arithmetic. The testing does not cover several arithmetic edge cases, including critical ones. While there are some comments around precision and rounding issues, there is room for improvement. There are areas where unsafe casting is performed. This should be addressed either by changing the code or adding a comment justifying the decision not to use safe casting.

### Important Links

- [Documentation](https://docs.salty.io)
- [Technical Overview](https://tech.salty.io)
- [Video Technical Walkthrough](https://www.youtube.com/watch?v=bmAjm8J3q3Y)
- [Twitter](https://x.com/salty_io)
- [Discord](https://discord.gg/saltyio)

## Testability

The system's testability is commendable though occasionally, there are intricate lengthy tests that are a bit hard to follow, this shouldn't be considered a flaw though, cause once familiarized with the setup, devising tests and creating PoCs becomes a smoother endeavour. The developers have provided a high-quality testing sandbox for implementing, testing, and expanding ideas, but just as for every testing scope and in regards to the current level of testing _there's always room for further refinement and improvement._

## Centralization Risks

Would also be key to note that the protocol has many decentralized aspects, and decentralization is embedded in the design. For example, unlike other DAO proposals that often allow for the execution of arbitrary calldata, Salty.IO limits the actions available to the DAO. Furthermore, the exchange is to be 100% decentralised at lunch. However, from another angle this protocol is not permissionless, since all users must be whitelisted to do anything other than swapping. i.e, to say if a new country is added to the blacklist, every user must be whitelisted again in order to add liquidity, take loans, or stake.

## Systemic Risks

Certain areas of complexity systematically are well documented and implemented in a sustainable way, we noted several areas of excess complexity. For example, there are numerous contracts in the protocol; instances in which a function makes an external call just to fetch an address for yet another external call add undue complexity and offer opportunities for streamlining. Additionally, some functions have unclear scopes, or their scopes include too many components, for example due to [this](https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/dao/DAO.sol#L62-L63) the url could get set to a malicious one and effectively lead to a hack for users.

## Other Recommendations

### **Enhance Documentation Quality**

I'd like to start by commending the quality of the docs. They are generally well-crafted. However, there are some gaps concerning what's within the scope. It's important to remember that code comments also count as documentation, and there have been little inconsistencies in this area. Comprehensive documentation not only provides clarity but also facilitates the onboarding process for new developers.

### **Onboard More Developers**

Having multiple eyes scrutinizing a protocol can be invaluable. More contributors can significantly reduce potential risks and oversights. Evidence of where this could come in handy can be gleaned from codebase typos. Drawing from the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory), such lapses may signify underlying code imperfections.

### **Leverage Additional Auditing Tools**

Many security experts prefer using Visual Studio Code augmented with specific plugins. While the popular [Solidity Visual Developer](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor) has already been integrated with the protocol, there's room for incorporating other beneficial tools.

### **Enhance Event Monitoring**

Current implementation subtly suggests that event tracking isn't maximized. Instances where events are missing or seem arbitrarily embedded have been observed. A shift towards a more purposeful utilization of events is recommended.

### **Refine Naming Conventions**

There's a need to improve the naming conventions for contracts, functions, and variables. In several instances, the names don't resonate with their respective functionalities, leading to potential confusion, an example of this can be seen on the `SystemContext.sol` contract, where `BlockInfo` points to two different contexts which is wrong concepts, namely "The number and the timestamp of the current L1 batch stored packed." & " The number and the timestamp of the current L2 block.", whereas this could be okay for the latter, `BatchInfo` would suit more for the former.

- Add more developers or team members,

## Security Researcher Logistics

- 1 hour dedicated to writing this analysis.
- 4 hours exploring the previous protocol's audits .
- 2 hours were allocated for discussions with sponsors on the private discord group regarding potential vulnerabilities.
- The remaining time on finding issues and writing the report for each of them on the spot, later on _editing a few with more knowledge gained on the protocol as a whole, or downgrading them to QA reports_

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, being that it's like an update contest and most _easy to catch_ bugs have already been spotted and mitigated from the previous audits.


### Time spent:
25 hours