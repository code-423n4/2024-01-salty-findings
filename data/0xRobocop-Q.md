# L-01 Bootstrap ballot can get bricked.

When `finalizeBallot` is called and the majority voted yes, then the code initiates the distribution of SALT tokens calling the `distributionApproved` function of the `InitialDistribution` contract which should have been transferred the 100M SALT tokens previously.

The issue arises because users can still cast their vote even after the ballot completion time has passed. So it is possible for users to vote negative and outnumber the "yes" votes:

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L48

If SALT supply were sent non-atomically with the call to `finalizeBallot` it will cause the SALT tokens to get stucked in the `InitialDistribution` contract: 

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L74C3-L84C26

Recommendations:

- Do not allow to vote if `block.timestamp > completion`.
- Or make the transfer of SALT tokens and the call to `finalizeBallot` atomically.

# L-02 Not claimed SALT from airdrop will get stucked. 

The initial distribution of SALT tokens take into account an initial airdrop where whitelisted users that casted their votes can claim their SALT rewards. The issue is that in case some users do not claim these SALT tokens, there is no other way to take this tokens out of the contract.

Recommendation: Send the remaining SALT balance from the airdrop contract to the DAO after a specified deadline.

# L-03 Some DAO proposals can be hard to propose.

Some proposals such as `proposeCountryInclusion`, `proposeCountryExclusion`, `proposeSetContractAddress` and `proposeWebsiteUpdate` can only be created once. For example, in other words, proposing a ballot for changing the address for priceFeed_1 is only possible if there are not active ballots for the same purpose. 

So, this allows a griefing attack vector where a well-funded user may try to always spam the creation of these proposals with a dummy address after the finalization of the previous one. This can be a challenge for governance participants, since they first need to coordinate efforts to reach the quorum in order to finalize the active dummy proposal and then ensure that they can propose the valid one previous in the block than the attacker.

Recommendation: 

- Information can included for the identification of the ballot in order to discriminate them even if they are the same type of proposal. For example, in the case of the price feeds, the proposed new address can be used to differentiate between the attacker one and the valid one.



# L-04 Reaching quorum to finalize a ballot is a too hard requirement.

The current logic on making proposals only allow a user to have at most one active proposal:

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L98

The issue is that in order to finalize a proposal even if it was not approved it need to reach a certain quorum of votes:

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L385

This is a too hard requirement, since it makes the user dependent on the participation that its proposal can get regardless if the direction is positive or negative. Some proposals encourage the community to vote (because they can only be proposed once, like change a parameter), but other ones such as `proposeCallContract` do not.

Users can still unstake and migrate their tokens to another account, but this will take almost a year in order to unstake completely.

Recommendation: 

- To avoid a DoS state on the proposals made by users due to a small activity of voters, it should be possible to finalize non-reached quorum ballots after some period of time. 
