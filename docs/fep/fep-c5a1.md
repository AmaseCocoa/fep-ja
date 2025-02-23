---
slug: "c5a1"
authors: Lynn Foster <lynn@mikorizal.org>
status: DRAFT
dateReceived: 2024-01-31
trackingIssue: https://codeberg.org/fediverse/fep/issues/256
discussionsTo: https://codeberg.org/fediverse/fep/issues/256
---
# FEP-c5a1: To-do's
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-c5a1.md)から翻訳に協力することができます。

## Summary

This document describes an implementation of simple to-do's or tasks based on [ActivityPub](https://www.w3.org/TR/activitypub/) protocol and [Valueflows](https://valueflo.ws/) vocabulary.  A to-do is a simple work commitment, and can be created for oneself or another person. Optionally, when the to-do is done, that can be recorded also.

## History

This is a parallel submission to [FEP-3264 Federated Work Coordination](https://codeberg.org/fediverse/fep/src/branch/main/fep/3264/fep-3264.md), which supports much more complex project or production planning. Both planning submissions continue the idea of Valueflows extensions to ActivityPub/ActivityStreams based on use case, started by silverpill with [ FEP-0837 Federated Marketplace](https://codeberg.org/fediverse/fep/src/branch/main/fep/0837/fep-0837.md).

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119](https://tools.ietf.org/html/rfc2119.html).

## Creating a to-do

A to-do is implemented as a Valueflows `Commitment`.

Consuming implementations which don't have planning features MAY display plan-related objects similarly to `Note` objects.

In all cases, the objects can also be discussed using `Note` objects and `replyTo`.

Valueflows defines a commitment is defined as **A planned economic flow that has been promised by an agent to another agent.**

- `id` (REQUIRED): the commitment's unique global identifier.
- `type` (REQUIRED): the type of the object MUST be `Commitment`.
- `attributedTo` (REQUIRED): the actor who published the commitment.
- `published` (RECOMMENDED): the date and time at which the commitment was published.
- `resourceConformsTo` (OPTIONAL): the skill or type of work involved. Could be any URI.
- `effortQuantity` (OPTIONAL): the estimated or expected amount and unit of the work. This is an object with two properties:
  - `hasUnit` (REQUIRED): name of the unit, according to [Ontology of units of Measure](http://www.ontology-of-units-of-measure.org/) classification.
  - `hasNumericalValue` (REQUIRED): amount of the resource.
- `provider` (RECOMMENDED): the actor who commits to providing the resource, including doing the work.  If not included, it is assumed to be the `to` actor.
- `receiver` (RECOMMENDED): the actor who will be receiving the resource.  If not included, it is assumed to be the `attributedTo` actor.
- `content` (RECOMMENDED): the description of what is to be done. The type of content SHOULD be `text/html`.
- `to` (REQUIRED): the audience of the commitment.

Example:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "om2": "http://www.ontology-of-units-of-measure.org/resource/om-2/",
      "vf": "https://w3id.org/valueflows/ont/vf#",
      "Commitment": "vf:Commitment",
      "receiver": "vf:receiver",
      "provider": "vf:provider",
      "resourceConformsTo": "vf:resourceConformsTo",
      "effortQuantity": "vf:effortQuantity",
      "hasUnit": "om2:hasUnit",
      "hasNumericalValue": "om2:hasNumericalValue"
    }
  ],
  "type": "Create",
  "id": "https://project.example/activities/ad2f7ee1-6567-413e-a10b-72650cbdc788/create",
  "actor": "https://project.example/actors/alice",
  "object": {
    "type": "Commitment",
    "id": "https://project.example/todos/ddde9d6f-6f3b-4770-a966-4dkjh8w32e59",
    "attributedTo": "https://project.example/actors/alice",
    "content": "Please proofread the document at https://project.example/docs/45, and let me know what you think could be improved.",
    "published": "2024-05-18T19:22:03.918737Z",
    "resourceConformsTo": "https://www.wikidata.org/wiki/Q834191",
    "effortQuantity": {
      "hasUnit": "hour",
      "hasNumericalValue": "2"
    },
    "provider": "https://project.example/actors/bob",
    "receiver": "https://project.example/actors/alice"
  },
  "to": "https://project.example/actors/bob"
}
```

## Accepting a To-do

Accepting or rejecting a to-do is OPTIONAL for this FEP, although it may be required based on the understanding between the actors.

The `object` of `Accept` activity MUST be the `id` of the `Commitment` object previously sent to the actor.

Activity MAY contain `content` property for further coordination.

Example:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Accept",
  "id": "https://project.example/todos/ddde9d6f-6f3b-4770-a966-9kh93jcmljei",
  "actor": "https://project.example/actors/bob",
  "object": "https://project.example/todos/ddde9d6f-6f3b-4770-a966-4dkjh8w32e59",
  "content": "Should be done within a couple days.",
  "to": "https://project.example/actors/alice"
}
```

## Rejecting a To-do

Accepting or rejecting a to-do is OPTIONAL for this FEP, although it may be required based on the understanding between the actors.

The `object` of `Reject` activity MUST be the `id` of the `Commitment` object previously sent to the actor.

Activity MAY contain `content` property indicating the reason for rejection.

