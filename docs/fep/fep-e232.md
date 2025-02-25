
# FEP-e232: オブジェクトリンク

|                    |                                                            |
|--------------------|------------------------------------------------------------|
| authors            | silverpill <@silverpill@mitra.social>                      |
| status             | FINAL                                                      |
| dateReceived       | 2022-08-01                                                 |
| dateFinalized      | 2023-12-03                                                 |
| trackingIssue      | [https://codeberg.org/fediverse/fep/issues/14](https://codeberg.org/fediverse/fep/issues/14) |
| discussionsTo      | [https://socialhub.activitypub.rocks/t/fep-e232-object-links/2722](https://socialhub.activitypub.rocks/t/fep-e232-object-links/2722) | 
| original           | [https://codeberg.org/fediverse/fep/src/branch/main/fep/e232/fep-e232.md](https://codeberg.org/fediverse/fep/src/branch/main/fep/e232/fep-e232.md) |

## 概要

この文書は、[ActivityPub][ActivityPub]オブジェクトへのテキストベースのリンクを表現する方法を提案します。これはメンションに類似しています。例えば、`content`プロパティの値内のインライン引用などがありますが、この提案は特定のユースケースに限定されるものではありません。

## 要件

この文書における「MUST」、「MUST NOT」、「REQUIRED」、「SHALL」、「SHALL NOT」、「SHOULD」、「SHOULD NOT」、「RECOMMENDED」、「MAY」、「OPTIONAL」というキーワードは、[RFC-2119][RFC-2119]に記載された通りに解釈されます。

## オブジェクトリンク

ソフトウェアは、`@mention`や`#hashtag`のマイクロシンタックスに類似した方法で、ユーザーがオブジェクトリンクを定義できるようにすることが期待されます。オブジェクトリンクの定義方法はユースケースによって異なる可能性があり、この文書の範囲外です。

オブジェクトの`name`、`summary`、または`content`に他のオブジェクトへのリンクが含まれている場合、そのオブジェクトは`tag`プロパティを持つべきであり、各オブジェクトリンクは[Activity Vocabulary][ActivityVocabulary]に基づいて`Link`オブジェクトとして表現されます。この`Link`オブジェクトのプロパティは以下の通りです：

- `type` (必須): タイプは`Link`またはそのサブタイプでなければならない。
- `mediaType` (必須): メディアタイプは`application/ld+json; profile="https://www.w3.org/ns/activitystreams"`でなければならない。この仕様はActivityPubオブジェクトのみを扱いますが、実際にはメディアタイプが異なる場合があり、サーバーはこの要件に準拠しないオブジェクトリンクを受け入れる可能性があります。例えば、`application/activity+json`のメディアタイプは同等と見なされるべきです。
- `href` (必須): `href`プロパティには参照されるオブジェクトのURIが含まれていなければならない。
- `name` (任意): `name`はオブジェクトのコンテンツで使用されるマイクロシンタックスと一致するべきである。
- `rel` (任意): 関連する場合、`rel`はリンクが現在のリソースにどのように関連しているかを指定すべきである。`rel`を使用することで、特定の用途を示すことにより、オブジェクトリンクに追加の目的を提供することができる。

## 例

(このセクションは非規範的です。)

バグトラッカーの問題へのリンク：

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "type": "Note",
    "content": "バグは #1374 で報告されました",
    "tag": [
        {
            "type": "Link",
            "mediaType": "application/ld+json; profile=\"https://www.w3.org/ns/activitystreams\"",
            "href": "https://forge.example/tickets/1374",
            "name": "#1374"
        }
    ]
}
```

インライン引用：

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "type": "Note",
    "content": "これは引用です:<br>RE: https://server.example/objects/123",
    "tag": [
        {
            "type": "Link",
            "mediaType": "application/ld+json; profile=\"https://www.w3.org/ns/activitystreams\"",
            "href": "https://server.example/objects/123",
            "name": "RE: https://server.example/objects/123"
        }
    ]
}
```

`content`には`RE: <url>`のマイクロシンタックスが含まれていますが、消費する実装は適切な関連付けを行うためにそれを解析する必要はありません。

## 実装

- (streams)
- FoundKey
- Mitra
- Pleroma ([via MRF](https://git.pleroma.social/pleroma/pleroma/-/blob/v2.6.0/lib/pleroma/web/activity_pub/mrf/quote_to_link_tag_policy.ex))
- Threads ([発表](https://engineering.fb.com/2024/03/21/networking-traffic/threads-has-entered-the-fediverse/))
- [Friendica](https://github.com/friendica/friendica/pull/14032)
- Bridgy Fed
- [Hollo](https://hollo.social/@hollo/01920132-739e-7eff-9f5f-424282884eee)
- [Iceshrimp.NET](https://iceshrimp.dev/iceshrimp/Iceshrimp.NET/src/commit/bdfd3a8d4e788ef3bdec06f32f444ed7fcffc3c7/FEDERATION.md#supported-feps)

## 参考文献

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- S. Bradner, [RFCでの要件レベルを示すためのキーワード][RFC-2119], 1997
- James M Snell, Evan Prodromou, [Activity Vocabulary][ActivityVocabulary], 2017

[ActivityPub]: https://www.w3.org/TR/activitypub/
[RFC-2119]: https://tools.ietf.org/html/rfc2119.html
[ActivityVocabulary]: https://www.w3.org/TR/activitystreams-vocabulary/ 

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
