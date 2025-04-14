---
slug: "fffd"
authors: Adam R. Nelson <adam@nels.onl>
status: DRAFT
dateReceived: 2023-04-29
trackingIssue: https://codeberg.org/fediverse/fep/issues/95
discussionsTo: https://codeberg.org/fediverse/fep/issues/95
---

# FEP-fffd: Proxy Objects
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-fffd.md)から翻訳に協力することができます。


## Summary

A *proxy object* is an \[ActivityPub] object that is semantically identical to another entity, which may exist on another, non-ActivityPub protocol. For example, an ActivityPub-to-Nostr bridge creates Actors and Notes that are proxies for Nostr users and notes.

This document describes a data format to identify proxy objects and to specify the ActivityPub and non-ActivityPub entities they are equivalent to, with the intention that multi-protocol clients will automatically merge objects with their proxies, hiding the implementation details of bridges and cross-protocol publishing from users.

## 1. Requirements

The key words "MUST", "SHOULD", and "MAY" are to be interpreted as described in \[RFC2119].

## 2. Rationale

> This section is non-normative.

Many Fediverse servers speak multiple protocols besides ActivityPub, such as Nostr or Diaspora, and simultaneously publish posts across multiple protocols. Additionally, bridge servers exist to relay posts between ActivityPub and other protocols such as Nostr.

A use case also exists for creating a duplicate of an ActivityPub object: moving an account to a new instance. Posts copied to a user's new account should be marked as duplicates of the originals, in a way that preserves the original posts' likes and replies.

Despite these use cases, there is currently no standard way to communicate that an ActivityPub activity is a copy (or *proxy*) of a post on another protocol.

Consider this scenario:

- Alice's server speaks both ActivityPub and Diaspora protocols. She publishes a post, which is syndicated on both protocols.
- Bob's server speaks only ActivityPub. He replies to the ActivityPub version of Alice's post.
- Charlie's server speaks only Diaspora. He replies to the Diaspora version of Alice's post.
- Dee's server speaks both ActivityPub and Diaspora. She receives Alice's, Bob's, and Charlie's posts. How does Dee's server know that Bob's reply and Charlie's reply belong to the same post?

Proxy objects provide a potential solution to this problem.

## 3. Format

FEP-fffd does not define any new vocabulary or `@context` entries. Instead, it further defines the meaning of Links in the `url` property of an Object when they have a `rel` property of `"alternate"` or `"canonical"`.

Each Link in `url` with `"rel": "alternate"` or `"rel": "canonical"` is called a *proxy link*. Any Object with one or more proxy links is called a *proxy object*. The referent of a proxy link is called a *proxied object*, and SHOULD be considered semantically identical to the parent proxy object, modulo the limitations described in section 4.

A proxied object is not necessarily an ActivityPub object, or even an object accessible via a network request; its meaning is determined based on its protocol. The protocol and format of the proxied object are determined by the proxy link's URI scheme and `mediaType`; well-known protocols and defaults are defined in section 5. An application SHOULD ignore proxied objects in protocols or formats that the application does not understand.

If a proxy link has `"rel": "canonical"`, it indicates that its proxied object is the *canonical* (original, authoritative) version of the proxy object. A proxy object MUST NOT have more than one proxy link with `"rel": "canonical"`. This property SHOULD be used by bridges that relay posts made by third parties, to indicate that the bridged post is not the original. It SHOULD NOT be used by servers that publish to multiple protocols at once; in this case no one version of an object is more authoritative than another.

## 4. Merging

When a conforming application encounters a proxy object, it may merge it with its proxy objects under certain circumstances.

To *merge* a proxy object and its proxied object(s) means to display all of these objects as a single entity (such as a user or a post), while combining all collections and metadata belonging to these objects:

- The followers, following, liked, and outbox collections of an actor, if present, SHOULD be combined with those of a proxy when merging.
- The replies, likes, and shares of a non-actor object, if present, SHOULD be combined with those of a proxy object when merging.
- If one of the merged objects is canonical, its properties SHOULD override any conflicting properties in any other merged object.
- If none of the merged objects are canonical, conflicting properties MAY be resolved in any way the application chooses, including but not limited to: choosing the representation from the protocol with the most features, displaying a detailed description of the conflict, or refusing to merge objects with conflicting properties.

In some circumstances, an application may encounter malformed or malicious proxy links that could misrepresent objects not owned by the links' author, or it may encounter proxy links whose referents are malformed or missing. These situations sometimes prevent merging.

- If a proxy link's referent has been deleted (as indicated by an HTTP 410 Gone status, a Delete activity, or another protocol's equivalent), then:
    - If the proxied object is canonical, the proxy object SHOULD be deleted
    - If the proxied object is not canonical, the proxy object MAY be deleted; if it is not, all collection entries originally from the deleted object's protocol (if it is not ActivityPub) SHOULD be removed from the proxy object's collections, and their proxy objects, if any, SHOULD be deleted.
- If a proxy link is broken, but has not been explicitly deleted, then the proxy object MAY continue to exist, and the application MAY still display cached data or proxy objects for the proxied object's collection entries.
- If a proxied object is canonical, and that proxied object itself has a link to a different canonical representation (whether through FEP-fffd or another protocol's equivalent), then:
    - The application SHOULD follow the chain of canonical links up to a fixed, application-defined maximum number of links.
    - If this number of links is exceeded (possibly indicating a cycle), or if any link in the chain has more than one canonical link, the application SHOULD NOT merge any of the objects in the chain.
- If a proxied object is not canonical, the application SHOULD verify that the proxied object also considers itself a proxy for the proxy object. If the proxied object is an ActivityPub object, then the application SHOULD NOT merge it with the proxy object if it does not meet at least one of these criteria:
    - It has a proxy link pointing to the proxy object (that is, both objects are proxies for each other)
    - Both objects are actors, and both are `alsoKnownAs` each other.
    - Both object are owned by the same actor, or by actors that are `alsoKnownAs` each other.
- If a proxy link points to localhost or any loopback address, the application SHOULD NOT follow the link or attempt to merge the proxied object it represents.

## 5. Protocols

Several protocols are named in this document, but interaction with these protocols is left intentionally underspecified, as the behavior of non-ActivityPub protocols is outside the scope of this FEP. If a proxy link's URI scheme and/or `mediaType` match a protocol named in this section, a conforming application SHOULD either use the matching protocol to access the proxied object or ignore the proxy link entirely, but it MUST NOT interpret the link as a proxy link for a different protocol or format.

### 5.1. Well-known Alternate Protocols

- Nostr: Identified by the `nostr:` URI scheme, as defined in \[NIP-21]. The identifiers used in these URIs MUST be "bare" NIP-19 identifiers starting with `npub1` or `note1`. The `npub1` identifier type MUST be used only in proxy links for Actors.
- Diaspora: Identified by the `diaspora:` URI scheme, following the format defined in \[`diaspora://` URI scheme].
- DID: Identified by the `did:` URI scheme, as defined in \[DID URL Syntax], and MUST be used only in proxy links for Actors.
- ATProto: Identified by the `at` URI scheme, as defined in \[AT URI Scheme]. AT Repositories, Collections, and Records MUST be used only in proxy links for ActivityPub Actors, Collections, and Objects, respectively. Repository URIs starting with `at://did:` SHOULD be considered identical to the `did:` URIs they contain; including both a `did:` link and an `at://did:` link for the same DID is redundant.
- Secure Scuttlebutt: Identified by the `ssb:` URI scheme.

### 5.2. Well-known Media Types

- RSS: `application/rss+xml`; the `href` should be the URL of the feed, followed by a URL fragment whose content is the `<guid>` value of an entry in the feed.
- Atom: `application/atom+xml`; the `href` should be the URL of the feed, followed by a URL fragment whose content is the `<id>` value of an entry in the feed.
- ActivityPub:  `application/ld+json; profile="https://www.w3.org/ns/activitystreams"` or `application/activity+json`; the `href` should point to an ActivityPub Object.

If an application supports general-purpose transport protocols other than HTTP(S), such as Gemini or IPFS, it MAY interpret proxy links to these protocols in the same manner as it would interpret HTTP(S) proxy links, including applying these well-known media types.

### 5.3. Non-federated Web Content

By default, if a proxy link uses the `http` or `https` protocol, and either does not have a `mediaType` or has a `mediaType` of `text/html`, it is considered a link to some unspecified, application-defined non-federated Web content. An application MAY interpret this link as any kind of content or protocol other than one of the well-known protocols or formats defined in this section. Notably, this kind of proxy object MUST NOT be interpreted as an ActivityPub resource, even if the link responds with valid ActivityStreams data.

This default allows an ActivityPub object to proxy any non-federated Web content; for example, a Twitter-to-ActivityPub bridge may use a proxy link to a Twitter URL to identify a Note as a proxy object for a Twitter post.

## 6. Examples

> This section is non-normative. The JSON-LD `@context` property is omitted for brevity.

---

A post relayed by a third-party Twitter-to-ActivityPub bridge. Because the `canonical` proxy link is also the only `url` entry, it should also be used as a clickable link to the original post.

```json
{
  "id": "http://twitter-bridge.example/status/1234",
  "type": "Note",
  "actor": "http://twitter-bridge.example/@jack",
  "content": "just setting up my twttr",
  "url": {
    "type": "Link",
    "rel": "canonical",
    "href": "https://twitter.com/jack/status/20"
  }
}
```

---

A post published to ActivityPub, Diaspora, and Nostr simultaneously. Because there is one non-proxy `Link` in `url` with an `https` protocol, this non-proxy link should be used as a clickable link to the original post.

```json
{
  "id": "http://fediverse.example/status/1234",
  "type": "Note",
  "actor": "http://fediverse.example/@alice",
  "content": "Hello, world!",
  "url": [{
    "type": "Link",
    "href": "https://fediverse.example/@alice/1234"
  }, {
    "type": "Link",
    "rel": "alternate",
    "href": "diaspora://alice@fediverse.example/post/deadbeefdeadbeefdeadbeefdeadbeef"
  }, {
    "type": "Link",
    "rel": "alternate",
    "href": "nostr:note1gwdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeef"
  }]
}
```

---

An ActivityPub Note copied from a user's old instance to a new instance when moving to a new account.

```json
{
  "id": "http://newinstance.example/status/1234",
  "type": "Note",
  "actor": "http://newinstance.example/@alice",
  "content": "Hello, world!",
  "url": [{
    "type": "Link",
    "href": "https://newinstance.example/@alice/1234"
  }, {
    "type": "Link",
    "rel": "canonical",
    "mediaType": "application/ld+json; profile=\"https://www.w3.org/ns/activitystreams\"",
    "href": "https://oldinstance.example/status/5678"
  }]
}
```

---

## 参考文献

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119.html), 1997
- [NIP-21]  fiatjaf, martindsq, mplorentz, [NIP-21: `nostr:` URI scheme](https://github.com/nostr-protocol/nips/blob/master/21.md), 2023
- [`diaspora://` URI scheme]  Benjamin Neff, [diaspora* federation protocol](https://diaspora.github.io/diaspora_federation/index.html), 2017
- [DID URL Syntax] Manu Sporny, Markus Sabadello, Drummond Reed, Orie Steele, Christopher Allen, [Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/did-core/#did-url-syntax), 2022
- [AT URI Scheme] Bluesky, [ATProto Documentation](https://atproto.com/specs/at-uri-scheme), 2023

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
