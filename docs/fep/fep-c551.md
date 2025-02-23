---
slug: "c551"
authors: bengo <ben@bengo.co>
status: DRAFT
dateReceived: 2024-07-11
trackingIssue: https://codeberg.org/fediverse/fep/issues/366
discussionsTo: https://codeberg.org/fediverse/fep/issues/366
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-c551.md)から翻訳に協力することができます。

# FEP-c551: Use ECMAScript Modules to Create Conformance Tests for Fediverse Enhancement Proposals

## Summary

This is a proposal to enhance the fediverse by creating test cases for FEPs as ECMAScript Modules.

<!-- TOC -->

## Contents

* [Context](#context)
* [Test Specifications](#test-specifications)
* [Test Modules](#test-modules)
* [Test Objects](#test-objects)
* [Test Functions](#test-functions)
* [Test Inputs](#test-inputs)
* [Test Results](#test-results)

<!-- section break -->

## Context

[FEP-d9ad][] proposes to Create Conformance Tests for Fediverse Enhancement Proposals, and specifies components that all Conformance Tests may use and describe in their Test Specifications. It *does not* specify a format for implementing FEP-d9ad Conformance Tests in any programming language.

This FEP-c551 proposes to supplement human-readable FEP-d9ad Conformance Tests with implementations of the test as [Test Objects](#test-objects) exported from [Test Modules](#test-modules). Each [Test Object][] has a `run` function parameterized by a [Test Input] and returning a Promise of a [Test Result][].

## Overview

When a tester comes up with a new test for a FEP, they create a human-readable [Test Specification](#test-specifications) describing how to test whether some subject conforms to the FEP.

ECMAScript developers implement Test Specifications as automatable code by using ECMAScript to create [Test Functions][] that execute the test logic and [Test Objects][] that group the Test Function with more info like the test's name, required input, and possible outcomes. Test Objects are distributed in ECMAScript Modules published on the web, e.g. in `.js` or `.mjs` files.

Testers invoke the Test Function once for each Test Input, await any returned Promises, and receive a [Test Result][] describing the `outcome` of running the test.

## Test Specifications

Test Specifications are human-readable documents that specify the behavior of a test.

Test Specifications SHOULD include Conformance Test Component specifications from [FEP-d9ad][].

An example of a test specification is [fep-521a-test-case.md](https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a-test-case.md).

## Test Modules

FEP testers MAY publish implementations of their test specifications as an [ECMAScript Module][] following the recommendations in this proposal. Such modules may be referred to as Test Modules.

Test Modules SHOULD have no imports. This is to ensure portability of the test modules.

Test Modules MAY export a default export object that is a Test Object

Test Modules SHOULD be resilient to being parsed and evaluated in various ECMAScript runtimes (e.g. node.js or a web browser like Firefox).

An example of a test module can be found [in activitypub-testing-fep-521a](https://codeberg.org/socialweb.coop/activitypub-testing-fep-521a/src/branch/main/fep/521a/actor-objects-must-express-signing-key-as-assertionMethod-multikey.js).

### Example Test Module

```javascript
export default {
  name: 'invalid script module name',
  run: (input) => ({ outcome: 'passed' }),
  type: ['https://w3id.org/fep/c551#Test'],
  '@context': ["https://www.w3.org/ns/activitystreams"],
}
```

## Test Objects

Test Objects are ECMAScript Objects that represent a named, runnable test, e.g. a test specified by a [FEP-d9ad Conformance Test][].

Test Objects MUST have a property named `type` whose value is either the string `https://w3id.org/fep/c551#Test` or  an Array containing that string.

Test Objects MUST have a property named `name` whose value is a string.

Test Objects SHOULD have a property named `@context` whose value is an Array containing `https://www.w3.org/ns/activitystreams`.

An example of a test object is [exported in activitypub-testing-fep-521a](https://codeberg.org/socialweb.coop/activitypub-testing-fep-521a/src/commit/b6e49fd5f490b05f04a958f5f3c5c584e66f592b/fep/521a/actor-objects-must-express-signing-key-as-assertionMethod-multikey.js#L38).

Test Objects SHOULD have a property named `run` whose value is a [Test Function][]].

## Test Functions

Test Object `run` calls SHOULD return a `Promise` that resolves to a [Test Result][].

Test Object `run` functions SHOULD be resilient to being run in various ECMAScript runtimes (e.g. node.js or a web browser like Firefox).

## Test Inputs

A Test Input is the first parameter to a test's `run` function.

Test Input MUST be an object. A test with several logically distinct inputs should give each input a name, and add each named input as a property within a top-level input object.

Test Input values SHOULD conform to the specification of the called test's [Input](https://bengo.is/fep/d9ad/#input) spec.

## Test Results

[Test Results][] MUST have a property named `outcome` whose value is a string.

Test Results SHOULD have a property named `info` whose value is a string.

Test Results MAY have a property named `pointer` that contextualizes the `outcome`, e.g. an object with a property for each value that led to the `outcome`. For example, if a test outcome is `failed` because some number was too low, you can set the result `info` to "number too low" and `pointer` to `{ number: 100 }`.

<!-- section break -->

<section id="conformance">
Conformance requirements are indicated by sentences containing MUST a la <a href="https://datatracker.ietf.org/doc/html/rfc2119">RFC2119</a>.
</section>

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.

[ECMAScript Module]: https://tc39.es/ecma262/#sec-modules
[Test Module]: #test-modules
[Test Modules]: #test-modules
[Test Inputs]: #test-inputs
[Test Result]: #test-results
[Test Results]: #test-results
[Test Object]: #test-objects
[Test Objects]: #test-objects
[Test Function]: #test-functions
[Test Functions]: #test-functions
[FEP-d9ad]: https://bengo.is/fep/d9ad/
[FEP-d9ad Conformance Test]: https://bengo.is/fep/d9ad/
