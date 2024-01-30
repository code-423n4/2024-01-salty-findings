# 🛠️ Analysis - Salty.IO
***An Ethereum-based DEX with zero swap fees, yield-generating Automatic Arbitrage, and a native WBTC/WETH backed stablecoin.***

### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |Overview of the Decent Project| Summary of the whole Protocol |
|b) |Technical Architecture| Architecture of the smart contracts |
|c) |The approach I would follow when reviewing the code | Stages in my code review and analysis |
|d) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|e) |Test analysis | Test scope of the project and quality of tests |
|f) |Security Approach of the Project | Audit approach of the Project |
|g) |Codebase Quality | Overall Code Quality of the Project |
|h) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|h) |Full representation of the project’s risk model| What are the risks associated with the project |
|i) |Packages and Dependencies Analysis | Details about the project Packages |
|j) |New insights and learning of project from this audit | Things learned from the project |



## a) Overview of the Decent Project

Decent Project is an innovative blockchain initiative designed to address the challenges of interoperability and fluidity in transactions across multiple blockchain networks. At its core, Decent aims to simplify and streamline the process of executing transactions on various blockchains, enhancing the user experience in the decentralized finance (DeFi) and blockchain space. Here's an overview of its key aspects:

### Key Features and Functionalities:

1. **Cross-Chain Transactions**: Decent facilitates seamless transactions across different blockchain networks. This capability is pivotal in a landscape where assets and liquidity are spread across various chains.

2. **Swapping and Bridging Mechanisms**: The platform integrates sophisticated swapping and bridging functionalities, allowing users to convert and transfer assets between different tokens and blockchains efficiently. This feature is crucial for users who engage in activities across multiple chains.

3. **Fee Management**: The platform incorporates an effective fee management system, ensuring that transactions are economically viable for users and sustainable for the platform's longevity.

## b) Technical Architecture:

Decent's architecture is built around a set of smart contracts, each serving specific roles:


| File Name               | Core Functionality                                      | Technical Characteristics                                                                                               | Importance and Management                                                 |
|-------------------------|---------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| `UTB.sol`               | Universal Transaction Bridge                            | Orchestrates swaps and bridges, interfaces with `Swapper` and `BridgeAdapter`                                           | Central to transaction execution; manages swaps and bridges               |
| `UTBExecutor.sol`       | Executes transactions                                   | Executes payment transactions, handles ERC20 and native tokens                                                          | Executes transactions post-swap/bridge, crucial for final transaction step|
| `UTBFeeCollector.sol`   | Manages fee collection                                  | Collects fees in various scenarios, handles ERC20 and native currency                                                   | Essential for economic sustainability, manages transaction fees           |
| `BaseAdapter.sol`       | Foundation for bridge adapters                          | Provides base functionalities and checks for bridge adapters                                                            | Basis for creating specific bridge adapters, ensures secure function calls|
| `DecentBridgeAdapter.sol` | Manages bridging operations                            | Handles the complexities of bridging tokens across chains                                                               | Facilitates asset transfer between blockchains                           |
| `StargateBridgeAdapter.sol` | Specialized bridge adapter using Stargate protocol | Manages bridging operations using Stargate, handles cross-chain transactions                                            | Enables efficient bridging with Stargate technology                      |
| `SwapParams.sol`        | Library for swap parameters                             | Provides structures and potentially utility functions for swaps                                                         | Essential for defining swap operations, used across swappers             |
| `UniSwapper.sol`        | Swapper implementation using Uniswap                    | Executes token swaps using Uniswap’s liquidity, handles swap logic                                                      | Enables token swapping, integral for transactions requiring token exchange|
| `DcntEth.sol`           | Manages Decent's native tokens                          | ERC-20 compliant, includes minting and burning functionalities                                                          | Manages token supply, critical for token operations within Decent        |
| `DecentEthRouter.sol`   | Manages routing and cross-chain logic                   | Handles transaction routing, bridges with LayerZero technology                                                          | Key for managing cross-chain interactions, routes transactions           |
| `DecentBridgeExecutor.sol` | Executes bridge transactions                          | Executes transactions involving bridging, supports both WETH and native ETH                                             | Executes complex cross-chain transactions, vital for bridging operations |


