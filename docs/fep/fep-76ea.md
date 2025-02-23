---
slug: "76ea"
authors: Evan Prodromou <evan@socialwebfoundation.org>
status: DRAFT
dateReceived: 2024-10-04
discussionsTo: https://codeberg.org/evanp/fep/issues
trackingIssue: https://codeberg.org/fediverse/fep/issues/407
---
# FEP-76ea: Conversation Threads
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-76ea.md)から翻訳に協力することができます。
## Summary

This FEP defines a way to identify the conversation thread of an object with Activity Streams 2.0.

## Motivation

Threaded conversations are a common data structure for social software. This is defined as a tree with the **original post** at its root, replies to that post as child nodes, all replies to those replies as their children, and so on recursively.

Some social software restricts the depth of the thread, while others allow for unlimited depth.

Identifying the thread that an AS2 object is part of allows for the construction of a conversation view of the thread.

It is possible with Activity Streams 2.0 to construct a conversation thread by following the `inReplyTo` property of an object until the original post is found, and then expanding the `replies` property of the original post recursively. With ActivityPub, however, this can require a lot of different HTTPS requests to different servers, which can be slow and inefficient.

This FEP defines an extension property, `thread`, that can be used to identify the conversation thread of an object.

[ActivityPub] is the primary use case for Activity Streams 2.0, but not the only one. Where specific processing requirements of ActivityPub implementations are made, they are specifically noted. General processing hints for other use cases are also provided.

## Context

The context URL for this FEP is `https://purl.archive.org/socialweb/thread`.

The context is as follows:

```
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "thr": "https://purl.archive.org/socialweb/thread#",
  "thread": {
    "@id": "thr:thread",
    "@type": "@id"
  },
  "root": {
    "@id": "thr:root",
    "@type": "@id"
  }
}
```

## Terms

The context defines two properties.

### `thread`

The `thread` property is an `OrderedCollection` that contains all of the objects in the conversation thread. The collection is ordered in reverse chronological order, with the most recent object first.

The `thread` collection does not directly represent the tree structure of the conversation thread; it is a flat list of objects. The tree structure can be reconstructed by following the `inReplyTo` and/or `replies` properties of each
object in the collection.

The `thread` property extends the `context` property from the Activity Vocabulary.

The `thread` property does not replace the `replies` property of an object. `replies` contains the possibly curated collection of direct replies to the object; `thread` contains the full conversation tree, up- and down-thread.

### `root`

The `root` property is an `Object` that is the original post of the conversation thread. The `root` property is usually the last (earliest) object in the `thread` collection.

This property gives an easy way for a consumer to find the root post of the thread without having to search the `orderedItems` collection, navigate through multiple `OrderedCollectionPage` pages, or traverse the `inReplyTo` properties of the objects in the collection.

Note that `thread` and `root` are partially inverse properties. The `thread` property of the `root` property of a collection SHOULD contain the `id` of the `thread` collection. However, the `root` property of the `thread` property of an object MAY not contain the object's `id`, because the object is in the thread, but is not the root.

## Thread maintenance

As with the `replies` property, the processor implementing the original post of a thread SHOULD maintain the `thread` collection by adding new objects to the collection as they are received.

In [ActivityPub], this could be done when the processor receives an object with an `inReplyTo` property that matches an object in the `thread` collection.

To facilitate collection synchronization, the processor SHOULD distribute an `Add` activity to the audience of the original object with the new object as the `object` property and the thread as the `target` property.

However, private replies "down-thread" may not be addressed to the author of the original post and may not be available to the processor for the original post.

The processor implementing the original post MAY curate the thread collection by filtering objects from the collection. This could be done to remove spam, off-topic, or abusive content from the thread.

In [ActivityPub], if an object is removed from the thread, he processor SHOULD distribute a `Remove` activity to the audience of the original object with the new object as the `object` property and the thread as the `target` property.

The tree structure of the thread should be maintained; every object in the thread collection, except the root, should have an `inReplyTo` property that matches the `id` of another object in the collection. If the processor removes an object from the collection, it SHOULD remove all objects that are in reply to that object, and their replies, and so on.

The `replies` property of objects in the thread collection MAY be maintained by other processors. Curation of the `replies` collections or of the `thread` collection may mean that objects may be omitted from one collection or the other. However, the `replies` collection of the original post SHOULD be a subset of the `thread` collection.

## 例

### Example 1

