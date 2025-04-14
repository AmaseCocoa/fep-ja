---
slug: "d8c2"
authors: Evan Prodromou <evan@prodromou.name>
status: DRAFT
dateReceived: 2023-09-17
trackingIssue: https://codeberg.org/fediverse/fep/issues/165
discussionsTo: https://codeberg.org/fediverse/fep/issues/165
---
# FEP-d8c2: OAuth 2.0 Profile for the ActivityPub API
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-d8c2.md)から翻訳に協力することができます。

## Summary

This FEP defines a profile of OAuth 2.0 for use with the ActivityPub API.

## Motivation

[ActivityPub] defines the ActivityPub API, a RESTful HTTP API for stream-oriented social software. Also called "client to server" or "c2s", this API allows clients to create new `Activity` objects by posting to an actor's `outbox` collection.

The ActivityPub specification does not define an authorization mechanism for the API, although the [ActivityPubAuth] recommendations include some suggestions. Although there are many ways to implement client authorization for an API, OAuth 2.0 is a popular and well-understood framework.

OAuth 2.0 is very broad and encompasses a number of different techniques and use cases. [OAuth20Simplified] documents the most common profile of OAuth 2.0: authorization code flow and bearer tokens. Many OAuth 2.0 client libraries implement this profile.

The most common case for OAuth 2.0 is an API supplied by a single provider.
There are three main issues with using OAuth 2.0 for a standard API implemented by many providers.

1. *Client identifiers*. The OAuth 2.0 flow uses the client identifier to show important information about the client software to the user, and to avoid certain classes of spoofing attacks. With a single provider, the client developer can register a client ID out of band using the provider's developer Web site or other tools. Client developers cannot manually register client IDs with each provider of the ActivityPub API.
2. *Client metadata*. To make authorization decisions, the user needs to
having information about the client software such as its name, description,
icon, and publisher. With a single provider, the API service can require
client metadata through its registration interface and store it in a database. A client developer cannot manually register client metadata with each provider of the ActivityPub API.
3. *Scopes*. The OAuth 2.0 flow uses scopes to indicate the permissions that the client is requesting. With a single provider, the provider can define and document a set of scopes that are appropriate for the API. It would be difficult for client developers had to use different scopes for each different ActivityPub API implementer.

This profile addresses these issues by using the ActivityPub API itself to identify and describe the client software. It also provides a simple set of scopes appropriate for social software using ActivityPub.

## Specification

