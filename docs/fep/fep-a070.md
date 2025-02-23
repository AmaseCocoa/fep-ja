---
slug: "a070"
authors: a <a@trwnh.com>
status: DRAFT
dateReceived: 2023-06-13
trackingIssue: https://codeberg.org/fediverse/fep/issues/121
discussionsTo: https://codeberg.org/fediverse/fep/issues/121
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-a070.md)から翻訳に協力することができます。
# FEP-a070: Ordered properties for plain JSON consumers

## Summary

In a [Github-issue] filed against the normative AS2 context, it was pointed out that `attachment` and `tag` are unordered by default, although some implementations of "fediverse" software blindly assume them to always be ordered. This can be made unambiguous by using `@list` in JSON-LD, but for plain JSON consumers, a separate shorthand term must be defined. This FEP attempts to disambiguate between unordered and ordered arrays for those plain JSON consumers.

## Prior art in `items` vs `orderedItems`

(This section is non-normative.)

In the [AS2-vocab], there is `items`, which is used to express the items included in a Collection or OrderedCollection. However, in [ActivityPub], the OrderedCollection type is mandated to be reverse chronological order. Tangentially, it is valid to have a Collection with `items` that are either ordered or unordered.

The normative [AS2-context] deals with issue by defining two different terms: `items` and `orderedItems`. Both of these terms have the same `@id` of `https://www.w3.org/ns/activitystreams#items` and the same `@type` of `@id` to indicate that they contain nodes on the graph, but `orderedItems` is additionally defined with a `@container` of an ordered `@list`, overriding the default `@container` of an unordered `@set`.

The use of `orderedItems` is present in the examples for `items` within [AS2-vocab], but it is not defined separately because it is not a separate term. Following this pattern, we can define similarly "ordered" counterparts to "unordered" existing properties.

## An exploration of properties that producers may wish to explicitly order

(This section is non-normative)

attachment
: There is nothing that requires attachments to an object to be ordered; however, it is a popular expectation in several cases that the attachments should be ordered. For example, someone authoring an object may wish to attach three images in order, representing a triptych. If the array order of `attachment` were to be changed, this would change the author's intended representation.

tag
: Tags are generally freeform and not meant to be parsed in any order, but some use-cases may wish to order tags as well. For example, Tumblr allows users to set the order their tags are presented in, and users sometimes use these tags to communicate a series of comments or thoughts without having them be present in the conversational context.

name
: A thing may have multiple names, some of which are preferred more than others. Having an array for `name` is likely to confuse existing implementations at the time of writing this FEP, but it is something that could conceivably be useful for several use-cases.

oneOf/anyOf
: When a Question represents a poll with predefined options for possible answers, the order of the options may be relevant to understanding the poll. Consider a self-referential poll that asks you to predict which option will receive the most votes: "option 2", "option 3", or "option 1".

## Terms defined by this FEP

In the associated [context](./context.jsonld), we define the following terms:

### orderedAttachment

URI
: `https://www.w3.org/ns/activitystreams#attachment`

Notes
: https://www.w3.org/TR/activitystreams-vocabulary/#dfn-attachment

JSON-LD definition:

```json
{
	"@context": {
		"orderedAttachment": {
			"@id": "https://www.w3.org/ns/activitystreams#attachment",
			"@type": "@id",
			"@container": "@list"
		}
	}
}
```

### orderedTag

Term
: `orderedTag`

URI
: `https://www.w3.org/ns/activitystreams#tag`

Notes
: https://www.w3.org/TR/activitystreams-vocabulary/#dfn-tag

JSON-LD definition:

```json
{
	"@context": {
		"orderedTag": {
			"@id": "https://www.w3.org/ns/activitystreams#tag",
			"@type": "@id",
			"@container": "@list"
		}
	}
}
```

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [AS2-context] W3C, [activitystreams.jsonld](https://www.w3.org/ns/activitystreams.jsonld)
- [AS2-vocab] James M Snell, Evan Prodromou, [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/), 2017
- [Github-issue] trwnh, [orderedAttachment (and orderedTag?)](https://github.com/w3c/activitystreams/issues/537), 2023


## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication 

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
