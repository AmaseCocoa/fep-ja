---
type:
- TestCase
status: draft
name: Actor Objects must express signing key as assertionMethod Multikey
slug: fep-521a-actor-objects-must-express-signing-key-as-assertionmethod-multikey
description: |
  This rule checks whether a given Actor Object has a Multikey object in top-level assertionMethod property of the shape specified in FEP-521a.
uuid: 36f73f6e-8c14-4606-864d-32b9a74abc87
attributedTo:
- https://bumblefudge.com
exampleImplementation: https://codeberg.org/socialweb.coop/activitypub-testing-fep-521a

"@context":
- TestCase:
    "@id": http://www.w3.org/ns/earl#:TestCase
  type:
    "@type": "@id"

respec:
  config:
    editors:
    - name: bumblefudge
      url: "https://bumblefudge.com"
      w3cid: 143155
    latestVersion: https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a-test-case.md

---

# Actor Objects must express signing key as assertionMethod Multikey
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-521a-test-case.md)から翻訳に協力することができます。

## Background

[FEP-521a][] defines how Actor object MUST express the public key for its signing key in the `assertionMethod` property, as a Multikey object of a given shape and `id` URI shape.

## About this Test

This is a Test Case testing conformance with the one behavior specified in FEP-521a.

## Test Subject

The subject of this test is any data claiming to conform to the specification of an ActivityPub Actor Object and to the FEP-521a extension thereof.

This test is *not* directly applicable to an ActivityPub Server.
An ActivityPub Server serves 0 or more Actor Objects.
An ActivityPub Server for a big community might serve hundreds of ActivityPub Actor Objects.
An ActivityPub Server for a single human may serve only that person's ActivityPub Actor Object.

This test applies to Actor *Objects*, but *not* all Actor Objects are addressable by an HTTPS URL.
The URI that addresses an Actor Object is not the same as the Actor Object.
A given URL may resolve to different Actor Objects in different contexts, and a given Actor Object may not be universally addressable across context by any one URL.

## Inputs

