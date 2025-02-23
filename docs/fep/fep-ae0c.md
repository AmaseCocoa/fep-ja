---
slug: "ae0c"
authors: Steve Bate <svc-fep@stevebate.net>
status: DRAFT
category: informational
dateReceived: 2024-10-19
trackingIssue: https://codeberg.org/fediverse/fep/issues/424
discussionsTo: https://socialhub.activitypub.rocks/t/fediverse-relays/4626
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-ae0c.md)から翻訳に協力することができます。
# FEP-ae0c: Fediverse Relay Protocols: Mastodon and LitePub

## Summary

Relays are important components within the decentralized Fediverse architecture. They act as intermediary servers that facilitate communication between different instances, enabling users on Fediverse platforms to share public content without requiring actor following relationships.

These relays benefit small instances by enabling them to effectively participate in the federated social network, both as consumers and producers of Fediverse content. 

Several styles of relays existing in the Activity Fediverse. This FEP describe two popular styles of relays:

* [Mastodon-style relays](#mastodon-relay-protocol)
* [LitePub-style relays](#litepub-relay-protocol)

*NOTE: This is an informational FEP documenting the current status quo. It uses [RFC-2119] requirements keywords only as a convenience. Also, these are not standardized protocols. They will generally not be conformant with the ActivityPub standard although they use some concepts from it.*

## Terminology

For the purposes of this document, the following terminology will be used:

| Term | Description |
|------|-------------|
| **relay client actor** | An actor in a server that is a Relay Server subscriber. May also be referred to as a *client actor*. |
| **relay client server** | A server hosting one or more Relay Client Actors. May also be referred to as a *client server*. |
| **relay subscription** | A relationship established between a Relay Client Actor and a Relay Server using an [ActivityPub] `Follow` activity.
| **relay server actor** | An actor in a server that provides relaying of messages between Relay Client Actors. May also be referred to as a *relay actor*.  |
| **relay server** | A server hosting one or more Relay Server Actors. May also be referred to as a *relay server* or a *relay*. |
| **[HTTP Signature][Mastodon HTTP Signatures]** | HTTP-based signature (Cavage) used to verify message sender and contents. |
| **[LD Signature][Mastodon LD Signatures]** | JSON-LD signature used to verify messages regardless of transport. |

## Mastodon Relay Protocol

The Mastodon relay protocol relies on LD Signatures to verify relayed messages. This allows Mastodon to verify the relayed message although it is being sent by a different actor (the relay server actor).

### Relay Client Actor

A Relay Client Actor establishes following relationship with a relay server actor and then processes relayed messages sent to the actor's [ActivityPub] inbox. The relay client server will add the relay inboxes to the delivery target for content with public visibility.

#### Relay Subscription

Mastodon will POST an ActivityPub `Follow` request to the relay ActivityPub `inbox` URI. The `object` of the `Follow` request MUST be the fully expanded URI of the Public pseudo-collection (`https://www.w3.org/ns/activitystreams#Public`). The relay then responds to the  `Follow` request with an `Accept` or `Reject` activity. The response time for the acknowledgement can be arbitrarily long since the subscription MAY require manual approval. 

The request MUST be signed using the same [HTTP Signature][Mastodon HTTP Signatures] (Cavage) algorithms that Mastodon uses for ActivityPub federation. The relay will fetch the relay client actor document to obtain the actor's public key. For best interoperability, the actor ActivityPub document SHOULD be Mastodon-compatible. For example, `preferredUsername` SHOULD be provided in addition to all actor fields required by [ActivityPub] and the actor SHOULD provide a `sharedInbox` endpoint URL.

The relay client actor type SHOULD accurately reflect the actor type. However, note that some relay server implementations constrain the ActivityPub type of a client actor. For example, the relay server implementation might require the client actor to be an `Application` type and reject other types.

**Example Follow Request**

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "https://client.example/6ae15297",
    "type": "Follow",
    "actor": "https://client.example/actor",
    "object": "https://www.w3.org/ns/activitystreams#Public"
}
```

**Example Follow Accept Response**

The `Accept` activity MAY respond with the accepted `Follow` activity URI as the `object` or it may embed a copy of the original `Follow` activity. A `Reject` activity will have a similar structure.

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "https://relay.example/15c0b99f-23d4-4488-ba9d-d0c7bc2876a5",
    "type": "Accept",
    "actor": "https://relay.example/actor",
    "object": {
        "@context": "https://www.w3.org/ns/activitystreams",
        "id": "https://client.example/6ae15297",
        "type": "Follow",
        "actor": "https://client.example/actor",
        "object": "https://www.w3.org/ns/activitystreams#Public"
    }
}
```

#### Relay Unsubscription

To unsubscribe from a relay send an `Undo` with the original `Follow` activity (embedded, or the URI) as the `object`. There is typically no response to the `Undo`.

**Example Undo/Follow Request**

```json
{
    "@context": "https: //www.w3.org/ns/activitystreams",
    "id": "https://client.example/3f5ebd6d",
    "type": "Undo",
    "actor": "https://client.example/actor",
    "published": "2024-10-14T14:42:17.650139+00:00",
    "object": "https://client.example/6ae15297"
}
```


#### Publishing Messages to a Relay

To publish an activity to a Mastodon-style relay, the publisher MUST sign the message using the Mastodon-specific [LD Signature][Mastodon LD Signatures] algorithm. The benefit of using LD Signatures is that the receiving servers can verify the message content without refetching from the client server. This lowers the server load on the client server.

The disadvantage is that the LD Signatures are not easy to implement and Mastodon uses an outdated nonstandard form of the algorithm. The Mastodon documentation recommends not supporting LD Signatures for these reasons.
Furthermore, the Mastodon documentation does not accurately describe the LD Signature algorithm it implements. For more details, see the additional information about [Mastodon LD Signatures](#mastodon-ld-signatures) in this document.

The posted activity MUST be signed with a Mastodon-compatible [HTTP Signature][Mastodon HTTP Signatures].

Mastodon will relay the following activity types: `Create`, `Update`, `Delete`, `Move`. A relay actor MAY only forward those types although Mastodon will *accept* other relayed activities, such as `Announce`, without an LD Signature. In the `Announce` case, it will fetch the announced `object`.

#### Receiving Messages from Relays

Relayed messages are posted to the relay client actor's `inbox`. Relayed messages MUST have an HTTP Signature signed by the Relay Actor.

Messages received from Relay Server Actors MAY have an LD Signature. If the HTTP Signature and the LD Signature are both present, the activity `actor` becomes the effective sender after LD Signature verification.

If no LD Signature is present and the received message is an `Announce` activity, then a relay client MUST ensure the content is legitimate (not spoofed). This may be done by fetching the announced activity from the originating server or using remote content from a local cache. However, if the announced activity is already locally cached, then there would typically be no processing to be done with it since it is already known to the client server.

A client server receiving a relayed message MAY also deliver the messages to local recipients based on the ActivityPub audience targeting properties.

### Relay Server Actor

The following behaviors describe the typical implementation of a Mastodon-style relay server actor.

#### Follow

Ensure that `https://www.w3.org/ns/activitystreams#Public` is in the `object` property. Use the `actor` as the relay client actor URI and save the information about the subscriber. The relay server MAY decide to deny access based on factors like the signer's domain.

#### Undo/Follow

Verify that the `actor` is a known relay client and, if so, remove the client actor from the set of relay actor followers.

#### Activity Relaying

When a message is received from a client actor, the relay MUST verify the HTTP Signature of the activity and identify the originating actor. If the message is valid, it is then posted (with the relay actor's HTTP Signature) to the inboxes of the relay's followers. No delivery is performed based on the ActivityPub audience targeting properties. The relay MUST not send the relayed message to the originating relay client actor.

Typically a message is forwarded unchanged. However, a relay MAY do other processing on the message, such as wrapping a message without an LD Signature using an ActivityPub `Announce` activity before forwarding it (See [pub-relay]). Extended behaviors like this are not described in this FEP.

A relay actor SHOULD only relay messages from followers. A relay actor SHOULD only relay activities that it hasn't already relayed. Addressing properties, like `to`, MUST be in a list even for a single URI.

### Mastodon LD Signatures


*Note that the [Mastodon documentation][Mastodon LD Signatures] for their LD Signatures is incomplete and inaccurate. This section provides more details, but it may be necessary to review the Mastodon source code for additional clarification.*

An activity signed with a Mastodon LD Signature will have a signature document in the activity (using the `signature` property).

**Example Signature Document**

```json
{
  "@context": [
    "https: //www.w3.org/ns/activitystreams",
    "https://w3id.org/security/v1"
  ],
  "id": "https://client.example/3f5ebd6d",
  # ...
  "signature": {
      "type": "RsaSignature2017",
      "creator": "https://client.example/actor#main-key",
      "created": "2024-12-08T03:48:33.901Z",
      "signatureValue": "s69F3mfddd99dGjmvjdjjs81e12jn121Gkm1"
  }
}
```

The `https://w3id.org/security/v1` JSON-LD context defines the `signature` and related properties, but is not used by Mastodon for LD Signature processing.

When performing signature operations the signature document and the activity (without the signature document) are initially processed (hashed) separately. The SHA256 hash digests are concatenated that string is then signed.

#### Signing a JSON-LD Activity

1. Create a signature document with only the `creator` and `created` properties. Set the `@context` to `https://w3id.org/identity/v1`. (Note that this context doesn't appear to be accessible on the web any more. You may need a local copy with a custom JSON-LD context loader.) 
2. Create a canonical RDF representation of the signature document. This requires JSON-LD expansion using standard algorithms ([JSON-LD-ALGO]) and conversion to RDF using the **Universal RDF Dataset Canonicalization Algorithm 2015** ([RDF-CANON]). The serialized RDF is then hashed using [SHA256] and a hexdigest is created.
3. Create a SHA256 hex digest for the activity document (without the signature document) using a similar procedure.
4. Concatenate the SHA256 hex digests for the signature and activity documents and sign the result using SHA256 and the client actor's private key.
5. Encode the signature using [Base64] and set the `signatureValue` of the signature document to the result. 
6. Set the signature document `type` to "RsaSignature2017". 
7. Set the `signature` property of the activity to the signature document.

#### Verifying a JSON-LD Signature

1. The signature document is retrieved from the activity and the type is checked to be the nonstandard "RsaSignature2017". If not, verification fails.
2. Save the `signatureValue` from the signature document.
2. Remove the `type` and `id` and `signatureValue` properties from the signature document and generate a SHA256 hex digest for the modified signature document using the procedure described for signing documents.
4. Remove the `signature` from the activity and generate a SHA256 hex digest for it.
5. Concatenate the hex digests for the modified signature and activity documents.
6. Use the client's public key to verify the signature using SHA256.
 
## LitePub Relay Protocol

The [LitePub][litepub] protocol is based on [ActivityPub] and is used in Pleroma-compatible servers. The reference implementation is the [Pleroma Relay][pleroma-relay].

### Relay Client

A LitePub relay client actor must have a type of `Application` and an actor ID ending with `/relay`. For best interoperability, it should be compatible with Mastodon actor documents and have WebFinger support. Other implementations may use different actor ID structures (e.g., AodeRelay apparently uses `/actor` and works with Pleroma). General relay interoperability of these LitePub variants is not known.

#### Relay Subscription

The client relay actor will send a `Follow` to the relay server. The `Follow` `object` is the relay server actor URI.

The relay server MUST respond to the `Follow` request with an `Accept` or `Reject`. If accepted, the relay server sends a reciprocal `Follow` request for the LitePub client actor. The client server SHOULD respond with `Accept` or `Reject` activity. A relay server MAY decide to ignore the subscription if no acknowledgement is received within a reasonable time interval.

**Example Relay Follow Request**

```json
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://pleroma.example/schemas/litepub-0.1.jsonld",
        {
            "@language": "und"
        }
    ],
    "actor": "https://pleroma.example/relay",
    "bcc": [],
    "bto": [],
    "cc": [],
    "id": "https://pleroma.example/activities/3fe13910-73f4-4cdc-9c84-ec7013a3e764",
    "object": "https://relay.example/actor",
    "state": "pending",
    "to": [
        "https://relay.example/actor"
    ],
    "type": "Follow"
}
```

Notes:
1. The JSON-LD context is not valid for JSON-LD processing. The litepub-0.1.jsonld document contains an invalid WebFinger-related context URL.
2. The `state` property is not defined in the JSON-LD context.

#### Relay Unsubscription

To unsubscribe from a relay send an `Undo` with the original `Follow` activity as the `object`. There is typically no response to the `Undo`.


**Example Undo/Follow Request**

```json
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://pleroma.example/schemas/litepub-0.1.jsonld",
        {
            "@language": "und"
        }
    ],
    "id": "https://pleroma.example/activities/cf9c85e9-f83f-4a02-b598-880f15423f68",
    "object": {
        "actor": "https://pleroma.example/relay",
        "bcc": [],
        "bto": [],
        "cc": [],
        "context": "https://pleroma.example/contexts/d493d02b-7cc9-49dc-995c-d949af0b5417",
        "id": "https://pleroma.example/activities/3fe13910-73f4-4cdc-9c84-ec7013a3e764",
        "object": "https://relay.example/actor",
        "published": "2024-10-18T14:04:11.029802Z",
        "state": "cancelled",
        "to": [
            "https://relay.example/actor"
        ],
        "type": "Follow"
    },
    "published": "2024-10-18T14:04:11.029791Z",
    "to": [ "https://relay.example/actor" ],
    "cc": [],
    "type": "Undo",
    "actor": "https://pleroma.example/relay",
    "context": "https://pleroma.example/contexts/d493d02b-7cc9-49dc-995c-d949af0b5417"
}
```

#### Publishing Messages to a Relay

A LitePub relay client actor will send an `Announce` for a relayed object (like a `Note`). For best interoperability `Announce` should refer to the announced object using an URI (instead of embedding the object).

The `Announce` activity MUST be address to the relay server actor's followers collection. (TODO it's not known if the admin addressing is also required). The `published` property should be included since some relay server will reject activities without it.

```json
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://pleroma.example/schemas/litepub-0.1.jsonld",
        {
            "@language": "und"
        }
    ],
    "actor": "https://pleroma.example/relay",
    "to": [
        "https://pleroma.example/relay/followers",
        "https://pleroma.example/users/admin"
    ],
    "bto": [],
    "cc": [],
    "context": "https://pleroma.example/contexts/a59117d9-7f7c-48ec-83b4-5e183e7179b5",
    "id": "https://pleroma.example/activities/e24e46a2-8926-4a20-9f5f-638e06102159",
    "object": "https://pleroma.example/objects/c13bba3c-e7c1-45ac-939f-aa292d23ee8c",
    "published": "2024-10-18T14:06:37.736295Z",
    "type": "Announce"
}
```

#### Receiving Messages from a Relay

Messages received from a relay are typically wrapped in an `Announce` activity. AFter the `object` of the announce is fetched and validated, it is displayed on the federated timeline. It appears that `Pleroma` will accept a relayed `Create` activity (for Mastodon compatibility), but will refetch the `Create` `object` since the LD Signature isn't processed. (TODO verify this behavior.)

## Other Relay Server Considerations

The relay server hosting relay actors will have other functionality other than activity relaying.

### WebFinger

A relay server MUST implement [WebFinger] support for relay actors. This is necessary because of Mastodon's actor fetching implementation. It's possible that it's not required for LitePub-only relay servers.

### NodeInfo

A relay server MAY implement [NodeInfo] to advertise server activity and metadata.

### Optional Relay Server Behaviors

A relay server MAY support multiple relay protocols. However, there is no standard way to advertise those capabilities.

A relay server often hosts a single actor, but any number of relay actors may be hosted. For example, a relay server may have a relay actor for specific topics, hashtags, or moderation categories. A relay client can subscribe to any number of relay actors in a given server.

Some servers implement dynamic relay actor creation. A relay actor's `inbox` URI might be based on a hashtag or a topic name. When a client actor subscribes to this kind of inbox URI, the relay actor is created automatically. Obviously, there are risks to this approach when used by misbehaved clients.

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- James M Snell, Evan Prodromou, [Activity Vocabulary][AS2-Vocab], 2017
- Mastodon Documentation, [LD Signatures][Mastodon LD Signatures], [HTTP Signatures][Mastodon HTTP Signatures]
- S. Bradner, Key words for use in RFCs to Indicate Requirement Levels, [RFC-2119], 1997
- Matthew Sporny, Dave Longley, JSON-LD 1.1, [JSON-LD], 2020
- Dave Longley, Gregg Kellogg, JSON-LD 1.1 Processing Algorithms and API, [JSON-LD-ALGO], 2018
- Graham Klyne, Jeremy J. Carroll, RDF 1.1 Concepts and Abstract Syntax, [RDF], 2014
- Dave Longley, RDF Dataset Canonicalization, (URDNA2015) [RDF-CANON], 2022
- NIST, Secure Hash Standard (SHS), [SHA256], 2015
- Wikipedia, [Base64]
- P. Jones, WebFinger, RFC-7033 [WebFinger], 2013 
- Jonne Haß, [NodeInfo], GitHub
- Pleroma Relay, [pleroma-relay]
- LitePub Protocol Suite, [litepub]
- Takeshi Umeda, pub-relay [pub-relay]

[ActivityPub]: https://www.w3.org/TR/activitypub/
[Mastodon LD Signatures]: https://docs.joinmastodon.org/spec/security/#ld
[Mastodon HTTP Signatures]: https://docs.joinmastodon.org/spec/security/#http
[AS2-Vocab]: https://www.w3.org/TR/activitystreams-vocabulary
[RFC-2119]: https://datatracker.ietf.org/doc/html/rfc2119
[JSON-LD]: https://www.w3.org/TR/json-ld11/
[JSON-LD-ALGO]: https://www.w3.org/2018/jsonld-cg-reports/json-ld-api/
[RDF]: https://www.w3.org/TR/rdf11-concepts/
[RDF-CANON]: https://www.w3.org/community/reports/credentials/CG-FINAL-rdf-dataset-canonicalization-20221009/#canonicalization
[SHA256]: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
[Base64]: https://en.wikipedia.org/wiki/Base64
[WebFinger]: https://www.rfc-editor.org/rfc/rfc7033
[NodeInfo]: https://github.com/jhass/nodeinfo
[pleroma-relay]: https://git.pleroma.social/pleroma/relay
[litepub]: https://litepub.social/
[pub-relay]: https://github.com/noellabo/pub-relay

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.



