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

鍵識別子とアクターの識別子は同じ[origin](fep-fe34.md)であるべきです (SHOULD)

Multikeyオブジェクトにはキーの有効期限を示す`expires`プロパティが含まれる場合があります。実装では、有効期限が切れた鍵で署名された署名を受け入れてはいけません (MUST NOT)。
### キーID
識別子は、アクターIDにフラグメント識別子を追加することによって生成される必要があります。同じアクターの異なる公開鍵は、異なるフラグメントIDを利用して識別される必要があります。

フラグメント識別子を含む URI の解決は、[Controlled Identifiers](https://w3c.github.io/cid/#fragment-resolution)仕様のセクション3.4 フラグメント解決仕様で指定されたアルゴリズムを利用して実行されます。
### 鍵の種類
実装者は、[Multicodec](https://github.com/multiformats/multicodec/)プレフィックスが登録されている任意のタイプの暗号化キーを使用できます。
## 制御された識別子文書
`Multikey`オブジェクトは、[Controlled Identifiers](https://w3c.github.io/cid/#Multikey)仕様で説明されているように、制御された識別子ドキュメントと見なされるアクター オブジェクトに追加される必要があります。

鍵がActivityPubオブジェクトの署名に利用されることを意図している場合は、アクターオブジェクトの[`assertionMethod`](https://w3c.github.io/cid/#assertion)配列に追加する必要があります。

その他のユースケースは現在この提案の範囲外です。

実装では、この仕様に準拠していないオブジェクトを`assertionMethod`配列に追加することは推奨されません。配列内で非準拠の`assertionMethod`エントリに遭遇した実装は、それらを無視する必要があります。

### 例
```json
{
    "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://www.w3.org/ns/cid/v1"
    ],
    "type": "Person",
    "id": "https://server.example/users/alice",
    "inbox": "https://server.example/users/alice/inbox",
    "outbox": "https://server.example/users/alice/outbox",
    "assertionMethod": [
        {
            "id": "https://server.example/users/alice#ed25519-key",
            "type": "Multikey",
            "controller": "https://server.example/users/alice",
            "publicKeyMultibase": "z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2"
        }
    ]
}
```
## この提案とFEP-c390の違い
[FEP-c390](fep-c390.md)は、外部 ID を ActivityPub アクターにリンクする方法を説明しています。有効な ID 証明は、アクターと証明のサブジェクトが同じエンティティによって制御されていることを意味します。

この提案では、アクターの公開鍵を表現する方法について説明します。対応する秘密鍵はサーバーによって制御されます。
## テストベクトル
[fep-521a.feature](https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.feature)を参照
## 実装
- Mitra
- streams
- Hubzilla
- Fedify
## 参考文献
- Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- Ivan Herman, Manu Sporny, Dave Longley, [Security Vocabulary](https://w3c.github.io/vc-data-integrity/vocab/security/vocabulary.html), 2023
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119.html), 1997
- Dave Longley, Manu Sporny, Markus Sabadello, Drummond Reed, Orie Steele, Christopher Allen, - [Controlled Identifiers (CIDs) v1.0](https://w3c.github.io/cid/), 2025
- Protocol Labs, [Multicodec](https://github.com/multiformats/multicodec/)
- silverpill, [FEP-fe34: オリジンベースのセキュリティモデル](fep-fe34.md), 2024
## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。