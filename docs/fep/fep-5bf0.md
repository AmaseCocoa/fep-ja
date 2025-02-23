---
slug: "5bf0"
authors: Michael Puckett <michael@puckett.contact>
status: DRAFT
dateReceived: 2023-04-10
trackingIssue: https://codeberg.org/fediverse/fep/issues/80
discussionsTo: https://codeberg.org/fediverse/fep/issues/80
---

# FEP-5bf0: Collection sorting and filtering

!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-5bf0.md)から翻訳に協力することができます。
## Summary

This proposal would allow Collections to have a `streams` property, as Actors do. The streams would be of the type CollectionView, a proposed vocabulary extension that represents a sorted and/or filtered version of a Collection. ActivityPub clients could then render CollectionViews without having to perform such filtering or sorting operations themselves.

Metadata about how the sorting or filtering has been applied would be applied using new proposed vocabulary extensions that leverage SHACL for describing constraints.

## Motivations

Some ActivityPub clients rely only on C2S protocols for accessing Collections and displaying the nested items.

Currently, in order to support filtering or sorting, these clients need to retrieve all paginated items, assemble them, manually sort or filter them, and then re-paginate them.

This proposal would allow servers to perform these kinds of operations, either at runtime or ahead of time, to ease the burden on clients.

Other servers should be free to explore the Collections, but they can be easily ignored, along with the new properties.

## Implementation

A CollectionView extends from OrderedCollection and represents a filtered and/or sorted version of a Collection. Similarly, a CollectionViewPage extends from OrderedCollectionPage.

The method of filtering applied to the CollectionView can be indicated via its "filter" property, which maps to one or more SHACL Shapes.

The vocabulary would also provide a SHACL Shape for indicating that a property's value is among the items in a particular ActivityStreams Collection.

The method of sorting applied to the CollectionView can be indicated via its "sort" property, which maps to a SortShape. A SortShape extends SHACL's PropertyShape and adds the "order" property which can be mapped to "Ascending" or "Descending".

If there is no "sort" property, the order is the same as that of the original Collection.

## 例

Here, an Actor's Inbox returns all Activities posted by the Actor, and the server also provides filtered versions as streams for client consumption.

The first CollectionView returns only the Like Activities.

The second CollectionView returns only created Articles with replies, demonstrating multiple filters and filtering on nested properties.

The third CollectionView demonstrates how to indicates that a given property's value is in a partiular ActivityStreams Collection. In the example, the CollectionView is returning Activities by Alyssa's Co-workers. (Alyssa has a custom stream of mutual followers who she has labeled as Co-workers.)

```json
{
  "@context": {
    "@vocab": "https://www.w3.org/ns/activitystreams#",
    "fep": "https://w3id.org/fep#",
    "CollectionView": "fep:CollectionView",
    "viewOf": "fep:viewOf",
    "filter": "fep:filter",
    "sort": "fep:sort",
    "inCollection": "fep:inCollection",
    "SortShape": "fep:SortShape",
    "order": "fep:order",
    "sh": "http://www.w3.org/ns/shacl#",
    "PropertyShape": "sh:PropertyShape",
    "path": "sh:path",
    "hasValue": "sh:hasValue",
    "minCount": "sh:minCount"
  },
  "id": "https://example.social/@alyssa/inbox",
  "type": "OrderedCollection",
  "name": "Inbox",
  "totalItems": 1000,
  "first": "https://example.social/@alyssa/inbox/page/1",
  "last": "https://example.social/@alyssa/inbox/page/2",
  "streams": [
    {
      "id": "https://example.social/@alyssa/inbox/likes",
      "type": "CollectionView",
      "name": "Likes",
      "filter": {
        "type": "PropertyShape",
        "path": "type",
        "hasValue": "Like"
      },
      "sort": {
        "type": "SortShape",
        "path": "published",
        "order": "Descending"
      },
      "totalItems": 10,
      "first": "https://example.social/@alyssa/inbox/likes/page/1",
      "last": "https://example.social/@alyssa/inbox/likes/page/1"
    },
    {
      "id": "https://example.social/@alyssa/inbox/posts-with-replies",
      "type": "CollectionView",
      "name": "Posts with Replies",
      "filter": [
        {
          "type": "PropertyShape",
          "path": "type",
          "hasValue": "Create"
        },
        {
          "type": "PropertyShape",
          "path": ["object", "inReplyTo"],
          "minCount": 1
        }
      ],
      "sort": {
        "type": "SortShape",
        "path": "published",
        "order": "Descending"
      },
      "totalItems": 10,
      "first": "https://example.social/@alyssa/inbox/blog-posts/page/1",
      "last": "https://example.social/@alyssa/inbox/blog-posts/page/1",
      "viewOf": "https://example.social/@alyssa/inbox"
    },
    {
      "id": "https://example.social/@alyssa/inbox/notes-by-coworkers",
      "type": "CollectionView",
      "name": "Posts by Co-Workers",
      "filter": {
        "type": "InCollectionShape",
        "path": "actor",
        "inCollection": "https://example.social/@alyssa/friends/coworkers"
      },
      "sort": {
        "type": "SortShape",
        "path": "published",
        "order": "Descending"
      },
      "totalItems": 10,
      "first": "https://example.social/@alyssa/inbox/notes-by-coworkers/page/1",
      "last": "https://example.social/@alyssa/inbox/notes-by-coworkers/page/1",
      "viewOf": "https://example.social/@alyssa/inbox"
    }
  ]
}
```

