---
slug: "bad1"
authors: a <a@trwnh.com>
status: DRAFT
dateReceived: 2023-06-15
trackingIssue: https://codeberg.org/fediverse/fep/issues/124
discussionsTo: https://codeberg.org/fediverse/fep/issues/124
---
# FEP-bad1: Object history collection
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-bad1.md)から翻訳に協力することができます。

## Summary

[AS2-Core] provides examples 18, 19, 32 which represent the "history" of an object.

Particularly in example 32, we see an object being Created, Updated, and Deleted. However, there is no property dedicated to advertising a collection fit for this purpose. This FEP attempts to define one.

## 例 taken from ActivityStreams 2.0

(このセクションは非規範的です。)

For convenience, the following examples are reproduced from the ActivityStreams 2.0 [AS2-Core] specification. Examples 18 and 19 are found in normative sections; example 32 is found in a non-normative section.

Example 18:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "summary": "Object history",
  "type": "Collection",
  "totalItems": 2,
  "items": [
    {
      "type": "Create",
      "actor": "http://www.test.example/sally",
      "object": "http://example.org/foo"
    },
    {
      "type": "Like",
      "actor": "http://www.test.example/joe",
      "object": "http://example.org/foo"
    }
  ]
}
```

Example 19:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "summary": "Object history",
  "type": "OrderedCollection",
  "totalItems": 2,
  "orderedItems": [
    {
      "type": "Create",
      "actor": "http://www.test.example/sally",
      "object": "http://example.org/foo"
    },
    {
      "type": "Like",
      "actor": "http://www.test.example/joe",
      "object": "http://example.org/foo"
    }
  ]
}
```

Example 32:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "oa": "http://www.w3.org/ns/oa#",
      "prov": "http://www.w3.org/ns/prov#",
      "dcterms": "http://purl.org/dc/terms/",
      "dcterms:created": {
        "@id": "dcterms:created",
        "@type": "xsd:dateTime"
      }
    }
  ],
  "summary": "Editing history of a note",
  "type": "Collection",
  "items": [
    {
      "id": "http://example.org/activity/20150101000000",
      "type": [ "Create", "prov:Activity" ],
      "actor": {
        "id": "http://example.org/#eric",
        "name": "Eric"
      },
      "summary": "Eric wrote a note.",
      "object": {
        "id": "http://example.org/entry/20150101000000",
        "type": [ "Note", "prov:Entity" ],
        "attributedTo": "http://example.org/#eric",
        "content": "Remember... all I'm offering is the trooth. Nothing more."
      },
      "published": "2015-01-01T00:00:00Z"
    },
    {
      "id": "http://example.org/activity/20150101000059",
      "type": [ "Update", "prov:Activity", "oa:Annotation" ],
      "summary": "Eric edited a note.",
      "dcterms:created": "2015-01-01T00:00:59Z",
      "dcterms:creator": { "@id": "http://example.org/#eric" },
      "oa:hasBody": {
        "id": "http://example.org/entry/20150101000059",
        "type": [ "Note", "prov:Entity" ],
        "content": "Remember... all I'm offering is the truth. Nothing more.",
        "prov:wasAttributedTo": { "@id": "http://example.org/#eric" },
        "prov:wasRevisionOf": { "@id": "http://example.org/entry/20150101000000" }
      },
      "oa:hasTarget": { "@id": "http://example.org/entry/20150101000000" },
      "oa:motivatedBy": { "@id": "oa:editing" },
      "prov:generated": { "@id": "http://example.org/entry/20150101000059" },
      "prov:wasInformedBy": { "@id": "http://example.org/activity/20150101000000" }
    },
    {
      "id": "http://example.org/activity/20150101010101",
      "type": [ "Delete", "prov:Activity" ],
      "actor": "http://example.org/#eric",
      "summary": "Eric deleted a note.",
      "object": "http://example.org/entry/20150101000059",
      "published": "2015-01-01T01:01:01Z"
    }
  ]
}
```

From these examples, Example 18 is nearly identical to Example 19, except for the type being `Collection` in Example 18, and `OrderedCollection` in Example 19. An object's history collection will necessarily be ordered chronologically, although whether the ordering should be forward chronological or reverse chronological is an open question; at the time of writing this FEP, [ActivityPub] Section 5 currently contains the following language:

> An OrderedCollection MUST be presented consistently in reverse chronological order.

This language indicates that if `OrderedCollection` is used, the ordering MUST be reverse chronological.

## Defining the `history` special collection

An object's history is discovered through the `history` property of an object. The `history` MUST be an `OrderedCollection`.

The history stream contains all activities which target the object as `object`, where the `actor` matches the `attributedTo` actor. This might include Create, Update, and/or Delete activities.

This is differentiated from `context`, which per FEP-7888 may be a collection containing related objects and activities. It is possible in certain cases to obtain an object's history by filtering such a collection for all items that contain an `object` referencing a given object, but this is not a consistent or straightforward way of doing so.

## Terms defined by this FEP

In the associated [context](./context.jsonld), we define the following terms:

### history

URI
: `https://w3id.org/fep/bad1/history`

Notes
: A collection containing all activities performed by the author related to this object's representation.

JSON-LD definition (`@id` pending adoption of FEP-9606):

```json
{
	"@context": {
		"history": {
			"@id": "https://w3id.org/fep/bad1/history",
			"@type": "@id"
		}
	}
}
```

Example:

```json
{
	"@context": [
		"https://w3id.org/fep/bad1/history",
		"https://www.w3.org/ns/activitystreams"
	],
	"id": "https://example.com/some-file",
	"type": "Tombstone",
	"formerType": "Document",
	"url": "https://example.com/404",
	"history": {
		"id": "https://example.com/some-object/log",
		"type": "OrderedCollection",
		"orderedItems": [
			{
				"id": "https://example.com/some-file/log/3",
				"type": "Delete",
				"object": "https://example.com/some-file"
			},
			{
				"id": "https://example.com/some-file/log/2",
				"type": "Update",
				"object": {
					"id": "https://example.com/some-file",
					"url": "https://example.com/storage/hash2"
				}
			},
			{
				"id": "https://example.com/some-file/log/1",
				"type": "Create",
				"object": {
					"type": "Document",
					"url": "https://example.com/storage/hash1"
				}
			}
		]
	}
}
```

## 参考文献

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [AS2-Core] James M Snell, Evan Prodromou, [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/), 2017


## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。