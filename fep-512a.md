---
slug: "521a"
authors: silverpill <@silverpill@mitra.social>
type: implementation
status: DRAFT
dateReceived: 2023-07-08
trackingIssue: https://codeberg.org/fediverse/fep/issues/130
discussionsTo: https://socialhub.activitypub.rocks/t/fep-521a-representing-actors-public-keys/3380
---
# FEP-521a: Actorの公開鍵の表現
## 概要
この提案では、 [ActivityPub](https://www.w3.org/TR/activitypub/)アクターに関連付けられた公開鍵を表現する方法について説明します。
## 根拠
これまで、Fediverseサービスは[publicKey](https://w3c-ccg.github.io/security-vocab/#publicKey)プロパティを利用してアクターの公開鍵を表現していました。多くの実装は1アクターにつき一つの鍵のみを許可するため、追加の鍵が必要なケースをサポートするには新しいアプローチが必要でした。

さらに、`publicKey`プロパティは最新の[Security Vocabulary](https://w3c.github.io/vc-data-integrity/vocab/security/vocabulary.html)からは削除されました。
## 要件
このドキュメント内のキーワード「MUST」、「MUST NOT」、「REQUIRED」、「SHALL」、「SHALL NOT」、「SHOULD」、「SHOULD NOT」、「RECOMMENDED」、「MAY」、および「OPTIONAL」は、[RFC-2119](https://tools.ietf.org/html/rfc2119.html)で説明されているとおりに解釈されます。
## Multikey
各公開鍵は、[Controlled Identifiers](https://w3c.github.io/cid/#Multikey)仕様のセクション2.2.2 Multikeyで定義されているように、Multikey型を持つオブジェクトとして表現されなければなりません (MUST)。このオブジェクトには、次のプロパティが必要です。

- `id`: 公開鍵の一意のグローバル識別子。
- `type`: このプロパティの値は`Multikey`文字列でなければなりません。
- `controller`: このプロパティの値はアクターのIDと一致する必要があります。
- `publicKeyMultibase`: [Multibase](https://w3c.github.io/cid/#multibase-0)でエンコードされた[Multicodec](https://github.com/multiformats/multicodec/)の接頭辞と鍵。実装では`base-58-btc`アルファベットを利用する必要があります。