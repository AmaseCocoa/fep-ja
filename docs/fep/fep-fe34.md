---
slug: "fe34"
authors: silverpill <@silverpill@mitra.social>
status: DRAFT
dateReceived: 2024-11-15
discussionsTo: https://socialhub.activitypub.rocks/t/fep-c7d3-ownership/4292
trackingIssue: https://codeberg.org/fediverse/fep/issues/445
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-fe34.md)から翻訳に協力することができます。
# FEP-fe34: Origin-based security model

## Summary

Developing a comprehensive [ActivityPub] security framework based on the concept of [web origin][RFC-6454].

## Rationale

[ActivityPub] standard does not specify authentication and authorization mechanisms. However, in some cases it hints at the importance of the web origin:

>**3. Objects**

>... Servers SHOULD validate the content they receive to avoid content spoofing attacks. (A server should do something at least as robust as checking that the object appears as received at its origin, but mechanisms such as checking signatures would be better if available).

>**7.3 Update Activity**

>... The receiving server MUST take care to be sure that the Update is authorized to modify its object. At minimum, this may be done by ensuring that the Update and its object are of same origin.

Implementations often rely on origin and ownership checks for determining the validity of activities and objects, but exact requirements are not documented and can be easily overlooked, leading to vulnerabilities such as [GHSA-3fjr-858r-92rw](https://github.com/mastodon/mastodon/security/advisories/GHSA-3fjr-858r-92rw).

This proposal attempts to formalize existing practices and provide guidance for implementers.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119].

## Origin

Object identifiers are grouped together into protection domains called "origins". This concept is similar to the "web origin" concept described in [RFC-6454], and origins of object IDs are computed by the same algorithm.

The same-origin policy determines when a relationship between objects can be trusted.

>[!NOTE]
>There might be other ways to establish trust, but they are not covered by this document.

## Authentication

ActivityPub object is considered authentic if any of the following conditions are met:

1. It was fetched from the location that has the same origin as its ID.
2. It was delivered to inbox and the request contained a valid [HTTP signature][HttpSig] created using a key whose ID has the same origin as the object ID.
3. It contains a valid [FEP-8b32] integrity proof created using a key whose ID has the same origin as the object ID.
4. If it is embedded within another object, and its ID has the same origin as containing object ID.

If none of these conditions are met, the object MUST be discarded.

Consumers SHOULD attempt to fetch the object by its ID if other authentication methods are not available.

>[!NOTE]
>In some cases, consumers can process unauthenticated objects if the risk is deemed acceptable.

### Anonymous objects

An object without an ID can only exist when embedded within another object. It is considered authentic when the parent object is authentic.

## Ownership

Ownership is indicated by a property of an ActivityPub object. The name of this property differs depending on the object type:

- Owner of an actor is indicated by its `id` property.
- Activities have an `actor` property, which describes the actor that performed the activity. This actor is considered to be the owner of the activity.
- An object (that is, not an actor and not an activity) can have an `attributedTo` property, which describes the actor to which the object is attributed. This actor is considered to be the owner of the object.
- Public keys and verification methods have `owner` and `controller` properties.

The owner of an object MUST be an actor.

Identifier of an object and identifier of its owner MUST have the same origin.

>[!NOTE]
>This document uses terms "actor", "activity", "collection" and "object" according to the classification given in [FEP-2277].

>[!WARNING]
>According to [Activity Vocabulary][ActivityVocabulary], `actor` and `attributedTo` properties can contain references to multiple actors. These scenarios are not covered by this document and implementers are expected to determine the appropriate authorization procedures on a case-by-case basis.

>[!NOTE]
>In subsequent sections, "objects" and "activities" will be referred to as simply "objects".

## Authorization

The actor that creates an object MUST be its owner.

If activity modifies or deletes an object, its owner SHOULD match the object's owner. If owners are different, their IDs MUST have the same origin.

Examples:

- `Create`, `Update` and `Delete` activities, and objects indicated by their `object` property SHOULD have the same owner.
- `Undo` activity and object indicated by its `object` property SHOULD have the same owner.
- `Add` and `Remove` activities, and objects indicated by their `target` property SHOULD have the same owner.
- `Announce` and `Like` activities don't modify objects indicated by their `object` property, therefore their owners can be different.

### Ownership transfer

When ownership changes, the new owner ID MUST have the same origin as the old owner ID.

### Access control

When a protected object is fetched, the `GET` request MUST contain a [HTTP signature][HttpSig] created using a key whose owner SHOULD belong to object's intended audience. If key owner doesn't belong to intended audience, its ID MUST have the same origin as one of the actors in object's intended audience.

### Implicit ownership

In some cases ownership can be implicit. Examples:

- Inbox and outbox collections are expected to be owned by the actor to which they are attached.
- All pages of a collection are expected to be owned by the same actor.

Authorization recommendations provided in this document still apply in such cases.

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- James M Snell, Evan Prodromou, [Activity Vocabulary][ActivityVocabulary], 2017
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119], 1997
- A. Barth, [The Web Origin Concept][RFC-6454], 2011
- silverpill, [FEP-8b32: Object Integrity Proofs][FEP-8b32], 2022
- Ryan Barrett, nightpool, [ActivityPub and HTTP Signatures][HttpSig], 2024

[ActivityPub]: https://www.w3.org/TR/activitypub/
[ActivityVocabulary]: https://www.w3.org/TR/activitystreams-vocabulary/
[RFC-2119]: https://tools.ietf.org/html/rfc2119.html
[RFC-6454]: https://www.rfc-editor.org/rfc/rfc6454.html
[FEP-8b32]: https://codeberg.org/fediverse/fep/src/branch/main/fep/8b32/fep-8b32.md
[FEP-2277]: https://codeberg.org/fediverse/fep/src/branch/main/fep/2277/fep-2277.md
[HttpSig]: https://swicg.github.io/activitypub-http-signature/

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
