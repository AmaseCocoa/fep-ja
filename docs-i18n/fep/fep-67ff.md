# FEP-67ff: FEDERATION.md
|               |                                                                  |
|---------------------|--------------------------------------------------------------------|
| slug                | "67ff"                                                             |
| authors             | silverpill <@silverpill@mitra.social>                             |
| status              | FINAL                                                              |
| dateReceived        | 2023-09-05                                                         |
| dateFinalized       | 2024-09-22                                                         |
| trackingIssue       | [Tracking Issue](https://codeberg.org/fediverse/fep/issues/157)    |
| discussionsTo       | [Discussions To](https://socialhub.activitypub.rocks/t/fep-67ff-federation-md/3555) |
| original           | [https://codeberg.org/fediverse/fep/src/branch/main/fep/67ff/fep-67ff.md](https://codeberg.org/fediverse/fep/src/branch/main/fep/67ff/fep-67ff.md) |

## 概要

`FEDERATION.md` は、連携サービスとの相互運用性を達成するために必要な情報を含むファイルです。この提案は、Darius Kazemi によって SocialHub フォーラムの [ドキュメント化された連携の挙動についての半標準的な方法？](https://socialhub.activitypub.rocks/t/documenting-federation-behavior-in-a-semi-standard-way/453) というトピックで最初に提案されました。

## 要件

この文書におけるキーワード「MUST」「MUST NOT」「REQUIRED」「SHALL」「SHALL NOT」「SHOULD」「SHOULD NOT」「RECOMMENDED」「MAY」「OPTIONAL」は、[RFC-2119](https://tools.ietf.org/html/rfc2119.html) に記載された通りに解釈されます。

## 構造

`FEDERATION.md` ファイルは任意の構造と内容を持つことができます。唯一の要件は以下の通りです：

- 有効な Markdown ドキュメントでなければなりません。
- プロジェクトのコードリポジトリのルートに存在しなければなりません。プロジェクトのドキュメントが他の場所にある場合、`FEDERATION.md` ファイルはその場所へのリンクを含めることができます。
- 実装された連携プロトコルのリストを含むべきです。
- サポートされている Fediverse Enhancement Proposals (FEPs) のリストを含むべきです。

## テンプレート

(このセクションは非規範的です。)

```markdown
# Federation

## サポートされている連携プロトコルと標準

- [ActivityPub](https://www.w3.org/TR/activitypub/) (サーバー間)
- [WebFinger](https://webfinger.net/)
- [Http Signatures](https://datatracker.ietf.org/doc/html/draft-cavage-http-signatures)
- [NodeInfo](https://nodeinfo.diaspora.software/)

## サポートされているFEPs

- [FEP-f1d5: FediverseソフトウェアにおけるNodeInfo](https://codeberg.org/fediverse/fep/src/branch/main/fep/f1d5/fep-f1d5.md)

## ActivityPub

<!-- アクティビティと拡張について説明します。 -->

## 追加のドキュメント

<!-- ドキュメントページへのリンクを追加します。 -->
```

## 実装

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

## 参考文献

- Darius Kazemi, [ドキュメント化された連携の挙動についての半標準的な方法？](https://socialhub.activitypub.rocks/t/documenting-federation-behavior-in-a-semi-standard-way/453), 2020
- S. Bradner, [RFCにおける要件レベルを示すためのキーワード](https://tools.ietf.org/html/rfc2119.html), 1997

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
