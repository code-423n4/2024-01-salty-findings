# Q/A Report


Salty.IO  - pina

## Summary

| |Issue|
|-|:-|
| [[L-01](#l-01)] | Potential Stuck `SALT` Tokens in `Emissions.sol` |
| [[L-02](#l-02)] | Inappropriate Event Emission in `DAO::setContract()` |
| [[L-03](#l-03)] | Lack of Validation in `Proposal::proposeCountryInclusion` for Existing Countries |
| [[L-04](#l-04)] | Truncation of amountToSendToTeam in Reward Processing |





### Low Risk Issues
### [L-01]<a name="l-01"></a> Potential Stuck `SALT` Tokens in `Emissions.sol`

The `Emissions` contract, designed to distribute `SALT` tokens, encounters a critical issue where a certain amount of tokens can become unrecoverable. Initially funded with 52 million `SALT` tokens (updated to 50 million as per the latest documentation), the contract may reach a state where the `saltBalance` is greater than zero, yet no further transfers are possible due to a division issue. This occurs when the calculation `(100 * 1000 weeks) > (saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000())` yields zero.

Given the current configuration, an estimated up to 0.00000000012096 SALT tokens could remain locked in the contract. This scenario depends on the variable timeSinceLastUpkeep, which can range between 1 and 1,000 weeks, especially if `rewardsConfig::emissionsWeeklyPercentTimes1000` is modified by the DAO.

*Proposed Solution*
A more effective solution would be to implement an option that, upon reaching this critical balance range, transfers all remaining tokens and modifies the validation for entering this function. This change would save gas for the upkeep operation, as the functionality would no longer be required under these conditions.


```solidity
File: src/rewards/Emissions.sol

        uint256 saltBalance = salt.balanceOf(address(this));
        // Target a certain percentage of rewards per week and base what we need to distribute now on how long it has been since the last distribution
        uint256 saltToSend =
            (saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000()) / (100 * 1000 weeks);
        if (saltToSend == 0) {
            return;
        }

```


*GitHub* : [L1](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L55)

### [L-02]<a name="l-02"></a> Inappropriate Event Emission in `DAO::setContract()`

The setContract() function in the DAO is emitting an event even when no contract address matches, leading to potential misinformation or confusion about contract updates.


The event SetContract is emitted regardless of whether a contract address match occurs in the if-else conditions. This can lead to a false indication that a contract has been set or changed, even when no actual update has occurred.

*Proposed Solution*
Adjust the code to ensure that the SetContract event is emitted only when a contract address successfully matches and is set. This can be achieved by moving the emit statement into each if condition where a contract address match is confirmed.

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

function _executeSetContract(Ballot memory ballot) internal {
        bytes32 nameHash = keccak256(bytes(ballot.ballotName));

        if (nameHash == keccak256(bytes("setContract:priceFeed1_confirm"))) {
            priceAggregator.setPriceFeed(1, IPriceFeed(ballot.address1));
        } else if (nameHash == keccak256(bytes("setContract:priceFeed2_confirm"))) {
            priceAggregator.setPriceFeed(2, IPriceFeed(ballot.address1));
        } else if (nameHash == keccak256(bytes("setContract:priceFeed3_confirm"))) {
            priceAggregator.setPriceFeed(3, IPriceFeed(ballot.address1));
        } else if (nameHash == keccak256(bytes("setContract:accessManager_confirm"))) {
            exchangeConfig.setAccessManager(IAccessManager(ballot.address1));
        }
        
@>      emit SetContract(ballot.ballotName, ballot.address1);
    }

```


*GitHub* : [L-02](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L35)


### [L-03]<a name="l-03"></a> Lack of Validation in `Proposal::proposeCountryInclusion` for Existing Countries


The proposeCountryInclusion function in the DAO does not check if a country already exists in the list before allowing a proposal for its inclusion.


The function allows for the proposal of a country's inclusion without verifying whether the country is already included. This oversight can lead to redundant proposals and potentially cause confusion or inefficiencies within the DAO's operational processes.

*Proposed Solution*
Implement a validation check within proposeCountryInclusion to verify if the proposed country is already on the list. This will prevent the submission of unnecessary proposals and streamline the DAO's operations.

```solidity
File: src/dao/Proposals.sol

    function proposeCountryInclusion(string calldata country, string calldata description)
        external
        nonReentrant
        returns (uint256 ballotID)
    {
        require(bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code");
@>      // there is not require

        string memory ballotName = string.concat("include:", country);
        return _possiblyCreateProposal(ballotName, BallotType.INCLUDE_COUNTRY, address(0), 0, country, description);
    }

```


*GitHub* : [L3](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L222-L228)


### [L-04]<a name="l-04"></a> Truncation of amountToSendToTeam in Reward Processing

The current implementation in Proposals.sol may lead to value truncation when calculating the amount of SALT to send to the team. To ensure accuracy and fairness, it's advisable to use more precise calculations or avoid truncation.

`uint256 amountToSendToTeam = (claimedSALT *10) / 100`

The use of claimedSALT / 10 for calculating amountToSendToTeam potentially results in a truncated value, leading to a minor but significant loss in precision. This could affect the distribution of rewards to the team, especially in large transactions.

*Proposed Solution*
Consider implementing a more precise calculation method to determine the amountToSendToTeam. This could involve using a more accurate formula or a library that handles fractional values better, ensuring no loss of rewards due to rounding errors.

```solidity
File: src/dao/Proposals.sol

function processRewardsFromPOL() external
		{
		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );

		// The DAO owns SALT/USDS and USDS/DAI liquidity.
		bytes32[] memory poolIDs = new bytes32[](2);
		poolIDs[0] = PoolUtils._poolID(salt, usds);
		poolIDs[1] = PoolUtils._poolID(usds, dai);

		uint256 claimedSALT = collateralAndLiquidity.claimAllRewards(poolIDs);
		if ( claimedSALT == 0 )
			return;

		// Send 10% of the rewards to the initial team
@>		uint256 amountToSendToTeam = claimedSALT / 10;

```


*GitHub* : [L4](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L341)



