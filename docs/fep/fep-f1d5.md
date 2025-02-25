---
slug: "f1d5"
authors: CJ <cjslep@gmail.com>, silverpill <@silverpill@mitra.social>
status: FINAL
dateReceived: 2020-12-13
dateFinalized: 2023-06-02
trackingIssue: https://codeberg.org/fediverse/fep/issues/50
discussionsTo: https://codeberg.org/fediverse/fep/issues/50
---
# FEP-f1d5: FediverseソフトウェアのNodeinfo

## 概要

NodeInfo は、サーバーレベルのメタデータを一般に公開する方法を標準化することを目的としたプロトコルです。これにより、ツールやクライアントはこのメタデータを利用してサーバーの健全性を評価したり、Fediverse で使用するサーバーやソフトウェアに関するエンドユーザーの選択を容易にしたりできるようになります。

## 歴史

NodeInfoは、diaspora、friendica、redmatrixソフトウェアでの使用を目的とした[ActivityPub]プロトコルの前に開発されました。NodeInfoがカプセル化している元のプロトコルには、diaspora、pumpio、gnusocialが含まれます。 

NodeInfoの仕様は、そのスキーマにおいて非常に厳格であり、しばしば正規表現による検証や、列挙された可能な値の閉じたセットを要求します。この点に対する反発として、NodeInfo2フォークが作成され、一部のフィールドの検証を削除し、メタデータの論理的再構成が行われました。NodeInfoとNodeInfo2を基にして、[ServiceInfo]が一時的に検討されました。 

このFEPは、特定のプロトコルの詳細を文書化することを目的としていません。そのための情報は、[NodeInfo][NodeInfoRepository]および[NodeInfo2][NodeInfo2Repository]を参照してください。このFEPは、歴史を明らかにし、現在のアプローチの欠点を特定することで、Fediverseソフトウェアの開発者に文脈を提供することを試みています。 

## Requirements

このドキュメント内のキーワード「MUST」、「MUST NOT」、「REQUIRED」、「SHALL」、「SHALL NOT」、「SHOULD」、「SHOULD NOT」、「RECOMMENDED」、「MAY」、および「OPTIONAL」は、[RFC-2119][RFC-2119]で説明されているとおりに解釈されます。

Fediverseソフトウェアは[NodeInfo][NodeInfoRepository]を実装する必要があります。

## 注意事項

本FEPの執筆時点において、コミュニティによって特定されたNodeInfoの現状に対する主な異議は以下の通りである。なお、示される技術的代替案は例示的であり、規範的なものではないことに留意されたい。

* `software.name` の正規表現は不必要に厳格である。例えば、大文字、スペース、非英語アルファベット、ハイフン以外の特殊文字は許可されていない。
* `software.version` フィールドは必須であり、これも不必要に厳格である。ソフトウェアにバージョン情報を強制的に開示させることは、潜在的なセキュリティ問題となる可能性がある。
* `inbound` および `outbound` 要素は、単純な文字列ではなく、列挙型の閉じたセットとして指定されている。プロトコルバージョン管理は名前変更として現れ、新しい列挙型を追加しなければならず、その結果、バージョン管理が不明瞭になる。
* Fediverseソフトウェアは、必須であるため、`openRegistrations`の概念を持たなければならない。
* HTTP署名、WebFinger、OAuthなどの他の機能を特定し、バージョン管理するための拡張可能な手法が欠如している。仕様は非常に厳格である一方で、`metadata`はあまりにも緩すぎる。
* `usage.users`は非正規化されておらず、実装がソフトウェアに合った `(活動カウント, 日数)` のカスタムペアを提供できない。
* `usage.users`は、ユーザーのアイデンティティが特定のソフトウェアのインスタンスに結びついていると仮定している。ユーザーのアイデンティティが複数のサーバー、複数のグループ、または複数のユーザーコレクションに分散している場合、`total`ユーザーをどのようにカウントするかは不明である。複数のソフトウェアインスタンスは、それぞれがユーザーを「使用している」と主張できるため、グローバルにユーザーが重複してカウントされる結果となる可能性がある。
* `usage.users`の活動カウントも同様に、ユーザーのアイデンティティが特定のソフトウェアインスタンスに結びついていると仮定している。上記の理由により、`total`ユーザーのカウントは、すべてのソフトウェアで同じユーザーが重複してカウントされる結果をもたらす可能性がある。また、活動カウントの`activeHalfYear`および`activeMonth`も、グローバルに水増しされたカウントを引き起こす可能性がある。
* `activeHalfYear`および`activeMonth`は、それぞれ180日および30日の期間を表現するには不適切な名称である。「半年」は180日であることは0%の時間であり、約182.5日であることは75%の時間である。1か月は30日であることは33%の時間である。
* `localPosts`および`localComments`は、例えば音声ファイルや動画をホストするソフトウェア、コメントや投稿がないソフトウェアに対して `(種類, カウント)` のペアに非正規化されていない。
* `localPosts`および`localComments`は必須であり、コメントや投稿がないソフトウェアにとっては問題である。 

## 実装

### サーバー

このリストは包括的ではありません：

* Mastodon
* Matrix
* Pleroma
* PeerTube
* WriteFreely
* Friendica
* Diaspora
* PixelFed
* Misskey
* Funkwhale
* Smithereen
* Plume
* GNU Social
* lemmy
* zap
* Socialhome
* epicyon
* apcore
* FIRM

### クライアント

* [The-Federation.Info](https://the-federation.info/)
* [Hello Matrix Public Servers](https://www.hello-matrix.net/public_servers.php)

## 参考文献

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- Jonne Haß, [jhass/nodeinfo][NodeInfoRepository], 2014
- Jason Robinson, [jaywink/nodeinfo2][NodeInfo2Repository], 2016
- Jason Robinson, [ServiceInfo - specification for service metadata][ServiceInfo], 2019
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119], 1997

[ActivityPub]: https://www.w3.org/TR/activitypub/
[NodeInfoRepository]: https://github.com/jhass/nodeinfo
[NodeInfo2Repository]: https://github.com/jaywink/nodeinfo2
[ServiceInfo]: https://web.archive.org/web/20220201002230/https://talk.feneas.org/t/serviceinfo-specification-for-service-metadata/99
[RFC-2119]: https://tools.ietf.org/html/rfc2119.html

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