- Servers MUST provide the `oauthAuthorizationEndpoint` and `oauthTokenEndpoint` properties in the actor's `endpoints` collection.
- Clients MUST use the `oauthAuthorizationEndpoint` and `oauthTokenEndpoint` properties of the actor's `endpoints` collection.
- Clients MUST use the Authorization Code flow.
- Clients MUST use [PKCE] with the `S256` method.
- Clients MUST provide a `client_id` as the ActivityPub ID of the `Application`, `Service`, or other ActivityPub object representing the client (see [Client identifier](#client-identifier) below).
- The Activity Object resource at the `client_id` URI MUST have a `redirectURI` property with the redirect URI for the client (see [Redirect URI](#redirect-uri) below).
- Clients SHOULD NOT provide a `client_secret`.
- Servers MUST ignore the `client_secret` parameter, if provided.
- The `scope` parameter SHOULD be a space-separated list of scope values (see [Scopes](#scopes) below) as defined by this specification.
- The `scope` parameter MAY include extended scopes defined by the server or client.
- Servers MUST support [Bearer tokens](https://tools.ietf.org/html/rfc6750).
- Servers MAY add an `instrument` property for `Activity` objects created by the client, with the value of the `client_id` parameter.

## Client identifier

ActivityPub provides a rich vocabulary for describing objects in the social space. Each object in the ActivityPub world has a unique https: URI, which must be dereferenceable to a JSON-LD document describing the object.

This allows a distributed description of ActivityPub API clients that doesn't require out-of-band registration.

Objects dereferenced at the SHOULD be of type `Application` or `Service`. They MUST have an `id` property with the same value as the `client_id` parameter. They MUST have a `redirectURI` property with the redirect URI for the client (see [Context document](#context-document) below).

Clients SHOULD provide metadata to help users make authorization decisions, including:

- `nameMap` or `name`: The name of the client software.
- `icon`: An `Image` object with the icon for the client software.
- `summaryMap` or `summary`: A description of the application or service.
- `attributedTo`: The `name`, `id`, `icon` and `summary` properties of
  the actor responsible for the client software.

## Scopes

The `scope` parameter is a space-separated list of scope values. The following scope values are defined:

- `read`: The client is requesting permission to read the actor's ActivityPub data, including the `inbox`, `outbox`, `liked`, `followers`, and `following` collections, and any other ActivityPub resources on the server, with the actor's authorization. The client is also requesting to use the `proxyURL` property of the actor, if it exists, to request resources from other servers with the actor's authorization.
- `write`: The client is requesting permission to create `Activity` objects by posting to the actor's `outbox` collection. This includes `Create`, `Update`, `Delete`, `Follow`, `Undo`, and other Activity types.
- `write:sameorigin`: The client is requesting permission to create `Activity` objects by posting to the actor's `outbox` collection, but only if the `Activity`'s `object`, `target` and/or `origin` properties have IDs with the same origin as the client ID. This allows the user to grant a limited scope to an application or service to interact with other resources controlled by the client, but not to interact with resources from other sources. "Same origin" is defined as a URI with the same scheme, host, and port as the client ID.

Extended scopes MAY be defined by the server or client. Servers SHOULD ignore scopes that they do not recognize. Extended scopes SHOULD use the "primary:restriction" pattern for naming the scope.

## Context document

The context document for this specification is at `https://purl.archive.org/socialweb/oauth`. Its contents are as follows:

```json
{
  "@context": {
    "oauth": "https://purl.archive.org/socialweb/oauth#",
    "redirectURI": {
      "@id": "oauth:redirectURI",
      "@type": "@id"
    }
  }
}
```

## 例

### Follower recommender

A Web service that wants to use the ActivityPub API would define an ActivityPub object at `https://followrec.example/client`. This object has a `redirectURI` property with the URI of the Web application's authorization endpoint.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/oauth"
  ],
  "id": "https:/followrec.example/apps/myapp",
  "name": "Follow Recommender",
  "type": "Service",
  "icon": {
    "type": "Image",
    "url": "http://followrec.example/followrec.png",
    "width": 256,
    "height": 256
  },
  "summaryMap": {
    "en": "Follow Recommender is a service that recommends people to follow based on your existing community."
  },
  "attributedTo": {
    "name": "Alyssa P. Hacker",
    "id": "https://hackers.example/alyssa",
    "type": "Person",
    "icon": {
      "type": "Image",
      "url": "https://hackers.example/alyssa/icon.png",
      "width": 256,
      "height": 256
    },
    "summaryMap": {
      "en": "Alyssa P. Hacker builds cool stuff on the Internet."
    }
  },
  "redirectURI": "https://followrec.example/oauth/callback"
}
```

A person who wants to use this application can provide their ActivityPub actor ID either directly (`https://home.example/evanp`) or via a Webfinger lookup (`evanp@home.example`).

The Web application discovers the `oauthAuthorizationEndpoint` to be `https://home.example/oauth/authorize`, and uses it to construct an URI for the authorization request, including scopes and PKCE parameters.

```
https://home.example/oauth/authorize?response_type=code&client_id=https%3A%2F%2Ffollowrec.example%2Fclient&redirect_uri=https%3A%2F%2Ffollowrec.example%2Foauth%2Fcallback&scope=read+write&state=1234zyx&code_challenge=1234&code_challenge_method=S256
```

The user is redirected to the authorization endpoint, where they are prompted to authorize the application. The server at `home.example` retrieves the `Service` object at `https://followrec.example/client` and, at a minimum, verifies that the `redirectURI` property matches the `redirect_uri` parameter.

The `home.example` server then prompts the user to authorize the application. If the user authorizes the application, the server redirects the user to the `redirect_uri` parameter with a `code` parameter.

The Web application then uses the `oauthTokenEndpoint` to exchange the authorization code for an access token and optional refresh token.

```
POST /oauth/token HTTP/1.1
Host: home.example
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=1234zyx&client_id=https%3A%2F%2Ffollowrec.example%2Fclient&redirect_uri=https%3A%2F%2Ffollowrec.example%2Foauth%2Fcallback&code_verifier=1234
```

It can use these access tokens to read the user's `following` collections and use [triadic closure](https://en.wikipedia.org/wiki/Triadic_closure) to recommend new people to follow. It can also use the access token to post `Follow` activities to the user's `outbox` collection.

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "type": "Follow",
    "object": "https://otherserver.example/otheruser"
}
```

The server at `home.example` may add the `instrument` property to the resulting `Activity` to identify the responsible service.

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "https://home.example/activities/1234",
    "actor": "https://home.example/evanp",
    "type": "Follow",
    "object": "https://otherserver.example/otheruser",
    "instrument": "https://followrec.example/client",
    "published": "2021-09-01T12:34:56Z",
    "updated": "2021-09-01T12:34:56Z"
}
```

### Mobile checkin app

An iOS app uses the ActivityPub API to post location updates for a user. Because the app is a native program, it uses a static site provided by its version control system to host the client object at `https://developer.git.example/kfc/client.json`.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/oauth"
  ],
  "id": "https:/mobile.example/app",
  "name": "Kentucky Fried Checkin",
  "type": "Application",
  "icon": {
    "type": "Image",
    "url": "https://developer.git.example/kfc/icon.png",
    "width": 256,
    "height": 256
  },
  "summaryMap": {
    "en": "Kentucky Fried Checkin is a mobile app that allows you to post checkins to your ActivityPub timeline."
  },
  "attributedTo": {
    "name": "MobileCorp",
    "id": "https://mobilecorp.example/organization",
    "type": "Organization",
    "icon": {
      "type": "Image",
      "url": "https://mobilecorp.example/organization/logo.png",
      "width": 256,
      "height": 256
    },
    "summaryMap": {
      "en": "MobileCorp provides cool apps supporting the social web."
    }
  },
  "redirectURI": "checkin:oauth/callback"
}
```

Note that the `redirectURI` property is a custom URI scheme for the mobile app.

A person who wants to use this application can provide their ActivityPub actor ID either directly (`https://home.example/evanp`) or via a Webfinger lookup (`evanp@home.example`).

The checkin discovers the `oauthAuthorizationEndpoint` to be `https://home.example/oauth/authorize`, and uses it to construct an URI for the authorization request, including scopes and PKCE parameters.

```
https://home.example/oauth/authorize?response_type=code&client_id=https%3A%2F%2Ffollowrec.example%2Fclient&redirect_uri=https%3A%2F%2Ffollowrec.example%2Foauth%2Fcallback&scope=write&state=1234zyx&code_challenge=1234&code_challenge_method=S256
```

Note that the `scope` parameter only includes the `write` scope, because the app only needs to post to the user's `outbox` collection.

The authorization flow continues as with the [Follower recommender](#follower-recommender) example, until the mobile app has a valid access token.

The mobile app can then post `Arrive` activities to the user's `outbox` collection.

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "type": "Arrive",
    "summaryMap": {
      "en": "evanp arrived at the Empire State Building."
    },
    "location": {
      "id": "https://places.example/empire-state-building",
      "type": "Place",
      "name": "Empire State Building",
      "latitude": 40.7484,
      "longitude": -73.9857
    }
}
```

### Open Farm Game

A Web game at `openfarmgame.example` lets its users construct imaginary farms with crops, livestock, and buildings. It uses the ActivityPub API to post game events to a user's `outbox` collection. It defines its client object at `https://openfarmgame.example/client`.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://purl.archive.org/socialweb/oauth"
  ],
  "id": "https://openfarmgame.example/client",
  "name": "Open Farm Game",
  "type": "Service",
  "icon": {
    "type": "Image",
    "url": "https://openfarmgame.example/client/icon.png",
    "width": 256,
    "height": 256
  },
  "summaryMap": {
    "en": "Raise crops, grow livestock, and build your farming empire! Open Farm Game is the social farming application you can play with friends and family."
  },
  "attributedTo": {
    "name": "FarmGamer Inc.",
    "id": "https://openfarmgame.example/organization",
    "type": "Organization",
    "icon": {
      "type": "Image",
      "url": "https://openfarmgame.example/organization/logo.png",
      "width": 256,
      "height": 256
    },
    "summaryMap": {
      "en": "We help players become farmers."
    }
  },
  "redirectURI": "https://openfarmgame.example/oauth/callback"
}
```

The authorization flow works as with the [follow recommender](#follower-recommender) above. Because the actor primarily interacts with objects on the game server, the game only needs to request `write:sameorigin` scope.

When the user plants a new crop in their imaginary farm, the game posts a `Create` activity to the user's `outbox` collection.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {"farm": "https://openfarmgame.example/ns#"}
  ],
  "type": ["farm:Plant", "Create"],
  "summaryMap": {
    "en": "evanp planted corn."
  },
  "object": {
    "id": "https://openfarmgame.example/crops/1234",
    "type": ["farm:Crop", "Object"],
    "nameMap": {
      "en": "Corn"
    },
    "image": {
      "type": "Image",
      "url": "https://openfarmgame.example/crops/corn.png",
      "width": 256,
      "height": 256
    }
  }
}
```

Note that the `object` property of the `Create` activity has an `id` property with the same origin as the client ID. This allows the actor's home server to verify that the client is only creating objects on the game server.

## Security considerations

- [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics) provides a number of best practices for implementing OAuth 2.0.
- One risk of implementing OAuth 2.0 is that the user is redirected to the `redirect_uri` parameter after authorization is complete. This can be used as an attack to treat the authorization server as an open redirector. The API server should check that the `redirect_uri` parameter matches the `redirectURI` property of the client object.
- A valid `redirectURI` property should not change often.
- As with any protocol that requires fetching a client-provided URI, the server should take care in dereferencing the `client_id` parameter to avoid attacks such as very large responses, responses that take a long time to generate, or responses with poorly-formatted content.
- The ActivityPub object used to define the client includes metadata that can be spoofed, like the `name` or `icon`. An attacker could use the name, icon, or publisher of a popular application to trick users into authorizing the attacker's application. Tools such as shared blocklists, reputation systems, and user education can help mitigate this risk.

## 参考文献

- [OAuth2] Dick Hardt, [The OAuth 2.0 Authorization Framework](https://www.ietf.org/rfc/rfc6749.txt), 2012
- [OAuth20Simplified], Aaron Parecki, [OAuth 2.0 Simplified](https://www.oauth.com/), 2016
- [PKCE], N. Sakimura, J. Bradley, N. Agarwal, [Proof Key for Code Exchange by OAuth Public Clients](https://datatracker.ietf.org/doc/html/rfc7636), 2015.
- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [ActivityPubAuth] Various authors. [SocialCG/ActivityPub/Authentication Authorization](https://www.w3.org/wiki/SocialCG/ActivityPub/Authentication_Authorization), 2017


## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