## c) The approach I would follow when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://code4rena.com/audits/2024-01-saltyio

Accordingly, I would analyze and audit the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#build--test-instructions)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Salty](https://github.com/code-423n4/2024-01-salty/tree/main/src) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| Slither report of the project for some basic analysis|
|5|Test Suits|[Tests](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#build--test-instructions)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#scope)||
|7|Using Solodit for common vulnerabilities|[Solodit](https://solodit.xyz/)|Using solodit to find common vulnerabilites related to NFT protocol|
|8|Infographic|[Figma](https://www.figma.com/)|Tried to make Visual drawings to understand the hard-to-understand mechanisms|
|9|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs)|Code where I should focus more|

## d) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics


-  **File:** This field contains the name or path of the source file being analyzed.

-  **Logic Contracts:** This field indicates the number of Contracts involves

-  **Interfaces:** This field indicated specify the number or details of interfaces defined in the source file.

-  **Lines:** This field represents the total number of lines in the source file, including code lines, comments, and blank lines.

-  **nLines:** nLines typically stands for "normalized lines" and represents the total number of lines in the source file excluding blank lines. 

-  **nSLOC:** nSLOC stands for "normalized source lines of code," and it further refines nLines by excluding both blank lines and comments. It gives a more accurate measure of the code's complexity.

-  **Comment Lines:** This field specifies the number of lines in the source file that contain comments.

-  **Complex. Score:** This field may indicate a complexity score or metric for the source file. 

## Analysis of sloc of `Dao` contracts

[![Screenshot-from-2024-01-30-12-40-36.png](https://i.postimg.cc/gk1md16Z/Screenshot-from-2024-01-30-12-40-36.png)](https://postimg.cc/bs0K9Cgy)

## Analysis of sloc of `Launch` contracts

[![Screenshot-from-2024-01-30-12-41-44.png](https://i.postimg.cc/TwhF478X/Screenshot-from-2024-01-30-12-41-44.png)](https://postimg.cc/47kWK1N8)

## Analysis of sloc of `Pools` contracts

[![Screenshot-from-2024-01-30-12-42-23.png](https://i.postimg.cc/PxLV4CZX/Screenshot-from-2024-01-30-12-42-23.png)](https://postimg.cc/47g14NBq)

## Analysis of sloc of `price_feed` contracts

[![Screenshot-from-2024-01-30-12-43-47.png](https://i.postimg.cc/y6v0fy35/Screenshot-from-2024-01-30-12-43-47.png)](https://postimg.cc/s1GBxSc9)

## Analysis of sloc of `Rewards` contracts

[![Screenshot-from-2024-01-30-12-44-35.png](https://i.postimg.cc/2SK4x2xW/Screenshot-from-2024-01-30-12-44-35.png)](https://postimg.cc/5X8Y45T9)

## Analysis of sloc of `Stable` contracts

[![Screenshot-from-2024-01-30-12-45-10.png](https://i.postimg.cc/2yGWLw6V/Screenshot-from-2024-01-30-12-45-10.png)](https://postimg.cc/rRdsv55k)

## Analysis of sloc of `Staking` contracts

[![Screenshot-from-2024-01-30-12-45-46.png](https://i.postimg.cc/RZyHtqMQ/Screenshot-from-2024-01-30-12-45-46.png)](https://postimg.cc/bZ0rcySs)

## Analysis of sloc of src contracts

[![Screenshot-from-2024-01-30-12-47-00.png](https://i.postimg.cc/1tTVQJ05/Screenshot-from-2024-01-30-12-47-00.png)](https://postimg.cc/LYt8kk0r)

## Comment-to-Source Ratio:

**`DAO` contracts:** On average there are **4.69** code lines per comment (lower=better).

**`Launch` contracts:** On average there are **4.39** code lines per comment (lower=better).

**`Pools` contracts:** On average there are **1.82** code lines per comment (lower=better).

**`price_feed` contracts:** On average there are **4.49** code lines per comment (lower=better).

**`Rewards` contracts:** On average there are **3.54** code lines per comment (lower=better).

**`Stable` contracts:** On average there are **3.26** code lines per comment (lower=better).

**`Staking` contracts:** On average there are **2.98** code lines per comment (lower=better).

**`src` contracts:** On average there are **3.22** code lines per comment (lower=better).

# Call Graph of Important Contracts

## Call graph of `Launch` contract
[![Screenshot-from-2024-01-30-12-52-14.png](https://i.postimg.cc/RVRdrJp6/Screenshot-from-2024-01-30-12-52-14.png)](https://postimg.cc/w71LLMpg)

## Call graph of `Pools` contracts

[![Screenshot-from-2024-01-30-12-53-45.png](https://i.postimg.cc/R09vW8Qb/Screenshot-from-2024-01-30-12-53-45.png)](https://postimg.cc/G4MWNXfk)

## Call graph of `src` contracts

[![Screenshot-from-2024-01-23-21-31-57.png](https://i.postimg.cc/J4H3LypK/Screenshot-from-2024-01-23-21-31-57.png)](https://postimg.cc/4mZHv3kh)

## Contract Integration Graph

[![Screenshot-from-2024-01-23-23-43-44.png](https://i.postimg.cc/mZCbkQSd/Screenshot-from-2024-01-23-23-43-44.png)](https://postimg.cc/SjQ0tY38)


# High Level Domain Model

This domain model provides an overview of the key components  and how they are interconnected.

[![Screenshot-from-2024-01-23-22-57-18.png](https://i.postimg.cc/L52fwmTL/Screenshot-from-2024-01-23-22-57-18.png)](https://postimg.cc/k6YBt3Lg)

# Sequence diagram of Important functions

## SwapAndExecute()

[![Whats-App-Image-2024-01-23-at-10-59-13-PM.jpg](https://i.postimg.cc/qvx8chx0/Whats-App-Image-2024-01-23-at-10-59-13-PM.jpg)](https://postimg.cc/5XjYbtrk)

## bridgeAndExecure()

[![bridge.png](https://i.postimg.cc/rwbmh0bg/bridge.png)](https://postimg.cc/mPYBkDTF)

## e) Test analysis

 **Foundry Testing:**
   
   Foundry, a modern smart contract testing framework, was utilized to test the reNFT contracts. This involved several key steps:
   
   a. **Installation and Setup:**
      - Foundry was installed using the command `curl -L https://foundry.paradigm.xyz | bash`, followed by `foundryup` to ensure the latest version was in use.
      - Dependencies were installed using `forge install`and `pnpm i`, ensuring all necessary components were available for the testing process.
      - Then to run the tests, I simply added the relevant files to the .env, referencing .env.example.
   
   b. **Execution of Tests:**
      - Tests were run using `forge test`, executing a suite of predefined test cases that covered various functionalities and scenarios within the reNFT contracts.
      - A gas report was generated using `forge test --gas-report`. This report provided insights into the gas efficiency of the contracts, which is crucial for optimizing transaction costs on the blockchain.
   
   c. **Test Coverage and Documentation:**
      - The overview of the testing suite, as referred to in the provided documentation, likely details the scope, scenarios, and objectives of each test, ensuring a comprehensive assessment of the contracts.
   

### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content. In particular, tests have been written successfully.

-   2) Overall line coverage percentage provided by your tests : 75

### What could they have done better?

-  1) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


[![test-cases.jpg](https://i.postimg.cc/1zgD5wCt/test-cases.jpg)](https://postimg.cc/v1s40gdF)

Ref:https://xin-xia.github.io/publication/icse194.pdf

[![nabeel-1.jpg](https://i.postimg.cc/6qtBdLQW/nabeel-1.jpg)](https://postimg.cc/bDVXPnbW)

- 2) Test Coverage of the protocol is around 75% which is very less, the recommended test coverage of any protocol is above 90% so it is recommended to increase the coverage to at least 90%

## f) Security Approach of the Project

### Successful current security understanding of the project;

1- The project hasn't underwent any audits(nothing stated in the docs), this innovative assessments on Code4rena is the first, where multiple auditors are scrutinizing the code.

### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

3- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

4- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

5- I also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

6 - As the project team, you can consider applying the multi-stage audit model.

[![sla.png](https://i.postimg.cc/nhR0kN3w/sla.png)](https://postimg.cc/Sn96Q1FW)

Read more about the MPA model;
https://mpa.solodit.xyz/

7 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19


## g) Codebase Quality

Overall, I consider the quality of the ReNFT protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The code exhibits good maintainability and reliability practices. It is well-commented, making it easier to understand the logic and functionality of the contracts. However, it should be noted that the documentation of the protocol is lacking, which could impact long-term maintainability.
| **Code Comments** | The code is adequately commented, providing insights into the purpose and functionality of various functions and sections. These comments enhance code readability and help developers understand the implementation details.
| **Documentation** | Unfortunately, the documentation of the protocol is notably lacking. There is a need for comprehensive documentation that explains the core concepts, interactions, and usage of the smart contracts. Such documentation is essential for developers and auditors to understand the protocol thoroughly.
| **Testing** | There are test cases but it's important to strive for higher test coverage to ensure the reliability and robustness of the protocol. A test coverage of 75% indicates that a significant portion of the code has been tested, but there may still be areas that lack coverage. It's advisable to aim for as close to 100% coverage as possible to minimize the risk of undiscovered bugs and vulnerabilities. 
| **Code Structure and Formatting** | The code follows a structured and well-organized format. Functions and variables are appropriately named, enhancing code readability. However, additional documentation within the code, especially for complex functions and interactions, would further improve code structure and understanding.


## h) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-01-decent/blob/main/bot-report.md

**Previous Audits**
There isn't any Previous Audit

**4naly3er report**
https://github.com/code-423n4/2024-01-decent/blob/main/4naly3er-report.md

## i) Full representation of the project’s risk model

### Systemic Risks
**Interoperability Issues:** As a cross-chain solution, Decent relies heavily on the stability and security of other blockchains. Issues in connected networks can cascade into the Decent ecosystem.

###  Technical Risks
**Smart Contract Vulnerabilities:** Bugs or flaws in smart contracts can lead to loss of funds or malfunctioning of the platform.

**Scalability Concerns:** As transaction volumes grow, the platform must scale without compromising performance or security.

### Integration Risks

**Compatibility with Different Blockchains:**  Ensuring that Decent works seamlessly across multiple chains requires constant updates and monitoring of changes in those ecosystems.

**Cross-Chain Security:** Security inconsistencies across different blockchains can expose vulnerabilities in cross-chain transactions.

##  j) Packages and Dependencies Analysis 📦

| Package | Version | Usage | 
| --- | --- | --- | 
| [`Solmate`](https://www.npmjs.com/package/solmate?activeTab=dependents) | [![npm]([![images.png](https://i.postimg.cc/MK89GbgX/images.png)](https://postimg.cc/pyqfG8Wt))](https://www.npmjs.com/package/solmate?activeTab=dependents) |  Project uses latest version of Solmate 

## k) New insights and learning of project from this audit:

After thoroughly reviewing the Decent project's codebase and documentation, several new insights and learnings have emerged.

1. **Innovative Approach to Cross-Chain Interactions**: Decent's utilization of a combination of bridging and swapping mechanisms to facilitate cross-chain transactions is a notable innovation. This approach addresses one of the most significant challenges in the blockchain ecosystem - the seamless transfer of value and interactions across different networks.

2. **Fee Management and Optimization**: The way Decent handles transaction fees, particularly in cross-chain contexts, provides valuable insights into cost optimization in blockchain applications.

3. **Scalability and Extensibility**: Decent's architecture, especially the use of adapter patterns for bridging and swapping, demonstrates a scalable approach to blockchain development. The ability to add new swappers and bridge adapters without overhauling the core system architecture allows for future extensibility.



Note: I didn't tracked the time, the time I mentioned is just an estimate


### Time spent:
5 hours



### Time spent:
5 hours

### Time spent:
5 hours