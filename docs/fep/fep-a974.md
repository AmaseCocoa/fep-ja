---
slug: "a974"
authors: James Smith <james@floppy.org.uk>
status: DRAFT
discussionsTo: https://socialhub.activitypub.rocks/t/proposed-fep-a974-all-actor-types-should-be-followable/5012
dateReceived: 2025-02-05
trackingIssue: https://codeberg.org/fediverse/fep/issues/487
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-a974.md)から翻訳に協力することができます。
# FEP-a974: All Actor types should be followable

## Summary

In order to foster interoperability and good semantics, any valid unblocked Actor should be visible and followable on any platform when searched for. The `type` of the Actor should not matter for initial following, though can be used later as appropriate.

## Motivation

In order to federate, an ActivityPub service will expose a set of Actors; these are ActivityPub objects that (a) can perform activities and (b) can be followed (see the [ActivityPub Specification][ActivityPub]).

Each Actor has a `type`. The [Activity Vocabulary][ActivityVocab] defines five "core" types: `Person`, `Group`, `Service`, `Organization`, and `Application`. These five types work well for many applications, but not all, and ActivityPub explicitly allows Actors to have any `type`.

In many services, it is desirable to use other object types for actors; either defined Object types or even custom types. For instance:

* on a reading-oriented platform, a user may want to follow a specific series of books to see when new items are added. The book series would be semantically represented as an `OrderedCollection`, rather than a `Group`.
* on a music platform, a user may want to follow a playlist; this would also be an `OrderedCollection`, or could even a custom `Playlist` type if the platform wanted to communicate specific semantics.

However, some ActivityPub platforms may choose to only show actors of the five core types in search; if a platform wanted to use more semantically-meaningful types for its actors, those actors would be unavailable on such sites, which will inevitably lead to bug reports to the services using the non-core types.

While filtering at the Activity/Object type is sensible and inevitable for any ActivityPub platform, filtering at the Actor type level limits the ability of new services to use semantically-correct types as appropriate, and will over time make the type effectively useless, as new services will choose one of the five core types even if inappropriate, in order for their actors to be visible.

This issue has been discussed in the community before (see [Mastodon issue #22322][Masto22322]); this FEP attempts to move that discussion into  a single simple statement of compatibility.

## Decision

Compliant ActivityPub services should not filter on Actor `type` in searches or actor-level activities (Follow, Accept, Undo, Block,  etc). Any unblocked Actor should be followable on any service.

## Impact

Services are of course free to filter activities that are delivered later on; this FEP puts no limit on that. The feed for an Actor that posts only `Document` object activities may appear completely empty to a user of a microblogging platform; it remains the right and privilege of any ActivityPub service to only process the activities it wants to.

However, by making all Actor types followable, new services can choose what activities to send where, safe in the knowledge that their actors will at least be visible and that the activities will be received.

A practical example: [Manyfold](Manyfold) allows Fediverse users to follow individual 3D Models (which could have a `Document` or even `3DModel` actor type), and when they change, it posts `Update` activities where those models are the `object`. However, Manyfold knows that microblog applications don't understand those activities (and nor should they). Therefore, for compatibility, it sends what it terms a "compabitility note", a human-readable `Create Note` activity with the same information as the `Update 3DModel`, thus allowing microblog users to follow models and get updates wherever convenient. A decision on what to send where could use the compatibility detection proposed in [FEP-9fde][FEP-9fde] in future.

Potential negative impacts may happen if a service makes extra assumptions about an Actor based on its use of the core types (e.g. the background to [Mastodon issue #22322][Masto22322] which talks about different semantics being used for `Group` actors), and discussion is invited on those impacts.

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- James M Snell, Evan Prodromou, [Activity Vocabulary][ActivityVocab], 2017
- James Smith, [Manyfold ActivityPub Documentation][Manyfold], 2025

[ActivityPub]: https://www.w3.org/TR/activitypub/
[ActivityVocab]: https://www.w3.org/TR/activitystreams-vocabulary/
[FEP-2277]: https://codeberg.org/fediverse/fep/src/branch/main/fep/2277/fep-2277.md
[FEP-9fde]: https://codeberg.org/fediverse/fep/src/branch/main/fep/9fde/fep-9fde.md
[Manyfold]: https://manyfold.app/technology/activitypub
[Masto22322]: https://github.com/mastodon/mastodon/issues/22322

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
