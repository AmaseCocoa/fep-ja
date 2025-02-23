---
slug: "c648"
authors: Evan Prodromou <evan@prodromou.name>
status: DRAFT
dateReceived: 2023-06-14
trackingIssue: https://codeberg.org/fediverse/fep/issues/123
discussionsTo: https://codeberg.org/fediverse/fep/issues/123
---
# FEP-c648: Blocked Collection
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-c648.md)から翻訳に協力することができます。

## Summary

Users need to review and revise the list of actors they have blocked. This FEP proposes a new collection property, the Blocked Collection, which contains the actors that a user has blocked and all metadata about the `Block` activity.

## History

The [ActivityPub] specification defines a `Block` activity, which is used to block an actor. However, the specification does not define an efficient way to retrieve the list of actors that a user has blocked.

The `followers` and `following` collection properties of an actor hold objects in the
actor's social graph.

[activitypub-express] implements
a `blocked` property in the `streams` collection of an actor, including the blocked objects only. The developers' experience was that storing objects only made it hard to `Undo` a block, since the full Activity object's `id` is needed. Metadata about the block activity, such as the date, is also lost.

In this proposal, the `blocked` collection holds `Block` activities.

## Details

The `blocked` property of an actor is a collection of `Block` activities, also known as a 'blocklist'.

The `blocked` property MUST be an `OrderedCollection` or a `Collection`.

The `blocked` collection SHOULD include all `Block` activities by the actor, except for those that have been reverted by an `Undo` activity.

Each activity in the `blocked` collection MUST be unique.

The `blocked` collection MUST be sorted in reverse chronological order, with the most recent activity first.

## Context

The context document for the `blocked` property is as follows:

```
{
  "@context": {
    "bl": "https://purl.archive.org/socialweb/blocked#",
    "blocked": {
      "@id": "bl:blocked",
      "@type": "@id"
    }
  }
}
```

The context document is available at the URL `https://purl.archive.org/socialweb/blocked`.

## 例

A publisher can include the `blocked` collection in the properties of an actor.

```
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://purl.archive.org/socialweb/blocked"
    ],
    "id": "https://example.com/evanp",
    "type": "Person",
    "name": "Evan Prodromou",
    "inbox": "https://example.com/evanp/inbox",
    "outbox": "https://example.com/evanp/outbox",
    "following": "https://example.com/evanp/following",
    "followers": "https://example.com/evanp/followers",
    "liked": "https://example.com/evanp/liked",
    "blocked": "https://example.com/evanp/blocked"
}
```

Retrieving the `blocked` collection would provide a list of `Block` activities.

```
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://purl.archive.org/socialweb/blocked",
        {"custom": "https://example.com/ns/custom"}
    ],
    "id": "https://example.com/evanp/blocked",
    "type": "OrderedCollection",
    "name": "Evan Prodromou's Blocked Collection",
    "orderedItems": [
        {
            "type": "Block",
            "id": https://example.com/evanp/block/2",
            "object": {
                "type": "Person",
                "id": "https://spam.example/spammer",
                "name": "Irritating Spammer"
            },
            "published": "2023-04-15T00:00:00Z"
        },
        {
            "type": ["custom:Disallow", "Block"],
            "id": https://example.com/evanp/block/2",
            "object": {
                "type": "Application",
                "id": "https://alarmclock.example/alarmclock",
                "name": "Badly-Behaved Alarm Clock App"
            },
            "published": "2022-12-25T00:00:00Z"
        }
    ]
}
```

Note that the second, earlier activity has two `type` values; one is `Block` and the other is a custom activity type defined in its own namespace.

## Security considerations

The `blocked` collection is very sensitive. Actors on the blocked list may be harassing or abusive. If they find themselves on a user's blocklist, they may retaliate against the user. Consequently, the `blocked` collection SHOULD NOT be publicly readable.

By default, implementations SHOULD NOT allow read access to the `blocked` collection to any actor other than the user that owns the collection.

Some users may want to share their blocklist with other actors. Shared blocklists are an important tool for user safety on monolithic social networks and on the social web. Implementations MAY allow a user to share their `blocked` collection with other actors. Implementations SHOULD inform the user of the risks of sharing their blocklist with the wrong actors.

## Implementations

The [onepage.pub] server implements the `blocked` collection.

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [onepage.pub] Evan Prodromou, [onepage.pub](https://github.com/evanp/onepage.pub/), 2023
- [activitypub-express](https://github.com/immers-space/activitypub-express)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
