# Past paid bounties

_This list includes valid submissions from past and current contract versions for which a bounty has been paid._

## Potential suicide of MultiSend library

We use a [MultiSend](https://github.com/safe-global/safe-contracts/blob/v1.2.0/contracts/libraries/MultiSend.sol) library to batch multiple transactions together. A transaction could be created that would self-destruct the contract. While this would not have put any funds at risk, user experience would have been seriously impacted.

We have updated the library as well as our interfaces. Details about the fix can be found on [Github](https://github.com/safe-global/safe-contracts/pull/156).

This bug was submitted by [Micah Zoltu](https://twitter.com/micahzoltu). It was regarded as a "Low Threat", and a bounty of 1,000 USD has been paid out.

## Transaction failure when receiving funds via `transfer` or `send`

Since the beginning of the bug bounty period, the contract update has been live on the Ethereum Mainnet. We performed extensive internal testing and discovered an edge case where a Safe could not receive funds from another contract via `send` or `transfer`. This was due to additional gas costs caused by the [emission of additional events](https://github.com/safe-global/safe-contracts/pull/135) and [gas price changes](https://eips.ethereum.org/EIPS/eip-1884) in the latest hard fork. This issue has been fixed, and more details can be found on [GitHub](https://github.com/safe-global/safe-contracts/issues/149).

## Duplicate owners during setup could render Safe unusable

A bug in the `setupOwners` function on `OwnerManager.sol` allows duplicate owners to be set when the duplicated address is next to itself in the `_owners` array. This could cause unexpected behavior. While stealing funds from existing Safes is impossible, it is unexpected, and user funds might be locked. During Safe creation, the threshold of a Safe could be set to something unreachable, making it impossible to execute a transaction afterward.

The Safe interfaces prevent this by checking for duplicates, but if users directly interact with the contracts, this can still happen. The issue is tracked on [GitHub](https://github.com/safe-global/safe-contracts/issues/244).

This bug was submitted by [David Nicholas](https://twitter.com/davidnich11). It was regarded as a "Medium Threat", and a bounty of 2,500 USD has been paid out.

## Setting a Safe as an owner of itself essentially reduces the threshold by 1

The contracts allow to set a Safe as an owner of itself. This has the same effect as lowering the threshold by 1, as it is possible for anyone to generate a valid signature for the Safe itself when triggering `execTransaction`. This is especially an issue for Safes with a threshold of 1. Anyone can execute transactions if a Safe with threshold 1 adds itself as an owner.

To our knowledge, there is no actual use case where it would make sense to set a Safe as an owner of itself. Hence, only a few number of Safes used themselves as owners. Most of these Safes could be contacted, and the Safe has been removed as an owner. The Safes still affected are Safes used for testing by us or Safes owned by a single owner with a threshold > 1 (so no immediate risk).

To fix this, the next contract update will prevent the Safe as its owner via `require(owner != address(this), "Safe can't be an owner")`. This check can be performed when adding owners and/or when checking signatures.

Details about this issue can be found on [GitHub](https://github.com/safe-global/safe-contracts/issues/229).

The bug was submitted by [Kevin Foesenek](https://github.com/keviinfoes). It was regarded as a "Medium Threat", and a bounty of 5,000 USD has been paid out.

## The function `getModulesPaginated` does not return all modules

The method [getModuledPaginated](https://github.com/safe-global/safe-contracts/blob/v1.3.0/contracts/base/ModuleManager.sol#L114) is used to return enabled modules page by page. For this, a `start` and a `pageSize` need to be specified, and the method will return an array of Safe Module addresses and `next`. This next can be used as the `start` to load the next page. When another page exists, then `next` is a module address. This module address, however, will not be present in any of the returned arrays. While this does not put any user assets at risk directly, it could lead to a wrong perception of the enabled modules of a Safe and, thereby, its state.

The workaround is to append the `next` to the returned array of module addresses if it is not the zero or sentinel address. Alternatively, the last element of the returned array can be used as the `start` for the next page.

This bug was submitted by [Renan Souza](https://github.com/RenanSouza2). It was regarded as a "Low Threat", and a bounty of 2,000 USD has been paid out.
