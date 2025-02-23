---
slug: "efda"
authors: a <a@trwnh.com>
status: DRAFT
discussionsTo: https://socialhub.activitypub.rocks/t/fep-efda-followable-objects/5030
dateReceived: 2025-02-13
trackingIssue: https://codeberg.org/fediverse/fep/issues/492
---
# FEP-efda: Followable objects
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-efda.md)から翻訳に協力することができます。


## Summary

ActivityStreams Vocabulary defines a `Follow` activity, and ActivityPub defines its side effects of manipulating a `followers` collection, but ActivityPub does not specify a full algorithm for how to follow something. This FEP aims to provide guidance on which objects can be followed:

- The object MUST have a `followers` collection present.
- If the object does not have an `inbox`, then you MAY recurse upwards through `attributedTo` until you find a resource with an `inbox`. The maximum recursion depth SHOULD be 1.

A Follow activity can then be constructed for that `object` and delivered to the discovered `inbox`. Additional requirements for the structure of the Follow activity are out-of-scope.

## Requirements

In order to follow an object, we use a Follow activity. But in order to use this Follow activity, we need to know the following:

1) What can be followed?
2) Where do we send the Follow for that `object`?

At the time of writing, current software practices within the fediverse enforce a limit on what can be followed. For example, Mastodon currently limits follows to actors that can be mapped to their internal concept of "accounts", and these "accounts" are in turn limited to being any of the five "actor types" described in [AS2-Vocab]: `Person`, `Group`, `Organization`, `Application`, `Service`.

Consequently, other software which does not have such a conceptual limit is forced to declare their actors as one of the five types, or else Mastodon users will not be able to follow their actors. [ActivityPub] says that there is no specified mapping between "users" and "actors", but Mastodon's interpretation of "actors" as "users" de facto disallows following anything but a strict subset of resources.

We may therefore express Mastodon's requirements as follows:

- The actor MUST have a `type` of any of `Person`, `Group`, `Organization`, `Application`, `Service`.
- The actor MUST have an `inbox` where the `Follow` will be sent.

A proposed modified requirement and algorithm are as follows:

- The object MUST have a `followers` collection present.
- If the object does not have an `inbox`, then you MAY recurse upwards through `attributedTo` until you find a resource with an `inbox`. The maximum recursion depth SHOULD be 1.

## Algorithm

Given an object O and a recursion limit L, a general algorithm for following it can be expressed like so:

(1) Initialize a variable INBOX.
(2) If `O.followers` is not present, return an error OBJECT_CANNOT_BE_FOLLOWED.
(3) If `O.inbox` is present, set INBOX to the referenced IRI.
(4) If INBOX is unset, then initialize a variable R whose initial value is O.
(5) While INBOX is unset:
(5.1) Check that L > 0. If false, return an error MAX_RECURSION_LIMIT.
(5.2) Dereference `R.attributedTo` and set the variable R to this referenced resource.
(5.3) If `R.inbox` is present, set INBOX to the referenced IRI.
(5.4) Set the variable L to the value L - 1.
(6) Initialize a document ACTIVITY:
(6.1) ...whose `actor` is yourself
(6.2) ...whose `type` is `Follow`
(6.3) ...whose `object` is O
(7) Make an HTTP POST request whose target is INBOX and whose body is ACTIVITY.

Additional requirements for the ACTIVITY can be defined at a protocol level, but the exact protocol and those requirements are out-of-scope for this FEP. The scope of this FEP is limited to determining which objects can be followed, and where to send the Follow.

## References

- James M Snell, Evan Prodromou, [Activity Vocabulary][AS2-Vocab], 2017
- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- trwnh, [Unresolved issues surrounding Follow activities][UNRESOLVED], 2019

[AS2-Vocab]: https://www.w3.org/TR/activitystreams-vocabulary/
[ActivityPub]: https://www.w3.org/TR/activitypub/
[UNRESOLVED]: https://socialhub.activitypub.rocks/t/unresolved-issues-surrounding-follow-activities/114

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
