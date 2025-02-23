---
slug: "4adb"
authors: Helge <helge.krueger@gmail.com>
status: DRAFT
dateReceived: 2023-05-13
trackingIssue: https://codeberg.org/fediverse/fep/issues/100
discussionsTo: https://codeberg.org/fediverse/fep/issues/100
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-4adb.md)から翻訳に協力することができます。
# FEP-4adb: Dereferencing identifiers with webfinger

## Summary

In this FEP, we will formalize the process of dereferencing an URI using webfinger in order for usage in ActivityPub. The main goal is to enable the usage of URIs of the form `acct:user@domain` or `did:example:12345` as ids for objects used in ActivityPub. While this FEP only discusses this in the context of actors, it should be applicable for general objects. In order for a smooth introduction, it is recommended to start deployment with actor objects.

This FEP first presents the algorithm and examples, then discusses the usage in the context of the Fediverse. This means the first two sections are for people wanting to implement this FEP, the following sections are for people wanting to decide if this FEP is a good idea.

## The algorithm

[Webfinger] allows us to associate an ActivityStreams object with
a pair given by an URI and a domain. This is done through the following steps:

1. Fetch `GET https://domain/.well-known/webfinger?resource={URI}`.
2. Determine the link property with `"type":"application/activity+json"`.
3. Dereference this object.

We will denote this object by `ActivityStreams(URI, domain)`. There are now two cases to resolve an URI:

1. The URI determines the domain denoted by `domain(URI)`
2. The URI doesn't determine the domain

In case 1, it is clear that we associate `ActivityStreams(URI, domain(URI))` to the URI. In case 2, we will use the domain associated with the `@id` of the document the document the URI appeared in. If the document was received through a POST request and doesn't contain an `id`, or the `id` is an URI, the domain the POST request originated from should be used. In this case a verification that the URI can be associated with the object MUST be performed. This can for example be achieved through [FEP-c390](https://codeberg.org/fediverse/fep/src/branch/main/feps/fep-c390.md).

## 例

We start with the [acct URI Scheme](https://www.rfc-editor.org/rfc/rfc7565.html). In order to deliver the activity

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Follow",
  "actor": "https://chatty.example/ben/",
  "object": "acct:alyssa@social.example",
  "to": "acct:alyssa@social.example",
}
```

the ActivityPub server should perform the lookup for `ActivityStreams("acct:alyssa@social.example", "social.example")`, i.e. the request

```http
GET https://social.example/.well-known/webfinger?resource=acct:alyssa@social.example
```

We now turn our attention to [DIDs](https://www.w3.org/TR/did-core/).
In order to resolve the author of

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://chatty.example/ben/456",
  "type": "Note",
  "attributedTo": "did:key:z6MkekwC6R9bj9ErToB7AiZJfyCSDhaZe1UxhDbCqJrhqpS5",
  "to": "acct:alyssa@social.example",
  "content": "Hello"
}
```

one has to resolve the URI `did:key:z6MkekwC6R9bj9ErToB7AiZJfyCSDhaZe1UxhDbCqJrhqpS5`. As this URI does not contain a domain, the domain from the id, i.e. `chatty.example` is used, so the request

```http
GET https://chatty.example/.well-known/webfinger?resource=did:key:z6MkekwC6R9bj9ErToB7AiZJfyCSDhaZe1UxhDbCqJrhqpS5
```

is made. The resulting ActivityStreams object should contain

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://chatty.example/bnm789",
  "attachment": {
    "type": "VerifiableIdentityStatement",
    "subject": "did:key:z6MkekwC6R9bj9ErToB7AiZJfyCSDhaZe1UxhDbCqJrhqpS5",
    "alsoKnownAs": "https://chatty.social/bnm789",
    "proof": { ... }
  },
  ...
}
```

following [FEP-c390](https://codeberg.org/fediverse/fep/src/branch/main/feps/fep-c390.md) so that the authority of ben to use the did can be verified.

## Background

Let's start with [Section 3.1](https://www.w3.org/TR/activitypub/#obj-id) of ActivityPub. It starts with the sentence

> All Objects in [ActivityStreams] should have unique global identifiers.

We first note that the word _unique_ does not appear in [ActivityStreams]. Second it is unclear why it is there and what it means:

- Does it mean that there is exactly one identifier associated with the object?
- Does it mean that the global identifier is only used for one object?

We will assume that it is the second interpretation. In particular, this means that we can associated multiple identifiers with the same ActivityStreams object.

Next comes the following line in [ActivityPub]

> Publicly dereferencable URIs, such as HTTPS URIs, with their authority belonging to that of their originating server. (Publicly facing content SHOULD use HTTPS URIs).

The essential point of this FEP is to extend the range of _publicly dereferencable URIs_ to contain basically any URI by using [Webfinger].

The current usage of Webfinger in the Fediverse is asymmetric. As discussed in [MastoGuide](https://guide.toot.as/guide/use-your-own-domain/), one can associate many URIs of the form `acct:user@domain.tld` with the same Actor, by just making webfinger return an appropriate response. However, only one acct-URI can be associated with an Actor. This is done by:

```uri
acct:{preferredUsername}@{domain of actor id}
```

where `preferredUsername` is from the actor object.

## Applications to the Actor Object

We now discuss applications of the approach of this FEP to the actor object.

### Preferred Account

We will follow [this suggestion](https://socialhub.activitypub.rocks/t/alsoknownas-and-acct/3132/20?u=helge) in [alsoKnownAs and acct:]. The main idea is to associate multiple accounts. For this, we will return to `alyssa@social.example`, whose Actor object would look like the following omitting irrelevant details

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://social.example/abc123",
  "preferredUsername": "alyssa",
  ...
}
```

