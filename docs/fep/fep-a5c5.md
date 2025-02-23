---
slug: "a5c5"
authors: AvidSeeker <avidseeker7@protonmail.com>
status: DRAFT
dateReceived: 2024-07-15
discussionsTo: https://socialhub.activitypub.rocks/t/4362
title: Web Syndication Methods
date: 2024-07-15
trackingIssue: https://codeberg.org/fediverse/fep/issues/368
---
# FEP-a5c5: Web Syndication Methods
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-a5c5.md)から翻訳に協力することができます。

## Summary

This document proposes a standard for web syndication methods across the
Fediverse by appending `.rss` or `.atom` to object URLs. This will allow users
to easily subscribe to feeds of timelines, posts, and other objects.
Additionally, this proposal addresses whether syndication methods should be
applicable to mirrored profiles across the Fediverse, recommending optional but
preferred implementation.

## Syndication Methods

### Creating Feeds

To create a syndication feed, servers must append `.rss` or `.atom` to the URL
of an object. For example:

- For a user timeline: `https://example.org/@user.rss` or `https://example.org/@user.atom`
- For a post: `https://example.org/posts/12345.rss` or `https://example.org/posts/12345.atom`

### Mirrored Profiles

For mirrored profiles, such as `https://example-mirror.org/@user@example.org`,
appending `.rss` or `.atom` should be supported, but it is optional. The
recommendation is to implement this feature to maintain consistency and user
convenience across different instances.

Example URLs:

- For a mirrored user timeline:
  `https://example-mirror.org/@user@example.org.rss` or
  `https://example-mirror.org/@user@example.org.atom`

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement
Proposal have waived all copyright and related or neighboring rights to this
work.
