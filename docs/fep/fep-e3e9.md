---
slug: "e3e9"
authors: Dmitri Zagidulin <@dmitri@social.coop>, bumblefudge <bumblefudge@learningproof.xyz>
status: DRAFT
dateReceived: 2024-06-29
trackingIssue: https://codeberg.org/fediverse/fep/issues/346
discussionsTo: https://codeberg.org/fediverse/fep/issues/346
---
# FEP-e3e9: Actor-Relative URLs
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-e3e9.md)から翻訳に協力することができます。

## Summary

> "All problems in computer science can be solved by another level of indirection" (the "fundamental theorem of software engineering")

-- Attributed to: Butler Lampson ([src](https://en.wikipedia.org/wiki/Indirection))

This FEP introduces an ID scheme for ActivityPub objects and collections that
has the following properties:

* IDs remains stable across domain migrations. That is, allows the controller of
  the objects to change object hosting providers without changing the object IDs.
* IDs are regular HTTP(S) URLs that are resolvable via an HTTP `GET` request
  (provided the client allows following `302` redirects).

The proposed mechanism identifies objects by adding query parameters to existing
Actor profile URLs. ActivityPub clients wishing to fetch the objects make an
HTTP `GET` request to this URL, as usual, carrying whatever authentication
mechanism is required currently, and then follow the HTTP `302` status code
redirect in the response to the current storage location of the object.

Example Actor-Relative URL:

`https://alice-personal-site.example/actor?service=storage&relativeRef=/AP/objects/567`

An AP client, encountering an Object ID with this URL makes an HTTP `GET` request
just as it would with any other Object ID:

```http
GET /actor?service=storage&relativeRef=/AP/objects/567 HTTP/1.1
Host: alice-personal-site.example
```

The server responds with a `302` redirect (which all HTTP clients are able
to automatically follow) pointing to the current storage location of the object.
For example:

```http
HTTP/1.1 302 Found
Location: https://storage-provider.example/users/1234/AP/objects/567
```

This redirection mechanism is enabled in all existing HTTP clients by
default (see https://developer.mozilla.org/en-US/docs/Web/API/Request/redirect),
and requires no additional re-tooling of ActivityPub client code.

## Actor-Relative URLs for Objects and Collections

On the Client side, the main change required is in the author/controller validation
procedure (since retrieving the objects at Actor-Relative URLs requires no
additional change beyond ensuring that following HTTP redirects is not disabled).

On the Server side (specifically, the server hosting the Actor profile), two
changes are required:

* (Data Model change) Adding a `service` section to the Actor profile, which
  is required for author/controller validation.
* (Protocol change) Enabling http `302` redirect responses when an Actor profile
  request is made that has the required query parameters (`service` and
  `relativeRef` params).

In addition:

* (Not required but recommended) Implementing [FEP-8b32: Object Integrity
  Proofs][FEP-8b32] is recommended, since it helps with author/controller
  validation even in the case that the Actor profile host is down or otherwise
  unavailable.

### Validating an Object's Author/Controller

Given the following example Actor profile:

```js
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://www.w3.org/ns/did/v1"
  ],
  "service": [{
     "id": "https://alice-personal-site.example/actor#storage",
     "serviceEndpoint": "https://storage-provider.example"
  }],
  // Rest of the Actor profile goes here
}
```

When fetching an ActivityPub Object or Collection identified by an Actor-Relative
URL (that is, when the Object or Collection ID contains the URL query parameters
`service` and `relativeRef`), a client MUST validate that the server hosting
the Object is authorized by the Actor profile:

1. The Client performs an HTTP `GET` request on the Object or Collection, as
   usual, including any currently required authorization headers.
2. The client performing the `GET` request MUST be able to support HTTP
   redirection. For example, if using the WHATWG `fetch` API, the request's
   `redirect` property cannot be set to `error`.
3. The Client follows the redirect and automatically fetches the object specified
   in the `Location` header of the `302` response (this behavior is the default
   in most HTTP clients).
4. The Client extracts the _current URL_ of the Object. This is the URL specified
   in the `Location` header of the redirect response; for example, if using
   the WHATWG `fetch` API, this is the last URL in the response's URL list,
   retrievable by accessing `response.url`.
5. The Client retrieves the Actor profile corresponding to this Object's author/
   controller (the `actor` or `attributedTo` property).
6. The Client extracts the value of the _authorized storage endpoint_ from the
   profile:

    a. The Client checks to see if the Actor profile contains the `service`
       property.
    b. If the `service` property is found, the Client searches through the
       array of service endpoints until it finds a service endpoint with the
       relative id ending in `#storage` (note: this is what the `service=storage`
       query parameter refers to, in the Actor-Relative URL). The Client extracts
       the `serviceEndpoint` property of this service description object.
       This is the _authorized storage endpoint_.
    c. If no _authorized storage endpoint_ is specified in the Actor profile
       (that is, if the Actor profile does not contain the `service` property,
       or if the `service` property is `null` or an empty array, or if the
       `service` array does not contain a service endpoint object with a relative
       `id` that ends in `#storage`, or if that service endpoint does not contain
       a `serviceEndpoint` property containing a URL), the Client SHOULD
       indicate to the user that the provenance of this Object cannot be determined,
       or that the storage location of the Object has not been authorized by
       the profile of the claimed author/controller.

7. The Client MUST validate that the _current URL_ of the object is authorized
   by the Actor's profile by checking that:

    a. The Object's _currentURL_ starts with the value of the _authorized storage
       endpoint_.
    b. The Object's _currentURL_ ends with the value of the `relativeRef` query
       parameter.
    c. For example, in JS pseudocode, using string concatenation:
       `response.url === (authorizedStorageEndpoint + query.relativeRef)`
    d. If these checks fail (if the _current URL_ of the object is not equal to
       the string concatenation of the _authorized storage endpoint_ and the
       `relativeRef` query parameter), the Client SHOULD
       indicate to the user that the provenance of this Object cannot be determined,
       or that the storage location of the Object has not been authorized by
       the profile of the claimed author/controller.

This validation procedure establishes a two-way link: from the Object to its
author/controller Actor profile (via the Object's `actor` or `attributedTo`
property), and from the Actor profile to the authorized storage service provider,
at whose domain the Object is currently stored.

### Client-Side Implementation

An ActivityPub client conforming to this FEP:

1. When encountering an Actor-Relative URL as an ID of an object, fetch it using the same
   HTTP `GET` mechanism that it currently does.
   * Note: An Actor-Relative URL is defined as a URL containing the `service` and
     `relativeRef` query parameters.
2. The client MUST follow the `302` redirect in the response.
3. The client MUST perform the validation steps outlined in the [Validating an Object's
   Author/Controller]() section above.

### Server-Side Implementation

On the server side (specifically, the server hosting the Actor profile), an ActivityPub
server conforming to this FEP:

1. For every request to the Actor profile object (for example, to 
  `https://alice-personal-site.example/actor`), examine the HTTP QUERY parameters.
  If the `service` and `relativeRef` query parameters are present in the request,
  treat this as an _Actor-Relative URL Request_ (by following the steps below).
2. Examine the Actor profile object for this request. If the profile does not contain
   a valid `serviceEndpoint` that corresponds to the `service` query parameter,
   the server MUST return a `422 Unprocessable Entity` HTTP status code error.
   To determine whether the profile contains a valid service endpoint:

   * If the Actor profile does not contain a top level `service` property: INVALID
   * If the Actor has a `service` property, but its value is `null` or `[]`: INVALID
   * Search through the array of service endpoints (the value of the `service`) property,
     until you find a service object with the id that ends in
     `<actor profile url>#<contents of the 'service' query param>`. See sample Actor
     profile and request below. If no valid service endpoint is found: INVALID

3. Assuming that a matching service endpoint is found, compose a _current location URL_
   from the `serviceEndpoint` contained in the profile concatenated with the contents
   of the `relativeRef` query parameter (see below for example).

4. Return a `302 Found` HTTP status code response, and set the `Location` response header
   to the value of the _current location URL_ composed in the previous step.
   Note: Servers SHOULD NOT return a `301` status response (a 301 response implies a
   _permanent_ relocation, and the whole point of this FEP is that Actor-Relative URLs are
   changeable at any point). Similarly, servers SHOULD not return a `303 See Other` status
   response.

### Example Server-Side Request and Response

Example request URL:

```
GET https://alice-personal-site.example/actor?service=storage&relativeRef=/AP/objects/567
```

The query parameters would be parsed on the server side as something similar to:

```json
{ "service": "storage", "relativeRef": "/AP/objects/567" }
```

Example Actor profile at that URL:

```js
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://www.w3.org/ns/did/v1"
  ],
  "service": [{
     "id": "https://alice-personal-site.example/actor#storage",
     "serviceEndpoint": "https://storage-provider.example"
  }],
  // Rest of the Actor profile goes here
}
```

Example _current location URL_ (from concatenating the `serviceEndpoint` value with the
`relativeRef` query parameter): `https://storage-provider.example/AP/objects/567`

Example response from the server:

```http
HTTP/1.1 302 Found
Location: https://storage-provider.example/AP/objects/567
```

## Object Storage Migration Using Actor-Relative URLs

Actor-Relative URLs can be used as an option for portable Object and Collection IDs that
remain unchanged even through migrating to a different object hosting provider (as long
as the Actor ID remains constant).

### Example Storage Provider Migration

Before migration, Alice uses the `https://old-storage-provider.example` as a
storage provider for her AP objects. She makes sure `https://old-storage-provider.example`
is specified as a service endpoint in her Actor profile.

`GET https://alice-personal-site.example/actor`

returns

```js
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://www.w3.org/ns/did/v1"
  ],
  "id": "https://alice-personal-site.example/actor",
  "type": "Person",
  "service": [{
     "id": "https://alice-personal-site.example/actor#storage",
     "serviceEndpoint": "https://old-storage-provider.example"
  }],
  "assertionMethod": { /* … */ },
  // All the other profile properties …
}
```

Alice then creates a Note and stores it with the storage provider (making sure to
add an Object Identity Proof). Example request:

```HTTP
POST /AP/objects/
Host: old-storage-provider.example

{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Note",
  "content": "This is a note",
  "attributedTo": "https://alice-personal-site.example/actor",
  "id": "https://alice-personal-site.example/actor?service=storage&relativeRef=/AP/objects/567"
}
```

returns

```HTTP
HTTP 201 Created
Location: https://old-storage-provider.example/AP/objects/567
```

Note that this created Object can now be fetched at TWO different URLs:

1. The direct URL (also called _current location URL_), 
   `https://old-storage-provider.example/AP/objects/567`
2. The indirect Actor-Relative URL
   `https://alice-personal-site.example/actor?service=storage&relativeRef=/AP/objects/567`

When it comes time to migrate to a different service provider, the new one being
located at `https://brand-new-storage.example`, Alice performs the following steps.

She updates her Actor profile service endpoint, to point to the new provider,
so that it looks like this:

```js
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://www.w3.org/ns/did/v1"
  ],
  "id": "https://alice-personal-site.example/actor",
  "type": "Person",
  "service": [{
     "id": "https://alice-personal-site.example/actor#storage",
     "serviceEndpoint": "https://brand-new-storage.example"
  }],
  "assertionMethod": { /* … */ },
  // All the other profile properties …
}
```

Note that the `serviceEndpoint` is the only property in the Actor profile that
has to change during migration.

Alice then transfers her Object to the new provider (for this example, she'll be
transferring the object individually, though in future FEPs, we expect
specification of APIs to transfer _all_ of the objects in one's storage):

```HTTP
POST /AP/objects/
Host: brand-new-storage.example

{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Note",
  "content": "This is a note",
  "attributedTo": "https://alice-personal-site.example/actor",
  "id": "https://alice-personal-site.example/actor?service=storage&relativeRef=/AP/objects/567"
}
```

returns:

```HTTP
HTTP 201 Created
Location: https://brand-new-storage.example/AP/objects/567
```

Notice that the object being stored at the new provider is byte-for-byte
identical to the object hosted at the old provider; its _indirect_ `id` and
contents do not change.

Throughout this service provider migration, the external indirect `id` of the
object _does not change_, for the purposes of all other AP mechanisms such as
Inbox delivery, Likes and Reposts, and so on.

## References

* [FEP-8b32: Object Integrity Proofs][FEP-8b32]

* Christine Lemmer Webber, Jessica Tallon, [ActivityPub][AP], 2018
* S. Bradner, Key words for use in RFCs to Indicate Requirement Levels, 1997

[FEP-8b32]: https://codeberg.org/fediverse/fep/src/branch/main/fep/8b32/fep-8b32.md

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement
Proposal have waived all copyright and related or neighboring rights to this work.