This test requires the following [inputs](https://www.w3.org/TR/act-rules-format/#input):

* `actor` - the actor object under tested
  * type: binary data
  * constraints
    * will be interpreted as JSON.
      * If not parseable as JSON, the test result MUST be `inapplicable`.
    * must be an actor as per core AP test e7ee491d-88d7-4e67-80c8-f74781bb247c
      * i.e. has inbox and outbox
      * If not actor, the test result outcome MUST be `inapplicable`.
    * must contain a non-empty `assertionMethod` array
      * if does not, the test result outcome MUST be `inapplicable`.

## Applicability

This test applies directly to the `actor` input.

* If `actor` is not an Actor Object, the outcome MUST be `inapplicable`.
* If `actor` is not a JSON object, the outcome MUST be `inapplicable`.
* If `actor` JSON does not have a `type` property, the outcome MUST be `inapplicable`.
* If `actor` JSON does not have an `assertionMethod` property, the outcome MUST be `inapplicable`.

### Test Targets

* each entry in `assertionMethod` array is a distinct test target, referred to below as `assertionMethod[x]`
* each entry should be a JSON object; each entry that is not is inapplicable (warning)
* each entry not typed as `Multikey` is inapplicable (warning)
* each entry typed as `Multikey` passes or fails the tests of its validity as a Multikey

## Expectations

1. `assertionMethod[x].id` - MUST be a string
2. `assertionMethod[x].controller` - MUST match the `id` property, if present, of the `assertionMethod` array's parent (i.e., the Actor object)
3. `assertionMethod[x].publicKeyMultibase` - MUST be a [base58btc-encoded](https://www.w3.org/TR/controller-document/#multibase-0) and appropriately-prefixed (in this case, beginning with `z`) expression of a binary public key expression

## Assumptions

### 1. How to Determine Whether Actor Object is expressing a key in the FEP-defined shape

For the purposes of determining whether the input `actor` is expressing a key in the shape defined by the FEP:

* the input is valid JSON
* the input, once parsed as JSON
  * has an `assertionMethod` property, containing an array of 0 or more objects
  * if present, each object therein...
    * has an `id` property, containing a string
    * has a `type` property, containing a string
    * has a `controller` property, containing a string
    * ...has a `publicKeyMultibase` property, containing a string

### 2. Property value expectations

For the purposes of determining whether the input `actor` is expressing 1 or more keys all the following must be true:

* the input is valid JSON
* each member of the `assertionMethod` array, once parsed as JSON, ...
  * has an `id` property, which is a URI
  * has an `assertionMethod.id` property, which is a string unique within the actor object
  * has an `assertionMethod.type` property whose value is the string "Multikey" or an Array containing the string "Multikey"
  * has an `assertionMethod.controller` property whose value is a string identical to the parent object's `id` property value
  * has an `assertionMethod.publicKeyMultibase` property equal to a string of [base58btc-alphabet](https://datatracker.ietf.org/doc/html/draft-msporny-base58-03) characters, i.e. matching the regular expression `[1-9A-HJ-NP-Za-km-z]+`

## Test Cases

These are test cases for this test case, and can be used to verify that an implementation of this test case specification will be [consistent](https://www.w3.org/WAI/standards-guidelines/act/implementations/#understanding-act-consistency) with other implementations.

### Missing assertionMethod

input

actor:

```json
{
  "type": "Person",
  "inbox": "https://example.com/inbox",
  "outbox": "https://example.com/outbox"
}
```

test result

* outcome: `inapplicable`

### Misshapen assertionMethod Array

input

actor:

```json
{
    "type": "Person",
    "inbox": "https://example.com/inbox",
    "outbox": "https://example.com/outbox",
    "assertionMethod": "z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2"
}
```

test targets

* none
  * outcome: `inapplicable`, warning (malformed assertionMethod array)

### Misshapen assertionMethod Members

input

actor:

```json
{
    "type": "Person",
    "inbox": "https://example.com/inbox",
    "outbox": "https://example.com/outbox",
    "assertionMethod": [{
        "inappropriateKey": "z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2"
    }]
}
```

test targets

* none
  * outcome: `inapplicable`, warning (malformed assertionMethod member)

### Malformed publicKeyMultibase Value

input

actor:

```json
{
    "type": "Person",
    "inbox": "https://example.com/inbox",
    "outbox": "https://example.com/outbox",
    "assertionMethod": {[
        "id": "https://example.com/#ed25519-key",
        "type": "Multikey",
        "controller": "https://example.com/",
        "publicKeyMultibase": "6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2"
    ]}
}
```

test targets

* actor.assertionMethod[0]
  * outcome: `inapplicable`, warning (malformed assertionMethod member)

### Valid Actor

input

* `actor`:

    ```json
    {
    "type": "Person",
    "inbox": "https://example.com/inbox",
    "outbox": "https://example.com/outbox",
    "id": "https://example.com/",
    "assertionMethod": [
        {
            "id": "https://example.com/#ed25519-key",
            "type": "Multikey",
            "controller": "https://https://example.com/",
            "publicKeyMultibase": "z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2"
        },
        {
            "inappropriateKey": "z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2"
        }
    ]
    }
    ```

test targets

* `assertionMethod[0]`
  * outcome: `passed`
* `assertionMethod[1]`
  * outcome: `inapplicable`, warning (non-conformant entries)

## Glossary

### `outcome`

An outcome is a conclusion that comes from evaluating a test on a test subject.
An outcome can be one of the three following types:

* `inapplicable`: No part of the test subject matches the applicability
* `passed`: A test target meets all expectations
* `failed`: A test target does not meet all expectations

[FEP-521a]: https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md

## Requirements Mapping

* FEP requirement: - Actor Objects must express signing key as assertionMethod Multikey
  * Required for Conformance to [FEP-521a][FEP-521a]
  * Outcome Mapping
    * when test target `assertionMethod` has outcome `passed`, requirement is satisfied
    * when test target `assertionMethod` has outcome `failed`, requirement is not satisfied
    * when test target `assertionMethod` has outcome `inapplicable`, further testing is needed to determine whether this requirement is satisfied

## Change Log

* 2024-06-25T00:00:00.000Z - implemented as a [free-standing package](https://codeberg.org/socialweb.coop/activitypub-testing-fep-521a) and submitted to FEP repo
* 2024-04-11T21:41:20.725Z - first draft by bumblefudge

## Issues List
