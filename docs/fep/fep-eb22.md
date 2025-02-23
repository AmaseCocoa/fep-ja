---
slug: "eb22"
authors: Manton Reece <manton@micro.blog>
status: DRAFT
dateReceived: 2024-07-25
discussionsTo: https://socialhub.activitypub.rocks/t/fep-eb22-supported-activitystreams-types-with-nodeinfo/4469
trackingIssue: https://codeberg.org/fediverse/fep/issues/386
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-eb22.md)から翻訳に協力することができます。
# FEP-eb22: Supported ActivityStreams types with NodeInfo

## Summary

Servers can advertise what features of the API they support, such as creating a poll or boosting a post. Clients can recognize if a server doesn't support a feature and hide it from the UI.

## Background

As the fediverse grows, we'll naturally see a wide range of servers and clients, and not all of them will exactly match the features that popular clients like Mastodon support. To support a diverse mix of clients with different user experiences, clients and servers should communicate about which features should be shown in the client UI.

Some services might not include public likes or polls, for example. Some might not include boosting. It would be confusing for a boost icon to show an error message when clicked if that feature was not available. Instead, the client should detect whether boosting is available, and if not simply hide the icon when connected to that server.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this specification are to
be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119.html).

## Proposal

Document in NodeInfo the ActivityStreams types and properties that correspond to features the server supports. This information can be used by both ActivityPub clients as well as Mastodon API clients. We use ActivityStreams types because they are already part of a fediverse standard, rather than inventing new keys to describe existing features.

Add a new top-level field `types` to NodeInfo with three fields, all of which are optional:

* `activities`: The types of activities the server broadly supports.
* `objects`: The types of objects that the server supports in the relevant activities.
* `properties`: For each activity or object, what properties are allowed in that object.

If no types are present in NodeInfo, a client should assume a server supports all types, just as most clients do today.

If no objects are present for an activity, a client should assume a server supports all common objects. If a limited number of objects are present, a client should assume a server only supports those objects.

If no properties are present for an object, a client should assume a server supports all common properties. If a limited number of properties are present, a client should assume a server only supports those properties.

## Examples

The following examples are simplified versions of what most servers should support. Most popular servers will support more activity and object types than this, but these examples keep the list short for readability. Other common activities include `Follow`, `Undo`, `Accept`, `Block`, and others as listed in the Activity Vocabulary specification.

### Supporting common social features and book reviews (`Review`)

This example adds `Review`, an object used by BookWyrm for book reviews. Because only properties for `Review` are specified, a client can assume that all common objects and properties for the other activities are supported.

```
{
  "types": {
    "activities": [
      "Create",
      "Like",
      "Announce",
      "Question",
      "Move",
      "Follow"
    ],
    "objects": [
      "Note",
      "Article",
      "Image",
      "Review"
    ],
    "properties": {
      "Review": [ "title", "body", "rating", "inReplyToBook" ]
    }
  }
}
```

### Supporting common social features, but _not_ supporting polls (`Question`)

This example leaves off the `Question` type and the `oneOf` property used for polls.

```
{
  "types": {
    "activities": [
      "Create",
      "Like",
      "Announce"
    ],
    "objects": [
      "Note",
      "Article",
      "Image"
    ],
    "properties": {
      "Note": [ "summary", "content", "published", "inReplyTo" ],
      "Article": [ "name", "content", "published" ]
    }
  }
}
```

### Supporting common social features including polls, but _not_ supporting titled posts (`name`)

This example includes only the `content` and `published` properties for `Article`, meaning the client should not prompt for a title for long-form articles.

```
{
  "types": {
    "activities": [
      "Create",
      "Like",
      "Announce",
      "Question",
    ],
    "objects": [
      "Note",
      "Article",
      "Image"
    ],
    "properties": {
      "Question": [ "oneOf" ],
      "Note": [ "summary", "content", "published", "inReplyTo" ],
      "Article": [ "content", "published" ]
    }
  }
}
```

### Supporting common social features, but _not_ supporting boosts (`Announce`)

This example lists the bare minimum types, leaving off the explicit list of objects and properties. Clients should assume the server supports whatever properties are common for those types, such as `Note`, `Article`, and others.


```
{
  "types": {
    "activities": [
      "Create",
      "Like",
      "Question",
      "Move"
    ]
  }
}
```

### Supporting common social features and account migration (`Move`)

This example lists `object` and `target` fields that are used for account migration with the `Move` activity.

```
{
  "types": {
    "activities": [
      "Create",
      "Like",
      "Announce",
      "Question",
      "Move"
    ],
    "properties": {
      "Move": [ "object", "target" ]
    }
  }
}
```

## Related proposals

There have been a few other attempts at addressing how to describe what a server supports:

* [FEP-9fde: Mechanism for servers to expose supported operations](https://codeberg.org/fediverse/fep/src/branch/main/fep/9fde/fep-9fde.md): This extension also uses NodeInfo, but with reverse domain name identifiers and versions to document what operations a server supports. This ties features with the clients that first added support rather than more generally with the common activity types.
* [FEP-6481: Specifying ActivityPub extension support with NodeInfo](https://codeberg.org/fediverse/fep/src/branch/main/fep/6481/fep-6481.md): This lists "extensions" to the common ActivityStreams types. BookWyrm, for example, adds a book "Review" type that could be documented in this way.
* [Micropub: Query for Supported Vocabulary](https://indieweb.org/Micropub-extensions#Query_for_Supported_Vocabulary): From the IndieWeb community, the Micropub API is an interface for posting to a web site. This Micropub extension returns a list of post types that a server supports.

## References

* [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core)
* [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/)
* [BookWyrm ActivityPub](https://docs.joinbookwyrm.com/activitypub.html)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
