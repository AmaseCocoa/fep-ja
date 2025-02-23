---
slug: "9fde"
authors: Nik Clayton <nik@ngo.org.uk>
dateReceived: 2024-03-14
status: DRAFT
trackingIssue: https://codeberg.org/fediverse/fep/issues/282
discussionsTo: https://codeberg.org/fediverse/fep/issues/282
---
# FEP-9fde: Mechanism for servers to expose supported operations
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-9fde.md)から翻訳に協力することができます。

| Version | Date       | Changes                                                                                                                                                                                                                |
| ------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3.1     | 2024-03-13 | - Convert to FEP format<br>- Incorporate feedback from Johannes Ernst and Steve Bates<br>- Renamed `clientApis` to `operations` so this is usable for server-server operations too<br>- Added "Open questions" section |
| 3.0     | 2024-01-22 | Re-write; include the operation information in the nodeinfo                                                                                                                                                            |
| 2.0     | 2023-09-25 | Re-write; replace the original suggestion to use the OpenAPI definition with a simpler specification                                                                                                                   |
| 1.0     | 2023-08-14 | Initial draft                                                                                                                                                                                                          |

## Summary

This document proposes an extension to the [NodeInfo schema](http://nodeinfo.diaspora.software/) ([FEP-f1d5](https://codeberg.org/fediverse/fep/src/branch/main/fep/f1d5/fep-f1d5.md)) that would allow developers of Mastodon and Mastodon-like servers to unambigiously communicate the operations their servers support, and allow developers of software that interoperates with those servers to detect those features, promoting interoperability and easier adoption of new features.
## Synopsis

This document is written for:

- The maintainers of the NodeInfo specification
- Developers of Mastodon and Mastodon-like servers
- Developers of clients for those servers

After reading this document you should:

- Understand the general problem this is intended to solve
- Understand the proposed solution
- Understand alternatives to the solution, and why they are not appropriate
- Understand the open questions for this proposal
- Be able to provide feedback on the proposal

## Requirements

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this specification are to be interpreted as described in [RFC-2119](https://datatracker.ietf.org/doc/html/rfc2119.html).
## Overview

> [!IMPORTANT] Other servers are also clients
> 
> The primary perspective of this document's author is as an end-user client developer dealing with the Mastodon client API, and the examples are written from that perspective. However, this proposal can also be used to describe operations between different servers in the Fediverse using other protocols (e.g., ActivityPub). For any given interaction between a pair of servers one of them is going to be acting as the client in the interaction, and will need to know the operations the server supports.

For the purposes of this document a "Mastodon or Mastodon-like" server is a server that presents the [Mastodon client API](https://docs.joinmastodon.org/api/), optionally with extensions to that API that provide additional functionality. These servers include, but are not limited to:

- Mastodon
- Glitch
- Hometown
- Pleroma
- Akkoma
- Firefish
- Iceshrimp
- Sharkey
- Friendica
- GoToSocial

Clients of these servers have an API discovery problem. Since different servers support different (but similar) APIs the client has to determine what API operations the server supports.

Given the wide variety of servers that are available, and their many forks, it's not feasible for clients to maintain an accurate list of all the possible server software names while mapping the names to API features.

Instead the server should have a mechanism for advertising the operations it supports.

The client would use this when determining what features to show the user, without needing to employ complex, error-prone heuristics.

This would also provide a clear mechanism for Mastodon and Mastodon-like servers to incrementally deploy new features and deprecate old ones without inconveniencing clients.

It also provides a clear mechanism to advertise server functionality without continually bolting it on to the "instance info" mechanism in the inconsistent fashion that has been done so far.

The rest of this document sets out the specific problems I'm interested in solving, with motivating examples, and then describes how the new approach would solve these problems.

## Problems
### The supported API is not easily discoverable

Changes are made to the Mastodon API in a manner that is not easily discoverable by clients.

For example, [Add POST /api/v1/conversations/:id/unread by ClearlyClaire · Pull Request #25509 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/pull/25509) adds a new API endpoint (`api/v1/conversations/:id/unread`).

The only way a client can discover that this API exists is to maintain, per-client, a mapping between Mastodon server version and the API supported at each version.

This is:

1. A lot of work for each client
2. Something that every client needs to do
3. Easy to get wrong
4. Doesn't scale across multitudes of different servers

### No standard way for Mastodon servers to advertise that some functionality is disabled or unavailable

The [Instance](https://docs.joinmastodon.org/entities/Instance/) information contains a `configuration` block that has some, but not all the information necessary to determine the features a server supports.

Other servers have extended this information in incompatible ways (e.g., the `pleroma` block).

Other servers may not implement the functionality at all. For example, GoToSocial did not implement support for the Mastodon client-side filter API until version 0.15.0.

### No standard way for Mastodon-like servers to advertise their additional functionality to clients

Mastodon-like servers implement some or all of the Mastodon API.

In many cases they also extend the API, providing additional functionality (local-only posting, quoting, markdown formatting, bookmarks, etc.)

In some cases that functionality has already been incorporated in Mastodon (e.g., bookmarks), in other cases there are plans to include that functionality in Mastodon (e.g., quoting, markdown formatting).

This leads to three problems.

1. There is no simple way for clients to know which parts of the Mastodon API the server supports
1. There is no simple way for clients to know if the server supports additional operations
2. If Mastodon decides to implement an API that was first introduced in a Mastodon-like server there is no way for clients to detect this, without recompiling the client with new information about what features a given Mastodon server version implements

### Server developers have too much to do

Server developers already have a lot of work to do. Any proposal should therefore be straightforward to implement. Additional complexity, such as changing the contents of existing API responses, or requiring developers of different servers to tightly coordinate when new functionality is introduced is going to make it less likely that groups adopt any proposed solutions.

## Proposed solution

A given Mastodon or Mastodon-like server supports a set of **operations**.

To expose those to the user a Mastodon client needs to know:

- Which operations does the server support?
- What's the overlap between the operations the server supports and the operations the client supports?

Therefore we need:

1. A unique identifer for each operation that a set of servers supports identically
2. A mechanism for a server to report the operations it supports

### A unique identifier for each operation

Operations are identified by the reverse FQDN of the server software that first implemented that operation, then an arbitrary number of additional dot-separated labels determined by the server authors.

This ensures that operation IDs are unique without needing tight coordination between different server developer groups.

For example:

- `org.joinmastodon.api.statuses.post`
- `org.joinmastodon.api.statuses.translate`
- `io.github.glitch-soc.api.statuses.bookmark`
- `dev.iceshrimp.api.notes.reactions.create`

> [!NOTE] Precise reverse FQDN to use for each server is to be decided
> 
> This example use the reverse FQDNs for the server's primary websites or documentation sites, but each server group would determine and document the reverse FQDN for their server's operations.

> [!NOTE] Dot-separated labels SHOULD map 1:1 to API endpoint components
> 
> In these examples the dotted components after the `api` correspond to the path components of the API endpoint. While there is no requirement that they do so server implementors are RECOMMENDED to do so, as it makes it easier for developers to mentally map between the operation identifier and the API path when they are working with the code.

Each label within the operation identifier is represented in lower-case US-ASCII (a character set containing 128
characters associated with code points in the range 0..7F) . If a server team wishes to define an operation that uses a non-US-ASCII label they MUST encode the operation label to US-ASCII using the `ToASCII` transformation described in [RFC 3490 - Internationalizing Domain Names in Applications (IDNA)](https://datatracker.ietf.org/doc/rfc3490/).
### Each operation has one or more versions

Each operation exists at one or more [semver-compatible](https://semver.org/) (v2.0.0) versions. Semver is used because it is a widely deployed standard, easily understandable, and client libraries that can parse this format are available across many different programming languages.

For example, in the Mastodon API documentation "[Post a new status](https://docs.joinmastodon.org/methods/statuses/#create)" describes the API for posting a new status. That API has changed three times in the Mastodon server implementation.

1. Initial implementation
2. Support for `scheduled_at`
3. Support for `poll`

There are no backwards-incompatible breaking changes across those versions so this is the same operation at three different versions; per Semver the major version stays the same and the minor version is incremented.

- `1.0.0` - initial implementation
- `1.1.0` - support for `scheduled_at`
- `1.2.0` - support for `polls`

> [!IMPORTANT] These version numbers are unrelated to the version number of the software that introduced the operation

#### Example: Bookmarks

Bookmarking statuses originated in the glitch-soc fork and was incorporated in to Mastodon.

Therefore, the ID for the bookmark operations -- if they are compatible with the glitch-soc implementation -- use the `io.github.glitch-soc.api` prefix.

- `io.github.glitch-soc.api.statuses.bookmark` @ `1.0.0` - bookmark a status
- `io.github.glitch-soc.api.statuses.unbookmark` @ `1.0.0` - remove a status from bookmarks
- `io.github.glitch-soc.api.timeline.bookmarks` @ `1.0.0` - fetch a timeline of the user's bookmarks
- `io.github.glitch-soc.api.timeline.bookmarks` @ `1.1.0` - fetch a timeline of the user's bookmarks, supporting `min_id` and `max_id` simultaneously

### Client discovery of supported operations and endpoints

Clients must be able to discover which operations the server supports and the endpoints to use for those operations.

To do this the nodeinfo (determined via `/.well-known/nodeinfo`) schema should be extended to support a new `operations` property.

The property's value is a map from a string key -- the operation ID -- to a set of one or more Semver versions of the operation that the server supports.

For example:

```
"operations": {
    ...
    "org.joinmastodon.api.some.operation": ["1.0.0", "1.1.0", "1.2.0", "2.0.0"]
    ...
}
```

> [!NOTE] Unordered versions
> 
> The supported version operations is not ordered; client code MUST treat this as a set, not a list.

> [!NOTE] Not limited to Mastodon / Mastodon-like servers
> 
> This `operations` map is not limited to operations supported by Mastodon/Mastodon-like servers. This is a general mechanism that can be used by servers to expose information about their supported operations and could be used by other Fediverse software like Lemmy, KBin, etc.

Because of the Semver rules for breaking changes servers MAY omit earlier versions from the list if they are included in a later version. In the previous example the `1.0.0` and `1.1.0` versions can be omitted as a server supporting `v1.2.0` of an operation implicitly supports all preceding versions with the same major number.

```
"operations": {
    ...
    "org.joinmastodon.api.some.operation": ["1.2.0", "2.0.0"]
    ...
}
```

> [!NOTE] There is no need to specify the operation semantics
> 
> The semantics of each {operation, version} pair are already known by the client (for each operation it supports). Semantics like whether these endpoints are `GET`, `POST`, `DELETE`, or `PATCH`, the exact names of the URL query parameters, the API endpoint, etc.
> 
> In other words, servers MUST NOT advertise support for an existing operation ID and change anything about how that operation works. The server developers should either define and document a new operation ID, or implement the operation as a new version (bumping the major version if it is a breaking change).

## ActivityPub extension support

Most of this document has presented examples using the Mastodon client API (and related APIs) but, as noted, this proposal also provides a clean mechanism to allow servers to report other extensions they support.

Here is an example from [FEP 6481 - # Specifying ActivityPub extension support with NodeInfo](https://codeberg.org/fediverse/fep/src/branch/main/fep/6481/fep-6481.md) which is presented as:

> [Manyfold](https://manyfold.app/technology/activitypub.html) intends to define a new Activity Vocabulary Object type for its own rich content, `3dModel`. The specification will define an extension IRI, for instance `https://w3id.org/manyfold/3dModel#v1`.
>
> Applications that wish to receive Activities with this type can state their support for the extension in their NodeInfo file as follows; Manyfold instances will then know that they can send that object type and it will be understood.

```json
{
  ...,
  "metadata": {
    "activitypub": {
      "extensions": [
        "https://w3id.org/manyfold/3dModel#v1"
      ]
    }
  }
}
```

In this proposal that could represented like this:

```json
"operations": {
  "app.manyfold.activitypub.accept.3dmodel": ["1.0.0"],
  ...
}
```

(the precise name of the operation would need to be determined)

Any other server that can accept a `3dModel` would expose the same operation.
## To deploy this...

### Server developers

Servers where the set of supported operations is **not** user configurable would need to maintain a static map of operations to versions, and return that map as part of the nodeinfo response.

If the set of operations is user configurable (e.g., perhaps the server software supports a translation API but the server operator has not enabled translation support) the nodeinfo response would need to be dynamically generated from the current software configuration.

In both cases developing a new operation or changing an existing operation would require the developers to:

1. Determine the operation's version number, following semver backwards-compatible rules
2. Document the behaviour of the new operation / version
3. Include the new operation / version in the server's response

### Client developers

To provide the best user experience client developers SHOULD fetch the operations map when the user logs in.

If the client supports a particular operation at a particular version the client can query the map and determine whether the concrete version they need is in the map, or met by a higher version. Semver client libraries are available for Kotlin and Java (Android) and Swift (iOS), as well as many other languages.

If the server does not support the operation the client MAY fall back to a different operation, or disable the particular operation in the UI.

To use the example from earlier, [Add POST /api/v1/conversations/:id/unread by ClearlyClaire · Pull Request #25509 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/pull/25509) which adds a new API endpoint (`api/v1/conversations/:id/unread`).

The server would report this as:

```
"operations": {
    ...
    "org.joinmastodon.api.conversations.id.unread": ["1.0.0"]
    ...
}
```

and a client that wanted to conditionally support this would query the operations map for `org.joinmastodon.api.conversations.id.unread` with any version entry with a major version of `1`, and if the operation/version pair is not found then disable the "Mark a conversation unread" UI affordances where they occur.

## Is there a proof of concept?

Yes.

I have started implementing the client side of this in [Pachli](https://pachli.app). At the moment this uses server version parsing heuristics to maintain a Pachli-specific map of operations and supported versions ([Server.kt](https://github.com/pachli/pachli-android/blob/main/core/network/src/main/kotlin/app/pachli/core/network/Server.kt) ) and then query the server's reported capabilities and adjust the UI accordingly.

For example, [this snippet](https://github.com/pachli/pachli-android/blob/74ca75632392728df0bc4eb41680e7146193ec09/app/src/main/java/app/pachli/components/preference/AccountPreferencesFragment.kt#L183-L199) conditionally enables the "edit filters" UI only if the user's server supports filtering.

Maintaining the server-specific operations map in Pachli is error prone, slow to update, and does not benefit the wider ecosystem of Mastodon clients and servers, hence this proposal.

## This solves...

This solves the problems described earlier:

- "[[#The supported API is not easily discoverable]]"
	- The client can easily discover the specific operations the server supports, and adjust UX accordingly
- "[[#No standard way for Mastodon servers to advertise that some functionality is disabled]]"
	- The `operations` property must reflect the active configuration of the server.
- "[[#No standard way for Mastodon-like servers to advertise their functionality to clients]]
	- If a Mastodon-like server implements a Mastodon-compatible API endpoint it lists that endpoint using the relevant `org.mastodon...` operation identifier.
- [[#Server developers have too much to do]]
	- This proposal doesn't modify any existing API responses
	- For a given server the list of supported operations can be statically configured, and does not change after the server has launched
	- The work of developing a dictionary of supported operations can be sharded amongst different groups
		- Server developers have a vested interest in contributing details of operations specific to their server, so more third party clients support them
		- Client developers have a vested interest in reviewing and contributing details of operations specific to servers their users use, to make their clients more attractive to potential users
		- No coordination is required between different groups of server developers to develop operation IDs
	- Developers are incentivised to re-use existing operations instead of inventing new ones
		- Implementing an existing operation in a compatible manner with another server increases the speed with which your users will be able to use the feature in their preferred clients.

## Not in scope

This proposal doesn't address how clients can discover any limits associated with the operations. For example, how many characters are allowed per post, or the number of options that can be included in a poll.

That information is already included in the server's `/api/v2/instance` call (in the language of this proposal, the `org.mastodon.api.instance` operation).

I did consider extending the `operations` definition so that each operation mapped to an object that contained multiple keys, like this:

```json
"operations": {
    "org.joinmastodon.api.statuses.post": {
        "1.0.0": {
            "endpoint": "/api/v1/statuses",
            "limits": {
                "max_characters": 500,
                // ...
            },
            "mimeTypes": ["text/plain"],
            // ...
        },
        "1.1.0": { /* ... */ }
    }
}
```

That would significantly complicate this proposal, increasing the risk that it's not adopted. There's also no clear value in doing this.

## Alternatives considered

### Reporting capabilities alongside operations

It's tempting to think that operations could be broken down in to smaller parts.

For example, instead of different versions for the "post a status" operation you could include more specific capabilities in the operation description:

```json
"operations": {
    ...
    "org.joinmastodon.api.statuses.post": {
        "contentWarning": true,
        "polls": true,
        "media": true,
        ...
    }
    ...
}
```

This indicates this server supports the "post a new status" operation with statuses that include content warnings, polls, and media.

You don't do that because it results in a combinatorial explosion of the different sub-types of operations that clients need to support, without any significant benefit.

Even the example above is incomplete; for example, some servers support including images in content warnings, so a simple boolean for the `contentWarning` property is insufficient.

So treating the thing-that-has-to-be-versioned as the operation (post a status, translate, reblog, etc) seems to be the better level of granularity.

### Reporting capabilities in API responses

A server could include metadata in each response that contains an object that describes the operations that can be performed on that object. For example, the [Status](https://docs.joinmastodon.org/entities/Status/) object could be modified to include an `operations` property that looks like this:

```json
{
  "id": "103270115826048975",
  "created_at": "2019-12-08T03:48:33.901Z",
  ...
  "operations": {
      "org.joinmastodon.api.statuses.reply": ["POST", "https://example.com/api/v1/statuses"],
      "org.joinmastodon.api.statuses.view": ["GET", "https://example.com/api/v1/statuses/103270115826048975"],
      "org.joinmastodon.api.statuses.favourite": ["POST", "http/api/v1/statuses/103270115826048975/favourite"],
      ... etc
   }
  }
}
```

This is the [Hypermedia as the engine of application state (HATEOAS)](https://en.wikipedia.org/wiki/HATEOAS) model.

It's an interesting approach, and a possible future direction. But it would require significant work on the part of server developers to implement as it would affect every response returned by the server.

On the other hand the approach in this proposal is static content in the nodeinfo response. It's significantly easier to implement and iterate on.

### Clients keep a hardcoded server version : capabilities map

This could go the other way, and instead require servers to have a consistent name and parseable version number, and expect clients to keep a map of "server A at version V can perform operations X, Y, and Z".

I think this is the wrong approach for two reasons:

**First**:

1. It requires every client development team to independently maintain a mapping between server versions and capabilities
2. It requires client updates whenever a server is released that supports a capability the client already supports on another server

Re that last point a worked example might make it clearer.

Suppose there are two server types, A and B. A supports operations X and Y, B supports X, Y, and Z.

A client is released which supports operations X, Y, and Z, and is hardcoded with knowledge about which server type supports a given operation.

A new version of server type A is released which now supports operation Z as well. But users of the client who connect to server type A cannot benefit from this until a new version of the client is released with updated information about the capabilities of server type A.

With the proposal in this document this problem does not occur; if a client supports operation Z (at a given version) and a server advertises that it supports that operation then the client can choose to use it without needing a new release.

This is better for our users.

**Second**:

Server developers do not seem to be interested in reporting useful versions for their software, for example:

- Sharkey: [feat: report server version following semver.org requirements (#371) · Issues · TransFem.org / Sharkey · GitLab](https://activitypub.software/TransFem-org/Sharkey/-/issues/371#issuecomment-1543)
- Iceshrimp: [#502 - Please use semver compatible versions - iceshrimp/iceshrimp - Iceshrimp development](https://iceshrimp.dev/iceshrimp/iceshrimp/issues/502#issuecomment-3519)
- Firefish: [Enforce semver for reported Firefish versions (#10844) · Issues · firefish / firefish · GitLab](https://firefish.dev/firefish/firefish/-/issues/10844)
- Mastodon: [Server should refuse to start if configured version is not semver.org compatible · Issue #28843 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/issues/28843)

### Return an OpenAPI definition for the supported API

[OpenAPI](https://swagger.io/specification/) is a popular schema for defining an API. The server could just return the OpenAPI schema for the API that it supports.

I did consider this (an earlier version of this proposal was built around it). But it complicates the data the client needs to process, and includes data that the client will ignore.

Consider the `/api/v1/timelines/home` endpoint, which would have an operation ID something like `org.joinmastodon.api.timelines.home` under this proposal.

This is the OpenAPI definition for that endpoint, copied from the GoToSocial project's OpenAPI definition ([swagger.yaml](https://github.com/superseriousbusiness/gotosocial/blob/main/docs/api/swagger.yaml), the descriptions have been deleted to keep this a reasonable length):

```yaml
    /api/v1/timelines/home:
        get:
            description: |-
                The statuses [... deleted ...]
        operationId: homeTimeline
            parameters:
                - description: [deleted]
                  in: query
                  name: max_id
                  type: string
                - description: [deleted]
                  in: query
                  name: since_id
                  type: string
                - description: [deleted]
                  in: query
                  name: min_id
                  type: string
                - default: 20
                  description: [deleted]
                  in: query
                  name: limit
                  type: integer
            produces:
                - application/json
            responses:
                "200":
                    description: Array of statuses.
                    headers:
                        Link:
                            description: [deleted]
                            type: string
                    schema:
                        items:
                            $ref: '#/definitions/status'
                        type: array
                "400":
                    description: bad request
                "401":
                    description: unauthorized
            security:
                - OAuth2 Bearer:
                    - read:statuses
            summary: See statuses/posts by accounts you follow.
            tags:
                - timelines
```

Most of the information in that definition is redundant *for the client*.

It's absolutely essential information to have for the server developer, and for producing documentation.

But the client should already have this compiled in. The contract between the client and the server is that *if the server reports that it supports the `org.joinmastodon.api.timelines.home` operation at v1.0.0*:

- the endpoint is `/api/v1/timelines/home`
- the valid parameters are `max_id`, `since_id`, `min_id`, and `limit`.
- the response is JSON, encoding an array of `Status`
- there will be pagination details in the `Link` header
- authorization is required

So returning an OpenAPI definition to the client significantly complicates things for no benefit.

OpenAPI is also endpoint-oriented; by which I mean that the definition leads with the endpoint (`/api/v1/statuses`) and then describes the single operation that is present at that endpoint.

This is backwards to what we need, where the operation comes first, and multiple operations might be supported at the same endpoint.

### Use the API path as the key

Instead of the proposed format, use the API path as the map key, like this:

```json
"operations": {
  "/api/some/path": ["org.joinmastodon.x.y.z:1.0.0", "org.joinmastodon.x.y.z:1.1.0", ...]
}
```

I reject this for two reasons:

1. It suggests that servers might decide to implement the same operation under different URL paths. That is unnecessary extra complexity, and per the previous section, the contract between the client and the server is if the server advertises support for operation X the definition of that operation includes that URL path it is served under.
2. It's unfriendly to client developers; a client using this information is trying to answer the question "Does the server support operation X?", not "Does API endpoint /a/b/c exist?". So the operation identifier should be the lookup key for the map, not the endpoint.
## Open questions

These are questions this proposal does not explicitly answer. I have suggestions, but feedback is appreciated.

### Can servers set an API prefix?

While I do not believe servers should be able to arbitrarily change all aspects of the path an operation is served under it may be useful for servers to be able to specify a path prefix for some or all operations.

E.g., a server that supports Mastodon and Friendica APIs might want to offer them at paths that starts `mastodon` and `friendica` respectively.

To do this the map value would be a second object with `prefix` and `versions` keys.

```json
"operations": {
    "org.joinmastodon.api.conversations.id.unread": {
        "prefix": "/mastodon",
        "versions": ["1.0.0"]
    },
    "ca.friendi.api.conversations.id.unread": {
        "prefix": "/friendica",
        "versions": ["1.0.0"]
    }
}
```

If the `prefix` was omitted the default would be `/`.

Adopting this format, even if the only key is `versions` is probably a good idea initially to support future changes in a backwards-compatible manner.
### What happens if an operations "owner" rebrands or disappears?

If a FQDN owner entity rebrands or disappears (e.g., Calckey renamed to Firefish) what happens to the operation identifiers they have defined?

I don't think a rebrand should necessitate a wholesale renaming of the operation identifiers, that's unnecessary churn. They may decide to define future operations using the new name.

Since the operation identifies are not actual domains / URLs there is no requirement for them to actually resolve to anything, so I don't foresee a technical problem with this approach if a project that defined some operation identifiers shuts down. As long as there is still documentation that defines the API for those operations nothing is lost.

### Is the reverse-FQDN-with-suffix approach the best format for operations keys?

I think so.

I did consider using [Uniform Resource Name (URN)](https://en.wikipedia.org/wiki/Uniform_Resource_Name) or [Internationalised Resource Identifiers (IRI)](https://en.wikipedia.org/wiki/Internationalized_Resource_Identifier) . There are advantages; for example, the operation identifier could be a URN or IRI that resolves to a document (perhaps the OpenAPI definition) for the API for that operation.

However, that introduces the difficult topic of case-sensitivity when clients are comparing the list of operations the server returns with the list of operations they support.

If the operation key was a URN or IRI it is, by definition, case-sensitive in parts. This provides an opportunity for server developers to incorrectly report an identifier with the wrong case, and for client developers to look for operations with the wrong case, impeding interoperability.

Suggesting "Clients should case-fold operation identifiers to lower case before comparing them" does not solve the problem, as rules for correctly lower-casing text are not well specified, and may differ from implementation to implementation, again impeding interoperability.

The approach of using a reverse FQDN with additional labels, and ensuring it is encoded with IDN sidesteps those problems.

### Is there a 1:1 mapping between operations and API calls?

Should every supported API call have an associated operation, or is it OK to consider multiple API calls as a single operation the server supports?

I think a 1:1 mapping makes most sense, as it allows server developers to incrementally deploy new features.

For example, at the time of writing Friendica only partially supports voting; posts with votes can be displayed and will be returned by the API, but the operations to create a post with a poll, or to vote on a poll are not implemented.

With a 1:1 mapping beween API and operations a Friendica server can advertise "I can return posts that can contain polls; I do not support API calls that post polls or vote on them".

If "voting" was treated as a single operation then Friendica would be forced to advertise "I do not support polls", and a client might unnecessarily not show polls attached to Friendica posts.

Anything other than a 1:1 mapping could also lead to different server developer teams "bundling" API calls into single operations in different ways, making things more complex for client developers, not less.

### Should clients indicate the operation identifier when making a request?

When a server supports multiple versions of an API at a single endpoint it can be challenging to reliably determine the version of the API the client is calling. Server software often has to resort to "sniffing" the request and deducing the client's intentions by the presence or absence of specific properties in the request.

This makes it more difficult to write the code to process the request and to determine what error details should be returned to the client.

I think trying to solve this problem is outside the scope of this proposal. However, if this proposal is adopted I recommend server developers include the operation ID as a mandatory specific property in all future changes to their API, so this problem is gradually solved.

### Does this need to be a new top-level key in the nodeinfo?

No. This could also be in the metadata, exposed as:

```json
{
  "version": "2.2",
  ...
  "metadata": {
    "operations": {
      // operation data here
    }
  }
}
```


## Related links / prior art

Not an exhaustive list:

- Mastodon Issues
	- [API Documentation via Swagger / RAML / Others · Issue #1404 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/issues/1404)
	- [How to get OpenAPI spec (JSON or YAML) · Issue #16328 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/issues/16328)
	- [OpenAPI documentation generation with rswag specs · Issue #20572 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/issues/20572)
- Mastodon PRs (these all attempted to add OpenAPI definitions)
	- [docs(open-api): Add OpenAPI Specification by oneslash · Pull Request #20000 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/pull/20000)
	- [Feat/add rswag in order to generate verified openapi docs by casaper · Pull Request #20607 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/pull/20607)
	- [[proposal] Machine readable API specification via OpenAPI by takayamaki · Pull Request #25043 · mastodon/mastodon · GitHub](https://github.com/mastodon/mastodon/pull/25043)
- Blog posts
	- [Automatically Generating OpenAPI Docs for Mastodon - DEV Community](https://dev.to/appmap/automatically-generating-openapi-docs-for-mastodon-42ig)
- Mastodon-like servers
	- [GotoSocial OpenAPI definition of the Mastodon API](https://github.com/superseriousbusiness/gotosocial/blob/main/docs/api/swagger.yaml)
	- [[feature] Mastodon-compatible API extensions discovery mechanism · Issue #1985 · superseriousbusiness/gotosocial · GitHub](https://github.com/superseriousbusiness/gotosocial/issues/1985)
- FEPs
	- [FEP 6481 - # Specifying ActivityPub extension support with NodeInfo](https://codeberg.org/fediverse/fep/src/branch/main/fep/6481/fep-6481.md)

## 参考文献

- [NodeInfo](http://nodeinfo.diaspora.software/)
- [RFC-2119 - Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119.html)
- [RFC 3490 - Internationalizing Domain Names in Applications (IDNA)](https://datatracker.ietf.org/doc/rfc3490/)
- [Semantic Versioning](https://semver.org/)
- [OpenAPI Specification](https://swagger.io/specification/)
## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
