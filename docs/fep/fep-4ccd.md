---
slug: "4ccd"
authors: Evan Prodromou <evan@prodromou.name>
status: DRAFT
dateReceived: 2023-06-21
trackingIssue: https://codeberg.org/fediverse/fep/issues/129
discussionsTo: https://codeberg.org/fediverse/fep/issues/129
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-4ccd.md)から翻訳に協力することができます。
# FEP-4ccd: Pending Followers Collection and Pending Following Collection

## Summary

This specification defines two collections, `pendingFollowers` and `pendingFollowing`, with which users can review and manage their pending follow requests.

## Motivation

[ActivityPub] represents a directed social graph with `followers` and `following` properties on actors. To initiate a relationship, a `Follow` activity is sent from the potential follower to the followed actor, who can `Accept` or `Reject` it.

Between the time that a `Follow` is sent and the time that it is accepted or rejected, the relationship is in a pending state. This is not represented in the ActivityPub data model.

The new `pendingFollowers` collection can be used to review incoming `Follow` activities to `Accept` or `Reject` them.

The new `pendingFollowing` collection can be used to review outgoing `Follow` activities to `Undo` them.

Because the full activity data is needed to `Accept`, `Reject` or `Undo`, these collections should include `Follow` activities, and not just the actors requesting to follow.

## Details

`pendingFollowers` is a property of an actor. It is a collection of `Follow` activities that have been sent **to** the actor, but not yet been accepted or rejected.

`pendingFollowing` is a property of an actor. It is a collection of `Follow` activities that have been sent **by** the actor, but not yet been accepted or rejected.

Both properties MUST be an `OrderedCollection` or `Collection`. Items in the collection MUST be in reverse chronological order.

Items in the collections MUST be `Follow` activities. They MUST be unique by `id`.

Each `object` of a `Follow` activity in the `pendingFollowing` collection MUST be unique by `id`.

Each `actor` of a `Follow` activity in the `pendingFollowers` collection MUST be unique by `id`.

When an `Accept`, `Reject`, or `Undo` activity with a `Follow` activity as `object` is processed, that `Follow` activity MUST NOT be included in the `pendingFollowers` and `pendingFollowing` collections in the future.

## Context

The context document for this specification is `https://purl.archive.org/socialweb/pending`. Its contents are as follows:

```
{
  "@context": {
    "pdg": "https://purl.archive.org/socialweb/pending#",
    "pendingFollowers": {
      "@id": "pdg:pendingFollowers",
      "@type": "@id"
    },
    "pendingFollowing": {
      "@id": "pdg:pendingFollowing",
      "@type": "@id"
    }
  }
}
```

## 例

A publisher can include the `pendingFollowers` and `pendingFollowing` collection in the properties of an actor.

```
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://purl.archive.org/socialweb/pending"
    ],
    "id": "https://example.com/evanp",
    "type": "Person",
    "name": "Evan Prodromou",
    "inbox": "https://example.com/evanp/inbox",
    "outbox": "https://example.com/evanp/outbox",
    "following": "https://example.com/evanp/following",
    "followers": "https://example.com/evanp/followers",
    "liked": "https://example.com/evanp/liked",
    "pendingFollowers": "https://example.com/evanp/pendingFollowers",
    "pendingFollowing": "https://example.com/evanp/pendingFollowing"
}
```

Retrieving the `pendingFollowers` collection would show incoming follow requests
for this actor.

```
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://purl.archive.org/socialweb/pending"
    ],
    "id": "https://example.com/evanp/pendingFollowers",
    "type": "OrderedCollection",
    "name": "Pending followers for Evan Prodromou",
    "orderedItems": [
        {
            "type": "Follow",
            "id": "https://example.net/alyssa/follow/7",
            "summary": "Alyssa wants to follow Evan",
            "content": "Hey, Evan! It's Alyssa from the conference.",
            "actor": {
                "id": "https://example.net/alyssa",
                "type": "Person",
                "name": "Alyssa P. Hacker"
            },
            "published": "2023-06-21T12:00:00Z"
        },
        {
            "type": ["http://custom.example/ns/Archive", "Follow"],
            "id": "https://social.example/jokebot3000/follow/287",
            "summary": "Jokebot 3000 wants to follow Evan to archive his jokes",
            "actor": {
                "id": "https://social.example/jokebot3000",
                "type": "Application",
                "name": "Jokebot 3000"
            },
            "published": "2023-05-07T12:00:00Z"
        }
    ]
}
```

Note that the second, earlier `Follow` activity has a custom `type` property. Note also that the `object` of the `Follow` activities, which will be the same for every activity, is elided for clarity and space.

```
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://purl.archive.org/socialweb/pending",
        {"sports": "https://sports.example/ns#"}
    ],
    "id": "https://example.com/evanp/pendingFollowing",
    "type": "Collection",
    "name": "Pending following for Evan Prodromou",
    "items": [
        {
            "type": ["sports:Fan", "Follow"],
            "id": "https://example.com/evanp/fan/309",
            "summary": "Evan is a fan of Jimena",
            "object": {
                "id": "https://tennis.example/jimena",
                "type": "Person",
                "name": "Jimena Suarez"
            },
            "published": "2023-04-19T12:00:00Z"
        },
        {
            "type": "Follow",
            "id": "https://example.net/evanp/follow/214",
            "summary": "Evan wants to follow Montreal Weather Updates",
            "object": {
                "id": "https://weather.example/canada/quebec/montreal",
                "type": "Service",
                "name": "Montreal Weather Updates"
            },
            "published": "2023-02-11T12:00:00Z"
        }
    ]
}
```

Note that the first `Follow` activity has a custom `type` property. Note also that the `actor` of the `Follow` activities, which will be the same for every activity, is elided for clarity and space. Finally, note that even though the collection's type is `Collection` and the items property is `items`, the activities still must be in reverse chronological order.

## Security considerations

The `pendingFollowers` and `pendingFollowing` collections are sensitive information about
an actor's social connections. For privacy, some services and actors do not share the `following` or `followers` collections. If not similarly protected, the `pendingFollowers` and `pendingFollowing` collections could be used to infer information about the actor's social connections before they are established.

Some services or actors do not forward `Reject` activities to the actor of a `Follow` activity. Harassing or abusive actors may try to determine if the actor has rejected their follow request by fetching the `pendingFollowers` collection.

For these reasons, publishers SHOULD NOT make the `pendingFollowers` and `pendingFollowing` collections visible to unauthenticated users. Publishers SHOULD NOT make the `pendingFollowers` and `pendingFollowing` collections visible to authenticated users who are not the actor.

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
