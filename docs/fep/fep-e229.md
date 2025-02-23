---
slug: "e229"
authors: a <a@trwnh.com>
status: DRAFT
dateReceived: 2024-04-02
trackingIssue: https://codeberg.org/fediverse/fep/issues/293
discussionsTo: https://codeberg.org/fediverse/fep/issues/293
---
# FEP-e229: Best practices for extensibility
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-e229.md)から翻訳に協力することができます。

## Summary

Current popular implementations of ActivityPub do not handle extensibility very well. This FEP seeks to highlight some basic requirements for extensibility, and offer suggested advice to implementers who wish to avoid compatibility issues, particularly for LD-unaware consumers.

## General recommendations

### LD-unaware consumers

#### Normalize types into type-sets

It is an unfortunate and erroneous belief that objects in [AS2-Core] or [AP] can have only one type. This assumption breaks proper extensibility. Wherever a generic ActivityStreams consumer needs to know whether it is dealing with an [AS2-Vocab] type or [AS2-Core] mechanism like Collections, it cannot do so unless that type is present in the `type` set. However, extension vocabularies may need to declare additional types as interfaces that have been fulfilled by the given object. For this reason, LD-unaware consumers doing type checks need to take care to normalize `type` into a set, and check that their desired type is contained within that set.

For example, `"type": "Collection"` would be normalized into `"type": ["Collection"]`.

#### Ignore JSON-LD context if you don't understand it

LD-unaware consumers MUST NOT attempt naive string comparison against the JSON-LD context declaration. There are several possible reasons why a received document might be valid AS2 but not declare a `@context`. One possibility is that the declared Content-Type is `application/activity+json` and the producer is LD-unaware. Another possibility is that the producer is LD-aware, but using a different context IRI that defines the same terms. Yet another possibility is that the producer is embedding inline term definitions. Regardless of the reason, either the consumer understands it or does not understand it.

### LD-aware consumers

#### Assume the AS2 context if none is provided

Given that including the [AS2-Context] is only a SHOULD and not a MUST, it is possible for some LD-unaware producers to serialize a document without a `@context` declaration. If the Content-Type is `application/activity+json`, then you MUST assume or inject the [AS2-Context] into the document, per [AS2-Core] section 2.1.

### LD-unaware producers

#### Declare IRIs for terms that are expected to be shared

By default, the [AS2-Context] document declares `@vocab` to be `_:`, meaning that the default vocabulary namespace is the blank namespace. Extension types and properties can be implemented as-is by LD-unaware producers, and the JSON-LD expansion algorithm will expand `term` to `_:term`. JSON-LD compaction will not strip these properties, but without the `@vocab: _:` declaration, they would be stripped. This may be sufficient for experimental or implementation-specific terms that are not expected to be used by anyone else, but it is bad practice for extensibility; the use of the blank namespace for properties is obsolete and may be removed in a future version of JSON-LD.

### LD-aware producers

#### Avoid unnecessary term prefixes

Compact IRI prefixes can have multiple terms map to the same prefix, depending on which context the producer uses for compaction. For example, say we have a prefix for `http://example.com/`. You may encounter some documents with `example:term`, some documents with `ex:term`, some documents with `http://example.com/term`, and so on. LD-aware consumers can "simply" apply JSON-LD expansion to make all terms unambiguous, and then apply JSON-LD compaction against their local preferred context. LD-unaware consumers instead have to deal with unbounded possible equivalent terms, and will either have to add support for them on a case-by-case basis, or reinvent and reimplement JSON-LD expansion. This issue can be ameliorated by taking care to reuse existing conventional prefixes. An example of this is the [RDFa-Context] "initial context".

#### Consider producing documents compacted against *only* the AS2 context document

Since JSON-LD expanded form is unambiguous, it may be a good idea to use it wherever possible. This slightly reduces human readability due to the additional verbosity, but it results in exactly one possible representation of your extension data. LD-unaware consumers will possibly have to learn the structure of JSON-LD expanded form. LD-aware consumers can "simply" re-compact the document against any additional contexts they understand.

For example, consider the current use of "profile fields" prior to [FEP-fb2a] "Actor metadata". Ignoring that Mastodon currently uses `sc` as a term prefix for an incorrect definition, such a term prefix would be unnecessary if partially-uncompacted JSON-LD was used:

```json
{
	"@context": "https://www.w3.org/ns/activitystreams",
	"id": "https://example.com/~alyssa",
	"type": "Person",
	"name": "Alyssa P. Hacker",
	"attachment": [
		{
			"type": "http://schema.org/PropertyValue",
			"http://schema.org/name": "Pronouns",
			"http://schema.org/value": "she/her"
		}
	]
}
```

