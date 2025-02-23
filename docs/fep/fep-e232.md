---
slug: "e232"
authors: silverpill <@silverpill@mitra.social>
status: FINAL
dateReceived: 2022-08-01
dateFinalized: 2023-12-03
trackingIssue: https://codeberg.org/fediverse/fep/issues/14
discussionsTo: https://socialhub.activitypub.rocks/t/fep-e232-object-links/2722
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-e232.md)から翻訳に協力することができます。
# FEP-e232: Object Links

## Summary

This document proposes a way to represent text-based links to [ActivityPub][ActivityPub] objects which are similar to mentions. One example of such link is inline quote within the value of the `content` property, but this proposal is not limited to any particular use case.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119][RFC-2119].

## Object links

It is expected that software will allow users to define object links using some kind of microsyntax, similar to `@mention` and `#hashtag` microsyntaxes. The exact way of defining object links may vary depending on the use case and is out of scope of this document.

If an object's `name`, `summary`, or `content` has qualified links to other objects, that object SHOULD have the `tag` property, where each object link is represented as a `Link` object, as suggested by [Activity Vocabulary][ActivityVocabulary]. The properties of this `Link` object are:

- `type` (REQUIRED): the type MUST be `Link` or a subtype.
- `mediaType` (REQUIRED): the media type MUST be `application/ld+json; profile="https://www.w3.org/ns/activitystreams"`. This specification only deals with ActivityPub objects but in practice the media type can be different and servers MAY accept object links which do not comply with the requirement. For example, a media type of `application/activity+json` SHOULD be treated as equivalent.
- `href` (REQUIRED): the href property MUST contain the URI of the referenced object.
- `name` (OPTIONAL): the `name` SHOULD match the microsyntax used in object's content.
- `rel` (OPTIONAL): if relevant, the `rel` SHOULD specify how the link is related to the current resource. Using `rel` can provide additional purpose to object links by signaling specific intended use-cases.

## Examples

(This section is non-normative.)

A link to an issue in a bug tracker:

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "type": "Note",
    "content": "The bug was reported in #1374",
    "tag": [
        {
            "type": "Link",
            "mediaType": "application/ld+json; profile=\"https://www.w3.org/ns/activitystreams\"",
            "href": "https://forge.example/tickets/1374",
            "name": "#1374"
        }
    ]
}
```

An inline quote:

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "type": "Note",
    "content": "This is a quote:<br>RE: https://server.example/objects/123",
    "tag": [
        {
            "type": "Link",
            "mediaType": "application/ld+json; profile=\"https://www.w3.org/ns/activitystreams\"",
            "href": "https://server.example/objects/123",
            "name": "RE: https://server.example/objects/123"
        }
    ]
}
```

Note that the `content` includes the `RE: <url>` microsyntax but consuming implementations are not required to parse that in order to make the appropriate associations.

## Implementations

- (streams)
- FoundKey
- Mitra
- Pleroma ([via MRF](https://git.pleroma.social/pleroma/pleroma/-/blob/v2.6.0/lib/pleroma/web/activity_pub/mrf/quote_to_link_tag_policy.ex))
- Threads ([announcement](https://engineering.fb.com/2024/03/21/networking-traffic/threads-has-entered-the-fediverse/))
- [Friendica](https://github.com/friendica/friendica/pull/14032)
- Bridgy Fed
- [Hollo](https://hollo.social/@hollo/01920132-739e-7eff-9f5f-424282884eee)
- [Iceshrimp.NET](https://iceshrimp.dev/iceshrimp/Iceshrimp.NET/src/commit/bdfd3a8d4e788ef3bdec06f32f444ed7fcffc3c7/FEDERATION.md#supported-feps)

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119], 1997
- James M Snell, Evan Prodromou, [Activity Vocabulary][ActivityVocabulary], 2017

[ActivityPub]: https://www.w3.org/TR/activitypub/
[RFC-2119]: https://tools.ietf.org/html/rfc2119.html
[ActivityVocabulary]: https://www.w3.org/TR/activitystreams-vocabulary/

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