By the algorithm discussed in background, we can associate the URI `acct:alyssa@social.example` with this. Suppose now that Alyssa owns the domain `alyssa.cool`, and set up WebFinger to answer to

```http
GET https://alyssa.cool/.well-known/webfinger?resource=me@alyssa.cool
```

with a link to the above actor. Alyssa would then be disappointed because Fediverse software is not displaying her cooler username. To remedy this, we propose the addition to the actor object

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "xrd": "http://docs.oasis-open.org/ns/xri/xrd-1.0#",
      "aliases": {
        "@id": "xrd:Alias",
        "@type": "@id",
        "@container": "@list"
      },
    }
  ],
  "id": "https://social.example/abc123",
  "preferredUsername": "alyssa",
  "aliases": ["acct:me@alyssa.cool", "acct:alyssa@social.example"],
  ...
}
```

a new generation of Fediverse software will then be able to display the account of Alyssa as `me@alyssa.cool`.

### DNS Names

Continuing with Alyssa from above, by relying on [FEP-612d](https://codeberg.org/fediverse/fep/src/branch/main/feps/fep-612d.md), she may even associate `@alyssa.cool` with her account. Unfortunately, a proper URI format is still missing.

### Using did:key

Let's return to Ben and him wanting to use a decentralized identifier, see [did-core](https://www.w3.org/TR/did-core/) and [did-method-key](https://w3c-ccg.github.io/did-method-key/)

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "xrd": "http://docs.oasis-open.org/ns/xri/xrd-1.0#",
      "aliases": {
        "@id": "xrd:Alias",
        "@type": "@id",
        "@container": "@list"
      },
    }
  ],
  "id": "https://chatty.social/bnm789",
  "preferredUsername": "ben",
  "aliases": ["did:key:z6MkekwC6R9bj9ErToB7AiZJfyCSDhaZe1UxhDbCqJrhqpS5", "acct:ben@chatty.social"],
  ...
}
```

The problem here is that, we do not know if the the did-key belongs to BEN. To remedy this, we will use [FEP-c390](https://codeberg.org/fediverse/fep/src/branch/main/feps/fep-c390.md). This means that we need to add

```json
    "attachment": {
        "type": "VerifiableIdentityStatement",
        "subject": "did:key:z6MkekwC6R9bj9ErToB7AiZJfyCSDhaZe1UxhDbCqJrhqpS5",
        "alsoKnownAs": "https://chatty.social/bnm789",
        "proof": { ... }
    }
```

with an appropriate proof to Ben's actor object.

## References

- [acct URI Scheme] P. Saint-Andre [RFC 7565](https://www.rfc-editor.org/rfc/rfc7565.html), 2015
- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [ActivityStreams] J. Snell, E. Prodromou, [ActivityStreams](https://www.w3.org/TR/activitystreams-core/), 2017
- [alsoKnownAs and acct:](https://socialhub.activitypub.rocks/t/alsoknownas-and-acct/3132?u=helge) on SocialHub
- [did-core] Manu Sporny, Dave Longley, Markus Sabadell, Drummond Reed, Orie Steele, Christopher Allen, [Decentralized Identifiers](https://www.w3.org/TR/did-core/) (DIDs) v1.0, 2022
- [did-method-key], Dave Longley, Dmitri Zagidulin, Manu Sporny, [did-method-key](https://w3c-ccg.github.io/did-method-key/) 20221
- [FEP-c390] silverpill, [FEP-c390](https://codeberg.org/fediverse/fep/src/branch/main/feps/fep-c390.md), 2022
- [FEP-612d] Helge, [FEP-612d](https://codeberg.org/fediverse/fep/src/branch/main/feps/fep-612d.md) 2023
- [MastoGuide] jippi  [Use your own user @ domain for Mastodon discoverability](https://guide.toot.as/guide/use-your-own-domain/)
- [Webfinger] P. Jones, G. Salgueiro, M. Jones, J. Smarr, [RFC 7033](https://datatracker.ietf.org/doc/html/rfc7033),2013

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
