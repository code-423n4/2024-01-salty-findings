# LOW-001: Malleable signatures are not rejected by `SigningTools` library

[SigningTools.\_verifySignature](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L11:L29)

[AccessManager.\_verifyAccess](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L51:L56)

## Description

Signatures with high _s-values_ (i.e. _s-value_ greater than `secp256k1n/2`) must be considered invalid in order to comply with [EIP-2](https://eips.ethereum.org/EIPS/eip-2). However, `ecrecover` allows for malleable (non-unique) signatures due to the ECDSA recover precompiled contract remains unchanged.

The `SigningTools._verifySignature` function overlooks the _s-value_ range:

```solidity
// Verify that the messageHash was signed by the authoratative signer.
function _verifySignature(bytes32 messageHash, bytes memory signature ) internal pure returns (bool)
    {
    require( signature.length == 65, "Invalid signature length" );

    bytes32 r;
    bytes32 s;
    uint8 v;

    assembly
        {
        r := mload (add (signature, 0x20))
        s := mload (add (signature, 0x40))
        v := mload (add (signature, 0x41))
        }

    // @audit-issue s-value range is not checked
    address recoveredAddress = ecrecover(messageHash, v, r, s);

    return (recoveredAddress == EXPECTED_SIGNER);
    }
```

## Impact

Currently there is no impact due to `SigningTools._verifySignature` is only being used by the `AccessManager.grantAccess` logic (which uses the `geoVersion` value as a replay protection nonce, and where allowing access a wallet more than once has no security impact). However, devs must be aware of the risks of using the `SigningTools` library elsewhere.

## Proof Of Concept

NA

## Tools Used

Manually reviewed.

## Recommended Mitigation Steps

Either use the latest [OpenZeppelin ECDSA cryptography library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/01ef448981be9d20ca85f2faf6ebdf591ce409f3/contracts/utils/cryptography/ECDSA.sol) (recommended, as it has been audited and not only checks that the signature is not malleable but also checks that the signer address is not the zero address) or make sure to implement the logic that validates the _s-value_ range in the `SigningTools._verifySignature` function.

# LOW-002 - Improve `setInitialFeeds` and `setPriceFeed` price feed addresses validations

## Description

Both `PriceAggregator.setInitialFeeds` and `PriceAggregator.setPriceFeed` functions do not prevent from setting an address more than once across the price feed storage variables (i.e. `priceFeed1`, `priceFeed2` and `priceFeed3`). Moreover, the `PriceAggregator.setPriceFeed` function does not prevent from setting the same existing value (which in fact ends up setting `priceFeedModificationCooldownExpiration` further in the future).

## Impact

Setting a duplicated price feed address by mistake has the following consequences:

- It makes the price aggregation logic to return the price of the duplicated oracle.
- It increases the risk of an oracle manipulation attack when an on-chain oracle address (i.e. Uniswap TWAP, Salty pool) is duplicated.
- It halts the protocol if the duplicated oracle stops working (due to the price aggregation logic requiring at least two sources different from zero).

Worth mentioning that it takes at least 30 days (up to 45 days) to amend a single price feed address (due to `PriceAggregator.setPriceFeed` only allowing to set one address at a time and setting `priceFeedModificationCooldownExpiration` several days ahead).

For these reasons it is worth adding checks on the price feed addresses to be stored.

## Proof Of Concept

NA

## Tools Used

Manually reviewed.

## Recommended Mitigation Steps

A trivial solution for the `PriceAggregator.setInitialFeeds` function is to check the addresses against each other:

```solidity
    function setInitialFeeds(
        IPriceFeed _priceFeed1,
        IPriceFeed _priceFeed2,
        IPriceFeed _priceFeed3
    ) public onlyOwner {
        require(
            address(priceFeed1) == address(0),
            "setInitialFeeds() can only be called once"
        );
        // HERE
        require(
            address(_priceFeed1) != address(_priceFeed2),
            "setInitialFeeds() feed1 == feed2"
        );
        // HERE
        require(
            address(_priceFeed2) != address(_priceFeed3),
            "setInitialFeeds() feed2 == feed3"
        );
        // HERE
        require(
            address(_priceFeed3) != address(_priceFeed1),
            "setInitialFeeds() feed3 == feed1"
        );

        priceFeed1 = _priceFeed1;
        priceFeed2 = _priceFeed2;
        priceFeed3 = _priceFeed3;
    }
```

A trivial solution for the `PriceAddregator.setPriceFeed` function is to check that the new address (i.e. `newPriceFeed`) does not equal the existing one:

```solidity
    function setPriceFeed(
        uint256 priceFeedNum,
        IPriceFeed newPriceFeed
    ) public onlyOwner {
        // If the required cooldown is not met, simply return without reverting so that the original proposal can be finalized and new setPriceFeed proposals can be made.
        if (block.timestamp < priceFeedModificationCooldownExpiration) return;

        if (priceFeedNum == 1) {
            // HERE
            require(
                priceFeed1 != newPriceFeed,
                "setPriceFeed() priceFeed1 already set"
            );
            priceFeed1 = newPriceFeed;
        } else if (priceFeedNum == 2) {
            // HERE
            require(
                priceFeed2 != newPriceFeed,
                "setPriceFeed() priceFeed2 already set"
            );
            priceFeed2 = newPriceFeed;
        } else if (priceFeedNum == 3) {
            // HERE
            require(
                priceFeed3 != newPriceFeed,
                "setPriceFeed() priceFeed3 already set"
            );
            priceFeed3 = newPriceFeed;
        } else {
            revert("setPriceFeed() unsupported case");
        }

        priceFeedModificationCooldownExpiration =
            block.timestamp +
            priceFeedModificationCooldown;
        emit PriceFeedSet(priceFeedNum, newPriceFeed);
    }
```

More complex solutions could be adopted, for instance making sure that the value to be stored in a price feed storage variable only belongs to a particular oracle address (e.g. only Chainlink price feeds can be stored in `priceFeed2`).
