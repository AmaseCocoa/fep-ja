---
slug: "c118"
authors: Tim Bray <tbray@texgtuality.com>
status: DRAFT
dateReceived: 2023-01-16
trackingIssue: https://codeberg.org/fediverse/fep/issues/57
discussionsTo: https://codeberg.org/fediverse/fep/issues/57
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-c118.md)から翻訳に協力することができます。
# FEP-c118: Content licensing support

## Summary

Currently, popular Fediverse software does very little to establish the legal status of posts.
Controversy over indexing and scraping the Fediverse is common. 
The hope is that providing a legal framework to express the desires of users as to how their content may be
re-used might bring order to this debate.

## Scenario

Dramatis Personae:
* A somewhat-technical new Fediverse arrival, hereinafter *Noob*
* Existing Fediverse Voices, hereinafer *EFV1*, *EFV2*, etc

*Noob*: Hey, I want to crawl the Fediverse and build an index so we can…
*EFV1*: Stop! Go back! Don't!

*EFV2*: We prefer not to do that here because we want to avoid into big-tech surveillance capitalism.

*Noob*: But I mean they're all just URLs, right?

*EFV3*: Burn the witch!

*EFV4*: Unclean!

*Noob*: Oh, I guess you don't want me to do this?

*EFV5*: Never!

*Noob*: Shouldn't you post the rules somewhere? I mean, lots of people are already doing this.

*EFV6*: Defederate his instance!

## Discussion

This scenario is happening really a lot, multiple times per week in December/January 2022-3.

At the moment, the Fediverse privacy story is unsatisfactory. Unless a user really understands the
visibility levels, every post they make instantly becomes part of the permanent indelible Internet
memory. There are no technical and very few legal barriers to prevent anyone from doing anything with
retrieved post content.

In developed societies, when there are behaviors that are logistically possible but society would like
to regulate, legal tools are frequently applied. Examples would be dangerous driving and public smoking.

A content-licensing framework for the Fediverse could include some or all of the following:

1. Prevent access to posts without the accessor having somehow acknowledged the content license that applies.
2. Build a menu of content-license choices, probably starting from a Creative Commons basis.
3. One parameter of the content licenses should be temporal; i.e., license access to posts but only for two weeks (or some other interval).
4. Make it easy and straightforward for individuals posting to the Fediverse to pick a default content license for their posts, and also to apply other licensing choices to individual posts.
5. Arrange that when one user follows another, the following user must acknowledge the default content licensing options of the followed user.
6. Since most users will just take the defaults, each instance should carefully choose and prominently display its content-licensing defaults.

I note that this document template comes with a carefully-considered Creative Commons license. Do not the contributions
of the humans who are enriching the Fediverse deserve a comparable level of protection?

## Caveat

Note that establishing a legal content-licensing framework will not prevent
certain bad actors from scraping the Fediverse and mis-using the harvested data.
That's OK because it will establish a legal framework that will present a significant barrier
to *commercial* data harvesters, and provide a tool to combat certain classes of mis-use.

## See also:

- Eugen Rochko, [Cage the Mastodon](https://blog.joinmastodon.org/2018/07/cage-the-mastodon/) in particular "Design Decisions"
- @pamela@bsd.network, [Hacky folks, please resist finding ways to scrape the fediverse….](https://bsd.network/@pamela/109287805657081451)
- GitHub discussion, [Controlling availability to search](https://github.com/w3c/activitypub/issues/221)
- Tim Bray [Private and Public Mastodon](https://www.tbray.org/ongoing/When/202x/2022/12/30/Mastodon-Privacy-and-Search)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication 

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
