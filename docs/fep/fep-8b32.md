---
slug: "8b32"
authors: silverpill <@silverpill@mitra.social>
status: DRAFT
dateReceived: 2022-11-12
trackingIssue: https://codeberg.org/fediverse/fep/issues/29
discussionsTo: https://socialhub.activitypub.rocks/t/fep-8b32-object-integrity-proofs/2725
---
!!! Warning
    このFEPはまだ翻訳が完了していません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-8b32.md)から翻訳に協力することができます。
# FEP-8b32: オブジェクト整合性証明

## 概要
この提案では、[ActivityPub](https://www.w3.org/TR/activitypub/)サーバーとクライアントが自己認証アクティビティとオブジェクトを作成する方法について説明します。

HTTP 署名は、サーバー間のやり取り中の認証によく使用されます。ただし、これにより認証がアクティビティの配信に結び付けられ、プロトコルの柔軟性が制限されます。

整合性証明は、デジタル署名とそれを検証するために必要なパラメータを表す属性のセットです。これらの証明は任意のアクティビティまたはオブジェクトに追加でき、受信者はアクターの ID とデータの整合性を検証できます。これにより、認証がトランスポートから切り離され、アクティビティのリレー、埋め込みオブジェクト、クライアント側の署名など​​、さまざまなプロトコルの改善が可能になります。

## 歴史

Mastodon は[2017年から](https://github.com/mastodon/mastodon/pull/4687)Linked Data 署名をサポートしており、その後、他の多くのプラットフォームでもサポートが追加されました。これらの署名は整合性証明に似ていますが、他の標準に置き換えられた古い[Linked Data Signatures 1.0](https://github.com/w3c-ccg/ld-signatures/)仕様に基づいています。



## 要件

このドキュメント内のキーワード「MUST」、「MUST NOT」、「REQUIRED」、「SHALL」、「SHALL NOT」、「SHOULD」、「SHOULD NOT」、「RECOMMENDED」、「MAY」、および「OPTIONAL」は、[RFC-2119][RFC-2119]で説明されているとおりに解釈されます。

## 完全性の証明
提案された認証メカニズムは、[データ整合性][Data Integrity]仕様に基づいています。

### 証明の生成

証明は、データ整合性仕様の[セクション4.2「証明の追加」](https://w3c.github.io/vc-data-integrity/#add-proof)に従って作成する必要があります。


証明生成のプロセスは、次の手順で構成されます。

- **正規化**は、何らかの決定論的アルゴリズムに従って、JSON オブジェクトをハッシュに適した形式に変換することです。
- **ハッシュ化**は、暗号ハッシュ関数を使用して変換されたデータの識別子を計算するプロセスです。

- **署名生成**は、入力データの整合性を変更から保護する値を計算するプロセスです。

結果の証明は、元のJSONオブジェクトの`proof`キーに追加されます。オブジェクトには複数の証明が含まれる場合があります。

整合性証明で使用される属性のリストは、データ整合性仕様の[セクション2.1 証明](https://w3c.github.io/vc-data-integrity/#proofs)で定義されています。証明タイプはセクション3.1 `DataIntegrityProof`で指定されているように`DataIntegrityProof`である必要があります。`proofPurpose`属性の値は`assertionMethod`でなければなりません。

証明の`verificationMethod`属性の値は、公開鍵のHTTP(S)URLまたは[DID URL][DID-URL]とすることができる。検証方法を表現する[コントローラドキュメント][ControllerDocument]は、アクターオブジェクト、または[ActivityPub]アクターに証明的に関連付けることができる他のドキュメント([DID][DIDs]ドキュメントなど)でなければならない[MUST]。検証メソッドは、コントローラドキュメントの`assertionMethod`プロパティと関連付けなければならない[MUST]。コントローラドキュメントがアクターオブジェクトである場合、実装者は、[FEP-521a]に記載されているように、`assertionMethod`プロパティを使用すべきである[SHOULD]。

### Proof verification

Recipients of an object SHOULD perform proof verification if it contains integrity proofs. Verification process MUST follow the *Data Integrity* specification, section [4.4 Verify Proof](https://w3c.github.io/vc-data-integrity/#verify-proof). It starts with the removal of the `proof` value from the JSON object. Then verification method is retrieved from the controller document as described in *Controller Documents* specification, section [3.3 Retrieve Verification Method](https://www.w3.org/TR/controller-document/#retrieve-verification-method). Then the object is canonicalized, hashed and signature verification is performed according to the parameters specified in the proof.

If both HTTP signature and integrity proof are used, the integrity proof MUST be given precedence over HTTP signature. The HTTP signature MAY be dismissed.

### Algorithms

Implementers are expected to pursue broad interoperability when choosing algorithms for integrity proofs.

[eddsa-jcs-2022][eddsa-jcs-2022] cryptosuite is RECOMMENDED:

- Canonicalization: [JCS][JCS]
- Hashing: SHA-256
- Signatures: EdDSA

>[!WARNING]
>`eddsa-jcs-2022` cryptosuite specification is not stable and may change before it becomes a W3C Recommendation.

### Backward compatibility

Integrity proofs and linked data signatures can be used together, as they rely on different properties (`proof` and `signature`, respectively).

If compatiblity with legacy systems is desired, the integrity proof MUST be created and inserted before the generation of the linked data signature.

If both `proof` and `signature` are present in a received object, the linked data signature MUST be removed before the verification of the integrity proof.

### Special cases

#### Activities

The controller of the verification method MUST match the actor of activity, or be associated with that actor.

## 例

### Signed object

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/security/data-integrity/v1"
  ],
  "id": "https://server.example/objects/1",
  "type": "Note",
  "attributedTo": "https://server.example/users/alice",
  "content": "Hello world",
  "proof": {
    "@context": [
      "https://www.w3.org/ns/activitystreams",
      "https://w3id.org/security/data-integrity/v1"
    ],
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "verificationMethod": "https://server.example/users/alice#ed25519-key",
    "proofPurpose": "assertionMethod",
    "proofValue": "...",
    "created": "2023-02-24T23:36:38Z"
  }
}
```

### Signed activity

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/security/data-integrity/v1"
  ],
  "id": "https://server.example/activities/1",
  "type": "Create",
  "actor": "https://server.example/users/alice",
  "object": {
    "id": "https://server.example/objects/1",
    "type": "Note",
    "attributedTo": "https://server.example/users/alice",
    "content": "Hello world"
  },
  "proof": {
    "@context": [
      "https://www.w3.org/ns/activitystreams",
      "https://w3id.org/security/data-integrity/v1"
    ],
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "verificationMethod": "https://server.example/users/alice#ed25519-key",
    "proofPurpose": "assertionMethod",
    "proofValue": "...",
    "created": "2023-02-24T23:36:38Z"
  }
}
```

### Signed activity with embedded signed object

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/security/data-integrity/v1",
    {
      "object": {
        "@id": "as:object",
        "@type": "@json"
      }
    }
  ],
  "id": "https://server.example/activities/1",
  "type": "Create",
  "actor": "https://server.example/users/alice",
  "object": {
    "@context": [
      "https://www.w3.org/ns/activitystreams",
      "https://w3id.org/security/data-integrity/v1"
    ],
    "id": "https://server.example/objects/1",
    "type": "Note",
    "attributedTo": "https://server.example/users/alice",
    "content": "Hello world",
    "proof": {
      "@context": [
        "https://www.w3.org/ns/activitystreams",
        "https://w3id.org/security/data-integrity/v1"
      ],
      "type": "DataIntegrityProof",
      "cryptosuite": "eddsa-jcs-2022",
      "verificationMethod": "https://server.example/users/alice#ed25519-key",
      "proofPurpose": "assertionMethod",
      "proofValue": "...",
      "created": "2023-02-24T23:36:38Z"
    }
  },
  "proof": {
    "@context": [
      "https://www.w3.org/ns/activitystreams",
      "https://w3id.org/security/data-integrity/v1"
    ],
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "verificationMethod": "https://server.example/users/alice#ed25519-key",
    "proofPurpose": "assertionMethod",
    "proofValue": "...",
    "created": "2023-02-24T23:36:38Z"
  }
}
```

## Test vectors

See [fep-8b32.feature](./fep-8b32.feature)

## Implementations

- [Mitra](https://codeberg.org/silverpill/mitra/src/commit/f096ed54e350f4a0121289bcc0d1d5f83b5bbf8b/FEDERATION.md#object-integrity-proofs)
- Vervis
  ([generation](https://codeberg.org/ForgeFed/Vervis/commit/e8e587af26944d3ea8d91f5c47cc3058cf261387),
  [verification](https://codeberg.org/ForgeFed/Vervis/commit/621275e25762a1c1e5860d07a6ff87b147deed4f))
- Streams
- [Hubzilla](https://hub.somaton.com/channel/mario?mid=4214a375-3a18-4acb-b546-75c6c4818e2f)
- [Fedify](https://todon.eu/users/hongminhee/statuses/112638238338153870)

## Use cases

- [Forwarding from inbox](https://www.w3.org/TR/activitypub/#inbox-forwarding)
- [Conversation Containers](https://fediversity.site/help/develop/en/Containers)
- [FEP-ef61: Portable Objects](https://codeberg.org/fediverse/fep/src/branch/main/fep/ef61/fep-ef61.md)
- [FEP-ae97: Client-side activity signing](https://codeberg.org/fediverse/fep/src/branch/main/fep/ae97/fep-ae97.md)

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119], 1997
- Dave Longley, Manu Sporny, [Verifiable Credential Data Integrity 1.0][Data Integrity], 2024
- Manu Sporny, Dave Longley, Markus Sabadello, Drummond Reed, Orie Steele, Christopher Allen, [Decentralized Identifiers (DIDs) v1.0][DIDs], 2022
- Dave Longley, Manu Sporny, Markus Sabadello, Drummond Reed, Orie Steele, Christopher Allen, [Controller Documents 1.0][ControllerDocument], 2024
- silverpill, [FEP-521a: Representing actor's public keys][FEP-521a], 2023
- Dave Longley, Manu Sporny, [Data Integrity EdDSA Cryptosuites v1.0][eddsa-jcs-2022], 2024
- A. Rundgren, B. Jordan, S. Erdtman, [JSON Canonicalization Scheme (JCS)][JCS], 2020

[ActivityPub]: https://www.w3.org/TR/activitypub/
[RFC-2119]: https://tools.ietf.org/html/rfc2119.html
[Data Integrity]: https://w3c.github.io/vc-data-integrity/
[DIDs]: https://www.w3.org/TR/did-core/
[DID-URL]: https://www.w3.org/TR/did-core/#did-url-syntax
[ControllerDocument]: https://www.w3.org/TR/controller-document/
[FEP-521a]: https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md
[eddsa-jcs-2022]: https://w3c.github.io/vc-di-eddsa/#eddsa-jcs-2022
[JCS]: https://www.rfc-editor.org/rfc/rfc8785

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
