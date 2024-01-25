## [L-1] Users can vote more than their stake amount if the minimum unstake period is less than 2 weeks.

### Code Line

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L259

### Impact

Malicious actors can pass proposals without holding the required tokens. This can lead to proposals with potential issues in the system to pass.

### Vulnerability Details

A staked amount of SALT can have a minimum unstake period of 1 week, while the voting duration for a proposal can range from 3 to 14 days.

Let's imagine an attacker who has 1 million SALT staked, giving them a voting power of 1 million. To reach the quorum for a vote, only 10% of the total stake is needed. For example, if there are 12 million SALT tokens in total stake, then 1.2 million tokens are required to pass the quorum.

If, for some reason, the DAO sets the minimum unstake period to 1 week and the voting duration is between 7 and 14 days, the user can exploit this scenario. They can create a malicious proposal and vote 1 million SALT for it. Afterward, they can unstake their 1 million SALT within a week and retain 20%, which is 200k. They can then stake this 200k SALT again and vote in the same proposal because it has not yet expired.

This allows the user to vote twice with the same SALT. However, the reward for doing this is not substantial. If the user unstakes their 1 million SALT in 1 week, they will lose all of their remaining 800k SALT. Additionally, if the proposal is malicious, other voters can vote against it and cause it to fail. Nonetheless, history has shown significant governance attacks in the past resulting from these types of accounting errors. While the impact may not be high at this time, a malicious actor could potentially exploit this issue to harm the system in a better way.

### Recommendation

Do not allow the unstake duration to be less than 2 weeks.