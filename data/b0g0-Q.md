## [L-01] Signature verification function does not prevent signature malleability

## Issue Description
A common attack vector when it comes to using signatures it their malleability. In simple terms this allows an already used signature to be modified without changing it's signed data and reusing it a second time ([article](https://medium.com/draftkings-engineering/signature-malleability-7a804429b14a)). The way to prevent this is to check that s value of the signature is in the lower half order, and the v value to be either 27 or 28. However the `_verifySignature` inside `SigningTools.sol` library doesn't do this and allows such forged signatures to be accepted. This is used during voting to start the exchange and could lead to double votes

```
function _verifySignature(
        bytes32 messageHash,
        bytes memory signature
    ) internal pure returns (bool) {
        require(signature.length == 65, "Invalid signature length");

        bytes32 r;
        bytes32 s;
        uint8 v;

        //@audit [L] -> does not check for upper/lower value of the signature

        assembly {
            r := mload(add(signature, 0x20))
            s := mload(add(signature, 0x40))
            v := mload(add(signature, 0x41))
        }

        address recoveredAddress = ecrecover(messageHash, v, r, s);

        return (recoveredAddress == EXPECTED_SIGNER);
    }
```

## Recommended Mitigation Steps
Consider using OpenZeppelin ECDSA library (which handles this) instead of implementing it yourself -> https://docs.openzeppelin.com/contracts/2.x/api/cryptography#ECDSA-recover-bytes32-bytes-

## [L-02] If exchange start is not approved after the Boostrap Ballot has completed it will never be launched

## Issue Description
Calling  `BootstrapBallot.finalizeBallot()` could be called only once after the `completionTimestamp` has passed. However voting is not restricted by that timestamp and could continue afterwards. In case the ballot did not receive more *YES* than *NO* votes calling  `BootstrapBallot.finalizeBallot()` will finalize it eternally without any mechanism to launch the exchange in case enough *YES* votes have been given after that (maybe just not all voter have voted before `completionTimestamp` has expired)

```solidity
function finalizeBallot() external nonReentrant {
        require(!ballotFinalized, "Ballot has already been finalized");
        require(
            block.timestamp >= completionTimestamp,
            "Ballot is not yet complete"
        );

        if (startExchangeYes > startExchangeNo) {
            exchangeConfig.initialDistribution().distributionApproved();
            //@audit - there is a nested nasty loop here- can deployment go out of gas for too many whitelisted pools
            exchangeConfig.dao().pools().startExchangeApproved();

            startExchangeApproved = true;
        }

        emit BallotFinalized(startExchangeApproved);

        ballotFinalized = true;
    }
```

## Recommended Mitigation Steps
Consider modifying the function to be called repeatedly, so that in case YES become more than *NO* votes the exchange would be launched

## [L-03] No limit on the string length of proposal descriptions

## Issue Description
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L155

When creating a proposal to be voted by the DAO the user can provide a description that would give information about the proposal. However in none of the proposal creation functions inside the `Proposals.sol` contract exists a restriction for the length of the string. This allows a proposer to maliciously store a giant piece of text in the proposal. This in turn would generate a lot of additional costs to the DAO members who vote. Since the EVM would have to load that giant text in memory every time someone want to vote on it (gas progressively increases with the size of the string). The additional gas costs of those inefficient memory loads would have to be paid by the voters.

Take a look at this article that explores the gas costs of long strings in Solidity -> [Article](https://medium.com/@marvinirwin/gas-estimation-and-large-strings-in-the-ethereum-virtual-machine-feat-remix-and-klaytn-649e6db073f)

## Recommended Mitigation Steps
Consider using the following check to prevent long description:
```solidity
require(bytes(description).length <= someReasonableLength,"description too long")
```

 