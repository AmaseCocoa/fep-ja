---
slug: "2e40"
authors: Helge Krueger <helge.krueger@gmail.com>
status: WITHDRAWN
dateReceived:  2023-02-13
dateWithdrawn: 2023-09-04
trackingIssue: https://codeberg.org/fediverse/fep/issues/62
discussionsTo: https://codeberg.org/fediverse/fep/issues/62
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-2e40.md)から翻訳に協力することができます。
# FEP-2e40: The FEP Vocabulary Extension Process

## Summary

Current usage of ActivityPub relies on the ActivityStreams namespace [AS-NS]
combined with custom extensions [Mastodon NS]. As far as I can tell, no
best practices exist or a formal process to add new namespaces.

This FEP will

- Create a FEP Vocabulary based on identified best practices
- Define a process to add new entries to this FEP Vocabulary without a risk of Term collision
- Define a process to elevate Terms to be common
- Define a process to create specialized Vocabularies
- Using [FEP-61CE] as an example how this process can be used.

__Note__: Withdrawn as no longer compatible with the FEP repository structure, see [this issue](https://codeberg.org/fediverse/fep/issues/134).

## Background and Terminology

The JSON-LD context is introduced in [3.1 The Context](https://www.w3.org/TR/json-ld/#the-context) of [JSON-LD]. The context of an object is specified by its `@context` property.

One can think of the context as defining certain strings to be equivalent. For example `Note`, `as:Note`, and `https://www.w3.org/ns/activitystreams#Note` all represent the same thing. More details can be found in [3.2. IRIs](https://www.w3.org/TR/json-ld/#iris). Following [JSON-LD], we will refer to all three strings mentioned above as a __Term__. The focus here is on having a format that is _machine-readable_. Having a standardized format allows various algorithms to be applicable see [Canon] and [LD-API].

The second useful aspect of this is that one can define the used terms through the provided URL: [https://www.w3.org/ns/activitystreams#Note](https://www.w3.org/ns/activitystreams#Note). Clicking on it will let you easily find the definiton of the [Note Type](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-note). This format is _human-readable_.

We will refer to the combination of Context and easily accessible documentations for the terms a __Vocabulary__.

## Requirements

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this specification are to be interpreted as described in [RFC-2119].

## The FEP Vocabulary

We now formalize the process of creating an adding to the FEP Vocabulary. This section formalizes the location of the two files containing the _machine readable_ json-ld context and _human readable_ markdown description. How to add new terms will be discussed in the next section.

This FEP creates the file `/feps/assets/fep-2e40/namespace.json` with content

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "fep": "https://w3id.org/fep#"
    }
  ]
}
```

A description of this context is contained in `/feps/assets/fep-2e40/namespace.md`. The goal of inclusion of the ActivityStreams context is to increase the chances of most objects having the simple form

```json
{
  "@context": "https://w3id.org/fep",
  ...
}
```

which is as close as one can get to pure JSON with JSON-LD. These two files form the __FEP Vocabulary__. Through a pull-request to [W3-ID], these two files will be made available at `https://w3id.org/fep`.

## Process to add a new Term to the FEP Vocabulary

The authors of a new FEP-xxxx following [FEP-a3ed] MAY include a section or sections titled "Create FEP Term: $TERM". The $TERM MUST follow the following rules inspired by points 2-5 of [3. The Registration Process](https://www.w3.org/TR/did-spec-registries/#the-registration-process) in [DID-Reg].

1. The $TERM MUST be indicative of its function. Avoid generic terms such as "myProperty" or "foo".
2. If there are copyright, trademark, or any intellectual property rights concerns, the addition and use MUST be authorized in writing by the intellectual property rights holder under a F/RAND license. Examples include terms that use trademarked brand names, property names that utilize the titles of copyrighted works, and patented technology that would cause the use of the extension to require licensing a patent.
3. Any addition MUST NOT create unreasonable legal, security, moral, or privacy issues that will result in direct harm to others. Examples of unacceptable additions include any containing racist language, technologies used to persecute minority populations, and unconsented pervasive tracking.

Furthermore, the author MAY perform the following action

1. Add a new term of the form `fep-xxxx-$TERM` to `/feps/assets/fep-2e40/namespace.json`.
2. Add a description for this term to `/feps/assets/fep-2e40/namespace.md`.

The changes to the two files MUST be described in the FEP, and they MUST only add new content or change content added by the FEP.
An example is provided in the next section. While these actions are optional at the DRAFT stage of a FEP, they MUST be performed before a FEP can reach its FINAL stage. We will refer to a term introduced by this process as a __FEP term__.

Instead of having many sections with similar names, the author of a FEP MAY combine them.

Instead of defining a new _FEP term_, the author of a  FEP MAY decide to include terms from another namespace as a FEP Term. For example, we decide to introduce `conversation` as follows

```json
 "fep-xxxx-conversation": {
    "@id": "http://ostatus.org#conversation",
    "@type": "@id"
 },
```

This has the advantage that terms, which are currently used but not properly documented, can be given a proper definition. Furthermore, this will help keep the `@context` block small. When doing so, the author MUST either provide a reference to the current definition of the object OR write a new definition of the object. If a reference is provided, it should be considered best pratice to quote the reference.

## Example "Create FEP term: eventSource"

In [FEP-61ce], I plan to introduce the term `eventSource`. As this will most likely be the first FEP using the herein defined procces, this will lead to `/feps/assets/fep-2e40/namespace.json` being changed to

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "fep": "https://www.w3id.org/fep#",
      "fep-61ce-eventSource": {
        "@id": "fep:fep-61ce-eventSource",
        "@type": "@id"
      }
    }
  ]
}
```

with usage information

> The term `eventSource` is to be as part of the `endpoints` of an [ActivityPub] Actor. It specifies an endpoint, where the Client can receive push notifications using the Server Side Events protocol of activities being added to collections on the server. By default the inbox collection of the Actor is used. By specifying the `X-ActivityPub-Collection` header a different collection can be specified to retrieve push notifications from.
>
> Example usage:
>
> ```json
> {
>  "@context": ["https://www.w3id.org/fep",
>      {"eventSource": "fep-61ce-eventSource"}
>   ],
>  "type": "Person",
>  "id": "https://example.com/client_actor",
>  "inbox": "https://example.com/client_actor/inbox",
>  "outbox": "https://example.com/client_actor/outbox",
>  "preferredUsername": "actor",
>  "endpoints": {
>    "proxyUrl": "https://example.com/client_actor/proxyUrl",
>    "eventSource": "https://example.com/client_actor/eventSource"
>  }
> }
> ```
>

## Promotion to common terms

While the above example is already nice looking, it still has the uglyness of needing to define `eventSource` as `fep-61ce/eventSource`. We will call the process of adding the line

```json
"eventSource": "fep-61ce-eventSource"
```

to `/feps/assets/fep-2e40/namespace.json` a __promotion to common term__.

In order to be elligible for promotion, a term MUST NOT conflict with any term currently in the vocabulary. For example, if the term `visualization` was already introduced in a previous FEP, it is not acceptable to introduce `visualisation` and expect promotion of this term.

Any term elligible for promotion CAN be promoted to a common term by the author of the FEP after the FEP has been declared FINAL. If an author does not seek to promotion of a term, it SHOULD be noted in the section introducing this term.

## Secondary FEP Vocabularies

We have now introduced a process for the common or primary vocabulary with context in `/feps/assets/fep-2e40/namespace.json` and documentation in `/feps/assets/fep-2e40/namespace.json`. We will now introduce _secondary vocabularies_. These are meant to be an option to introduce many domain specific terms without having to add them to the primary vocabulary.

An author of a FEP MAY include a section titled "Introducing Secondary FEP Vocabulary fep-DOMAIN". Here DOMAIN is a short name representing the domain. Then files `/feps/fep-xxxx/namespace.json` and `/feps/fep-xxxx/namespace.md` can be created. An example of a file of the `@context` file is given below. By including the original `fep` context, one can keep duplication to a minimum. Similarly, the documentation file should only explain the namespace and which additional terms are promoted or overridden. For the documentation of the _fep terms_, one should refer back to the original documentation.

Furthermore, the author MAY adjust the configuration of [W3-ID] to reflect his changes. All obligations that are MAY during the draft stage will turn into MUST before the FEP can be finalized.

## Example: The barber vocabulary

The barber community in the FediVerse has decided that, they need a marker if people shave themself. So they have introduce FEP-ba1b and defined the term `shavesHimself`. It was decided not to promote this term to the primary vocabulary. Instead the barber community has decided to introduce FEP-ba2b, in which they introduce the secondary FEP vocabulary `fep-barber`. The context of this vocabulary then takes the form:

```json
{
  "@context": [
    "https://www.w3id.org/fep",
    {
      "shavesHimself": "fep-ba1b-shavesHimself"
    }
  ]
}
```

By including the primary FEP vocabulary, it is ensured that changes to the primary vocabulary carry over to the secondary `fep-barber` vocabulary. Furthermore, by the promotion of `shavesHimself` taking place after the inclusion of the primary vocabulary, it is ensure that the primary vocabulary cannot override it.

## Acknowledgements

The comments by _aschrijver_ and _trwnh_ on SocialHub have helped me greatly in clarifying and formulating the concepts discussed in this FEP.

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [AS-NS] Amy Guy [ActivityStreams 2.0 Terms](https://www.w3.org/ns/activitystreams), 2018
- [AS-Git] M. Sporny  [Add alsoKnownAs property to AS vocabulary #511](https://github.com/w3c/activitystreams/issues/511), 2020
- [Canon] A. Rundgren, B. Jordan, S. Erdtman [RFC 8785 JSON Canonicalization Scheme (JCS)](https://www.rfc-editor.org/rfc/rfc8785), 2020
- [DID Reg] Orie Steele, Manu Sporny [DID Specification Registries](https://www.w3.org/TR/did-spec-registries), 2023
- [FEP-a4ed] pukkamustard [FEP-a4ed: The Fediverse Enhancement Proposal Process](fep-a4ed.md)
- [FEP-61ce] Helge Krueger [FEP-61CE: Server Sent Events for Actor], in preparation
- [JSON-LD] Gregg Kellogg, Pierre-Antoine Champin, Dave Longley [JSON-LD](https://www.w3.org/TR/json-ld/), 2020
- [LD-API]  Gregg Kellogg, Pierre-Antoine Champin, Dave Longley [JSON-LD-API](https://www.w3.org/TR/json-ld-api/), 2020
- [Mastodon NS] Eugen Rochko et al [JSON-LD Namespacing](https://docs.joinmastodon.org/spec/activitypub/#namespaces)
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119.html)
- [W3-ID] Contibutors [Permanent Identifiers for the Web](https://github.com/perma-id/w3id.org)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
