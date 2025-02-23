---
slug: "d556"
type: implementation
authors: Steve Bate <svc-fep@stevebate.net>
status: DRAFT
dateReceived: 2024-01-20
trackingIssue: https://codeberg.org/fediverse/fep/issues/243
discussionsTo: https://codeberg.org/fediverse/fep/issues/243
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-d556.md)から翻訳に協力することができます。
# FEP-d556: Server-Level Actor Discovery Using WebFinger

## Summary

Server-level [ActivityPub] actors support server-wide functionality rather than representing a user or the software equivalent (sometimes called a *bot*). This proposal describes how to discover a server-level actor's URI using [WebFinger].

## Terminology

The term *server* is not well-defined. For the purposes of this document, an server is an *origin* [SameOriginPolicy] having the same URL prefix (scheme, host, port). The term does not imply anything about network or software architecture. An server could consist of many server processes behind a load-balancing reverse proxy. Or, inversely, a single server process could host many servers (multi-tenant architecture).

Some implementations could have multiple actors to support different server-level roles (moderation, administration, etc.). In this document, the term *server-level actor* will be used to describe these kind of actors. The term *Server Actor* or *Application Actor* is a special, but common, case where there is a single server-level actor.

The term *Server* is used extensively in the [ActivityPub Recommendation][ActivityPub], although it is mostly undefined beyond which activities a server may process. The term is closely related to Mastodon's use of the word *instance*, although this is not the only way the word is used in online discussions.

> NOTE: The standard role and responsibilities of server-level actors are not defined here (or elsewhere, at the time of this submission). Several implementations have something they call an Instance Actor or Application Actor, but they may or may not be interoperable since no standard behaviors have been defined at this time.

## Use Cases

Although this FEP does not define specific uses of server-level actors, it's useful to know how they are, or could be, used in practice. The following are a some potential use cases:

* **Signing Fetch Requests:** This appears to be the most common use case. The requirement for this is a combination of: limiting access to actor profiles by requiring HTTP Signatures (i.e., "authorized fetch") and tightly-coupling actor profiles with their public keys. This results in profile/key fetch loops. ([InstanceActor]). To mitigate this undesirable behavior, one technique is to have a third-party actor (often called an "instance actor") sign every fetch request. Actor discovery is not required for this use case, but it's mentioned here because it appears to be the motiving use case for [FEP-2677], which has some similarities to this one.

* **Relay Support:** A server-level actor can be used for subscribing to a relay (often using an [ActivityPub] `Follow` request) and receiving `inbox` messages.

* **Server-level Subscriptions:** Some implementations, like [Pleroma], provide an actor that can be followed to receive [all messages from an "instance"][InstanceActorIssue].

* **Moderation:** A server-level actor may be used to federate moderation-related content (actor or domain blocks, post flags, etc.) or provide a publication proxy to shield the identity of moderators performing the actions.

* **Announcements:** An server-level actor could be used to public server news. For example, it could publish content including announcements about new features, maintenance schedules, or updates.

* **Object Attribution:** Some server implementations allow some objects to be attributed to the server rather than an individual user or account.

* **Administration:** A server-level actor could be used to share information about software issue (including reports from users), available updates, and security vulnerabilities and mitigations.
  
## Discovery

To discover an server-level actor's URI, query [WebFinger] with the server prefix as the resource query parameter.

Example Request:
```
GET /.well-known/webfinger?resource=https://server.example/
```
Response:
```json
{
    "subject": "https://server.example/",
    "links": [
        {
            "rel": "https://www.w3.org/ns/activitystreams#Service",
            "type": "application/activity+json",
            "href": "https://server.example/actor"
        }
    ]
}
```
The `subject` would typically be the resource URI. This proposal does not depend on any specific URI for `subject`, although the ActivityPub actor URI is recommended.

The Server-level Actor's URI will be the `href` property of a `link` with a `rel` (relation type) property of `https://www.w3.org/ns/activitystreams#Service` ([W3C AS2 Service Primer][ActivityPubService]). The type of the Server-level Actor itself is not required to be the same as the relation type.

The `https://www.w3.org/ns/activitystreams#Service` `rel` value may be replaced with `self` if there is no ambiguity between the server-level actor and user's actor in a single actor server (see discussion of [single-actor servers](#single-actor-servers)).

A `http://webfinger.net/rel/profile-page` `rel` ([WebFinger Relations][WebFingerRels]) can be used to link to server metadata (possibly with multiple content types). However, the structure of the target metadata has not been defined at this time. For example, the following links refer to profile data in HTML and JSON-LD formats.

