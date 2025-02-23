---
slug: "0ea0"
authors: silverpill <silverpill@firemail.cc>
status: DRAFT
dateReceived: 2023-04-18
trackingIssue: https://codeberg.org/fediverse/fep/issues/88
discussionsTo: https://codeberg.org/fediverse/fep/issues/88
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-0ea0.md)から翻訳に協力することができます。
# FEP-0ea0: Payment Links

## Summary

This FEP describes a way to attach payment information to [ActivityPub](https://www.w3.org/TR/activitypub/) actors and objects. That information might be a link to donation page, a link for buying an artwork, or anything else that can be represented with a URI.

## History

[PeerTube](https://docs.joinpeertube.org/api/activitypub#video) videos may have `support` property, which contains a text explaining how to support the content creator.

[FEP-8c3f: Web Monetization](https://codeberg.org/fediverse/fep/src/branch/main/fep/8c3f/fep-8c3f.md) was published in 2022. The ensuing discussion on SocialHub forum led to the creation of Payment Links proposal.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119](https://tools.ietf.org/html/rfc2119.html).

## Payment links

Payment link is an object with the following properties:

- `type` (REQUIRED): the type MUST be `Link`.
- `name` (RECOMMENDED): the `name` property SHOULD contain a human-readable description of the payment link.
- `href` (REQUIRED): the `href` property MUST contain a payment URI. This can be a URL of a website, or any other kind of URI, such as [payto URI](https://datatracker.ietf.org/doc/html/rfc8905).
- `rel` (REQUIRED):  the `rel` property MUST contain the string `payment` or an array containing that string. The `payment` relation type is defined in [Link Relations Registry](https://www.iana.org/assignments/link-relations/link-relations.xhtml).

Payment links MUST be added to `attachment` array of an actor or an object.

## 例

Payment link attached to an actor:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Person",
  "id": "https://social.example/users/alice",
  "inbox": "https://social.example/users/alice/inbox",
  "outbox": "https://social.example/users/alice/outbox",
  "attachment": [
    {
      "type": "Link",
      "name": "Donate",
      "href": "payto://iban/DE75512108001245126199",
      "rel": "payment"
    }
  ]
}
```

Payment link attached to an object:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Image",
  "id": "https://gallery.example/photos/123",
  "attributedTo": "https://gallery.example/users/alice",
  "name": "Painting of a cat",
  "attachment": [
    {
      "type": "Link",
      "name": "Buy",
      "href": "https://gallery.example/photos/123/order",
      "rel": [
        "payment",
        "https://gallery.example/ns#buy"
      ]
    }
  ]
}
```

## Payment links as actor metadata

(このセクションは非規範的です。)

Implementers may treat payment links attached to actor object in the same way as actor metadata fields. In that case, `name` translates into field label and `href` translates into field value.

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [FEP-8c3f: Web Monetization] Diogo Peralta Cordeiro, Phablulo Joel, [FEP-8c3f: Web Monetization](https://codeberg.org/fediverse/fep/src/branch/main/fep/8c3f/fep-8c3f.md), 2022
- [Link Relations Registry] IANA, [Link Relations](https://www.iana.org/assignments/link-relations/link-relations.xhtml), 2005

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
