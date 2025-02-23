---
slug: "c16b"
authors: ilja <ilja@ilja.space>
status: DRAFT
dateReceived: 2024-08-10
discussionsTo: https://socialhub.activitypub.rocks/t/fep-c16b-formatting-mfm-functions/4448
relatedFeps: FEP-dc88
trackingIssue: https://codeberg.org/fediverse/fep/issues/383
---
# FEP-c16b: MFM機能のフォーマット

## 概要

この FEP では、カスタムクラスと [data-* 属性] を使用した HTML を使用して ActivityPub 投稿コンテンツで MFM をフォーマットする方法を推奨しています。さらに、この FEP では、この HTML 表現が使用されていることを示す新しい拡張用語を提供します。

## 根拠

この文書のキーワード「MUST」、「MUST NOT」、「REQUIRED」、「SHALL」、「SHALL NOT」、「SHOULD」、「SHOULD NOT」、「RECOMMENDED」、「MAY」、および「OPTIONAL」は、[RFC-2119](https://tools.ietf.org/html/rfc2119.html)で説明されているように解釈されます。「Fediverse 実装」または「実装」は、[ActivityPub](https://www.w3.org/TR/activitypub/)で説明されているように、ActivityPub 準拠のクライアント、ActivityPub 準拠のサーバー、または ActivityPub 準拠の連合サーバーとして解釈されます。



## 謝辞

(このセクションは非規範的です。)

このFEPの核となるアイデアは、Foundkey issue tracker [1]のJohan150によるものです。具体的には、カスタムクラスと`data-*`属性を持つ`span`要素を使ってHTMLでMFM関数を表現する提案です。

## 歴史

(このセクションは非規範的です。)

Fediverse 実装では、テキストの入力としてマークアップ言語を許可するのが一般的です。このコンテンツの統合は、通常、このテキスト入力を別の実装が簡単に理解できる適切な HTML 表現に変換することによって行われます。この HTML 表現は、[ActivityStreams](https://www.w3.org/TR/activitystreams-core) オブジェクトの`content`プロパティを使用して ActivityPub 上で統合されます。一方、ActivityPub によって追加された`source`プロパティは、オプションで元の入力と入力形式を提供するために使用できます。


Misskey は独自の [Markup language For Misskey](https://misskey-hub.net/ja/docs/for-users/features/mfm/) (MFM とも呼ばれる) を使用しています。MFM は主に HTML、Markdown、Katex、および`$[name content]`形式のカスタム MFM 関数の組み合わせで構成されています。これらの MFM 関数の意図を適切に表示するには、通常、複雑なCSSやJavascriptが必要です。そのため、`content`では簡略化された HTML表現のみが提供されます。この表現では多くの情報が削除されるため、受信側の実装では作成者が伝えようとした内容を常に適切に表示できるとは限りません。MFMを正しく表示したい受信側の実装の唯一のオプションは、`mediaType`プロパティの値が`text/x.misskeymarkdown`の場合に`source`プロパティの`content`を再解析することです。これにより、不要なオーバーヘッドが発生するだけでなく、特に 2 つの実装が異なるパーサーを使用している場合に互換性の問題も発生します。

## MFM機能

(このセクションは非規範的です。)

MFM機能は、名前、オプションで値を持つ場合と持たない場合がある一つ以上の属性、及びコンテンツで構成されます。形式は`$[name.attribute1,attribute2=value content]`です。

### 例

(このセクションは非規範的です。)

```
$[x2 Misskey expands the world of the Fediverse]
$[jelly.speed=2s Misskey expands the world of the Fediverse]
$[spin.x,speed=0.5s Misskey expands the world of the Fediverse]
```


## MFM 機能の HTML 表現


HTML で MFM 関数を表す場合、`span`要素を使用する必要があります。`span`要素には、 MFM 関数の名前であるクラスが必要です。MFM 関数に属性がある場合、要素には各属性の属性が必要です。ここでは、該当する属性の名前です。MFM 関数の属性に値がある場合、属性には同じ値が必要です。`mfm-name``name``span``data-*``data-mfm-attributename``attributename``data-*`



### 例

(このセクションは非規範的です。)

これにより、前の例は次のようになります。

```
<span class="mfm-x2">Misskey expands the world of the Fediverse</span>
<span class="mfm-jelly" data-mfm-speed="2s">Misskey expands the world of the Fediverse</span>
<span class="mfm-flip" data-mfm-x data-mfm-speed="0.5s">Misskey expands the world of the Fediverse</span>
```


## その他のMFMコンポーネント

この FEP は MFM 機能の表現に重点を置いていますが、MFM はこれらの MFM 機能だけで構成されるわけではありません。`content`プロパティ内の HTML 表現は、受信側の実装が MFM が伝える内容を正しく表示できるように、正確かつ完全である必要があります。

HTML と Markdown は、`content`プロパティ内で一般的に正しく表現されており、どちらも Fediverse で広く使用されています。したがって、MFM 関数と同じ意味では問題とは見なされません。

Katex は、一般的にプロパティで適切に表現されないという同じ問題を抱えていますcontent。 Katex 入力を HTML として適切に表現するには、[FEP-dc88](fep-dc88.md) を使用する必要があります。

## 発見

(このセクションは非規範的です。)

MFM 対応だが FEP-c16b 非準拠の実装との互換性が必要な場合、`"mediaType": "text/x.misskeymarkdown"`を使用して`source`を連合する必要がある可能性があります。一方、この実装からの受信ソースは再解析する必要がある可能性があります。そのため、 `content`を直接使用できることを FEP-c16b 準拠の実装に通知するための検出メカニズムが必要です。


この目的のために、[FEP-888d](fep-888d.md)で説明されているように、新しい拡張用語が提案されています。


### htmlMfm

`content`がFEP-c16bに準拠していることを示すために、実装では`htmlMfm`が`true`な値を持つ拡張項を使用できます。`content`がFEP-c16bに準拠していない場合、実装では`htmlMfm`値を持つ拡張項を使用してはなりませんが、`htmlMfm`値を持つ拡張項を使用できます。


* 説明: `content`がFEP-c16bに準拠していることを示すフラグ
* URI: `https://w3id.org/fep/c16b#htmlMfm`
* ドメイン: [`as:Object`](https://www.w3.org/ns/activitystreams#Object)
* 範囲: ブール値


## 例

(このセクションは非規範的です。)

```json
{
	"@context": [
		"https://www.w3.org/ns/activitystreams",
		{
			"htmlMfm": "https://w3id.org/fep/c16b#htmlMfm"
		}
	],
	"content": "<span class=\"mfm-spin\" data-mfm-x data-mfm-speed=\"0.5s\">Misskey expands the world of the Fediverse</span>",
	"source": {
		"content": "$[spin.x,speed=0.5s Misskey expands the world of the Fediverse]",
		"mediaType": "text/x.misskeymarkdown"
	},
	"htmlMfm": true
}
```


## 参考文献

- [data-* attributes]: Part of the [HTML Living Standard](https://html.spec.whatwg.org/multipage/dom.html#embedding-custom-non-visible-data-with-the-data-*-attributes)
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119), 1997
- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [1] Johan150, [Federate MFM in content field using HTML](https://akkoma.dev/FoundKeyGang/FoundKey/issues/343#issuecomment-7344), 2023
- [ActivityStreams] James M Snell, Evan Prodromou, [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core), 2017
- [Markup language For Misskey], [MFM](https://misskey-hub.net/ja/docs/for-users/features/mfm/)
- [FEP-dc88] Calvin Lee, [FEP-dc88: Formatting Mathematics](https://codeberg.org/ilja/fep/src/branch/main/fep/dc88/fep-dc88.md), 2023
- [FEP-888d] a, [FEP-888d: Using https://w3id.org/fep as a base for FEP-specific namespaces](https://codeberg.org/ilja/fep/src/branch/main/fep/888d/fep-888d.md), 2023


## 著作権

CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。