```json
{
    "subject": "https://server.example/",
    "links": [
        {
            "rel": "https://www.w3.org/ns/activitystreams#Service",
            "type": "application/activity+json",
            "href": "https://server.example/actor"
        },
        {
            "rel": "http://webfinger.net/rel/profile-page",
            "type": "text/html",
            "href": "https://server.example/profile"
        },
        {
            "rel": "http://webfinger.net/rel/profile-page",
            "type": "application/ld+json",
            "href": "https://server.example/profile"
        }
    ]
}
```

If multiple server-level actor links are returned, the links can be disambiguated by adding metadata to the links using standard [WebFinger] properties. For example, an implementation could have different server-level actors that serve different purposes. 

It's also possible that another FEP will define standard `rel` URIs for common roles. In that case, those FEP role URIs SHOULD be preferred. 

> NOTE: The definition of standard server-level actor roles is outside the scope of this FEP.

```json
{
    "subject": "https://server.example/",
    "links": [
        {
            "rel": "https://www.w3.org/ns/activitystreams#Service",
            "type": "application/activity+json",
            "href": "https://server.example/actor",
            "properties": {
              "http://schema.org/roleName": "administration"
            }
        },
        {
            "rel": "https://www.w3.org/ns/activitystreams#Service",
            "type": "application/activity+json",
            "href": "https://server.example/actor",
            "properties": {
              "http://schema.org/roleName": "moderation"
            }
        }
    ]
}
```

In this example, the same actor used used for administration and moderation. However, the example would also be valid if the actors were different. It's possible that for some use cases a role might be further refined. For example, additional properties might specify a geographical region for a role.


## <a id="single-actor-servers"></a> Single Actor Servers

A developer of a single-actor (user actor) server may want that user to have a URI corresponding to the server prefix although it's not intended to be an server-level actor. This scenario, which is not expected to be a common one, can be supported by returning multiple links in the [WebFinger] response.

```json
{
    "subject": "https://server.example/",
    "links": [
        {
            "rel": "https://www.w3.org/ns/activitystreams#Service",
            "type": "application/activity+json",
            "href": "https://server.example/server-actor"
        },
        {
            "rel": "self",
            "type": "application/activity+json",
            "href": "https://server.example/user-actor"
        }
    ]
}
```

If an application is only interested in a the Server Actor or User Actor specifically, it can use the `rel` query parameter to filter the links, as described in the [WebFinger] specification (if supported by the [Webfinger] service implementation).

For example, to only query the User Actor URI, the query would be:

```
GET /.well-known/webfinger?resource=https://server.example/&rel=self
```

```json
{
    "subject": "https://server.example/",
    "links": [
        {
            "rel": "self",
            "type": "application/activity+json",
            "href": "https://server.example/user-actor"
        }
    ]
}
```

# Implementations

Known implementations include:

* FIRM
* [Mastodon] implements something similar to this proposal. 

## Mastodon Example

```
GET /.well-known/webfinger?resource=https://mastodon.social/
Host: https://mastodon.social
```
or using Mastodon account-based URI:
```
GET /.well-known/webfinger?resource=acct:mastodon.social@mastodon.social
Host: https://mastodon.social
```

```json
{
  "subject": "acct:mastodon.social@mastodon.social",
  "aliases": [
    "https://mastodon.social/actor"
  ],
  "links": [
    {
      "rel": "http://webfinger.net/rel/profile-page",
      "type": "text/html",
      "href": "https://mastodon.social/about/more?instance_actor=true"
    },
    {
      "rel": "self",
      "type": "application/activity+json",
      "href": "https://mastodon.social/actor"
    },
    {
      "rel": "http://ostatus.org/schema/1.0/subscribe",
      "template": "https://mastodon.social/authorize_interaction?uri={uri}"
    }
  ]
}
```

Some differences between the Mastodon implementation and this proposal include:

* It does not support standard [WebFinger] filtering by `rel`.

* The `subject` is the Mastodon-specific account URI for the server-level actor rather than the recommended [ActivityPub] actor URI.

Since no user-related actor link is provided for the server resource, the `self` `rel` value can be used without ambiguity

## Related Proposals

[FEP-2677] suggests using [NodeInfo] for a similar purpose. There are several disadvantages of this compared to using [WebFinger]. 

