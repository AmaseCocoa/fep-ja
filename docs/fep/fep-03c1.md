---
slug: "03c1"
authors: helge <@helge@mymath.rocks>
status: DRAFT
dateReceived: 2023-11-10
trackingIssue: https://codeberg.org/fediverse/fep/issues/205
discussionsTo: https://codeberg.org/fediverse/fep/issues/205
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-03c1.md)から翻訳に協力することができます。
# FEP-03c1: Actors without acct-URI

## Summary

Most current Fediverse applications use an acct-URI as unique display name
for actors. Usually, this display is done by displaying `acct:user@domain.example`
as `@user@domain.example`. This FEP states that if there is no
acct-URI associated with an actor, the actor should be displayed
as its id. So the actor with id `https://actor.example/path` will be
displayed as `https://actor.example/path`.

In addition to the example below, we wish to point out that further independence of webfinger will enable new features such as using domain names as handles.

## Example: RSS

Consider building an application that posts new entries of an RSS
feed to the Fediverse. Let's assume we are interested in the updates
of the [bovine python library](https://pypi.org/project/bovine/)
available through RSS at [https://pypi.org/rss/project/bovine/releases.xml](https://pypi.org/rss/project/bovine/releases.xml).
Then an uri for this actor might look like

```url
https://rss.example/actor?feed=https%3A%2F%2Fpypi.org%2Frss%2Fproject%2Fbovine%2Freleases.xml
```

This would enable `rss.example` to automatically generate the respective actor and generate an actor object like

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "https://rss.example/actor?feed=https%3A%2F%2Fpypi.org%2Frss%2Fproject%2Fbovine%2Freleases.xml",
    "type": "Service",
    "inbox": "https://rss.example/inbox?feed=https%3A%2F%2Fpypi.org%2Frss%2Fproject%2Fbovine%2Freleases.xml",
    "outbox": "https://rss.example/outbox?feed=https%3A%2F%2Fpypi.org%2Frss%2Fproject%2Fbovine%2Freleases.xml",
    "followers": "https://rss.example/followers?feed=https%3A%2F%2Fpypi.org%2Frss%2Fproject%2Fbovine%2Freleases.xml",
    "name": "PyPI recent updates for bovine",
    "url": "https://pypi.org/project/bovine/",
    "summary": "Recent updates to the Python Package Index for bovine"
}
```

There is now no obvious choice for `preferredUsername` and thus acct-URI.
If one wanted to create one, the best choice would probably be to hash the url of the feed. This unfortunately has the consequence of not generating a human readable id, but quite the opposite.

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
