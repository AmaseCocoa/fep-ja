---
slug: "eb48"
title: Hashtags
dateReceived: 2024-07-16
discussionsTo: https://socialhub.activitypub.rocks/t/4369
title: Timeline Preferences
authors: AvidSeeker <avidseeker7@protonmail.com>
status: DRAFT
trackingIssue: https://codeberg.org/fediverse/fep/issues/373
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-eb48.md)から翻訳に協力することができます。

# FEP-eb48: Hashtags

## Summary

This proposal introduces a standardized method for identifying and displaying
hashtags in posts across the Fediverse. The rules define what constitutes a
hashtag and how it should be parsed and displayed, ensuring consistency and
predictability across different platforms and clients.

## Hashtag Rules

A string is considered a hashtag if it meets the following criteria:

1. It starts with the `#` symbol.
2. It is followed by one or more alphanumeric characters (letters A-Z, a-z, or digits 0-9).
3. It may include underscores (`_`) but must not include any other special characters, spaces, or punctuation within the hashtag itself.

## Examples

The following examples illustrate how hashtags should be identified and
highlighted:

- `#hashtag`
- "`#hashtag`"
- " `#hashtag`"
- (`#hashtag`/#hashtag)
- ( `#hashtag`/#hashtag)
- ( `#hashtag` /#hashtag)
- ( `#hashtag` / `#hashtag`)
- -`#hashtag`
- \_`#hashtag`
- !`#hashtag`
- ?`#hashtag`
- @`#hashtag`
- ;`#hashtag`
- ,`#hashtag`
- .'`#hashtag`
- [`#hashtag`
- &`#hashtag`
- ^`#hashtag`

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement
Proposal have waived all copyright and related or neighboring rights to this
work.