* Although [WebFinger] is not required by the [ActivityPub] Recommendation, it is required for federation with most ActivityPub-based implementations (e.g., Mastodon and compatible implementations). [NodeInfo] is not required for federation, so requiring it's use for this purpose increases the complexity of federation with no benefits.
* [WebFinger] has been standardized by the Internet Engineering Task Force (IETC). [NodeInfo] is defined informally.
* [WebFinger] is already used to resolve resource identifiers and provide links to server-level metadata (e.g., profile page URLs). [NodeInfo] is primarily used for gathering and aggregating server metadata.
* [FEP-2677] adds a new non-standard `rel` relation to the [NodeInfo] index document. This may have surprising effects on some consuming implementations. This proposal is using [WebFinger] in standard ways.
* Given an [ActivityVocabulary] actor type is being used for the WebFinger `rel` value, a `as:Service` ([Primer][ActivityPubService]) is the type suggested by the W3C ActivityStreams Primers for this kind of resource rather than `as:Application` ([Primer][ActivityPubApp]). (Note this is distinct from the type specified in the server-level actor resource that's linked from WebFinger.)
* [FEP-2677] only defines a singleton server-level actor. This proposal allows that use case but has more flexibility for advanced implementations.
* [FEP-2677] Requires actors to have an `as:Application` type. This proposal has no constraints on the actor type. The `as:Service` URI is only used for the link relation type.
  
Although the definition isn't clear, the "Application Actor" in [FEP-2677] appears to be a proxy for a software "application" (not defined, but appears to be a similar concept to "server" in this proposal). For example, there's a discussion about attaching application metadata to the actor. In this proposal, there is no server proxy actor (although that's not prohibited). There is a server WebFinger resource with linked server-level service actors, but the server resource is not necessarily an actor itself.

[FEP-2c59] discusses how to discover [WebFinger] resource URIs from an [ActivityPub] actor resource. This is not related to server-level actor discovery.

[FEP-4adb] discusses dereferencing identifiers with WebFinger. It's similar to this proposal but not specifically related to discovering server-level actors.

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub], 2018
- James M Snell, Evan Prodromou, [ActivityStreams Vocabulary][ActivityVocabulary], 2017
- W3C ActivityStreams Primer - [Application type][ActivityPubApp]
- W3C ActivityStreams Primer - [Service type][ActivityPubService]
- Eugen Rochko, [Mastodon], 2016
- Jonne Haß, [NodeInfo 2.1][NodeInfo]
- MDN, [Same-origin Policy][SameOriginPolicy]
- Brad Fitzpatrick, [WebFinger], 2013
- WebFinger\.net [Link Relations][WebFingerRels]

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.

[ActivityPub]: https://www.w3.org/TR/activitypub/ "The ActivityPub protocol is a decentralized social networking protocol based upon the ActivityStreams 2.0 data format. It provides a client to server API for creating, updating and deleting content, as well as a federated server to server API for delivering notifications and content."
[ActivityVocabulary]: https://www.w3.org/TR/activitystreams-vocabulary "This specification describes the Activity vocabulary. It is intended to be used in the context of the ActivityStreams 2.0 format and provides a foundational vocabulary for activity structures, and specific activity types."
[ActivityPubApp]: https://www.w3.org/wiki/Activity_Streams/Primer/Application_type "W3c AS2 Primer for the Application type"
[ActivityPubService]: https://www.w3.org/wiki/Activity_Streams/Primer/Service_type "W3c AS2 Primer for the Service type"
[InstanceActor]: https://seb.jambor.dev/posts/understanding-activitypub-part-4-threads/#the-instance-actor "This article documents some request sequences for authorized fetch loops. The article mistakenly associates this behavior with ActivityPub, although it's a non-ActivityPub quirk mostly based on Mastodon implementation designs."
[InstanceActorIssue]: https://github.com/mastodon/mastodon/issues/10453 "Original GitHub issue associated with adding an instance-wide actor to Mastodon."
[Mastodon]: https://joinmastodon.org/ "Self-hosted, globally interconnected microblogging software"
[NodeInfo]: http://nodeinfo.diaspora.software/protocol.html "NodeInfo defines a standardized way to expose metadata about an installation of a distributed social network"
[Pleroma]: https://pleroma.social/ "Pleroma is a microblogging server software that can federate (i.e., exchange messages with) other servers that support the same federation standards (OStatus and ActivityPub)."
[SameOriginPolicy]: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy "The same-origin policy is a critical security mechanism that restricts how a document or script loaded by one origin can interact with a resource from another origin."
[WebFinger]: https://tools.ietf.org/html/rfc7033 "Protocol for discovering information about people or other entities on the Internet using standard HTTP methods"
[WebFingerRels]: https://webfinger.net/rel "WebFinger link relations defined at webfinger.net"
[FEP-2677]: ../2677/fep-2677.md "FEP-2677: Identifying the Application Actor"
[FEP-2c59]: ../2c59/fep-2c59.md "FEP-2c59: Discovery of a Webfinger address from an ActivityPub actor"
[FEP-4adb]: ../4adb/fep-4adb.md "FEP-4adb: Dereferencing identifiers with webfinger"