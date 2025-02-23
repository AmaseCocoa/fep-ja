---
slug: "fb2a"
authors: a <a@trwnh.com>
status: DRAFT
dateReceived: 2022-12-09
trackingIssue: https://codeberg.org/fediverse/fep/issues/45
discussionsTo: https://codeberg.org/fediverse/fep/issues/45
---
# FEP-fb2a: Actor metadata
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-fb2a.md)から翻訳に協力することができます。


## Summary

It is useful for actors to publish additional structured information about themselves without necessarily defining an extension property or additional vocabulary. This FEP describes a way for actors to publish generic key-value pairs representing their metadata.

## History

Mastodon v2.4.0 (March 2018) implemented "bio fields" [1], a feature that allows adding structured data to profiles. This feature was federated via the `attachment` field, filtering for array items that had a type of `PropertyValue` derived from schema.org's vocabulary. Each item used `name` from the ActivityStreams Vocabulary, and `value` from the schema.org context. The schema.org namespace was defined as `schema` and (erroneously) mapped to `http://schema.org#` (instead of `http://schema.org/` or `https://schema.org`) within the JSON-LD context property. 

Misskey (December 2018) implemented "user fields" [2], following the same federation logic as Mastodon (filtering for a type of `PropertyValue`, then taking `name` and `value`).

Pleroma (August 2019) implemented "custom profile fields" [3], following the same federation logic as Mastodon (filtering for a type of `PropertyValue`, then taking `name` and `value`).

## 1. Using ActivityStreams Note instead of schema.org PropertyValue

Rather than depending on an additional (and unnecessary) vocabulary, it makes sense to define a more "native" way of expressing the same idea of a key-value pair representing structured metadata about the actor. To this end, this FEP proposes using the existing `Note` type from the ActivityStreams 2.0 Vocabulary (instead of schema.org's `PropertyValue`), as well as the existing `content` property (instead of schema.org's `value`). Note that the `name` property exists within both the ActivityStreams 2.0 Vocabulary and the schema.org vocabulary, with largely the same semantic meaning; however, the use of schema.org vocabulary is out of scope of this FEP.

Thus, we can define a standard for actor metadata, largely drawing from prior art.

## 2. Defining generic actor metadata as key-value pairs

General-purpose actor metadata fields SHOULD be included in the `attachment` array on the actor. If a more specific property exists and is a better fit for the specific metadata being expressed, then implementations MAY use that instead of or in addition to the more generic actor metadata.

- Each generic metadata field MUST have a type of `Note`.
- Each metadata field MUST have a property of `name` representing the name (key) of the field.
- Each metadata field MUST have a property of `content` representing the content (value) of the field.

## 3. Defining generic actor metadata as links

Actor metadata fields may also take the form of a link rather than a content value.

- Each generic link MUST have a type of `Link`.
- Each link field MUST have a property of `href` representing the value of the link.
- Each link field SHOULD have a property of `name` representing a label for the link.
- Each link field SHOULD use `rel` values if an appropriate link relation exists, such as `"me"`.

## 4. Backwards compatibility with legacy implementations of profile fields using incorrect schema.org IRIs

(このセクションは非規範的です。)

Existing implementations currently using the incorrect IRIs `http://schema.org#PropertyValue` and `http://schema.org#value` may wish to maintain backwards compatibility during a transitional period by serving both legacy representations as well as representations according to this FEP. The following algorithm may be used to support the legacy implementations while also favoring the implementation within this FEP:

- Filter the `attachment` array for items of type `Note` or `Link`. Take `name` and `content` from each remaining item if the type is `Note`, or take `name` and `href` if the type is `Link`. If the type is `Link` and the `rel` contains `"me"`, attempt to verify this link using rel-me verification.
- If none are found (or if items remain in the set of `attachment`), filter the `attachment` array for items of type `http://schema.org#PropertyValue`. Take `name` and `http://schema.org#value` from each remaining item. If `name` is a duplicate of an existing `name`, ignore the item.

After some transitional period, implementations may wish to simplify their logic by filtering only for items of type `Note` and drop support for `http://schema.org#PropertyValue`, `http://schema.org#value`, and the schema.org context entirely (assuming those implementations do not use any other vocabulary from the schema.org context).

## 5. Examples of current legacy implementations compared to serialization according to this FEP

Consider a profile which has the following profile fields:

```
Pronouns
: they/them

My portfolio
: https://example.com/
```

A legacy implementation might currently serialize these as such:

```
{
	"@context": [
		"https://www.w3.org/ns/activitystreams",
		{
			"sc": "http://schema.org#"
			"PropertyValue": "sc:PropertyValue",
			"value": "sc:value"
		}
	],
	"id": "https://social.example/someone"
	"type": "Person",
	"attachment": [
		{
			"type": "PropertyValue",
			"name": "Pronouns",
			"value": "they/them"
		},
		{
			"type": "PropertyValue",
			"name": "My portfolio",
			"value": "<a href="https://example.com" target="_blank" rel="nofollow noopener noreferrer me" translate="no"><span class="invisible">https://</span><span class="">example.com</span><span class="invisible"></span></a>"
		}
	]
}
```

For implementations that do not include the same incorrect IRI term mapping in their contexts, compaction would result in the following:

```
{
	"@context": "https://www.w3.org/ns/activitystreams",
	"id": "https://social.example/someone"
	"type": "Person",
	"attachment": [
		{
			"type": "http://schema.org#PropertyValue",
			"name": "Pronouns",
			"http://schema.org#PropertyValue": "they/them"
		},
		{
			"type": "http://schema.org#PropertyValue",
			"name": "My portfolio",
			"http://schema.org#PropertyValue": "<a href="https://example.com" target="_blank" rel="nofollow noopener noreferrer me" translate="no"><span class="invisible">https://</span><span class="">example.com</span><span class="invisible"></span></a>"
		}
	]
}
```

Implementation according to this FEP might result in a simpler and more semantically correct serialization but equivalent representation like so:

```
{
	"@context": "https://www.w3.org/ns/activitystreams",
	"id": "https://social.example/someone"
	"type": "Person",
	"attachment": [
		{
			"type": "Note",
			"name": "Pronouns",
			"content": "they/them"
		},
		{
			"type": "Link",
			"name": "My portfolio",
			"href": "https://example.com",
			"rel": ["nofollow", "noopener", "noreferrer", "me"]
		}
	]
}
```

## Implementations

- WordPress
- Streams
- NodeBB

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [1] Gargron, [Add bio fields (#6645)](https://github.com/mastodon/mastodon/pull/6645)
- [2] mei23, [Show user fields (#3590)](https://github.com/misskey-dev/misskey/pull/3590)
- [3] minibikini, [Add custom profile fields (!1488)](https://git.pleroma.social/pleroma/pleroma/-/merge_requests/1488)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication 

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