An example of a `Note` object with a `thread` property:

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/thread"
  ],
  "id": "https://example.com/note/123",
  "type": "Note",
  "attributedTo": "https://example.com/user/1",
  "to": [
    "https://remote.example/user/17",
    "https://remote.example/user/17/followers"
  ],
  "content": "I concur!",
  "thread": "https://remote.example/thread/117",
  "inReplyTo": "https://remote.example/note/117"
}
```

### Example 2

An example of an `Image` object with a `thread` property. The `Image` is a root or original post with no `inReplyTo` property:

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/thread"
  ],
  "id": "https://example.com/image/123",
  "type": "Image",
  "name": "A photo of a cat",
  "attributedTo": "https://example.com/user/1",
  "to": "https://example.com/user/1/followers",
  "url": {
    "type": "Link",
    "mediaType": "image/jpeg",
    "href": "https://example.com/image/123.jpg"
  },
  "replies": "https://example.com/replies/123",
  "thread": {
    "id": "https://example.com/thread/123",
    "to": "https://example.com/user/1/followers",
    "type": "OrderedCollection",
    "totalItems": 4,
    "orderedItems": [
      {
        "id": "https://fourth.example/note/721",
        "attributedTo": "https://fourth.example/user/4",
        "to": [
          "https://example.com/user/1",
          "https://example.com/user/1/followers",
          "https://other.example/user/2"
        ],
        "inReplyTo": "https://other.example/note/338"
      },
      {
        "id": "https://third.example/note/992",
        "attributedTo": "https://third.example/user/3",
        "to": "https://example.com/user/1",
        "inReplyTo": "https://example.com/image/123"
      },
      {
        "id": "https://other.example/note/338",
        "attributedTo": "https://other.example/user/2",
        "to": [
          "https://example.com/user/1",
          "https://example.com/user/1/followers"
        ],
        "inReplyTo": "https://example.com/image/123"
      },
      "https://example.com/image/123"
    ]
  }
}
```

Note that not all objects in the `thread` collection need to be addressed to the same audience. The audience of the `thread` collection is the audience of the original post.

### Example 3

This is a `Note` object that is a reply to two different objects, and thus is part of two different threads.

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/thread"
  ],
  "id": "https://example.com/note/789",
  "attributedTo": "https://example.com/user/1",
  "to": "as:Public",
  "content": "These are both good points.",
  "inReplyTo": [
    "https://remote.example/note/57",
    "https://other.example/note/456"
  ],
  "thread": [
    "https://remote.example/thread/57",
    "https://other.example/thread/456"
  ]
}
```

### Example 4

Objects in a thread that have been deleted by their author can be represented in the `thread` collection with a `Tombstone` object.

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/thread"
  ],
  "id": "https://example.com/note/345",
  "attributedTo": "https://example.com/user/1",
  "to": "as:Public",
  "content": "Activity Streams 2.0 is awesome!",
  "replies": "https://example.com/replies/345",
  "thread": {
    "id": "https://example.com/thread/345",
    "to": "as:Public",
    "type": "OrderedCollection",
    "orderedItems": [
      {
        "id": "https://third.example/note/567",
      },
      {
        "type": "Tombstone",
        "id": "https://remote.example/note/456",
        "inReplyTo": "https://example.com/note/345",
        "deleted": "2024-10-03T00:00:00Z"
      },
      "https://example.com/note/345"
    ]
  }
}
```

### Example 5

The `thread` collection can be paged, as with other collections.

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/thread"
  ],
  "id": "https://example.com/note/678",
  "attributedTo": "https://example.com/user/1",
  "to": "as:Public",
  "content": "Is Wario A Libertarian?",
  "replies": "https://example.com/replies/678",
  "thread": {
    "id": "https://example.com/thread/678",
    "to": "as:Public",
    "type": "OrderedCollection",
    "totalItems": 244780,
    "first": "https://example.com/thread/678/page/12239",
    "last": "https://example.com/thread/678/page/1"
  }
}
```

# Example 6

The `root` property can be used to identify the original post of a thread.

```
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/thread"
  ],
  "id": "https://example.com/thread/654",
  "type": "OrderedCollection",
  "totalItems": 457,
  "first": "https://example.com/thread/654/page/23",
  "last": "https://example.com/thread/654/page/1",
  "root": "https://example.com/note/654"
}
```

## Security Considerations

Not all objects in the `thread` collection may be addressed to the same audience. Representations of the collection SHOULD NOT include the `content` property or other sensitive information from objects in the collection that are not addressed to the recipient of the representation.

In [ActivityPub], the `orderedItems` property of the `thread` collection MAY be filtered for the recipient of the representation.

## Previous work

The `ostatus:conversation` property is used in Mastodon and elsewhere to identify the thread of an object, but it is not necessarily dereferenceable.

Some implementations of ActivityPub use the `context` property to represent the thread of an object. This FEP provides a more specific property, which frees up the "intentionally vague" `context` property for other uses. It also avoids the confusing clash with the `@context` property of JSON-LD.

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018

[ActivityPub]: https://www.w3.org/TR/activitypub/

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
