---
slug: "a4ed"
authors: pukkamustard <pukkamustard@posteo.net>
status: FINAL
dateReceived: 2020-10-16
dateFinalized: 2021-01-18
trackingIssue: https://codeberg.org/fediverse/fep/issues/201
discussionsTo: https://codeberg.org/fediverse/fep/issues/201
---
# FEP-a4ed: The Fediverse Enhancement Proposal Process
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-a4ed.md)から翻訳に協力することができます。

## Summary

A Fediverse Enhancement Proposal (FEP) is a document that provides information to the Fediverse community. The goal of a FEP is to improve interoperability and well-being of diverse services, applications and communities that form the Fediverse.

This document describes the scope, format and process of publishing Fediverse Enhancement Proposals.

## Scope and Objectives

A Fediverse Enhancement Proposal (FEP) should be a concise and focused documentation of a specific topic that is of interest to the Fediverse community.

A proposal should always have the intention of improving the interoperability and well-being of diverse services, applications and community of the Fediverse.

The Fediverse includes applications, services and communities using the ActivityPub protocol and other protocols that foster decentralized and diverse social media and culture.

Proposals may include descriptions of technical protocols and mechanisms, documentation of experimental work or current best practices.

Proposals are not limited to technical topics and may focus on social and cultural aspects.

Proposals may be entertaining and humorous (unlike this proposal).

## Language, Document Structure and Format

All Fediverse Enhancement Proposals must be written in English, be properly formatted as [CommonMark][CommonMark] and be reasonably grammatical.

Authors should use inclusive language and examples and refrain from using [oppressive terminology][Internet-Draft-terminology].

### Proposal Title and Identifier

Every Fediverse Enhancement Proposal must have a descriptive title.

An identifier is computed from the proposal title as the first 4 digits of the sha256 hash (in hex). The identifier can be computed from the title with standard Unix tools:

```
$ echo -n "The Fediverse Enhancement Proposal Process" | sha256sum | cut -c-4
a4ed
```

By using the hash of the title as identifier we reduce the burden on facilitators to assign unique ids. This requires proposal titles to be unique.

### Proposal Type

Fediverse Enhancement Proposals are classified into following types:

- Informational documents
- Implementation proposals

### Metadata

Proposal metadata is placed at the top of the document as key-value pairs between opening and closing ~---~.

Following metadata key-value pairs may be placed at the top of a proposal:

- `type`: Indicates the type of the proposal. Can be either `informational` or `implementation`.
- `slug`: the proposal identifier. The value must be enclosed within double quote marks, e.g. `"a4ed"`.
- `authors`: A comma separated list of authors of the proposal. Each item of the list should contain an author's name followed by a contact address in angle brackets (e.g. `Alyssa P. Hacker <alyssa.p.hacker@email.example>`, `Alyssa P. Hacker <@alyssa.p.hacker@fediverse.example>`).
- `status`: Indicates the proposal status. Can be either `DRAFT`, `WITHDRAWN` or `FINAL`.
- `dateReceived`: Date of when the proposal was added to the repository (when status is set to `DRAFT`).
- `dateWithdrawn`: Date of when the proposal status was set to `WITHDRAWN` (only for proposals with status `WITHDRAWN`).
- `dateFinalized`: Date of when the proposal status was set to `FINAL` (only for proposals with status `FINAL`).
- `trackingIssue`: Link to the tracking issue for the proposal.
- `discussionsTo`: Link to the discussion.
- `relatedFeps`: A comma separated list of related FEPs (e.g. `FEP-a4ed, FEP-141a, FEP-686f`).
- `replaces`: A comma separated list of FEPs that are replaced by the proposal.
- `replacedBy`: Identifier of a FEP that replaces the proposal.

### Required Sections

Every FEP should include at least the following sections:

- Summary: A short (no more than 200 words) summary of the proposal.
- Copyright: Indicating that the proposal has been placed in the public domain.

Following sections may be included in a proposal:

- History: An overview of previous related efforts and how they relate to the proposal.
- Implementations: If applicable an overview of services or applications that implement the proposal at time of submission.
- References: A list of documents and resources referenced by the proposal.

### Copyright

Fediverse Enhancement Proposals must be placed in the public domain by the authors with a CC0 1.0 Universal (CC0 1.0) Public Domain Dedication.

## The Fediverse Enhancement Proposals Process

```mermaid
stateDiagram-v2
    accTitle: FEP process flowchart
    accDescr {
        The process begins from the "DRAFT" status, followed by either of the "FINAL" or "WITHDRAWN" status.
        The "WITHDRAWN" status can go back to the "DRAFT" status.
    }
    [*] --> DRAFT
    DRAFT --> FINAL
    DRAFT --> WITHDRAWN
    WITHDRAWN --> DRAFT
    FINAL --> [*]
```

