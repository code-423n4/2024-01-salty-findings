## [N-01] File does not contain an SPDX Identifier

There are 3 instances of this issue in 3 files:

    File: PoolUtils.sol	

    1: pragma solidity =0.8.22;

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol

    File: PoolMath.sol	

    1: pragma solidity =0.8.22;

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol

    File: SigningTools.sol	

    1: pragma solidity =0.8.22;

https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol

## [N-02] Contract Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

1) splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
2) clearly documenting the functions and implementations the owner can change,
3) documenting the risks associated with privileged users and single points of failure, and
4) ensuring that users are aware of all the risks associated with the system.

## [N-03] Consider addings checks for signature malleability

Use OpenZeppelin’s *ECDSA* contract rather than calling *ecrecover()* directly

There is 1 instance of this issue:

    File: SigningTools.sol	

	26: address recoveredAddress = ecrecover(messageHash, v, r, s); 
    28: return (recoveredAddress == EXPECTED_SIGNER);

https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol

## [N-04] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

There are 5 instances of this issue in 4 files:

    File: Proposals.sol	

	51: mapping(uint256=>mapping(Vote=>uint256)) private _votesCastForBallot;

	55: mapping(uint256=>mapping(address=>UserVote)) private _lastUserVoteForBallot;

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol

    File: Pools.sol	

	49: mapping(address=>mapping(IERC20=>uint256)) private _userDeposits;

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol

    File: StakingRewards.sol	

	36: mapping(address=>mapping(bytes32=>UserShareInfo)) private _userShareInfo;

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol

    File: AccessManager.sol	

    26: mapping(uint256 => mapping(address => bool)) private _walletsWithAccess;

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol