---
slug: "2c59"
authors: Evan Prodromou <evan@prodromou.name>
status: DRAFT
dateReceived: 2024-01-04
trackingIssue: https://codeberg.org/fediverse/fep/issues/226
discussionsTo: https://codeberg.org/fediverse/fep/issues/226
---
# FEP-2c59: Discovery of a Webfinger address from an ActivityPub actor

!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-2c59.md)から翻訳に協力することができます。
## Summary

[Webfinger][Webfinger] is used on the fediverse to abstract out variations in [ActivityPub][ActivityPub] actor URL formats, giving a uniform way of addressing an actor. With a Webfinger address, a client can discover the actor's ActivityPub actor URL. This specification defines an explicit way to reverse the process, and discover a preferred Webfinger address from an ActivityPub actor URL.

## Motivation

[ActivityPub][ActivityPub] is a protocol for federated social networking. It uses HTTPS URLs to identify actors, that is, the people, organizations, applications and groups that use the protocol.

The URL format is not standardized, and there are many variations. For example, a user with the username `evan` on the `activitypub.example` domain could have an actor URL of:

- A path on the domain, like `https://activitypub.example/users/evan`
- Including an ID number, like `https://activitypub.example/users/1234`
- A specific subdomain for social data, like `https://social.activitypub.example/evan`
- One subdomain per user, like `https://evan.activitypub.example`

[Webfinger][Webfinger] is a discovery protocol for people on the Internet. A Webfinger address is an email-address-like identity in the form `user@activitypub.example`. Following the Webfinger protocol, a client can discover information about the person at that address, including their ActivityPub actor URL.

This discovery is unidirectional and can cross domain boundaries. This allows potentially misleading or harmful use of WebFinger to identify actors on the network without their consent. For example, the owner of `idiot.example` could create a Webfinger address `pompous@idiot.example` that points to `https://activitypub.example/users/evan`.

In addition, ActivityPub servers that display actors' Webfinger addresses in their UIs need a reliable way to know which Webfinger is preferred for a given actor ID.

To solve this problem, [Mastodon][Mastodon] constructs a canonical Webfinger address for an actor by extracting the `preferredUsername` from the actor's profile, and prepending it to the domain name of the actor's identity URL. It then does a Webfinger lookup on that address, and if the actor URL returned matches the identity URL, it uses that Webfinger address.

This technique can cause issues for certain actor URLs. First, `preferredUsername` is not a required property for an ActivityPub actor.

Second, it becomes inconvenient to use subdomains or alternate domains for actor URLs. If the original WebFinger is `evan@organization.example`, and the ActivityPub URL is `https://social.organization.example/evan`, the Webfinger will be canonicalized to the longer `evan@social.organization.example`. A similar problem arises if the actor URL uses one subdomain per user, like `https://evan.organization.example/`.

This specification defines a way to discover a Webfinger address from an ActivityPub actor without constructing it from properties and URL parsing. It gives domain owners the option to use a registered domain for their Webfinger addresses, and host their ActivityPub server on a subdomain or a different domain. The Mastodon method can be used as a fallback.

## Specification

- An ActivityPub actor SHOULD include a `webfinger` property, which is a canonical Webfinger address for the actor.
- The `webfinger` property is functional, that is, there can be at most one `webfinger` property per actor.
- If provided, the JRD link data for the Webfinger address in the `webfinger` property MUST link directly to the actor URL verbatim, without redirects or aliases.
- The identity in the `webfinger` property SHOULD be a plain Webfinger address, like `user@domain.example`.
- The identity in the `webfinger` property MAY be an `acct:` URL, like `acct:user@domain.example`.

## Context

The context document for this specification is `https://purl.archive.org/socialweb/webfinger`. Its contents are as follows:

```
{
  "@context": {
    "wf": "https://purl.archive.org/socialweb/webfinger#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "webfinger": {
      "@id": "wf:webfinger",
      "@type": "xsd:string"
    }
  }
}
```

## 例

A publisher can include the `webfinger` property for an actor.

```
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://purl.archive.org/socialweb/webfinger"
    ],
    "id": "https://social.example.com/evanp",
    "type": "Person",
    "name": "Evan Prodromou",
    "inbox": "https://social.example.com/evanp/inbox",
    "outbox": "https://social.example.com/evanp/outbox",
    "following": "https://social.example.com/evanp/following",
    "followers": "https://social.example.com/evanp/followers",
    "liked": "https://social.example.com/evanp/liked",
    "webfinger": "evanp@example.com"
}
```

A publisher can include the `webfinger` property for a `Person` object
referenced in the `actor` property of an activity.

```
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://purl.archive.org/socialweb/webfinger"
    ],
    "id": "https://social.example.com/evanp/activity/775",
    "to": ["Public"],
    "type": "Like",
    "summary": "Evan liked a note",
    "actor": {
        "type": "Person",
        "name": "Evan Prodromou",
        "id": "https://social.example.com/evanp",
        "webfinger": "evanp@example.com"
    },
    "object": {
        "type": "Note",
        "id": "https://social.example.com/other/note/221",
        "to": ["Public"]
    }
}
```

The publisher can include the `webfinger` property for an actor with an `acct:` prefix.

```
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://purl.archive.org/socialweb/webfinger"
    ],
    "id": "https://prefix.example/urlfan",
    "type": "Person",
    "name": "URL Fan",
    "inbox": "https://prefix.example/urlfan/inbox",
    "outbox": "https://prefix.example/urlfan/outbox",
    "following": "https://prefix.example/urlfan/following",
    "followers": "https://prefix.example/urlfan/followers",
    "liked": "https://prefix.example/urlfan/liked",
    "webfinger": "acct:urlfan@prefix.example"
}
```

## Security Considerations

The source of the `webfinger` property is important. A third party may spoof, intentionally or unintentionally, the `webfinger` property of another actor. The property should only be treated as the preferred Webfinger address for the actor if it comes from the actor `id` URL, or if it is delivered with the actor's [HTTP Signature][HTTP Signature] via the ActivityPub protocol.

## 参考文献

- [Webfinger] Brad Fitzpatrick, [WebFinger](https://tools.ietf.org/html/rfc7033), 2013
- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [Mastodon] Eugen Rochko, [Mastodon](https://joinmastodon.org/), 2016
- [Mastodon Webfinger], Mastodon contributors, [Webfinger - Mastodon documentation](https://docs.joinmastodon.org/spec/webfinger/), 2018
- [HTTP Signature], Cavage, M., [HTTP Signatures](https://tools.ietf.org/html/draft-cavage-http-signatures-10), 2017

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
