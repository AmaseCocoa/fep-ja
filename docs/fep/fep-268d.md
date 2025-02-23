---
slug: "268d"
authors: Daiki "tesaguri" Mizukami <tesaguriguma+fep-268d@gmail.com>
status: DRAFT
dateReceived: 2024-10-12
discussionsTo: https://socialhub.activitypub.rocks/t/fep-268d-search-consent-signals-for-objects/4654
relatedFeps: FEP-5feb
trackingIssue: https://codeberg.org/fediverse/fep/issues/421
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-268d.md)から翻訳に協力することができます。
# FEP-268d: Search consent signals for objects

## Summary

This FEP documents an extension property for [Activity Streams 2.0] to signal the consent for an object to be searched by a given actor.

## History

*This section is non-normative.*

Fedibird [introduced the `searchableBy` property](https://github.com/fedibird/mastodon/commit/91d6b018dfef47f19fcab13fb2d56f6e657f7db5) in August 2022.

kmyblue [implemented the `searchableBy` property](https://github.com/kmycode/mastodon/commit/af20b1d2aafe2e535611a2d3155306fcd1d8a89b) in April 2023.

Mastodon [began implementing the `indexable` property](https://github.com/mastodon/mastodon/pull/23808) [FEP-5feb] in February 2023 and [released it](https://github.com/mastodon/mastodon/releases/tag/v4.2.0-rc1) in September 2023.

## Background

*This section is non-normative.*

[FEP-5feb] introduced the `indexable` property, which allows an Activity Streams actor to signal their preference for whether objects attributed to them should be made publicly available for search.

The `indexable` property is an actor-level property which takes a boolean value, allowing the actor to make all of their objects available for search by either anyone or no one at all. However, sometimes it is desirable to make a certain set of objects searchable/unsearchable or make an object searchable by a limited audience. The object-level `searchableBy` property documented in this FEP complements the `indexable` property with a more fine-grained audience targeting ability.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].

## Terms defined

<div id="searchableBy" resource="http://fedibird.com/ns#searchableBy" typeof="rdf:Property">
<h3>searchableBy</h3>
<dl>
<dt>URI</dt>
<dd><code>http://fedibird.com/ns#searchableBy</code></dd>
<dt>Label</dt>
<dd property="rdfs:label" lang="en">searchable by</dd>
<dt>Comment</dt>
<dd property="rdfs:comment" lang="en">Identifies one or more entities that are allowed to search for the subject.</dd>
<dt>Domain</dt>
<dd><code property="rdfs:domain" resource="as:Object">Object</code></dd>
<dt>Range</dt>
<dd><code property="rdfs:range" resource="as:Object">Object</code> | <code property="rdfs:range" resource="as:Link">Link</code></dd>
<dt>Required</dt>
<dd property="owl:minCardinality" content="0" datatype="xsd:nonNegativeInteger">No</dd>
<dt>Is defined by</dt>
<dd><a property="rdfs:isDefinedBy" href="https://w3id.org/fep/268d" resource="https://w3id.org/fep/268d">FEP-268d</a></dd>
</dl>

A non-normative example of a publicly searchable post:

```json
{
  "@context": [
    "https://w3id.org/fep/268d",
    "https://www.w3.org/ns/activitystreams"
  ],
  "id": "https://example.com/notes/1",
  "attributedTo": "https://example.com/users/1",
  "to": "https://www.w3.org/ns/activitystreams#Public",
  "cc": "https://example.com/users/1/followers",
  "type": "Note",
  "content": "Hello, world!",
  "searchableBy": "https://www.w3.org/ns/activitystreams#Public"
}
```

A post only searchable by the mentioned users and the author's followers:

```json
{
  "@context": [
    "https://w3id.org/fep/268d",
    "https://www.w3.org/ns/activitystreams"
  ],
  "id": "https://example.com/notes/42",
  "attributedTo": "https://example.com/users/1",
  "to": "https://alice.example/actor",
  "cc": ["https://example.com/users/1/followers", "https://www.w3.org/ns/activitystreams#Public"],
  "type": "Note",
  "content": "@Alice Happy birthday!",
  "tag": {
    "type": "Mention",
    "href": "https://alice.example/actor",
    "name": "@Alice"
  },
  "searchableBy": ["https://alice.example/actor", "https://example.com/users/1/followers"]
}
```

(where `https://example.com/users/1/followers` is the `followers` collection of `https://example.com/users/1`.)

A post only searchable by the author themselves:

```json
{
  "@context": [
    "https://w3id.org/fep/268d",
    "https://www.w3.org/ns/activitystreams"
  ],
  "id": "https://example.com/notes/123",
  "attributedTo": "https://example.com/users/1",
  "to": "https://example.com/users/1/followers",
  "cc": "https://www.w3.org/ns/activitystreams#Public",
  "type": "Note",
  "summary": "Note to self",
  "searchableBy": "https://example.com/users/1"
}
```

An actor with a default searchability of `as:Public`:

```json
{
  "@context": [
    "https://w3id.org/fep/268d",
    "https://www.w3.org/ns/activitystreams"
  ],
  "id": "https://example.com/users/1",
  "type": "Person",
  "searchableBy": "https://www.w3.org/ns/activitystreams#Public"
}
```

## Searchability of objects

This section describes how the searchability of an object for a given actor should be determined.

If the object has at least one `searchableBy` property value and the searching actor isn't included in the property values, the object MUST NOT be made available for search by that actor. Even in this case, the object SHOULD be made available for search if the object is attributed to the searching actor themselves, and MAY be made available for search if the searching actor has previously interacted with the object (e.g. by `Like`-ing the object).

If the `searchableBy` property values of the object include the searching actor, the object SHOULD be made available for search by the actor, unless the searching actor is otherwise forbidden to see the object by access controls such as the Activity Streams audience targeting (`to`, `bto`, `cc`, `bcc` and `audience` properties) of the object and `Block`s by the object's attributed actors.

Objects with a `searchableBy` value of `https://www.w3.org/ns/activitystreams#Public` SHOULD be made available for search by any users.

If the object does not have any `searchableBy` values but is attributed to an actor with at least one `searchableBy` value, the object SHALL be treated as inheriting the `searchableBy` value of the attributed actor.

Note that, in JSON-LD, an empty array value (`[]`) is semantically equivalent to `null` or "undefined". In consequence, if a publisher wishes to make an object searchable by no one (instead of falling back on the attributed actor's `searchableBy` value), a placeholder object needs to be used as the `searchableBy` value of that object. The RECOMMENDED placeholder in that case is the object's `attributedTo` value, if any.

If the object does not fall under any of the conditions described in this section, the searchability of the object MAY be determined by an implementation-defined strategy.

Note that the searchability signaled by the `searchableBy` property is only applicable to Activity Streams representation of objects. Searchability of non-Activity Streams representations (most notably, HTML) of objects with non-public `searchableBy` property value SHOULD be signaled by other means. Non-normative examples of such means include the Robots Exclusion Protocol [RFC-9309] [ROBOTSTXT], the `<meta name="robots">` HTML tag [ROBOTSTXT-META] and the `X-Robots-Tag` HTTP header [GOOGLE-ROBOTS].

## Interaction with FEP-5feb (`toot:indexable`)

*This section is only applicable to conformant [FEP-5feb] implementations.*

If an object does not have any `searchableBy` values and is attributed to an actor with no `searchableBy` values but with an `http://joinmastodon.org/ns#indexable` value, the searchability of the object SHOULD be determined according to [FEP-5feb].

If an object has at least one `searchableBy` value, the property SHALL take precedence over the `indexable` property. In particular, if an object is not searchable according to the `searchableBy` property, the object must not be made available for search, even if the object is targeted to the `as:Public` collection and the object is attributed to an actor with `"indexable": true`.

## Security considerations

*This section is non-normative.*

As mentioned in the normative section, consuming implementations should take care to ignore JSON entries with an empty array value (`"searchableBy": []`). Failure to do so would have a security implication if the implementation is verifying RDF-based signatures of objects, because the empty array value does not affect the RDF dataset (and hence the resulting signature value), so that an attacker can remove/insert a `"searchableBy": []` entry from/to a signed object without making the signature verification fail, allowing them to tamper with the object's searchability if the empty array value were handled differently from "undefined".

## Implementations

*This section is non-normative.*

- [Fedibird](https://github.com/fedibird/mastodon/tree/fedibird)
- [kmyblue](https://github.com/kmycode/mastodon)

## References

- [Activity Streams 2.0] James M Snell, Evan Prodromou, <cite>[Activity Streams 2.0](https://www.w3.org/TR/activitystreams-core/)</cite>, 2017
- [FEP-5feb] Claire, <cite>[FEP-5feb: Search indexing consent for actors](https://w3id.org/fep/5feb)</cite>, 2023
- [RFC-2119] S. Bradner, <cite>[Key words for use in RFCs to Indicate Requirement Levels](https://doi.org/10.17487/RFC2119)</cite>, 1997
- [RFC-9309] M. Koster, <cite>[Robots Exclusion Protocol](https://doi.org/10.17487/RFC9309)</cite>, 2022
- [ROBOTSTXT] Martijn Koster, <cite>[A Standard for Robot Exclusion](https://www.robotstxt.org/orig.html)</cite>, 1994
- [ROBOTSTXT-META] Martijn Koster, <cite>[About the Robots \<META> tag](https://www.robotstxt.org/meta.html)</cite>, 2007
- [GOOGLE-ROBOTS] Google, <cite>[Robots Meta Tags Specifications](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag#xrobotstag)</cite>

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
