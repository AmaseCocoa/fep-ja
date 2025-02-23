---
slug: "1b12"
authors: Felix Ableitner <me@nutomic.com>
status: FINAL
dateReceived: 2022-11-12
dateFinalized: 2023-02-09
trackingIssue: https://codeberg.org/fediverse/fep/issues/22
discussionsTo: https://codeberg.org/fediverse/fep/issues/22
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-1b12.md)から翻訳に協力することができます。
# FEP-1b12: Group federation


## Summary

Internet forums are one of the oldest forms of social media. This document describes how they are implemented in existing Activitypub platforms using `Group` actors. It also introduces a new property to indicate that a given object belongs to a group.

## History

Friendica released federated forums in version [2019.03](https://github.com/friendica/friendica/releases/tag/2019.03).

Lemmy published the first public beta of federated groups in [v0.8.0 (October 2020)](https://join-lemmy.org/news/2020-10-20_-_Lemmy_Release_v0.8.0_-_Federation_beta!).

[FEP-400e] introduces publicly appendable collections, which can also be used to implement forums. However they are incompatible with the implementations described here.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this specification are to be interpreted as described in [RFC-2119].

## Group actor

A federated forum is represented by a `Group` actor as specified in [Activity Vocabulary]. This actor is not directly controlled by a human, but can perform its main functionality in a fully automated way, essentially acting as a bot. It has the same general properties as any `Person` actor.

### Following a Group

Public groups SHOULD support a standard `Follow`-`Accept` workflow. This ensures compatibility with existing implementations which support `Person` following using the same vocabulary. After receiving a valid `Follow` activity, the group SHOULD automatically respond with an `Accept/Follow`, and add the sender to its followers collection. Correspondingly actors can unfollow a group by sending an `Undo/Follow` activity.

## Audience property

In order to render content in a forum, it is necessary to know which particular forum the content belongs to. This way users can navigate up from a thread to the forum's main page, or view metadata like the description or moderators. With current implementations there is no easy way to determine which forum a given activity or object belongs to, if any.

Currently there are different approaches to specify which group a given object or activity belongs to. Lemmy, Friendica and lotide put the group ID in the `to` field. Peertube uses `attributedTo`. Both properties have the problem that they are also used for different purposes by some platforms, and are represented as arrays. So to retrieve the group from a received object, an implementation needs to loop through these properties and resolve each URL, until it hits one which resolves to a `Group`. Clearly this is very inefficient.

To simplify this process, we propose to specify the group identifier in the `audience` property. This property is part of Activitystreams, but not yet used in the wild. This way no extension is necessary, and neither will it cause problems for existing implementations. Additionally, platforms can continue to federate the group identifier in the existing format for backwards compatibility.

## Threads and comments

Each `Group` actor represents a single forum. Forums contain many user-submitted threads, which can be represented by different objects depending on the implementation. Thread objects SHOULD have a `name` property which represents the thread title.

Example thread:
```
{
    "type": "Page",
    "id": "https://sally.example.org/p/1",
    "attributedTo": "https://sally.example.org",
    "name": "Hello forum!",
    "audience": "https://example.org/my-forum"
}
```

Threads can have replies, which are usually represented as `Note`s. They MUST have a property `inReplyTo` referencing either the thread they belong to, or the parent reply in case of nested replies.

Example reply:
```
{
    "type": "Note",
    "id": "https://sally.example.org/p/3",
    "attributedTo": "https://sally.example.org",
    "inReplyTo": "https://sally.example.org/p/1",
    "content": "My first comment",
    "audience": "https://example.org/my-forum"
}
```

The properties `inReplyTo` and `audience` can be used to navigate up from a reply to a thread and forum. To navigate down from a group to threads and comments, groups MAY have a `replies` collection which contains all threads. Each thread MAY again have a `replies` collection which lists all top-level comments responding to the thread.

## The Announce activity

The main task of a group is to distribute content among its followers.

When a group receives a activity in its inbox, it SHOULD perform some automatic validation, such as checking for domain and user blocks. Groups MAY require additional validation, such as accepting content only from followers, or even manual approval from group moderators. In case an activity fails these checks, the group MAY respond to the sender with a `Reject` activity.

In case the incoming activity is deemed valid, the group MUST wrap it in an `Announce` activity, with the original activity as object. The wrapped activity MUST be preserved exactly as it was received, without changing or removing any properties. This ensures that forwarded activities can be verified with [Object Integrity Proofs]. Announce activities SHOULD get added to the group outbox. If the group exposes collections of threads and comments, relevant items should also be added to them.

Example:
```
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Announce",
  "actor": "https://example.org/my-forum",
  "id": "example.org/a/5",
  "object": {
    "@context": "https://www.w3.org/ns/activitystreams",
    "type": "Create",
    "id": "https://sally.example.org/a/2"
    "actor": "https://sally.example.org",
    "audience": "https://example.org/my-forum",
    "object": {
      "type": "Page",
      "id": "https://sally.example.org/p/1",
      "content": "Hello forum!",
      "audience": "https://example.org/my-forum"
    }
  }
}
```

After the group successfully verifies and wraps the received activity, it sends it to the inboxes of its followers. Followers then use the outer `Announce` activity to verify that the content was really approved by the group. After this step the `Announce` can be discarded and only the inner activity shown to users.

This mechanism can be used to publish any possible activity type. Examples include `Announce/Like`, `Announce/Delete/Note` or `Announce/Undo/Like`. Implementations may choose not to forward some activity types which are considered private, for example `Follow` activities.

## Group moderation

Group moderators are those actors who control the group, are able to change its metadata and remove malicious content. They are listed in the group’s `attributedTo` collection. Moderation is an optional feature, implementations can safely ignore this entire section. At the moment it is only implemented by Lemmy.

```
{
  "id": "https://example.org/my-forum",
  "type": "Group",
  "name": "Ten Forward",
  "attributedTo": "https://example.org/my-forum/moderators",
}
```

```
{
  "type": "OrderedCollection",
  "id": "https://example.org/my-forum/moderators",
  "orderedItems": [
    "https://example.org/picard",
    "https://example.org/riker"
  ]
}
```

Group moderators can be changed with `Add` and `Remove` activities:

```
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Announce",
  "actor": "https://example.org/my-forum",
  "id": "example.org/a/8",
  "object": {
    "id": "https://example.org/a/7",
    "type": "Add",
    "actor": "https://example.org/picard",
    "object": "http://example.org/data",
    "audience": "https://example.org/my-forum",
    "target": "https://example.org/my-forum/moderators"
  }
}
```

The actions which can be done by moderators are called moderation activities. These are implementation specific, examples include `Add`, `Remove` (to change the moderators collection), `Block` (ban malicious users) and `Update/Group` (change group metadata).

If an group or group follower supports moderation, it MUST validate incoming moderation activities before further processing. Such activities MUST have an actor who is listed in `attributedTo`. Group followers MUST additionally verify that the moderation activity was announced by the group.

Implementations SHOULD also accept moderation activities which come from the same server where the community is hosted, under the assumption that these are sent by server administrators. These moderation activities also need to be wrapped in `Announce` by the group.

## Implementations

This document is written based on existing group implementations in Lemmy, Friendica, Hubzilla, Lotide and Peertube. These already federate successfully in production.

The `audience` field is an exception as it is not in use yet. Lemmy will add support for it in version 0.17.0.

## References

- [FEP-400e] Gregory Klyushnikov, [FEP-400e: Publicly-appendable ActivityPub collections](https://codeberg.org/fediverse/fep/src/branch/main/fep/400e/fep-400e.md)
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119.html)
- [Activity Vocabulary], James M Snell, Evan Prodromou, [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/)
- [Object Integrity Proofs] silverpill, [FEP-8b32: Object Integrity Proofs](https://codeberg.org/fediverse/fep/src/branch/main/fep/8b32/fep-8b32.md)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
