---
slug: "f228"
authors: silverpill <@silverpill@mitra.social>
type: implementation
status: DRAFT
discussionsTo: https://codeberg.org/silverpill/feps/issues
dateReceived: 2025-02-17
trackingIssue: https://codeberg.org/fediverse/fep/issues/500
---
# FEP-f228: Backfilling conversations
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-f228.md)から翻訳に協力することができます。

## Summary

The most common conversation backfill method is based on recursive retrieval of posts indicated by `inReplyTo` property and posts contained in `replies` collections. This is inefficient.

[FEP-7888: Demystifying the context property][FEP-7888] suggests using the `context` property for grouping related objects (such as posts in a conversation). This property can resolve to a collection, which can be used for efficient backfilling without recursion.

Two different implementations of `context` collection exist: collection of posts and collection of activities.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119].

## Collection of posts

This collection represents a [thread] and contains all posts in a conversation, from the perspective of the conversation owner.

It is an `OrderedCollection`, and the order of items is chronological. It MUST contain at least one item, the top-level post. This post MUST have a `context` property referring to the collection. Other items might not have this property,

When `context` property is present on a post, it MUST resolve to a collection of posts.

There is a difference between contents of this collection and a reply tree defined by `inReplyTo` and `replies` relationships, because conversation owner might choose to not include certain replies.

## Collection of activities

This collection contains all activities related to posts in a conversation, including but not limited to:

- `Create`
- `Update`
- `Delete`
- `Like`

It is an `OrderedCollection`, and the order of items is chronological. It MUST contain at least one item, the `Create` activity for the top-level post. This activity MUST have a `context` property referring to the collection. Other activities might not have this property.

When `context` property is present on an activity, it MUST resolve to a collection of activities.

### Compatibility with Conversation Containers

In [Conversation Containers][FEP-171b] this collection would coincide with the conversation container. It will contain `Add` activities in addition to other activities.

### `history` property

Collections described in this document can be implemented separately.

If both of them are implemented, the `history` property can be added to a collection of posts, indicating a corresponding collection of activities. This property is defined in [FEP-bad1: Object history collection][FEP-bad1], although the use case here differs from the one described in that proposal.

### `contextHistory` property

`contextHistory` property can be used to make a reference from a post to a collection of activities.

## Reading collections

After top-level post of a conversation is discovered, the whole conversation can be retrieved using the following algorithm:

- If `contextHistory` property is present, retrieve collection of activities and stop.
- If `context` property is present, retrieve collection of posts and stop.
- If `replies` property is present, retrieve collection of replies, and repeat this step for every reply.

## 実装

Collection of posts:

- NodeBB
- Iceshrimp.NET
- WordPress

Collection of activities:

- Streams
- Hubzilla

## 参考文献

- a, [FEP-7888: Demystifying the context property][FEP-7888], 2023
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119], 1997
- silverpill, [FEP-171b: Conversation Containers][FEP-171b], 2024
- a, [FEP-bad1: Object history collection][FEP-bad1], 2023

[RFC-2119]: https://tools.ietf.org/html/rfc2119.html
[thread]: https://en.wikipedia.org/wiki/Thread_(online_communication)
[FEP-7888]: https://codeberg.org/fediverse/fep/src/branch/main/fep/7888/fep-7888.md
[FEP-171b]: https://codeberg.org/fediverse/fep/src/branch/main/fep/171b/fep-171b.md
[FEP-bad1]: https://codeberg.org/fediverse/fep/src/branch/main/fep/bad1/fep-bad1.md

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