In general, try to consider whether your target consumer is expected to understand the context you are declaring. For ActivityStreams-specific consumers, the [AS2-Context] is a requirement, and so it can be generally depended upon. Some specifications (such as [WebAnnotations]) may similarly require their own context to be declared, whereas some specifications may not require this. In general, it may be better to not require a context and to use only the partially-compacted form; this is because multiple context declarations makes it possible for some contexts to conflict, and the most recently-declared context will win out, leading to potentially undefined behavior. This behavior can be avoided by being more judicious about your context declarations and your choice of context documents to compact against.

#### Declare the ActivityStreams context *last*, if compacting against additional contexts

Because [AP] and [AS2-Core] mandate compaction against the [AS2-Context] while also mandating that terms cannot be overridden, it is best to have the [AS2-Context] be the most-recently-declared context. For example:

```json
{
	"@context": [,
		"https://schema.org",
		"https://www.w3.org/ns/activitystreams"
	],
	// ...
}
```

## How to define an extension

LD-unaware producers need to be aware of at least some of the ways that JSON-LD works; otherwise, their extensions will go into a blank namespace and may be stripped in future versions of JSON-LD. See the [recommendations for LD-unaware producers](#declare-iris-for-terms-that-are-expected-to-be-shared) above.

### Extension property

Extension properties will primarily be one of two kinds:

- Those whose value is a literal value. In uncompacted form, these would use `@value`.
- Those whose value is a node on the graph. In uncompacted form, these would use `@id`.

For LD-unaware producers, it is enough to produce JSON of the following form:

```json
{
	"@context": "https://www.w3.org/ns/activitystreams",
	"http://example.com/valueProperty": "some string or number or boolean",
	"http://example.com/idProperty": {
		"@id": "https://example.com/some-resource"
	}
}
```

For LD-aware producers, first note that compacting against any additional context will make parsing harder for LD-unaware consumers, as described in [above guidance for LD-aware producers](#consider-producing-documents-compacted-against-only-the-as2-context-document-compact-only-as2) -- any additional context you declare must be shared by the consumer, and the only guaranteed context is the ActivityStreams context. Still, for the benefit of LD-aware consumers, you SHOULD at least make a context document available for download out-of-band. One mechanism for doing so is described in [FEP-888d]. 

The previous example can be expressed as follows when you compact against additional context:

```json
{
	"@context": [
		{
			"valueProperty": "http://example.com/valueProperty",
			"idProperty": {
				"@id": "http://example.com/idProperty",
				"@type": "@id"
			}
		},
		"https://www.w3.org/ns/activitystreams"
	]
}
```

More complete guidance on term definitions is available within [JSONLD11-TERMS].

### Extension type

Extension types follow similar logic to extension properties with literal values.

For LD-unaware producers, it is enough to use a full IRI as an extension type, although note that some vocabularies have been declared as term prefixes as part of the [AS2-Context], and thus MUST be serialized as compact IRIs using the given prefix. For example, `vcard` is declared by the [AS2-Context], but `schema` is not. Thus:

```json
{
	"@context": "https://www.w3.org/ns/activitystreams",
	"type": ["Person", "http://schema.org/Person", "vcard:Individual"]
}
```

LD-aware producers may wish to declare additional context, keeping in mind [above guidance for LD-aware producers](#consider-producing-documents-compacted-against-only-the-as2-context-document-compact-only-as2):

```json
{
	"@context": [,
		"https://schema.org",
		"https://www.w3.org/ns/activitystreams"
	],
	"type": ["Person", "schema:Person", "vcard:Individual"]
}
```

Alternatively, to avoid importing the entire Schema.org context:

```json
{
	"@context": [
		{
			"schema": "http://schema.org/"
		},
		"https://www.w3.org/ns/activitystreams"
	],
	"type": ["Person", "schema:Person", "vcard:Individual"]
}
```

## References

- [AP] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [AS2-Context] [ActivityStreams 2.0 Terms](https://www.w3.org/ns/activitystreams), 2017
- [AS2-Core] James M Snell, Evan Prodromou, [Activity Streams 2.0](https://www.w3.org/TR/activitystreams-core/), 2017
- [AS2-Vocab] James M Snell, Evan Prodromou, [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/), 2017
- [FEP-888d] a, [FEP-888d: Using https://w3id.org/fep as a base for FEP-specific namespaces](https://w3id.org/fep/888d), 2023
- [FEP-fb2a] a, [FEP-fb2a: Actor metadata](https://w3id.org/fep/fb2a), 2022
- [JSONLD11-TERMS] Gregg Kellogg, Pierre-Antoine Champin, Dave Longley, [JSON-LD 1.1 Section 9.15.1 Expanded Term Definition](https://www.w3.org/TR/json-ld11/#expanded-term-definition), 2020
- [RDFa-Context] Ivan Herman, [RDFa Core Initial Context](https://www.w3.org/2011/rdfa-context/rdfa-1.1), 2011
- [WebAnnotations] Robert Sanderson, Paolo Ciccarese, Benjamin Young, [Web Annotations Data Model](https://www.w3.org/TR/annotation-model/), 2017

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