1. A Fediverse Enhancement Proposal can be submitted by individuals or groups of individuals (authors). See the `SUBMISSION.md` file for a list of accepted submission methods.
2. Within seven days one of the facilitators will read and respond to the proposal. The facilitator checks if the proposal conforms to the required structure and fits the scope and objective of the FEPs. The facilitator may request the authors to clarify, justify, or withdraw the proposal. Such a request must not reflect the personal bias of a facilitator. Rather, it will be made strictly to maintain a high quality of submissions. The facilitators reserve the right to reject a submission when a proposal amounts to blatant misuse of the process. The authors may seek feedback from the wider community if the submitted proposal is rejected or clarifications are requested.
3. If a FEP facilitator approves a submission it receives the status `DRAFT` and is added to the repository. The facilitator also creates a tracking issue for the proposal and adds `trackingIssue` attribute to the proposal metadata.
   - Registered proposals must have the following metadata attributes: `slug`, `authors`, `status`, `dateReceived`, `trackingIssue` and `discussionsTo`.
   - When the status is changed to `FINAL` or `WITHDRAWN`, facilitator adds `dateFinalized` or `dateWithdrawn` attributes to the proposal metadata.
4. While a proposal has the status `DRAFT`:
   - Authors are responsible for initiating community discussion and collecting feedback.
   - Authors may submit updates to the proposal which will be checked in to the repository by a facilitator. Others may also submit updates, but in this case a change must be approved by at least one author.
   - Authors may withdraw the submission upon which a facilitator will set the status of the submission to `WITHDRAWN`.
5. After at least 60 days the authors may request the proposal to be finalized. This is done by requesting final comments on the proposal.
   - If there are no community objections within 14 days and the authors can show that they have initiated sufficient awareness and discussion of the proposal, a facilitator will set the status of the submission to `FINAL`.
6. If after 1 year the authors have not requested the proposal to be finalized, a facilitator should inquire about the status of the proposal. If authors don't respond, a facilitator will set the status of the submission to `WITHDRAWN`.
7. A proposal with status `FINAL` can not be changed or updated in a way that would lead to adjustments to implementations. Minor corrections are allowed.
8. Any substantial change to finalized proposal must be submitted as a separate FEP.

- The new FEP MUST include a `replaces` metadata attribute pointing to FEPs it supersedes.
- If the new FEP becomes `FINAL`, the original one MUST point to it in an added `replacedBy` metadata attribute.

9. A proposal with status `WITHDRAWN` remains in the repository and can be resubmitted.
10. After a proposal becomes `FINAL` a facilitator will archive all discussions linked in the tracking issue and add the resulting archive links to the tracking issue.

### Facilitators

A list of facilitators is maintained in the `FACILITATORS.md` file at the root of the FEP repository.

### Submission Methods

A list of methods in which a proposal may be submitted is maintained in the `SUBMISSION.md` file at the root of the FEP repository.

### FEP Process Governance

FEP-a4ed (The Fediverse Enhancement Proposal Process) is a living document and can be updated despite having the `FINAL` status.

Anyone can propose a change to FEP-a4ed (or other documents related to the FEP process) using any method listed in `SUBMISSION.md` file. The change must be approved by at least two facilitators (one of them could be the submitter). For any part of the FEP process, there must be a sufficient number of facilitators who agreed to do the maintenance work. Changes should not be accepted before at least 1 month passes after the last update, to give facilitators and the community time to review them and provide feedback.

Minor changes (i.e. a correction of a typo or of a broken link) can be accepted immediately after getting two approvals from facilitators.

## History

The process and format described in this proposal is influenced by other community driven documentation efforts such as the [BitTorrent Enhancement Proposal Process][BEP-1], [Scheme Request for Implementation][SRFI] and the [IETF RFC Series][RFC-8729].

## References

- Dave Mason, [Scheme Request For Implementation - Process][SRFI]
- David Harrison, [The BitTorrent Enhancement Proposal Process][BEP-1], 2008
- Housley, R., Ed., and L. Daigle, Ed., [The RFC Series and RFC Editor][RFC-8729], 2020
- John MacFarlane, [CommonMark Spec Version 0.29][CommonMark], 2019
- Mallory Knodel, [Terminology, Power and Oppressive Language][Internet-Draft-terminology]

[SRFI]: https://srfi.schemers.org/srfi-process.html
[BEP-1]: http://bittorrent.org/beps/bep_0001.html
[RFC-8729]: https://www.rfc-editor.org/info/rfc8729
[CommonMark]: https://spec.commonmark.org/0.29/
[Internet-Draft-terminology]: https://tools.ietf.org/html/draft-knodel-terminology

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
