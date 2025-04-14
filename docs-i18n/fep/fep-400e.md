---
slug: "400e"
authors: Gregory Klyushnikov <activitypub@grishka.me>
status: FINAL
dateReceived: 2021-02-16
dateFinalized: 2022-02-04
trackingIssue: https://codeberg.org/fediverse/fep/issues/5
discussionsTo: https://codeberg.org/fediverse/fep/issues/5
---
# FEP-400e: Publicly-appendable ActivityPub collections
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-400e.md)から翻訳に協力することができます。


## Summary

In social media, it's a frequent pattern when there's a collection owned by someone that other people can contribute to. Examples include:

- Walls where others can post
- Forums as well as topics within
- Photo albums in groups where group members can add photos

Currently, there is no generic way to signify that an object was created as part of a collection and should only be considered in its context.

This proposal describes how ActivityPub servers and clients could specify collections to which objects created by their actors belong.

## Requirements

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this specification are to be interpreted as described in [RFC-2119].

## Publicly-appendable collections

A publicly-appendable collection is any collection where it is expected that someone other than its owner could add items but over which its owner retains complete authority. For example, a wall is a collection to which other people could add posts (`Note`s), but from which its owner could delete any posts as well as restrict who can add them.

A publicly-appendable collection SHOULD have a valid and globally-unique `id` that SHOULD point to either a `Collection` or an `OrderedCollection` object that contains the links to all its objects.

## Specifying collections in actors

If an actor has publicly-appendable collections, its server MAY include them as additional fields in its ActivityPub representation. For example, user actors may specify the link to their walls, or groups may specify the link to the collection of their photo albums.

Implementations MAY use the presence or absence of specific collection to determine whether the actor's server supports features that depend on that collection and alter their UIs accordingly.

## Using `target` in objects

If an ActivityPub object is being created as part of a collection, the object SHOULD include the `target` field that contains an abbreviated collection object, which SHOULD contain at least the following fields:

* `type` — either `Collection` or `OrderedCollection`.
* `id` — the `id` of the collection.
* `attributedTo` — the `id` of the owner of the collection. This is necessary to simplify the database design on the receiving side.

### Discussion

While [Activity Vocabulary] specifies `target` as a field with similar semantics in activities, it's important to include it in objects themselves so any software that only sees the object without its enclosing `Create` activity, e.g. when following a link form another object or retrieving the object from a user-provided URI, unambiguously knows that it should only be considered in the context of its collection.

## Adding an object to a collection

When an ActivityPub server receives in its inbox a correctly signed `Create` activity with an object that has the `target` field, it does the following:

- Retrieve the collection owner using either `attributedTo` or `id` fields of the abbreviated collection object.
  - If the collection owner does not exist, or if `attributedTo` doesn't match the actual owner of the collection specified by `id`, or if the collection owner is not a local actor, the server SHOULD abort processing and MAY return `400 Bad Request`.
  - If the object could not be added to the collection, for example due to the privacy settings configured by its owner, the server SHOULD either respond with `403 Unauthorized` or respond with `200 OK` and later send a `Reject{Create}` activity to the originating server.
- Store either the entire object or its `id` in its local storage as belonging to the specified collection.
- Send an `Add` activity to any parties that might be concerned with it. The `target` field in the activity SHOULD only be the collection `id`, and the `object` field SHOULD be the `id` of the object being added. It is RECOMMENDED that this activity is sent to all the collection owner's followers for the sake of data consistency, and it SHOULD be sent to the actor that created the object being added.
- Perform any implementation-specific processing, like sending notifications.

### Receiving an `Add` activity

When an ActivityPub server receives in its inbox a correctly signed `Add` activity, it SHOULD do the following:

- Retrieve the `actor` either from local storage or from the network.
- Retrieve the `object`.
- Check that `target` is the ID of a collection owned by `actor`. If it is not, abort processing and return `400 Bad Request`.
- Check that `target` in the activity matches `target.id` in the object and that `target.attributedTo` in the object matches the actor ID. If it does not, abort processing and return `400 Bad Request`.
- Store either the entire object or its `id` in its local storage as belonging to the specified collection.
- Perform any implementation-specific processing, like sending notifications.

### Example of an `Add` activity

```json
{
  "@context":"https://www.w3.org/ns/activitystreams",
  "actor":"https://example.com/users/1",
  "id":"https://example.com/posts/41864/activityAdd",
  "to":[
    "https://www.w3.org/ns/activitystreams#Public",
    "https://example.com/users/1/followers",
    "https://example.com/users/6946"
  ],
  "type":"Add",
  "object":"https://example.com/posts/41864",
  "target":"https://example.com/users/1/wall"
}
```

## Deleting an object from a collection

Since the collection owner has complete authority over the contents of the collection, they can delete any objects from it. When an object is deleted from a collection by its owner, their server SHOULD send a `Delete` activity to at least the server of the actor that created the object; it's also RECOMMENDED that this activity is sent to all the servers that `Add` was sent to. Those servers then SHOULD delete the object as if the deletion was initiated by its creator.

## Moving an object between collections

In some use cases, it might make sense to allow objects to be moved between collections, for example, a group moderator might want to move a photo between photo albums in a group, or a forum moderator might want to split some messages into a separate thread. It's only possible to move objects between collections that are owned by the same actor.

When moving an object between collections, the collection owner SHOULD send a `Move` activity to at least the server of the object creator, specifying the target collection and the `id` of the object; it's also RECOMMENDED that this activity is sent to all the servers that `Add` was sent to. Those servers then SHOULD update the `target` field in their stored copies of the object.

## Security considerations

The requirement of an `Add` activity sent by a collection owner largely prevents a bad actor from effectively adding something to a collection against the collection owner's will while also helping data consistency across servers. However, there's still one case when this is possible. When a server fetches an object that has a `target` field directly, for example when a user has entered its URL into a search box or when it is referenced by a field such as `inReplyTo`, there's presently no reliable way to verify whether the object actually belongs to the collection.

## 実装

This proposal is implemented in Smithereen for both user and group walls since the following commit: https://github.com/grishka/Smithereen/commit/de013593dde06a3091ecfbd32960a694d79c146e

## 参考文献

- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119.html)
- [Activity Vocabulary], James M Snell, Evan Prodromou, [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/)


## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。