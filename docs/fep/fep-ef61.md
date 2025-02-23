---
slug: "ef61"
authors: silverpill <@silverpill@mitra.social>
status: DRAFT
dateReceived: 2023-12-06
trackingIssue: https://codeberg.org/fediverse/fep/issues/209
discussionsTo: https://socialhub.activitypub.rocks/t/fep-ef61-portable-objects/3738
---
# FEP-ef61: Portable Objects
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-ef61.md)から翻訳に協力することができます。

## Summary

Portable [ActivityPub][ActivityPub] objects with server-independent IDs.

## Motivation

Usage of HTTP(S) URLs as identifiers has a major drawback: when the server disappears, everyone who uses it loses their identities and data.

The proposed solution should satisfy the following constraints:

- User's identity and data should not be tied to a single server.
- Users should have a choice between full control over their identity and data, and delegation of control to a trusted party.
- Implementing the solution in existing software should be as simple as possible. Changes to ActivityPub data model should be kept to a minimum.
- The solution should be compatible with existing and emerging decentralized identity and storage systems.
- The solution should be transport-agnostic.

## History

[Nomadic identity](https://joinfediverse.wiki/index.php?title=Nomadic_identity/en) mechanism makes identity independent from a server and was originally part of the Zot federation protocol.

[Streams](https://codeberg.org/streams/streams) (2021) made nomadic accounts available via the [Nomad protocol](https://codeberg.org/streams/streams/src/commit/11f5174fdd3dfcd8714974f93d8b8fc50378a193/spec/Nomad/Home.md), which supported ActivityStreams serialisation.

[FEP-c390](https://codeberg.org/fediverse/fep/src/branch/main/fep/c390/fep-c390.md) (2022) introduced a decentralized identity solution compatible with ActivityPub. It enabled permissionless migration of followers between servers, but didn't provide full data portability.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119][RFC-2119].

## Identifiers

An [ActivityPub][ActivityPub] object can be made portable by using an identifier that is not tied to a single server. This proposal describes a new identifier type that has this property and is compatible with the [ActivityPub] specification.

### ap:// URLs

`ap://` URL is constructed according to the [URI][RFC-3986] specification, but with a [Decentralized Identifier][DID] in place of the authority:

```text
ap://did:example:123456/path/to/object?name=value#fragment-id
\_/  \________________/ \____________/ \________/ \_________/
 |           |                |            |           |
scheme   authority           path        query     fragment
```

- The URI scheme MUST be `ap`.
- The authority component MUST be a valid [DID].
- The path is REQUIRED. It MUST be treated as an opaque string.
- The query is OPTIONAL. To avoid future conflicts, implementers SHOULD NOT use parameter names that are not defined in this proposal.
- The fragment is OPTIONAL.

### DID methods

Implementers MUST support the [did:key] method. Other DID methods SHOULD NOT be used, as it might hinder interoperability.

>[!NOTE]
>The following additional DID methods are being considered: [did:web](https://w3c-ccg.github.io/did-method-web/), [did:dns](https://danubetech.github.io/did-method-dns/), [did:webvh](https://identity.foundation/didwebvh/) (formerly `did:tdw`) and [did:fedi](https://arcanican.is/excerpts/did-method-fedi.html).

DID documents SHOULD contain Ed25519 public keys represented as verification methods with `Multikey` type (as defined in the [Controller Documents](https://www.w3.org/TR/controller-document/#Multikey) specification).

Any [DID URL][DID-URL] capabilities of a DID method MUST be ignored when working with `ap://` URLs.

### Dereferencing ap:// URLs

To dereference an `ap://` URL, the client MUST make HTTP GET request to a gateway endpoint at [well-known] location `/.well-known/apgateway`. The `ap://` prefix MUST be removed from the URL and the rest of it appened to a gateway URL. The client MUST specify an `Accept` header with the `application/ld+json; profile="https://www.w3.org/ns/activitystreams"` media type.

Example of a request to a gateway:

```
GET https://social.example/.well-known/apgateway/did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/path/to/object
```

ActivityPub objects identified by `ap://` URLs can be stored on multiple servers simultaneously.

If object identified by `ap://` URL is stored on the server, it MUST return a response with status `200 OK` containing the requested object. The value of a `Content-Type` header MUST be `application/ld+json; profile="https://www.w3.org/ns/activitystreams"`.

If object identified by `ap://` URL is not stored on the server, it MUST return `404 Not Found`.

If object is not public, the server MUST return `404 Not Found` unless the request has a HTTP signature and the signer is allowed to view the object.

>[!NOTE]
>This document describes web gateways, which use HTTP transport. However, the data model and authentication mechanism are transport-agnostic and other types of gateways could exist.

## Authentication and authorization

Authentication and authorization are performed in accordance with [FEP-fe34] origin-based security model.

The [origin][RFC-6454] of an `ap://` URL is computed by the following algorithm:

1. Let `uri-scheme` be the `ap` string.
2. Let `uri-host` be the authority component of the URL.
3. Let `uri-port` be the number 0.
4. Return the triple `(uri-scheme, uri-host, uri-port)`.

And the origin of a [DID URL][DID-URL] is computed by the following algorithm:

1. Let `uri-scheme` be the `ap` string.
2. Let `uri-host` be the DID component of the DID URL.
3. Let `uri-port` be the number 0.
4. Return the triple `(uri-scheme, uri-host, uri-port)`.

Actors, activities and objects identified by `ap://` URLs MUST contain [FEP-8b32] integrity proofs. Collections identified by `ap://` URLs MAY contain integrity proofs. If collection doesn't contain an integrity proof, [another authentication method](#collections) MUST be used.

The value of `verificationMethod` property of the proof MUST be a [DID URL][DID-URL] where the DID matches the authority component of the `ap://` URL.

>[!NOTE]
>This document uses terms "actor", "activity", "collection" and "object" according to the classification given in [FEP-2277].

## Portable actors

An actor object identified by `ap://` URL MUST have a `gateways` property containing an ordered list of gateways where the latest version of that actor object can be retrieved. Each item in the list MUST be an HTTP(S) URL with empty path, query and fragment components. The list MUST contain at least one item.

One identity (represented by [DID]) can control multiple actors (which are differentiated by the path component of an `ap://` URL).

Example:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/security/data-integrity/v1",
    "https://w3id.org/fep/ef61"
  ],
  "type": "Person",
  "id": "ap://did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/actor",
  "inbox": "ap://did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/actor/inbox",
  "outbox": "ap://did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/actor/outbox",
  "gateways": [
    "https://server1.example",
    "https://server2.example"
  ],
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "created": "2023-02-24T23:36:38Z",
    "verificationMethod": "did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2#z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2",
    "proofPurpose": "assertionMethod",
    "proofValue": "..."
  }
}
```

### Location hints

When ActivityPub object containing a reference to another actor is being constructed, implementations SHOULD provide a list of gateways where specified actor object can be retrieved. This list MAY be provided using the `gateways` query parameter. Each gateway address MUST be URL-endcoded, and if multiple addresses are present they MUST be separated by commas.

Example:

```
ap://did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/actor?gateways=https%3A%2F%2Fserver1.example,https%3A%2F%2Fserver2.example
```

This URL indicates that object can be retrieved from two gateways:

- `https://server1.example`
- `https://server2.example`

Implementations MUST discard query parameters when comparing `ap://` identifiers and treat identifiers with different query parameter values as equal.

### Inboxes and outboxes

Servers and clients MUST use gateways to deliver activities to inboxes or outboxes. Servers specified in the `gateways` property of an actor object MUST accept POST requests to respective gateway URLs.

Example:

```
POST https://social.example/.well-known/apgateway/did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/actor/inbox
```

Delivered activities might be not portable. If delivered activity is portable (has `ap://` identifier), the server MUST verify its [FEP-8b32] integrity proof. If the server does not accept deliveries on behalf of an actor, it MUST return `405 Method Not Allowed`.

ActivityPub clients MAY follow [FEP-ae97][FEP-ae97] to publish activities. In this case, the client MAY deliver signed activity to multiple outboxes, located on different servers.

Upon receiving an activity in actor's outbox, server SHOULD forward it to outboxes located on other servers where actor's data is stored. An activity MUST NOT be forwarded from outbox more than once.

Upon receiving an activity in actor's inbox, server SHOULD forward it to inboxes located on other servers where actor's data is stored.

### Collections

Collections associated with portable actors (such as inbox and outbox collections) MAY not have [FEP-8b32] integrity proofs. Consuming implementations MUST NOT process unsecured collections retrieved from servers that are not listed in the `gateways` array of the actor document.

## Portable objects

Example:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/security/data-integrity/v1",
    "https://w3id.org/fep/ef61"
  ],
  "type": "Note",
  "id": "ap://did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/objects/dc505858-08ec-4a80-81dd-e6670fd8c55f",
  "attributedTo": "ap://did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/actor?gateways=https%3A%2F%2Fserver1.example,https%3A%2F%2Fserver2.example",
  "inReplyTo": "ap://did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK/objects/f66a006b-fe66-4ca6-9a4c-b292e33712ec",
  "content": "Hello!",
  "attachment": [
    {
      "type": "Image",
      "url": "hl:zQmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n",
      "mediaType": "image/png",
      "digestMultibase": "zQmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n"
    }
  ],
  "to": [
    "ap://did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK/actor"
  ],
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "created": "2023-02-24T23:36:38Z",
    "verificationMethod": "did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2#z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2",
    "proofPurpose": "assertionMethod",
    "proofValue": "..."
  }
}
```

## Media

Integrity of an external resource is attested with a digest. When a portable object contains a reference to an external resource (such as image), it MUST also contain a [`digestMultibase`](https://w3c.github.io/vc-data-integrity/#resource-integrity) property representing the integrity digest of that resource. The digest MUST be computed using the SHA-256 algorithm.

The URI of an external resource SHOULD be a [hashlink][Hashlinks].

Example of an `Image` attachment:

```json
{
  "type": "Image",
  "url": "hl:zQmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n",
  "mediaType": "image/png",
  "digestMultibase": "zQmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n"
}
```

After retrieving a resource, the client MUST verify its integrity by computing its digest and comparing the result with the value encoded in `digestMultibase` property.

Resources attached to portable objects using hashlinks can be stored by gateways. To retrieve a resource from a gateway, the client MUST make an HTTP GET request to the gateway endpoint at [well-known] location `/.well-known/apgateway`. The value of a hashlink URI MUST be appended to the gateway base URL.

Example of a request:

```
GET https://social.example/.well-known/apgateway/hl:zQmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n
```

## Compatibility

### Identifiers

`ap://` URLs might not be compatible with existing [ActivityPub][ActivityPub] implementations. To provide backward compatibility, gateway-based HTTP(S) URLs of objects can be used instead of their actual `ap://` identifiers:

