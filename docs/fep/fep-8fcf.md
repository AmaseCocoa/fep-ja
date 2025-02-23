---
slug: "8fcf"
authors: Claire <claire.fep-1d7d@sitedethib.com>
status: FINAL
dateReceived: 2020-10-24
dateFinalized: 2022-02-07
trackingIssue: https://codeberg.org/fediverse/fep/issues/6
discussionsTo: https://codeberg.org/fediverse/fep/issues/6
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-8fcf.md)から翻訳に協力することができます。
# FEP-8fcf: Followers collection synchronization across servers

## Summary

In ActivityPub, follow relationships are established, updated and removed by
sending activities such as `Follow`, `Accept` or `Reject`, which are assumed to
be correctly and promptly processed upon receipt.

However, due to incompatible protocol extensions, software bugs, server crashes
or database rollbacks, the two ends of a `Follow` relationship may end up out of
sync.

This can be especially damaging when a remote instance has outdated information
about follow relationships that should have been revoked, as some
implementations may deliver activities addressed to the sender's `followers`
collection by using the `sharedInbox` mechanism and letting the recipient use
the sender's `followers` collection for local delivery and access control.

This proposal describes an optional mechanism for detecting discrepancies in
following relationships across instances, with minimal overhead and without loss
of privacy.

## Requirements

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”,
“SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this
specification are to be interpreted as described in [RFC-2119].

The proposed protocol for followers collection synchronization makes a number of
assumptions that may not be suitable to every implementation or deployment.

Implementations and deployments MUST NOT implement the mechanisms described in
this proposal unless they match the following requirements:
- actors managed by an instance are required to all share the same exact URI
  scheme and authority for their `id`, `inbox` and `sharedInbox` URIs
- such instances are required to manage all actors using the same URI scheme and
  authority for either their `id`, `inbox` or `sharedInbox` URIs (that is, for
  instance, two fediverse implementations cannot implement this proposal if they
  are set up on the same exact same domain name, unless implementing an
  additional mechanism to share follower information between them, which is out
  of scope for this proposal).

The reason for those requirements is to prevent the partial followers collection
described below from missing legitimate followers, which could result in
followers being removed for no reason.

Failing to implement this proposed synchronization mechanism should not impact
compatibility with other implementations, as it is completely optional.

## Partial follower collection

For efficiency and privacy purposes, we consider a subset of an actor's
followers collection. This subset is the set of an actor's followers whose `id`
shares an instance's specific URI scheme and authority.

For instance, if `https://example.org/users/1` has the following followers:
- `https://example.org/users/2`
- `https://testing.example.org/users/1`
- `https://next.example.org/users/foo`
- `https://testing.example.org/users/2`

The partial follower collection of `https://example.org/users/1` for the
instance serving `https://testing.example.org/users/1` is:
- `https://testing.example.org/users/1`
- `https://testing.example.org/users/2`

### Partial follower collection digest

To enable quick checking of partial followers consistency across instances, a
partial follower collection digest is computed.

This digest is created by XORing together the individual SHA256 digests of each
follower's `id`.

```
partialCollectionDigest = SHA256(follower1) XOR SHA256(follower2) XOR ... XOR SHA256(followerN)
```

For instance, the partial follower collection digest of
`https://example.org/users/1` for the instance serving
`https://testing.example.org/users/1` is:
`3a06e99569547f444c352ab7f52e4bab207abec5ca6f07b0045cfdc9723f8fa9 XOR f939a1585d4a8f02ee339210dbe7315d7003476663d6095f7d996fc4bc7a49b6 = c33f48cd341ef046a206b8a72ec97af65079f9a3a9b90eef79c5920dce45c61f`

## The `Collection-Synchronization` HTTP Header

The `Collection-Synchronization` HTTP header provides a mechanism for quickly
checking whether the sender's followers collection part that is relevant to the
recipient is consistent with the recipient's knowledge.

The header field name is `Collection-Synchronization` and its value is a list of
parameters and values, formatted according to the `signature` syntax defined in
[HTTP-Signatures], Section 4.1.

Example:
```
Collection-Synchronization: collectionId="https://example.org/users/1/followers", url="https://example.org/users/1/followers_synchronization", digest="c33f48cd341ef046a206b8a72ec97af65079f9a3a9b90eef79c5920dce45c61f"
```

### Collection Synchronization Header Parameters

The `Collection-Synchronization` header's parameters are defined as follows:

- `collectionId`: this is URI of the collection that supports synchronization.
  It must be the sender's `followers` collection.
- `url`: this is the URL of the partial followers collection intended for the
  receiving instance.
  Accessing it should require authentication from the receiving instance.
- `digest`: the partial follower collection digest intended for the receiving
  instance.

## Synchronization procedure

### On the sender end

When delivering an Activity to an `inbox` (or `sharedInbox`), an instance MAY
set a `Collection-Synchronization` header intended for the corresponding
instance (determined by the `inbox` URI scheme and authority).

When exactly to set this header is up to the sender, but it is recommended to
at least send it for any `Create` activity addressed specifically to the
sender's `followers` collection.

### On the receiving end

On the receiving end, upon receiving an Activity delivery with a
signed `Collection-Synchronization` header, the receiver MUST check that:
- the `collectionId` attribute matches the sender's `followers` collection `id`
- the `url` attribute also matches the same authority (so that the instance
  cannot get tricked into requesting the followers list of a third-party
  individual)

If any of those checks fails, the receiver MUST ignore the
`Collection-Synchronization` header.

The receiver SHOULD then compute the partial collection digest for the sender's
followers based on its own knowledge. If the digest does not match the `digest`
attribute of the header, it SHOULD then query the `url`, authenticating itself
to the remote server using [HTTP-Signatures] or another method.

Having fetched the up-to-date partial followers collection from the autoritative
server, the receiving end:
- SHOULD remove from its local copy of the followers collection any local actor
  not listed in the partial followers collection.
- MAY consider any pending outgoing follow listed in the partial followers
  collection as accepted.
- SHOULD send an `Undo Follow` for any other local follower listed in the
  partial followers collection but not known locally.

## Implementations

This proposal is implemented by Mastodon since the following Pull Request: https://github.com/tootsuite/mastodon/pull/14510

## References

- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119.html
- [HTTP-Signatures] A. Backman, J. Richer, M. Sporny, [Signing HTTP Messages](https://tools.ietf.org/html/draft-ietf-httpbis-message-signatures-00.html)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
