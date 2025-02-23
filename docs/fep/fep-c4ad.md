---
slug: "c4ad"
authors: AvidSeeker <avidseeker7@protonmail.com>
status: DRAFT
dateReceived: 2024-07-15
discussionsTo: https://socialhub.activitypub.rocks/t/4361
title: Viewership History
date: 2024-07-15
trackingIssue: https://codeberg.org/fediverse/fep/issues/367
---
# FEP-c4ad: Viewership History
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-c4ad.md)から翻訳に協力することができます。

## Summary

This document proposes a standard for managing viewership history across the
Fediverse. It addresses the common issue of posts being repeatedly shown to
users on different clients. The goal is to enable servers to track which posts
have been viewed by individual users and ensure that clients do not display
these posts again. This proposal aims to enhance user experience by preventing
the redundant display of already seen posts, commonly requested as "Hide
already seen posts" or "stop repeating already seen posts".

## Creating a Viewership Record

When a user views a post, the server should create a record of this event. This
record will include the following properties:

- `id` (REQUIRED): the unique identifier of the viewership record.
- `type` (REQUIRED): the type of the object MUST be `ViewershipRecord`.
- `actor` (REQUIRED): the actor who viewed the post.
- `object` (REQUIRED): the unique identifier of the post that was viewed.
- `published` (RECOMMENDED): the date and time at which the post was viewed.

## Querying Viewership Records

Clients should query the server for viewership records to determine whether a
post has been viewed by the user. The server should provide an endpoint for
clients to fetch viewership records for a specific user.

## Handling Viewed Posts

When fetching posts, clients must filter out posts that have been viewed by the
user based on the viewership records. This ensures that users are not shown the
same posts repeatedly.

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement
Proposal have waived all copyright and related or neighboring rights to this
work.
