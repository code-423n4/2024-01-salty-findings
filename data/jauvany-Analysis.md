**Overview**

Salty.IO is a Decentralized Exchange on Ethereum which uses Automatic Atomic Arbitrage (AAA) to generate yield and provide Zero Fees on all swaps. With AAA, market inefficiencies are arbitraged at swap time to create profits - which are then distributed to liquidity providers and stakers and used to form Protocol Owned Liquidity (POL) for the DAO. Additionally, Salty.IO provides USDS, an overcollateralized ERC20 stablecoin native to the protocol which uses WBTC/WETH LP as collateral. The exchange is 100% decentralized at launch - with all parameters, regional exclusions, whitelisting, and contracts controlled by the DAO itself.

# Codebase quality analysis
 
> Analysis of the codebase (What’s unique? What’s using existing patterns?):

**Unique:* 
- Codebase carries out specific governance mechanisms that are uniquely designed for Salty.io’s specific use case e.g  Initial tokens whitelisted on the exchange will be WBTC, WETH and DAI as well as the native USDS and SALT tokens.

**Existing Patterns:* 
- The Salty .io Protocol adheres to common contract management patterns, such as the use of OnlyRole and OnlyAdmin.

**Strengths:*
- Rich documentation provided

**Weaknesses:*
- Incomplete test coverage (99%). 100% test coverage is recomended.
 
# Systemic risks

- External Contract Dependencies: Salty.io relies on external contracts from Openzepplin, Uniswapv3, and Chainlink. If any of these contracts have vulnerabilities, it would affect the protocol.

- Test Coverage: The test coverage provided by Salty.io is 99% however, 100% test coverage is recommended.

- Like any smart contract-based system, Salty.io is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol. 

# Documents 

The documentation of the Salty.io protocol is quite comprehensive and detailed, providing a solid overview of how the Salty.io protocol is structured and how its various aspects function. 
I would also recommend adding quality Medium articles, it’s a great way to provide an indepth look at many of the topics in the project and is used by many blockchain projects.

# Other Audit Reports and Automated Findings 

See automated findings [here](https://github.com/code-423n4/2024-01-salty/blob/main/bot-report.md)
The 4naly3er report can be found [here](https://github.com/code-423n4/2024-01-salty/blob/main/4naly3er-report.md).

Previous ABDK audit can be found [here](https://github.com/abdk-consulting/audits/tree/main/othernet_global_pte_ltd).
Previous Trail of Bits audit can be found [here](https://github.com/trailofbits/publications/blob/master/reviews/2023-10-saltyio-securityreview.pdf).


# Architecture recommendations

> The Salty.io architecture seems solid in general, none the less here are some areas that could be improved:

- Testing and Simulations: I recommend creating a live testnet app. Here is an
[example](https://app.opendollar.com/) from The Open Dollar protocol. Conduct thorough testing of all contracts and functions and simulations to understand how they will behave under various market conditions.

- I recommend rewriting some of the tests in the codebase for this audit to use the actual contracts instead of mock addresses like in some cases. This will offer greater confidence during system deployment.

**Gas Optimizations**

- Review data types: Analyze the data types used in your smart contracts and
consider if they can be further optimized. For example, changing uint256 to
uint128 or uint94 can save gas and storage slots.
- Struct packing: Look for opportunities to pack structs into fewer storage slots. By carefully selecting appropriate data types for struct members, you can reduce the overall storage usage.
- Use constant values: If certain values in your contracts are constant and do
not change, declare them as constants rather than storing them as state variables. This can significantly save gas costs.
- Avoid unnecessary storage: Examine your code and eliminate any unnecessary storage of variables or addresses that are not required for contract functionality.
- Storage vs. memory usage: When working with arrays or structs, consider whether using storage instead of memory can save gas. Using storage allows direct access to the state variables and avoids unnecessary copying of data.
- Replacing the use of memory with calldata for read-only arguments in external  functions.

**Other recommendations**

- Regular code reviews and adherence to best practices.
- Conduct external audits by security experts.
- Consider open sourcing the contract for community review.
- Maintain comprehensive security documentation.
- Establish a responsible disclosure policy for vulnerabilities.
- Implement continuous monitoring for unusual activity.
- Educate users about risks and best practices.

# Approach taken in evaluating the codebase
 
> My analysis of the Salty.io Protocol Included understanding the architecture, mechanism, overall codebase and possible risks associated to the protocol. 

- Day 1: I spent time reading the different available articles in order to have a deep understanding of the protocol. 

- Day 2: I analyzed the codebase for better understanding, Performed a Mechanism review and investigated possible systemic risks, and centralization risks. 

- Day 3: I dedicated this day to coming up with possible Architecture recommendations after identifying possible risks and prepared the final analysis report.

**Conclusion**  
Analysing this codebase and its architectural choices has been a delightful experience. Inherently complex systems greatly benefit from strategically implemented simplifications, and I believe this project has successfully struck a harmonious balance between the imperative for simplicity and the challenge of managing complexity. I hope that I have been able to offer a valuable overview of the methodology utilised during the analysis of the contracts within scope, along with pertinent insights for the project team and any party interested in analysing this codebase.


### Time spent:
24 hours