```
https://social.example/.well-known/apgateway/did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/path/to/object
```

Publishers MUST use the first gateway from actor's `gateways` list when constructing compatible identifiers. Consuming implementations that support `ap://` URLs MUST remove the part of the URL preceding `did:` and re-construct the canonical `ap://` identifier. Objects with the same canonical identifier, but located on different gateways MUST be treated as different instances of the same object.

Publishers MUST NOT add the `gateways` query parameter to object IDs if compatible identifiers are used.

When HTTP signatures are necessary for communicating with other servers, each gateway that makes requests on behalf of an actor SHOULD use a separate secret key. The corresponding public keys MUST be added to actor document using the `assertionMethod` property as described in [FEP-521a].

### WebFinger addresses

WebFinger address of a portable actor can be obtained by the reverse discovery algorithm described in section 2.2 of [ActivityPub and WebFinger][WebFinger] report, but instead of taking the hostname from the identifier, it MUST be taken from the first gateway in actor's `gateways` array.

## Discussion

(このセクションは非規範的です。)

### 'ap' URL syntax

'ap' URLs are not valid URIs per [RFC-3986]. To make them valid, the authority component can be percent-encoded:

```
ap://did%3Akey%3Az6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/actor
```

### Discovering locations

#### Arbitrary paths

The `gateways` array can contain HTTP(S) URLs with a path component, thus enabling discovery based on the ["follow your nose"](https://indieweb.org/follow_your_nose) principle, as opposed to discovery based on a [well-known] location.

Example of a compatible object ID if the gateway endpoint is `https://social.example/ap`:

```
https://social.example/ap/did:key:z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2/path/to/object
```

#### Alternatives to `gateways` property

This proposal makes use of the `gateways` property, but the following alternatives are being considered:

- `aliases` and [`sameAs`](https://schema.org/sameAs) (containing HTTP(S) URLs of objects)
- `alsoKnownAs` (used for account migrations, so the usage of this property may cause issues)
- `url` (with `alternate` [relation type](https://html.spec.whatwg.org/multipage/links.html#linkTypes))

#### DID services

Instead of specifying gateways in actor document, they can be specified in [DID] document using [DID services](https://www.w3.org/TR/did-core/#services). This approach is not compatible with generative DID methods such as `did:key`, which might be necessary for some types of applications.

### Media access control

The proposed approach to referencing media with hashlinks does not support access control: anybody who knows the hash can retrieve the file.

To work around this limitation, a different kind of identifier can be used where digest is combined with the `ap://` identifier of its parent document. The gateway will not serve media unless parent document ID is provided, and will check whether request signer has permission to view the document and therefore the attached media.

### Compatibility

The following alternatives to gateway-based compatible IDs are being considered:

1. Use regular HTTP(S) URLs but specify the canonical `ap://` URL using the `url` property (with `canonical` relation type, as proposed in [FEP-fffd][FEP-fffd]). For pointers to other objects such as `inReplyTo` property, an embedded object with `url` property can be used instead of a plain URL.
2. Alter object ID depending on the capabilities of the peer (which can be reported by [NodeInfo][NodeInfo] or some other mechanism).

## 実装

- [Streams](https://codeberg.org/streams/streams)
- [Mitra](https://codeberg.org/silverpill/mitra) (gateway only)

## 参考文献

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119], 1997
- T. Berners-Lee, R. Fielding, L. Masinter, [Uniform Resource Identifier (URI): Generic Syntax][RFC-3986], 2005
- Manu Sporny, Dave Longley, Markus Sabadello, Drummond Reed, Orie Steele, Christopher Allen, [Decentralized Identifiers (DIDs) v1.0][DID], 2022
- Dave Longley, Dmitri Zagidulin, Manu Sporny, [The did:key Method v0.7][did:key], 2022
- M. Nottingham, [Well-Known Uniform Resource Identifiers (URIs)][well-known], 2019
- silverpill, [FEP-8b32: Object Integrity Proofs][FEP-8b32], 2022
- silverpill, [FEP-ae97: Client-side activity signing][FEP-ae97], 2023
- silverpill, [FEP-fe34: Origin-based security model][FEP-fe34], 2024
- A. Barth, [The Web Origin Concept][RFC-6454], 2011
- silverpill, [FEP-2277: ActivityPub core types][FEP-2277], 2025
- M. Sporny, L. Rosenthol, [Cryptographic Hyperlinks][Hashlinks], 2021
- silverpill, [FEP-521a: Representing actor's public keys][FEP-521a], 2023
- a, Evan Prodromou, [ActivityPub and WebFinger][WebFinger], 2024
- Dave Longley, Manu Sporny, [Verifiable Credential Data Integrity 1.0][Data Integrity], 2024
- Adam R. Nelson, [FEP-fffd: Proxy Objects][FEP-fffd], 2023
- Jonne Haß, [NodeInfo][NodeInfo], 2014

[ActivityPub]: https://www.w3.org/TR/activitypub/
[RFC-2119]: https://datatracker.ietf.org/doc/html/rfc2119.html
[RFC-3986]: https://datatracker.ietf.org/doc/html/rfc3986.html
[DID]: https://www.w3.org/TR/did-core/
[did:key]: https://w3c-ccg.github.io/did-method-key/
[DID-URL]: https://www.w3.org/TR/did-core/#did-url-syntax
[well-known]: https://datatracker.ietf.org/doc/html/rfc8615
[FEP-8b32]: https://codeberg.org/fediverse/fep/src/branch/main/fep/8b32/fep-8b32.md
[FEP-ae97]: https://codeberg.org/fediverse/fep/src/branch/main/fep/ae97/fep-ae97.md
[FEP-fe34]: https://codeberg.org/fediverse/fep/src/branch/main/fep/fe34/fep-fe34.md
[RFC-6454]: https://www.rfc-editor.org/rfc/rfc6454.html
[FEP-2277]: https://codeberg.org/fediverse/fep/src/branch/main/fep/2277/fep-2277.md
[Hashlinks]: https://datatracker.ietf.org/doc/html/draft-sporny-hashlink-07
[FEP-521a]: https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md
[WebFinger]: https://swicg.github.io/activitypub-webfinger/
[Data Integrity]: https://w3c.github.io/vc-data-integrity/
[FEP-fffd]: https://codeberg.org/fediverse/fep/src/branch/main/fep/fffd/fep-fffd.md
[NodeInfo]: https://nodeinfo.diaspora.software/

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
