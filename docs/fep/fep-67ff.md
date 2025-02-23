---
slug: "67ff"
authors: silverpill <@silverpill@mitra.social>
status: FINAL
dateReceived: 2023-09-05
dateFinalized: 2024-09-22
trackingIssue: https://codeberg.org/fediverse/fep/issues/157
discussionsTo: https://socialhub.activitypub.rocks/t/fep-67ff-federation-md/3555
---

# FEP-67ff: FEDERATION.md
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-67ff.md)から翻訳に協力することができます。
## Summary

`FEDERATION.md` is a file containing information necessary for achieving interoperability with a federated service. It was originally proposed by Darius Kazemi on SocialHub forum in [Documenting federation behavior in a semi-standard way?](https://socialhub.activitypub.rocks/t/documenting-federation-behavior-in-a-semi-standard-way/453) topic.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119](https://tools.ietf.org/html/rfc2119.html).

## Structure

The `FEDERATION.md` file can have arbitrary structure and content. The only requirements are:

- It MUST be a valid Markdown document.
- It MUST be located in the root of a project's code repository. If project's documentation is located in another place, the `FEDERATION.md` file may contain a link to that location.
- It SHOULD include a list of implemented federation protocols.
- It SHOULD include a list of supported Fediverse Enhancement Proposals (FEPs).

## Template

(このセクションは非規範的です。)

```markdown
# Federation

## Supported federation protocols and standards

- [ActivityPub](https://www.w3.org/TR/activitypub/) (Server-to-Server)
- [WebFinger](https://webfinger.net/)
- [Http Signatures](https://datatracker.ietf.org/doc/html/draft-cavage-http-signatures)
- [NodeInfo](https://nodeinfo.diaspora.software/)

## Supported FEPs

- [FEP-f1d5: NodeInfo in Fediverse Software](https://codeberg.org/fediverse/fep/src/branch/main/fep/f1d5/fep-f1d5.md)

## ActivityPub

<!-- Describe activities and extensions. -->

## Additional documentation

<!-- Add links to documentation pages. -->
```

## Implementations

- [gathio](https://github.com/lowercasename/gathio/blob/main/FEDERATION.md)
- [Streams](https://codeberg.org/streams/streams/src/branch/dev/FEDERATION.md)
- [Smithereen](https://github.com/grishka/Smithereen/blob/master/FEDERATION.md)
- [Mastodon](https://github.com/mastodon/mastodon/blob/main/FEDERATION.md)
- [Hometown](https://github.com/hometown-fork/hometown/blob/hometown-dev/FEDERATION.md)
- [Mitra](https://codeberg.org/silverpill/mitra/src/branch/main/FEDERATION.md)
- [Emissary](https://github.com/EmissarySocial/emissary/blob/main/FEDERATION.md)
- [Vervis](https://codeberg.org/ForgeFed/Vervis/src/branch/main/FEDERATION.md)
- [WordPress](https://github.com/Automattic/wordpress-activitypub/blob/master/FEDERATION.md)
- [Postmarks](https://github.com/ckolderup/postmarks/blob/main/FEDERATION.md)
- [Bovine](https://bovine-herd.readthedocs.io/en/latest/FEDERATION/) in [repo](https://codeberg.org/bovine/bovine/src/branch/main/bovine_herd/docs/docs/FEDERATION.md) and the [symlink](https://codeberg.org/bovine/bovine/src/branch/main/FEDERATION.md)
- [BookWyrm](https://github.com/bookwyrm-social/bookwyrm/blob/main/FEDERATION.md)
- [Hatsu](https://github.com/importantimport/hatsu/blob/main/FEDERATION.md)
- [tootik](https://github.com/dimkr/tootik/blob/main/FEDERATION.md)
- [Bridgy Fed](https://github.com/snarfed/bridgy-fed/blob/main/FEDERATION.md)
- [Friendica](https://git.friendi.ca/friendica/friendica/src/branch/develop/FEDERATION.md)
- [PieFed](https://codeberg.org/rimu/pyfedi/src/branch/main/FEDERATION.md)
- [Akkoma](https://akkoma.dev/AkkomaGang/akkoma/src/branch/stable/FEDERATION.md)
- [Iceshrimp.NET](https://iceshrimp.dev/iceshrimp/Iceshrimp.NET/src/branch/dev/FEDERATION.md)
- [Forte](https://codeberg.org/fortified/forte/src/branch/dev/FEDERATION.md)
- [NeoDB](https://github.com/neodb-social/neodb/blob/main/FEDERATION.md)
- [FIRM](https://github.com/steve-bate/firm/blob/main/FEDERATION.md)

## References

- Darius Kazemi, [Documenting federation behavior in a semi-standard way?][Documenting federation behavior in a semi-standard way?], 2020
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119], 1997

[Documenting federation behavior in a semi-standard way?]: https://socialhub.activitypub.rocks/t/documenting-federation-behavior-in-a-semi-standard-way/453
[RFC-2119]: https://tools.ietf.org/html/rfc2119.html

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