## Vocabulary Extensions

Here are the terms that would needed to be added to the FEP vocabulary:

```json
{
  "@context": {
    "fep": "https://w3id.org/fep#",
    "as": "https://www.w3.org/ns/activitystreams#",
    "sh": "http://www.w3.org/ns/shacl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#"
  },
  "fep:CollectionView": {
    "@id": "fep:CollectionView",
    "@type": "rdfs:Class",
    "rdfs:subClassOf": "as:OrderedCollection",
    "rdfs:label": "Collection View",
    "rdfs:comment": "Represents a sorted and/or filtered version of a Collection"
  },
  "fep:CollectionViewPage": {
    "@id": "fep:CollectionViewPage",
    "@type": "rdfs:Class",
    "rdfs:subClassOf": "as:OrderedCollectionPage",
    "rdfs:label": "Collection View Page",
    "rdfs:comment": "Represents a sorted and/or filtered version of a CollectionPage"
  },
  "fep:SortShape": {
    "@id": "fep:SortShape",
    "@type": "rdfs:Class",
    "rdfs:subClassOf": "sh:PropertyShape",
    "rdfs:label": "Sort Shape",
    "rdfs:comment": "A PropertyShape with an 'order' property"
  },
  "fep:filter": {
    "@id": "fep:filter",
    "@type": "rdf:Property",
    "rdfs:domain": "fep:CollectionView",
    "rdfs:range": "sh:Shape",
    "rdfs:label": "Filter",
    "rdfs:comment": "The method of filtering applied to the CollectionView"
  },
  "fep:sort": {
    "@id": "fep:sort",
    "@type": "rdf:Property",
    "rdfs:domain": "fep:CollectionView",
    "rdfs:range": "fep:SortShape",
    "rdfs:label": "Sort",
    "rdfs:comment": "The method of sorting applied to the CollectionView"
  },
  "fep:order": {
    "@id": "fep:order",
    "@type": "rdf:Property",
    "rdfs:domain": "fep:SortShape",
    "rdfs:range": "fep:SortOrderType",
    "rdfs:label": "Order",
    "rdfs:comment": "Indicates whether the sort order is 'Ascending' or 'Descending'"
  },
  "fep:SortOrderType": {
    "@id": "fep:SortOrderType",
    "@type": "rdfs:Class",
    "rdfs:label": "Sort Order Type",
    "rdfs:comment": "For indicating the sort order"
  },
  "fep:Ascending": {
    "@id": "fep:Ascending",
    "@type": "fep:SortOrderType",
    "rdfs:label": "Ascending",
    "rdfs:comment": "Indicates ascending sort order"
  },
  "fep:Descending": {
    "@id": "fep:Descending",
    "@type": "fep:SortOrderType",
    "rdfs:label": "Descending",
    "rdfs:comment": "Indicates descending sort order"
  },
  "fep:InCollectionShape": {
    "@id": "fep:InCollectionShape",
    "@type": "rdfs:Class",
    "rdfs:subClassOf": "sh:PropertyShape",
    "rdfs:label": "In Collection Shape",
    "rdfs:comment": "For filtering on whether a property's value is in a Collection.",
    "sh:js": "fep:inCollectionFunction"
  },
  "fep:inCollection": {
    "@id": "fep:inCollection",
    "@type": "rdf:Property",
    "rdfs:domain": "fep:InCollectionShape",
    "rdfs:range": "sh:IRI",
    "rdfs:label": "In Collection",
    "rdfs:comment": "Maps a Collection URL to fep:inCollectionFunction"
  },
  "fep:inCollectionFunction": {
    "@id": "fep:inCollectionFunction",
    "@type": "sh:JSFunction",
    "sh:jsFunction": "inCollection",
    "sh:jsLibrary": "https://w3id.org/fep/functions.js",
    "sh:parameter": {
      "@type": "sh:Parameter",
      "sh:path": "fep:inCollection"
    }
  },
  "fep:viewOf": {
    "@id": "fep:viewOf",
    "@type": "rdf:Property",
    "rdfs:label": "View of",
    "rdfs:domain": "fep:CollectionView",
    "rdfs:range": "as:Collection",
    "rdfs:comment": "A reference back to the original Collection"
  }
}
```

## Security

Servers could in theory make available a templated URL endpoint that allows for arbitrary sorting or filtering. This should be discouraged, as it could lead to database injections. Instead, only predetermined sorted/filtered CollectionViews should be made available via the `streams` property.

## 参考文献

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