Example:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Reject",
  "id": "https://project.example/todos/8c05f97f-1531-4b70-9ca8-4ee4a09f36a3",
  "actor": "https://project.example/actors/bob",
  "object": "https://project.example/todos/ddde9d6f-6f3b-4770-a966-4dkjh8w32e59",
  "content": "Sorry, my schedule is full for a month.",
  "to": "https://project.example/actors/alice"
}
```

## Recording what actually was done

Recording what was done is OPTIONAL for the FEP, although it may be required by the actors involved.

Actual economic activity is represented with an `EconomicEvent` in Valueflows.  In the case of to-do's, it would only occur in response to an earlier to-do commitment.  More than one `EconomicEvent` can be recorded for one `Commitment` to-do.

The representation of an economic event is a JSON document with the following properties:

- `id` (REQUIRED): the economic event's unique global identifier.
- `type` (REQUIRED): the type of the object SHOULD be `EconomicEvent`. If interoperability with other ActivityPub services is desirable, implementers MAY also use object types from [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/#object-types), such as `Note`.
- `attributedTo` (REQUIRED): the actor who published the economic event.
- `content` (OPTIONAL): the description of the economic event or communication about the economic event. The type of content SHOULD be `text/html`.
- `published` (RECOMMENDED): the date and time at which the economic event was published.
- `to` (REQUIRED): the audience of the economic event.
- `fulfills` (REQUIRED): the commitment the economic event is completely or partially fulfilling.
- `resourceConformsTo` (OPTIONAL): the type of an economic resource (for to-do's, can be a skill or type of work). Could be any URI.  It does not have to match the commitment, but if not included, can be assumed it is the same.
- `effortQuantity` (OPTIONAL): the amount and unit of the work done (for to-do's, usually a time quantity). This is an object with two properties:
  - `hasUnit` (REQUIRED): name of the unit, according to [Ontology of units of Measure](http://www.ontology-of-units-of-measure.org/) classification.
  - `hasNumericalValue` (REQUIRED): amount of the resource.
- `provider` (RECOMMENDED): the actor who commits to doing the work.  If not included, it is assumed to be the `attributedTo` actor.
- `receiver` (RECOMMENDED): the actor who will be receiving the benefit.  If not included, it is assumed to be the `to` actor.
- `finished` (OPTIONAL): set to true if this economic event completes the commitment, or the commitment is no longer open for some reason.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "om2": "http://www.ontology-of-units-of-measure.org/resource/om-2/",
      "vf": "https://w3id.org/valueflows/ont/vf#",
      "EconomicEvent": "vf:EconomicEvent",
      "fulfills": "vf:fulfills",
      "receiver": "vf:receiver",
      "provider": "vf:provider",
      "resourceConformsTo": "vf:resourceConformsTo",
      "effortQuantity": "vf:effortQuantity",
      "hasUnit": "om2:hasUnit",
      "hasNumericalValue": "om2:hasNumericalValue",
      "finished": "vf:finished"
    }
  ],
  "type": "Create",
  "id": "https://project.example/activities/ad2f7ee1-6567-413e-a10b-72650cbdc932/create",
  "actor": "https://project.example/actors/bob",
  "object": {
    "type": "EconomicEvent",
    "id": "https://project.example/inputs/ad2f7ee1-6567-413e-a10b-72650cbdc932",
    "attributedTo": "https://project.example/actors/bob",
    "published": "2024-10-21T14:16:41.843794Z",
    "fulfills": "https://project.example/todos/ddde9d6f-6f3b-4770-a966-4dkjh8w32e59",
    "resourceConformsTo": "https://www.wikidata.org/wiki/Q3485549",
    "effortQuantity": {
      "hasUnit": "hour",
      "hasNumericalValue": "1.5"
    },
    "provider": "https://project.example/actors/bob",
    "receiver": "https://project.example/actors/alice",
    "finished": true
  },
  "to": "https://project.example/actors/alice"
}
```

The `Commitment` can alternatively be marked as `finished`, without recording an `EconomicEvent`.  This could occur in cases where no work will be done, or work was done but will not be recorded, or some work was recorded earlier without marking the to-do `finished`.  So, `finished` does not imply that the to-do was done, only that it is not waiting to be done any more.

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "om2": "http://www.ontology-of-units-of-measure.org/resource/om-2/",
      "vf": "https://w3id.org/valueflows/ont/vf#",
      "Commitment": "vf:Commitment",
      "finished": "vf:finished"
    }
  ],
  "type": "Update",
  "id": "https://project.example/activities/ad2f7ee1-6567-413e-a10b-72650cbdc932/update",
  "actor": "https://project.example/actors/alice",
  "object": {
    "type": "Commitment",
    "id": "https://project.example/todos/ddde9d6f-6f3b-4770-a966-4dkjh8w32e59",
    "attributedTo": "https://project.example/actors/alice",
    "published": "2024-10-24T16:16:41.843794Z",
    "finished": true
  },
  "to": "https://project.example/actors/alice"
}
```

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [Valueflows] Lynn Foster, elf Pavlik, Bob Haugen [Valueflows](https://valueflo.ws/), 2024
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119.html), 1997
- [Activity Vocabulary] James M Snell, Evan Prodromou, [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/), 2017
- [Ontology of units of Measure] Hajo Rijgersberg, Don Willems, Xin-Ying Ren, Mari Wigham, Jan Top, [Ontology of units of Measure](http://www.ontology-of-units-of-measure.org/), 2017

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
