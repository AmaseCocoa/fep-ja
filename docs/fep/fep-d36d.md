---
slug: "d36d"
authors: Zack Dunn <zack@tilde.team>
status: DRAFT
dateReceived: 2023-07-01
trackingIssue: https://codeberg.org/fediverse/fep/issues/125
discussionsTo: https://codeberg.org/fediverse/fep/issues/125
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-d36d.md)から翻訳に協力することができます。
# FEP-d36d: Sharing Content Across Federated Forums

## Summary

New instances on the threadiverse (servers that implement ActivityPub with FEP-1b12) are often seeded with forums for common
interests, leading to multiple servers having similar forums. Users may dislike having to follow what they perceive to be
"duplicate" forums or keep up with multiple discussions on the same topic across multiple servers. This document describes a
method for allowing `Group` actors to share content to reduce posting of a single link multiple times, which reduces what users
see as "duplicate" posts and fragmented conversations across multiple forums.

## History

FEP-1b12 introduces federated forums and is implemented by [Lemmy](https://github.com/LemmyNet/lemmy/), [/kbin](https://codeberg.org/Kbin/kbin-core/), and [Friendica](https://github.com/friendica/friendica).

The site [reddit](https://old.reddit.com) has a feature for grouping its forums, called subreddits, into a new forum, called a
multireddit. A multireddit is a feed made up of the combination of each of its constituent subreddits and allows a user an easy
way to keep up with multiple related forums. Because subreddits can be in multiple multireddits, multireddits don't affect
moderation of links posted to individual subreddits and a link can be posted to more than one constituent subreddit within a
multireddit.

## Group to Group Follows

When a moderator of a federated forum determines that their forum overlaps in topic with another forum, they can direct the
`Group` actor to send a `Follow` activity to the other forum's `Group` actor. FEP-1b12 specifices that a group should
automatically respond with an `Accept/Follow`, but this document overrides that for `Follow` activities with an `actor` of type
`Group`. After receiving a `Follow` activity from another `Group`, the group MAY automatically respond with an `Accept/Follow`
or a moderator may instruct the group to reply with a `Reject/Follow`. After replying with a `Accept/Follow` activity, the
group that received the `Follow` activity MAY automatically add the first group to its `following` collection, creating a
symmetric relationship.

This document makes no change to the handling of an `Undo/Follow` activity. If a group receives an `Undo/Follow` from a `Group`
actor, it MAY automatically remove the other group from its `following` collection.

## Activity Handling

When a group receives an activity in its `inbox`, it SHOULD perform automatic validation as described in FEP-1b12. If that
validation includes deduplication (via the `url` property of the activity's `object`, the `url` of any attachements, or any
other method), that deduplication validation MUST include objects received from followed groups. If an activity fails this
deduplication validation, the group MUST respond with a `Reject` activity where the `object` property is the `object` from
the inbox activity and the `target` object is the object that the new object duplicates. This ensures that content is posted
only once across related forums and a forum can provide navigation to an original post when a user tries to post a duplicate.

If the incoming activity is deemed valid, the group MUST handle it according to FEP-1b12 handling of valid activities.

## References
[FEP-1b12] Felix Ableitner, [FEP-1b12: Group federation](https://codeberg.org/fediverse/fep/src/branch/main/fep/1b12/fep-1b12.md)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or
neighboring rights to this work.

