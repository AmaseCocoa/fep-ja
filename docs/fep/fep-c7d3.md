---
slug: "c7d3"
authors: silverpill <@silverpill@mitra.social>
status: WITHDRAWN
dateReceived: 2024-06-04
dateWithdrawn: 2025-02-21
discussionsTo: https://socialhub.activitypub.rocks/t/fep-c7d3-ownership/4292
trackingIssue: https://codeberg.org/fediverse/fep/issues/323
---
# FEP-c7d3: Ownership
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-c7d3.md)から翻訳に協力することができます。

## Summary

In this document we discuss the concept of ownership, as applied to [ActivityPub] objects.

>[!WARNING]
>This FEP has been superseded by [FEP-fe34](https://codeberg.org/fediverse/fep/src/branch/main/fep/fe34/fep-fe34.md).

## Rationale

[ActivityPub] standard does not specify authentication and authorization mechanisms.

Many implementations use `actor` and `attributedTo` properties (defined in [Activity Vocabulary][ActivityVocabulary]) to determine the validity of activities and objects. This proposal attempts to formalize the current practices and provide guidance for implementers.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119].

## Ownership

Ownership is indicated by a property of an ActivityPub object. The name of this property differs depending on the object type:

- Owner of an actor is indicated by its `id` property.
- Activities have an `actor` property, which describes the actor that performed the activity. This actor is considered to be the owner of the activity.
- An object (that is, not an actor and not an activity) can have an `attributedTo` property, which describes the actor to which the object is attributed. This actor is considered to be the owner of the object.
- Public keys and verification methods have `owner` and `controller` properties.

The owner of an object MUST be an actor.

>[!WARNING]
>According to [Activity Vocabulary][ActivityVocabulary], `actor` and `attributedTo` properties can contain references to multiple actors. These scenarios are not covered by this document and implementers are expected to determine the appropriate authentication and authorization procedures on a case-by-case basis.

>[!NOTE]
>In subsequent sections, "objects" and "activities" will be referred to as simply "objects".

## Origin

Object identifiers are grouped together into protection domains called "origins". This concept is similar to the "web origin" concept described in [RFC-6454], and origins of object IDs are computed by the same algorithm.

The same-origin policy determines when a relationship between objects can be trusted.

>[!NOTE]
>There might be other ways to establish trust, but they are not covered by this document.

## Identifiers and ownership

Identifier of an object and identifier of its owner MUST have the same origin.

## Authentication

The object is considered authentic if any of the following conditions are met:

1. It was fetched from the location that has the same origin as its owner's ID.
2. It was delivered to inbox and the `POST` request contained a valid [HTTP signature][HttpSig] created using a key whose owner has the same origin as the object owner.
3. It contains a valid [FEP-8b32] integrity proof created using a key whose owner has the same origin as the object owner.
4. If it is embedded within another object, and its owner has the same origin as the owner of the containing object.

If none of these conditions are met, the object MUST be discarded.

If signature verification is performed, the key owner SHOULD match the object owner.

>[!NOTE]
>In some cases, consumers can process unauthenticated objects if the risk is deemed acceptable.

### Delivered to inbox

If the object was delivered to inbox and its authentication fails, the recipient SHOULD fetch it and repeat the authentication procedure.

### Emdedded objects

If the embedded and the containing objects have owners with different origins, the authenticity of the embedded object MUST be verified independently either by fetching it from the server of origin, or by verifying its [FEP-8b32] integrity proof.

### Anonymous objects

An object without an ID can only exist when embedded within another object. It has the same owner as the parent object and it is considered authentic when the parent object is authentic.

### Unattributed objects

An object without an explicit owner is owned by the server. Such object MUST be considered authentic only if fetched from the location that has the same origin as its ID.

## Authorization

The actor that creates the object MUST be its owner.

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

## 参考文献

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
[HttpSig]: https://swicg.github.io/activitypub-http-signature/

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
