---
slug: "d9ad"
authors: bengo <ben@bengo.co>
status: DRAFT
dateReceived: 2024-07-02
trackingIssue: https://codeberg.org/fediverse/fep/issues/350
discussionsTo: https://codeberg.org/fediverse/fep/issues/350
---
# FEP-d9ad: Create Conformance Tests for Fediverse Enhancement Proposals
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-d9ad.md)から翻訳に協力することができます。


## Summary

This is a proposal to enhance the fediverse by creating test cases for FEPs.
The proposal describes a Conformance Test Rule format that FEP authors and testers may find useful when creating tests cases as proposed.

## Contents

* [Context](#context)
* [Proposal](#proposal)
* [Conformance Tests](#conformance-tests)
  * [W3C Accessibility Conformance Testing (ACT) Rules Format](#w3c-accessibility-conformance-testing-act-rules-format)
  * [FEP-d9ad Conformance Test Format](#fep-d9ad-conformance-test-format)
* [Conformance Test Components](#conformance-test-components)
  * [Applicability](#applicability)
  * [Change Log](#change-log)
  * [Description](#description)
  * [Expectations](#expectations)
  * [Glossary](#glossary)
  * [Identifier](#identifier)
  * [Input](#input)
  * [Issues List](#issues-list)
  * [Name](#name)
  * [Outcomes](#outcomes)
  * [Requirements Mapping](#requirements-mapping)
  * [Target](#target)
  * [Test Cases](#test-cases)
  * [Test Subject](#test-subject)
  * [Test Suite Memberships](#test-suite-memberships)
* [Appendix: Conformance](#appendix-conformance)

## Context

[FEP-a4ed: The Fediverse Enhancement Proposal Process][FEP] describes a process for proposing enhancements to the fediverse (i.e. 'FEPs').

However, this process says nothing about testing implementations claiming to conform to those proposals nor ways of making FEPs that are more conducive to consistent [conformance](#appendix-conformance) testing.

However, consistent testing is an important part of an interoperability and accessibility on the web:
> In order for web specifications to permit full interoperability and access to all, it is important that the quality of implementation be given as much attention as their development. Moreover, as the complexity of W3C specifications and their interdependencies increases, quality assurance becomes even more important to ensuring their acceptance and deployment in the market

&horbar;[NIST.gov - W3C Quality Assurance Activity Initiated](https://www.nist.gov/publications/w3c-quality-assurance-activity-initiated)

The goal of this FEP is to propose a way of proposing FEPs that may lead to more [consistent testing](https://www.w3.org/WAI/standards-guidelines/act/implementations/#understanding-act-consistency) of candidate implementations claiming to conform to a FEP.

For more context on conformance, see the [appendix on conformance](#appendix-conformance) for:

* [What is Conformance?](#what-is-conformance)
* [What is a Conformance Model?](#what-is-a-conformance-model)
* [Why does Conformance Matter?](#why-does-conformance-matter)

## Proposal

FEPs authors MAY [include a conformance clause](https://www.w3.org/TR/qaframe-spec/#include-conformance-clause-principle).

FEPs authors MAY [Indicate which conformance requirements are mandatory, which are recommended, and which are optional](https://www.w3.org/TR/qaframe-spec/#req-opt-conf-principle).

FEP authors MAY [write test assertions](https://www.w3.org/TR/qaframe-spec/#write-assertion-gp).

FEP authors and testers MAY [write tests](https://www.w3.org/TR/qaframe-spec/#write-sample-gp).

FEP testers MAY publish a FEP test as a new FEP.

FEP implementors MAY test their implementations.

FEP implementors MAY publish an Implementation Conformance Statement explaining how the implementation conforms to a FEP

Project and services claiming to implement a FEP SHOULD publish an Implementation Conformance Statement

FEP testers MAY write tests using [W3C Accessibility Conformance Testing (ACT) Rules Format 1.0](https://www.w3.org/TR/act-rules-format/#input)

## Conformance Tests

Conformance Tests provide guidance for developers of automated testing tools and manual testing methodologies, to help ensure consistent interpretation of the FEP.

### W3C Accessibility Conformance Testing (ACT) Rules Format

W3C ACT ([spec](https://www.w3.org/TR/act-rules-format/#input)) is a conformance test format that can be used to provide guidance for testing conformance to W3C's Web Content Accessibility Guidelines that are normatively referenced in other standards and [laws](https://en.wikipedia.org/wiki/Web_Content_Accessibility_Guidelines#WCAG_referenced_by_law) like [section508.gov](https://www.section508.gov/), [gov.co.uk](https://www.gov.uk/guidance/accessibility-monitoring-how-we-test), and Europe's [EN 301 549](https://www.etsi.org/human-factors-accessibility/en-301-549-v3-the-harmonized-european-standard-for-ict-accessibility).

While there may be other formats for Conformance Tests, the ACT Format is sufficient for expressing Conformance Tests not only for accessibility, but for FEPs as well. This author was unable to find any other commonly used conformance test formats. At the same time, because the ACT format was designed specifically for accessibility guidelines, the ACT format may be imperfect for FEP Conformance Tests. So this FEP does not specify a strict conformance test format. Instead, it specifies useful [components of a Conformance Test](#conformance-test-components), most of which are inspired by similar subcomponents of ACT Rules.

### FEP-d9ad Conformance Test Format

This FEP defines a kind of Conformance Test that may be used. It reuses many good decisions from the ACT Rule Format, while generalizing the format to be useful for expressing tests for things other than accessibility (e.g. FEPs).

At a high level, a Conformance Test specifies

* **[input](#input)** required to run the test
* **[applicability](#applicability)**, or whether the rest of the test even applies to the input
* **[targets](#target)**, derived from input, that should be tested
* **[expectations](#expectations)** whose assertions about the targets are tested
* **[outcomes](#outcomes)** the result of testing expectations for each test target
* **[requirements mapping](#requirements-mapping)** how the outcomes affect claims about a test subject's conformance to any specific requirements

## Conformance Test Components

Each of these components may be a part of a conformance test.

* [Applicability](#applicability)
* [Change Log](#change-log)
* [Description](#description)
* [Expectations](#expectations)
* [Glossary](#glossary)
* [Identifier](#identifier)
* [Input](#input)
* [Issues List](#issues-list)
* [Name](#name)
* [Outcomes](#outcomes)
* [Requirements Mapping](#requirements-mapping)
* [Target](#target)
* [Test Cases](#test-cases)
* [Test Subject](#test-subject)
* [Test Suite Memberships](#test-suite-memberships)

### Applicability

Applicability describes how to determine whether a test is even able to be applied to a particular [Input](#input) and produce a meaningful outcome other than `inapplicable`.

A test may have all kinds of outcomes, e.g. `inapplicable`. If there is a test specified to apply to an ActivityPub Actor Object and check that it is valid JSON, and the test is run with input of a fish, it's not even possible to check the fish for JSON Syntax conformance to derive any targets or apply expectations. It's a fish. The test doesn't apply *at all*, and in situations like this, the test run MAY have outcome `inapplicable`. The Applicability section in a Conformance Test specifies how to determine whether the test applies to the test [input](#input) and, if so, how to derive the test [targets](#target).

Inspired by

* [ACT Rule Applicability for Atomic Rules](https://www.w3.org/TR/act-rules-format/#applicability-atomic)

#### Example

An example of an Applicability clause for a hypothetical conformance test is:

> This test applies to an ActivityPub Actor Object. If the value of the `actor` input's "type" property is an array, there should be a test target for each entry in the array. If it is not an array, there should be one test target whose value is the value of the `actor` input's "type" property.

#### URI

<https://w3id.org/fep/d9ad/ns/test/applicability>

<!-- conformance test component separator -->

### Change Log

A log with entry for each change to the test over time.

For example, if a test if maintained over many years, it is likely that the editors will learn from experience with implementations and usage of the test implementations that there were unexpected inputs the test might be presented with, and then update the test inputs, target, or expectations to better handle the situation. This kind of update is encouraged, but it should be logged in the Conformance Test's Change Log.

#### URI

<https://w3id.org/fep/d9ad/ns/test/changeLog>

<!-- conformance test component separator -->

### Description

A plain language overview of what the test does and why.

Inspired by

* [ACT Rule Description](https://www.w3.org/TR/act-rules-format/#rule-description)

#### Example

> This is a conformance test that checks whether an ActivityPub Actor satisfies syntax requirements for the "type" property

#### URI

<https://w3id.org/fep/d9ad/ns/test/description>

### Expectations

Assertions about test targets.

An Conformance Test MUST contain one or more expectations.

The expectations describe what the requirements are for the test targets.

An expectation is an assertion about a test target.

Expectations determine test target [outcomes](#outcomes)

* When a test target meets all expectations, the test target passed the rule.
* If the test target does not meet all expectations, the test target failed the rule.
* If there are no test targets, the outcome for the rule is inapplicable.

Each expectation must be distinct, unambiguous, and be written in plain language.

Inspired by

* [ACT Rule Expectations](https://www.w3.org/TR/act-rules-format/#expectations)
* [ACT Rule Expectations for Atomic Rules](https://www.w3.org/TR/act-rules-format/#expectations-atomic)

#### URI

<https://w3id.org/fep/d9ad/ns/test/expectations>

<!-- conformance test component separator -->

### Glossary

Definitions for common terms.

The Glossary for a test MUST define each of the test's possible Outcomes.

Inspired by

* [ACT Rule Glossary](https://www.w3.org/TR/act-rules-format/#glossary)

<!-- conformance test component separator -->

#### Identifier

a globally unique identifier that identifies the Conformance Test

The Identifier MUST be a [URI](https://datatracker.ietf.org/doc/html/rfc3986#section-1.1).

Inspired by

* [ACT Rule Identifier](https://www.w3.org/TR/act-rules-format/#rule-identifier)
* [ActivityStreams 2.0 id](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-id)

##### 例

* `urn:uuid:93bafe14-b13f-4a9a-9d47-0a9316d50c97`
* `https://bengo.is/fep/d9ad`

#### URI

<https://w3id.org/fep/d9ad/ns/test/id>

<!-- conformance test component separator -->

### Input

This is what goes in to each run of the conformance test.

The test input is the entirety of how each run of the test can be configured.

An Input may itself have several components.

#### Example Input Specification

Consider a hypothetical Conformance Test that tests conformance with [ActivityPub][]. An Input to the test may have several components, for example:

* `id` - URI - An ActivityPub Object ID
* `authorization` - optional string - A value to pass with each request to fetch `id`

The Input and its components MAY be represented as a JSON Object with a property for each named component:

#### Example Input as JSON

```json
{
    "object": "https://bengo.is/fep/d9ad",
    "authorization": "mellon"
}
```

#### URI

<https://w3id.org/fep/d9ad/ns/test/input>

### Issues List

A list of unresolved issues with the test.

Tests should be published early and often, even and especially before there are no known issues with it. The issues may be considered over time, perhaps waiting for enough information to determine a resolution, and then the test may be updated accordingly. The Issues List makes test readers aware of unresolved issues that may lead to test changes later.

Inspired by

* [ACT Rule Issues List](https://www.w3.org/TR/act-rules-format/#issues-list)

#### URI

<https://w3id.org/fep/d9ad/ns/test/issuesList>

### Name

A short label for the test that can distinguish it in a list of tests.

The name MUST NOT include HTML markup.

The name MAY have distinct values for distinct languages.

The name may be a value that is not unique amongst all other test cases.
However, test names should be sufficiently descriptive to distinguish them from other tests in the same test suite.

Inspired by

* ACT Descriptive Title
* [ActivityStreams 2.0 name](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-name)
    > A simple, human-readable, plain-text name for the object. HTML markup MUST NOT be included. The name MAY be expressed using multiple language-tagged values.

#### Example Name

An example name for a hypothetical test:
> ActivityPub Actors MUST have a valid "type" property value according to FEP-003c

#### URI

<https://w3id.org/fep/d9ad/ns/test/name>

<!-- conformance test component separator -->

### Outcomes

A test's glossary SHOULD define every possible outcome that the test may assign to test targets as a result of the test expectations.

#### URI

<https://w3id.org/fep/d9ad/ns/test/glossary>

<!-- conformance test component separator -->

### Requirements Mapping

When a Conformance Test is designed to test conformance to one or more requirements documents, the test must list all requirements from those documents that are not satisfied when one or more of the outcomes of the test is failed.

Each requirement in the mapping must include the following:

* the requirement
  * name or summary
  * uri identifier
  * url to documentation

Inspired by

* [ACT Requirements Mapping](https://www.w3.org/TR/act-rules-format/#accessibility-requirements-mapping)

#### URI

<https://w3id.org/fep/d9ad/ns/test/requirementsMapping>

### Target

All parts of the test subject that will be checked by the test.

Each target MUST have a test [outcome](#outcomes). Each possible outcome SHOULD be defined in the glossary.

From a given [input](#input), the test may derive many targets. For example, from an input that is a JSON Object, a test could derive a test target for each JSON Object nested within the input Object. Then the test, for each target, could check for conformance to a hypothetical requirement that all objects within the input MUST have a certain property.

Inspired by

* [ACT Rule Input](https://www.w3.org/TR/act-rules-format/#input)
* [ACT Rule Input Aspects](https://www.w3.org/TR/act-rules-format/#input-aspects)

#### URI

<https://w3id.org/fep/d9ad/ns/test/input>

### Test Cases

Test Cases (i.e. test cases for the test itself) are a set of example inputs and corresponding results that implementors of the test can check to be confident they have implemented the test has specified in prose. The Test Cases may also be helpful to test readers seeking to understand the intention of the test [expectations](#expectations).

Every Conformance Test Case SHOULD include

* input
* targets derived from input
* outcome for each target
* how outcomes map to requirements

Inspired by

* [ACT Rule Test Cases](https://www.w3.org/TR/act-rules-format/#test-cases)

#### URI

<https://w3id.org/fep/d9ad/ns/test/testCases>

### Test Subject

An overview of what kind of thing the Conformance Test tests. For example, a FEP Conformance Test may apply to an ActivityPub Actor, or another ActivityPub Object, or some part of a server that hosts many ActivityPub Objects, or it could test something else entirely. The Test Subject is helpful for contextualizing the test [Input](#input)

Inspired By

* <https://en.wikipedia.org/wiki/System_under_test>

### Test Suite Memberships

The test suites that the test is a part of, if any.

For each test suite membership, the test should specify:

* name: Plain language name of the Test Suite
* url: URL to the Test Suite

#### URI

<https://w3id.org/fep/d9ad/ns/test/testSuiteMemberships>

## Conformance with this Specification

<section id="conformance">Conformance requirements are indicated by sentences containing MUST a la <a href="https://datatracker.ietf.org/doc/html/rfc2119">RFC2119</a>. A FEP-d9ad Conformant Conformance Test is a document satisfying all conformance requirements in this document.</section>

## Related Links

These links were helpful when researching conformance testing

* [w3.org - QA Framework: Specification Guidelines](https://www.w3.org/TR/qaframe-spec/)
* [ACT Rules Test Cases](https://act-rules.github.io/pages/implementations/testcases/)
* [ISO 17000 - Conformity Assessment](https://www.iso.org/obp/ui/#iso:std:iso-iec:17000:ed-1:v1:en)

## Appendix: Conformance

### What is Conformance?

> Conformance is the fulfillment of specified requirements by a product, process, or service. These requirements are detailed in a specification as part of a conformance clause and in the body of the specification. A conformance clause is the section of a specification that identifies all the criteria that must be satisfied in order to claim conformance to the specification.

&horbar;[w3.org - QA Framework: Specification Guidelines](https://www.w3.org/TR/qaframe-spec/#specify-conformance)

### What is a Conformance Model?

> What does it mean?
>
> The conformance model is the conceptual framework in which conformance is defined. It consists of and is defined by addressing at least these three topics:
>
> * What needs to conform and how — hereafter designated as class of products.
> * Any special designations or concepts used to distinguish conformance categories, types, etc. (e.g., profile/module/level, well-formed/valid, A/AA/AAA).
> * Ways that conforming implementations can vary from each other (e.g., optionality and extensions).
>
> Why care?
>
> The key is to communicate to the reader what conformance to the specification is all about. The model provides a framework for implementers, describes what they need to build in order to conform, and explains the different ways that they could claim conformance.  It provides users and customers with a basis on which to express their requirements.

&horbar;[w3.org - QA Framework: Specification Guidelines](https://www.w3.org/TR/qaframe-spec/#specify-conformance)

### Why does Conformance Matter?

> It is inevitable that people (e.g., vendors, purchasers) will either claim conformance or demand conformance to a technology. In fact, claiming conformance to a technology may be required in certain situations. Thus, it is important to provide a consistent and unambiguous way to make these claims. Identification of the specification version, class of products, and conformance label are some of the items that could be part of such wording.
>
> Why care?
>
> Having a framework, by which to make conformance claims for a particular usage of the technology, minimizes confusion by people who are interested in such claims. Many contexts use conformance claims, including legal as part of regulations, laws, or policies and commercial when selling or buying a product.

&horbar;[w3.org - QA Framework: Specification Guidelines](https://www.w3.org/TR/qaframe-spec/#specify-conformance)

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。

[FEP]: https://codeberg.org/fediverse/fep/src/branch/main/fep/a4ed/fep-a4ed.md#fep-a4ed-the-fediverse-enhancement-proposal-process
[ActivityPub]: https://www.w3.org/TR/activitypub/]
