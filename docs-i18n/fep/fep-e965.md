---
slug: "e965"
authors: bumblefudge <bumblefudge@learningproof.xyz>, bengo <@bengo@social.coop>
status: DRAFT
dateReceived: 2024-07-05
discussionsTo: https://socialhub.activitypub.rocks/t/fep-0f2a-announce-activity-for-migrations-and-tombstone-events/4349
trackingIssue: https://codeberg.org/fediverse/fep/issues/352
---
# FEP-e965: Move Activity for Migrations and Announce Activity for Tombstone Events
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-e965.md)から翻訳に協力することができます。

## Summary

This FEP normatively specifies exactly one narrow step in almost all the migration user-stories defined in [FEP-73cd: User Migration Stories][FEP-73cd]:

* the updates to an Actor object made after a migration and/or deactivation event, and
* the Move activity which a source server propagates to inform followers of said Actor object update

Our proposal clarifies semantics and behavior of the earlier [FEP-7628][FEP-7628] on which it strictly relies.
It also proposes a simple, additive approach to use the above to express "deactivated" Actors by "tombstoning" their Actor objects, i.e. adding "Tombstone" to their `type` array (already afforded by the Activity Streams vocabulary).
It also accomodates migrations to new forms of Actor object, such as "Nomadic"-style Portable Actors as described in [FEP-ef61: Portable Objects][FEP-ef61] and "Independently-hosted" Actor objects as described in [FEP-7952][FEP-7952], both for conforming and non-conforming consumers.
As such, fully implementing all optional features of this proposal would require implementing [FEP-521a: Representing actor's public keys][FEP-521a], which adds terms to the Actor object for publishing a verification method to verify assertions about the Actor independently of domain.

### Current Approaches

Migration is currently supported in a somewhat ad hoc and piecemeal way, without harmonized expectations for how to update, announce, or interpret an Actor object after a migration, or after a deactiviation.
Deactivation is sometimes referred to as a "tombstone" event, both in general usage in distributed systems and in the Activity Streams sense of the [Tombstone](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-tombstone) object type.
Adding the "Tombstone" member to the `type` array of an Actor object marks it as deactivated, as is already possible but more commonly implemented for deleted content or Activities than for Actors.
We did not do a review of existing codebases, and the only public prior art that we're aware of is the retrospecification of current practice in [FEP-7628: Move Actor][FEP-7628].

Beyond passively leaving a `Tombstone` hint for future queries, there have been no public proposals to our knowledge specifying how to actively express a given Actor's controller expressing an "intent [for that Actor] to be forgotten" to other servers where interactions with it may be stored, much less how to document that intent for legal purposes (which is explicitly out of scope here).

## Specification

### Conformance

MUST, MAY, and SHOULD used in the [RFC-2119] sense where they appear in CAPITAL LETTERS. Similarly, the references to "valid" URIs throughout should be interpreted as conforming to both [RFC-3987][] and the [Activity Streams guidance on URI usage][Activity Streams URIs].

Implementations SHOULD signal their support for this specification by including `"https://w3id.org/fep/7628"` in the `@context` array of their Actors, as this will clearly signal that the _ABSENCE_ of a `movedTo` or `copiedTo` property indicates a currently-active Actor.

Implementations MAY prove support for this specification by publishing a Conformance Report referencing the tests run.
A specification for possible tests is provided in [fep-e965-test-case](./fep-e965-test-case.md).

### Actor Object Migration and Deactivation Syntax

In the section, ["Move Activity"](https://codeberg.org/fediverse/fep/src/branch/main/fep/7628/fep-7628.md#move-activity) of [FEP-7628][], two variations of the Mastodon-style `Move` Activity are defined, as well as semantics for the `movedTo` and `copiedTo` properties that MUST be applied to the Actor object on the source server of the activity:

> If previous primary actor is deactivated after migration, it MUST have movedTo property containing the ID of the new primary actor. [...]
If previous primary actor is not deactivated, copiedTo property MUST be used.

We add a few more explicit requirements:

* `movedTo` MUST be a string or an array containing 1 string.
* `copiedTo` MUST be a string or an array of strings.
* both `movedTo` and `copiedTo` MUST NOT be present in the same Actor object.
* Consuming implementations SHOULD treat an Actor with both properties as malformed.

Many other current and future process and Activities could also be using the same semantics, including new "styles" or "profiles" of the many possible Actor objects allowed by the [ActivityPub] specification.
These include Actors that *do not change `id` after migrating*, whether they conform to the [Nomadic][FEP-ef61] Actor extension, or to the [separately-hosted][FEP-7952] Actor extension.
If an account is moving to one of these configurations, the `movedTo` or `copiedTo` value will be the `id` and location of an `ap://` URL, or to a URL controlled by the Actor object's data subject, respectively.

If the Actor object before the deactivation event included a public key for signing Activities expressed according to [Client-Signing][FEP-521a], and the same public key will NOT be published at the destination server for verifying post-migration Activities, then the source server MAY add an `expires` key and current-timestamp value to the key's `assertionMethod` object as described in [section #2.3.1: Verification Methods](https://www.w3.org/TR/vc-data-integrity/#verification-methods) of the [W3C Data Integrity](https://www.w3.org/TR/vc-data-integrity) specification (to which [FEP-521a] normatively refers).
Any consumer fetching this `assertionMethod` object for the purposes of verifying signatures according to the Data Integrity algorithm will thus invalidate signatures newer than the deactivation of that key.

If an account has been deleted intentionally and consuming implementations are expected to recognize this, regardless of whether or not a `movedTo` value has been set, a server MUST include the string "Tombstone" in the `type` array of the deactivated or moved Actor object.
Whether any other types are present is out of scope of this specification, to minimize side effects or complications for implementers.

If an account whose Actor object containing a valid `copiedTo` value has been deleted intentionally, this MUST be removed before adding `Tombstone` to the Actor's `type`.
A server performing this removal MAY move one valid URI from `copiedTo` to `movedTo` to aid discovery.

If a user account is being deactivated but the source server wants to enable a future migration to be authenticated cryptographically, it MAY add to the Actor object a public key authenticated to the account (if not already present), as per to [FEP-521a].

An Actor object set to `Tombstone` SHOULD also set a top-level [`as:deleted`](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-deleted) property containing a current XSD `dateTime` as a courtesy to consumers.

### Announcing a Migration or Deactivation Event

After these changes have been made to the Actor object on the source server, it should be announced according to the type of changes made.
If `movedTo` or `copiedTo` have been changed, then a [FEP-73cd][]-compatible Move activity with the Actor as its object SHOULD be sent out, at least to all of that Actor's Followers.
If the Actor is being retyped to `Tombstone`, an Announce activity with the Actor as object should be sent out, at least to all of that Actor's followers.

If a user account is being deactivated but the source server wants to enable a future migration to be authenticated cryptographically, it is RECOMMENDED that the Announce activity be signed as per [FEP-8b32].
In this way, a later Move activity **signed by the same key** could, at least theoretically, be accepted from a new server which authenticates it by [FEP-8b32].

### Interpreting a Migrated or Deactivated Actor Object

In the section, ["`movedTo` and `copiedTo` properties"](https://codeberg.org/fediverse/fep/src/branch/main/fep/7628/fep-7628.md#movedto-and-copiedto-properties) of [FEP-7628][], the following general rule for all Actor objects is proposed:

> Publishers SHOULD NOT deliver activities to actor's inbox if movedTo property is present.

We add the following behavioral expectations:

* Publishers SHOULD attempt to resolve the `movedTo` property to find out if it contains an inbox property.
  * If an inbox is found, publishers SHOULD apply security, privacy, and federation policies on the domain at which it is hosted before taking any further action.
  * If said inbox is permitted, publishers SHOULD attempt to deliver activities to the new inbox.
* If no `movedTo` value is set and one or more `copiedTo` values are set, publishers MAY resolve a `copiedTo` value to retrieve an `inbox` value and similarly process it.
  * In the case of a valid `copiedTo` inbox and allowance by policy, delivery MAY attempt delivery to both Actor inbox and `copiedTo` inbox(es).
* Consuming implementations that keep redirect or alias records MAY persist the above-resolved relationship to avoid repeating this resolution in the future.
* If a `movedTo` value has been set to a valid URI, but `type` does not include "Tombstone", consuming implementations SHOULD treat it as a deactivated actor per Postel's Law.

#### Behavior for unfamiliar Actor URIs

There are caveats to interpreting these values if the `movedTo` or `copiedTo` properties contain unconventional URLs, such as those generated by an implementation extended by the above-mentioned FEPs:

* If the `movedTo` or `copiedTo` value is a valid URL beginning with the prefix `ap://` and the `@context` value includes the relevant extension properties, the destination server of the migration is likely implementing [FEP-ef61] and may require custom resolution logic to return an Actor object.
* Similarly, if the `movedTo` or `copiedTo` value contains an actor-relative URL of the type defined in [FEP-7952], it should resolve as usual if the server is live, as long as the querying implementation allows for the HTTP redirect and has no policy against (or hardcoded assumptions incompatible with) `inbox` values on different domains than `id` values for a given Actor.
* If an actor returned contains a non-empty `movedTo` or a non-empty `copiedTo` value in turn, this should in turn be dereferenced, barring domain-based policies to the contrary.
* If a querying implementation cannot resolve a value of these types or further indirections, it SHOULD consider them equivalent to URLs that return 404 and MAY log an error or warning to user or system log as appropriate.
* It is RECOMMENDED that unresolvable `movedTo` values be displayed to end-users as corrupted or incomplete moves, rather than as deactivated accounts.

### Interpreting a Move Activity or an Announce Activity of a Deactivated Actor

Servers receiving a Move or an Announce activity with an Actor as its object SHOULD NOT increment a `shares` collection.
If a receiving server persists redirects or aliases to more smoothly remain aware of migrating or multi-homed users, or for other reasons, it MAY resolve the new Actor object and perform the above-described checks and MAY record said Actor update.

## Open Issues

1. Are there others to which an Actor-update Move or Announce should be addressed beyond just the old Actor's [Followers, somehow imported](https://socialhub.activitypub.rocks/t/fep-7628-move-actor/3583/3)? is it worth calling out server-instance Actors, since they might also want to know for... idunno moderation reasons?
2. Announce Activity example
3. Address Actor Equivalence Attestation objects explicitly, or leave up to implementer imagination?

## 参考文献

* [FEP-521a: Representing actor's public keys][FEP-521a]
* [FEP-73cd: Migration User Stories][FEP-73cd]
* [FEP-7628: Move Actor][FEP-7628]
* [FEP-7952: Roadmap for Actor and Object Portability][FEP-7952]
* [FEP-8b32: Object Integrity Proofs][FEP-8b32]
* [FEP-cd47: Federation-friendly Addressing and Deduplication Use-Cases][FEP-cd47]
* [FEP-ef61: Portable Objects][FEP-ef61]

* Christine Lemmer Webber, Jessica Tallon, [ActivityPub][AP], 2018
* S. Bradner, Key words for use in RFCs to Indicate Requirement Levels, 1997
* Dave Longley, Manu Sporny, [Verifiable Credential Data Integrity 1.0][DI for VCs], 2023
* Manu Sporny, Dave Longley, Markus Sabadell, Drummond Reed, Orie Steele,  Christopher Allen, [Decentralized Identifiers][DID] (DIDs) v1.0, 2022
* Dave Longley, Manu Sporny, [Data Integrity EdDSA Cryptosuites][DI Sigs] v1.0, 2023
* A. Rundgren, B. Jordan, S. Erdtman, [JSON Canonicalization Scheme][JCS] (JCS), 2020

[FEP-521a]: https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md
[FEP-73cd]: https://codeberg.org/fediverse/fep/src/branch/main/fep/73cd/fep-73cd.md#migration-user-stories
[FEP-7628]: https://codeberg.org/fediverse/fep/src/branch/main/fep/7628/fep-7628.md
[FEP-7952]: https://codeberg.org/bumblefudge/fep/src/branch/fep-7952--roadmap-for-actor-and-object-portability/fep/7952/fep-7952.md
[FEP-8b32]: https://codeberg.org/fediverse/fep/src/branch/main/fep/8b32/fep-8b32.md
[FEP-cd47]: https://codeberg.org/fediverse/fep/src/branch/main/fep/cd47/fep-cd47.md
[FEP-ef61]: https://codeberg.org/fediverse/fep/src/branch/main/fep/ef61/fep-ef61.md

[AP]: https://www.w3.org/TR/activitypub/
[Activity Streams URI]: https://www.w3.org/TR/activitystreams-core/#urls
[DI Sigs]: https://w3c.github.io/vc-di-eddsa/#eddsa-jcs-2022
[DI for VCs]: https://w3c.github.io/vc-data-integrity/
[DID]: https://www.w3.org/TR/did-core/
[JCS]: https://www.rfc-editor.org/rfc/rfc8785
[RFC-2119]: https://tools.ietf.org/html/rfc2119.html
[RFC-3987]: https://tools.ietf.org/html/rfc3987.html

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement
Proposal have waived all copyright and related or neighboring rights to this work.
