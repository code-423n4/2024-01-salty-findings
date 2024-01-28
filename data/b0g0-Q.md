## [L-01] Signature verification function does not prevent signature malleability

## Issue Description
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/SigningTools.sol#L11

A common attack vector when it comes to using signatures is their malleability. In simple terms this allows an already used signature to be modified without changing it's signed data and reusing it a second time ([article](https://medium.com/draftkings-engineering/signature-malleability-7a804429b14a)). The way to prevent this is to check that s value of the signature is in the lower half order, and the v value to be either 27 or 28. However the `_verifySignature` inside `SigningTools.sol` library doesn't do this and allows such forged signatures to be accepted. This is used during voting to start the exchange and could lead to double votes

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

## [L-02] BoostrapBallot does not use `deadline` in the signature when verifying  votes in the `vote()` method

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L48

## Issue Description
Using deadlines is a standard security measure when creating and verifying signatures. It ensures that generated signatures cannot be saved(or stolen) to be used at some later date than it was originally intended and possibly gain some advantages from it.


## Recommended Mitigation Steps
Consider including a deadline when generating the signature off-chain and provide it as a parameter to `vote()` so that it can be verified that it is not expired. This will provide additional line of defence against malicious uses of signatures

## [L-03] If exchange start is not approved after the Boostrap Ballot has completed it will never be launched

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

## [L-04] No limit on the string length of proposal descriptions

## Issue Description
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L155

When creating a proposal to be voted by the DAO the user can provide a description that would give information about the proposal. However in none of the proposal creation functions inside the `Proposals.sol` contract exists a restriction for the length of the string. This allows a proposer to maliciously store a giant piece of text in the proposal. This in turn would generate a lot of additional costs to the DAO members who vote. Since the EVM would have to load that giant text in memory every time someone want to vote on it (gas progressively increases with the size of the string). The additional gas costs of those inefficient memory loads would have to be paid by the voters.

Take a look at this article that explores the gas costs of long strings in Solidity -> [Article](https://medium.com/@marvinirwin/gas-estimation-and-large-strings-in-the-ethereum-virtual-machine-feat-remix-and-klaytn-649e6db073f)

## Recommended Mitigation Steps
Consider using the following check to prevent long description:
```solidity
require(bytes(description).length <= someReasonableLength,"description too long")
```

## [L-05] Discrepancy between documented functionality and implementation when creating `SendSalt` proposals

## Issue Description
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L205-L208

According to the description in `proposeSendSALT()` function only a single active ballot for sending Salt tokens from the DAO can exists at a time. The description  above `_possiblyCreateProposal()` says that if more than one wallet should be specified to receive the SALT tokens a splitter can be used. However nowhere in the contract exist a logic that can handle more than one recipient for a `SendSalt` proposal.

```solidity
 function proposeSendSALT(
        address wallet,
        uint256 amount,
        string calldata description
    ) external nonReentrant returns (uint256 ballotID) {
        ....

        // This ballotName is not unique for the receiving wallet and enforces the restriction of one sendSALT ballot at a time.
        // If more receivers are necessary at once, a splitter can be used.
        //@audit [L] -> no splitter is set
        string memory ballotName = "sendSALT";
        return
            _possiblyCreateProposal(
                ballotName,
                BallotType.SEND_SALT,
                wallet,
                amount,
                "",
                description
            );
    }
```

## Issue Description

If the protocol should be able to handle multiple SALT receivers then some string parsing logic should be implemented inside  the above function to extract the address from a string (e.g "address,address,address") or the address parameter should be an array

## [L-06] No check if provided `swapAmountIn` is 0 inside `Pools.swap()`

## Issue Description
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L366

Inside the swap function no check is made if the provided swap token amount is 0. This would cause useless calculations to be made and gas to be wasted.

## Issue Description
Add a validation if `swapAmountIn` parameters is not 0

## [L-07] No check if provided `token` is wbtc/weth inside `Proposals.proposeTokenWhitelisting()`

## Issue Description
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L162

Inside the function for proposing a new token whitelisting no validation is made if provided token is `wbtc` or `weth`. Every token in the protocol is pooled with both `weth` and `wbtc` so there is no point in creating pools paired with the same token `weth/weth` or `wbtc/wbtc`. 

## Issue Description
Consider adding the following validation inside `proposeTokenWhitelisting()`:

```solidity

 require(address(token) != exchangeConfig.wbtc(), "token cannot be wbtc");
 require(address(token) != exchangeConfig.weth(), "token cannot be weth");

```

This will prevent the creation of invalid pools with the same token