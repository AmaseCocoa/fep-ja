---
slug: "73cd"
authors: Bumblefudge <@learningproof.xyz> / <@by_caballero@mastodon.social>
status: DRAFT
relatedFeps: FEP-8b32, FEP-7628, FEP-c390, FEP-ae97, FEP-ef61, FEP-521a
dateReceived: 2024-02-07
trackingIssue: https://codeberg.org/fediverse/fep/issues/265
discussionsTo: https://codeberg.org/fediverse/fep/issues/265
---
# FEP-73cd: Migration User Stories
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-73cd.md)から翻訳に協力することができます。
## Summary

In the interest of clarifying and aligning on the problem-space of user account migration, multiple-account management, and export/import/migration of content/activity history, these user stories are offered to organize discussion and solution-sharing.

## Contributing

Any one of these user stories could be further elaborated in the spirit of the [SWAT exercise](https://www.w3.org/2005/Incubator/federatedsocialweb/wiki/SWAT0) that served as ActivityPub's initial "definition of done", perhaps in a separate FEP and/or SocialHub thread.

## Migration User Stories

PRs welcome! Add or update!

1, 2, and 3 describe follower/profile migration, while 4,5,6 and 7 describe migration of posted activities.

1. Alice wants to move her account from Alpha to Gamma, both of which are online and federated to one another, without losing her follow relationships in either direction. Four possible variants, not mutually exclusive:
    * A. Alice would like her account on Alpha terminated with some kind of human-readable redirect, i.e., links to old Alice@Alpha content display a warning that “Alice doesn't live here anymore”.
    * B. Alice would like Alpha to dynamically redirect any links to Alice@Alpha or to any specific content she posted/generated there to a reasonable default “homepage” for Alice@Gamma.
    * C. Alice would like Alpha to dynamically redirect any links to Alice@Alpha content to the migrated contents @Gamma (i.e. 301 HTTP codes and nginx-style URL rewrites).
    * D. Alice would like her account on Alpha to remain active and accept new posts as a personal account, but would like her followers to know about the new professional account as well. A selection of professional posts from Alpha will be ported over so that her new professional account can carry over a little backhistory (with a disclaimer that they were originally uploaded to Alpha).
    * E. In addition to her followers and followees, Alice would like to bring her "unfollows" from Alpha to Gamma, i.e. the specific accounts she has manually blocked or muted.
    * F. In addition to her followers, followees, and "unfollows," Alice would like to bring her Alpha's carefully-curated "defederation list," i.e. its server-level blocklist, from Alpha to Gamma, since Gamma is a younger and less robustly administered community than Alpha.
2. Bob is asked to leave Alpha by its moderation team, who have disabled new posts on that account but are allowing Bob to execute a one-way, permanent migration to a new server of Bob's choosing as a courtesy. Bob logs in to Alpha and permanently migrates to a new server with which Alpha is federated, allowing a server-to-server connection. Bob instead finds a new home on the server Beta, which is specifically de-federated by Alpha for incompatible moderation policies. Bob would like to announce to his followers his new account, without Alpha and Beta having to communicate with one another (since Alpha refuses connections with Beta altogether).
3. Charlie would like to move his profile, followers, and his years of original content to Gamma from Delta, because the latter was recently and unexpectedly taken offline by government intervention. Before going dark, Gamma had already authorized a custom client for Charlie, which he used to sign each posts with a self-managed private key, and Charlie had backed up his followers/following lists. Delta is able to load all of the above and authenticate them to display on his new account.
4. Bob would also like to move his previous content from Alpha to Beta, which are defederated. After having his service partially limited, he is able to export his archive of Alpha content (including media uploads but no moderation records, as Alpha does not generate any) and import it to Beta to be considered for republication.
5. Charlie also managed to expert an archive of his Gamma content after authorizing Charlie's custom client (including media uploads and moderation records), and is able to use said client to authenticate this content to Delta, who can consider it for republication despite Gamma being offline.
6. After using Delta for some time, Charlie moves on to server Epsilon, which Daniel wants to migrate to from Delta. Epsilon has no moderation policy because Daniel is its only user and he has full admin rights over it. Epsilon loads a recent backup, skipping over the moderation policies exported by Delta. 
7. At time of import, Delta does not support all the same features and Activity types that Gamma did at time of export. Two variants, not mutually exclusive:
    * A. Delta's import wizard warns Charlie to keep his backup and try again later. Years later, he does, and additional content is imported now that Delta supports a bigger subset of Gamma's Activity types, without duplicating the content previously imported.
    * B. Delta stored all the un-imported Gamma Activities in a separate archive for Charlie. Years later, when Charlie exports this historic data along with Delta data to Epsilon, both the un-imported Gamma content, the imported Gamma content, and the newer Delta content alike get included in his new backup, and it all gets imported to Epsilon.

## Use-Case to FEP Mapping

Note: this section is largely subjective and intended as an aid to decision-making, NOT as any kind of definitive statement about the utility or necessity of other FEPs.
Feel free to PR in links to blog posts or other tutorials that explain a given implementation of any user stories to the "profile" row in the form `[1](<link>), [2](<link>)...`

* Key:
  * [R]equired
  * [H]elpful
  * [U]nknown
  * [1/2/3] Optionality sets

|Use cases:|1A|1B|1C|1D|2|3|4|5|6|7A|7B|
|---|---|---|---|---|---|---|---|---|---|---|---|
|[FEP-7628 "Move Actor"](https://codeberg.org/fediverse/fep/src/branch/main/fep/7628/fep-7628.md) (`push` mode)|R1|||R1|R|||||||
|[FEP-7628 "Move Actor"](https://codeberg.org/fediverse/fep/src/branch/main/fep/7628/fep-7628.md) (`pull` mode)|R2|||||R||||||
|[FEP-c390 "Identity Proofs"](https://codeberg.org/fediverse/fep/src/branch/main/fep/c390/fep-c390.md)|||||H|H|R|||||
|[FEP-ae97 "Client-Side Activity Signing"](https://codeberg.org/fediverse/fep/src/branch/main/fep/ae97/fep-ae97.md)|||||||R|||||
|[FEP-ef61 "Portable Objects"](https://codeberg.org/fediverse/fep/src/branch/main/fep/ef61/fep-ef61.md)||||R|||R|H|H|||
|[FEP-8b32 "Object Integrity Proofs"](https://codeberg.org/fediverse/fep/src/branch/main/fep/8b32/fep-8b32.md) (req'd by ^)||||R^|||R^|H^|H^|||
|[FEP-521a "Actor's Public Keys"](https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md)||||H|||H|H|H|||
|FEP-TBD [Per-user? Per-server? both?] "Moderation Records"||||||||R|H|||
|FEP-TBD "Forwarding and Redirecting Migrated Actors"||H|R|R?|H|H|R|||||
|FEP-TBD "Activity Archives/Export Format"||||||||||R|R|
|Profiles needed to combine the above?||||||||||||

## Open Questions

* [arbitrary Actor Metadata k/v pairs as per FEP-fb2a](https://codeberg.org/fediverse/fep/src/branch/main/fep/fb2a/fep-fb2a.md) might be useful as a legacy/fallback for some of the above?
* likewise the [custom TXT record that FEP-612d proposes for mapping domains to actors](https://codeberg.org/fediverse/fep/src/branch/main/fep/612d/fep-612d.md) might be a useful way of doing "migration from dead server" or some such?
* exporting activity-histories should probably also export histories of each activity in that activity history, as per [FEP-bad1, Object History collection](https://codeberg.org/fediverse/fep/src/branch/main/fep/bad1/fep-bad1.md)
  * do FEP-bad1 histories span multiple services?
    * if so, does "this content was originally uploaded to another server" belong in a new FEP or what?
* migrating accounts should probably migrate [PENDING follows as per FEP-4ccd](https://codeberg.org/fediverse/fep/src/branch/main/fep/4ccd/fep-4ccd.md), and/or do a full [follower sync as per FEP-8fcf](https://codeberg.org/fediverse/fep/src/branch/main/fep/8fcf/fep-8fcf.md)

## References

* [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
* [ABC] Alyssa P. Hacker, [An example proposal](http://abc.example/abc.html), 2020

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
