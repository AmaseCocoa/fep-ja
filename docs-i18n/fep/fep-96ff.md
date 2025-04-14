---
slug: "96ff"
authors: Erin Shepherd <fep@erinshepherd.net>
status: DRAFT
dateReceived: 2024-02-17
trackingIssue: https://codeberg.org/fediverse/fep/issues/272
discussionsTo: https://codeberg.org/fediverse/fep/issues/272
---
# FEP-96ff: Explicit signalling of ActivityPub Semantics
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-96ff.md)から翻訳に協力することができます。
## Summary
A number of vulnerabilities have occurred in ActivityPub implementations due to
"type confusion" attacks - where unrelated files on the same hostnmae as an ActivityPub
implementation are processed as obejcts with ActivityPub semantics. 

Such attacks have been mitigated by carefuly validating the `Content-Type` header (and
by implementations ensuring that users cannot create files with the `application/activity+json`
or `application/ld+json` content types), but it would bolster such defences if messages
intended to be processed with ActivityPub semantics 

Additionally, ActivityPub nominally supports transfer syntaxes other than JSON-LD (such
as any other RDF syntax like Turtle; or potentially a more bandwidth efficient syntax such
as a hypothetical CBOR-LD). Strict content type filtering permanently prevents usage of 
such syntaxes in the future

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", " SHOULD", 
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as 
described in [RFC2119][RFC2119]. 

## Signalling of ActivityPub Semantics
An implementation signifies its intention to use ActivityPub semantics for a request
or response body by including a Link header with relation type `type` and contents
of `https://www.w3.org/TR/activitypub/`. This header usage is borrowed from [LDP][LDP],
which uses the same relation type to indicate support of LDP semantics.

```
Link: <https://www.w3.org/TR/activitypub/>;rel="type"
```

A conformant implementation MUST include this link relation in any HTTP messages it
intends to be processed with ActivityPub semantics.

A conformant implementation MUST process a HTTP message and where this link relation 
is present and the content type is an implementation supported transport syntax for 
ActivityStreams 2 with ActivityPub semantics.

A conformant implementation MAY process a HTTP message without any Link header with
relation "type", where the `Content-Type` header contains exactly the value 
`application/activity+json` or `application/ld+json` with a profile parameter 
containing the token `https://www.w3.org/ns/activitystreams` with ActivityPub
semantics (This enables backwards compatibility with existing implementations.)

A conformant implementation MUST NOT process any other HTTP messages with ActivityPub 
semantics.

Conformant implementations MUST support messages containing multiple Link headers, 
including multiple link headers with the "type" relation. The following examples 
are all valid and indicate that processing should occur with ActivityPub semantics:

```
Link: <https://www.w3.org/TR/activitypub/>;rel="type"

Link: <https://www.w3.org/TR/activitypub/>;rel="type", <https://example.com/>;rel="test"

link: <https://www.w3.org/TR/activitypub/>;rel="type", <https://example.com/>;rel="type"

LINK: <https://www.w3.org/TR/activitypub/>;rel="type"
Link: <https://example.com/>;rel="test"

Link: <https://www.w3.org/TR/activitypub/>;rel="type"
lInK: <https://example.com/>;rel="type"
```

Implementations are encouraged to periodically add additional Link relations to their
messages to ensure support by counterparties. (This process is typically termed
[greasing](https://www.rfc-editor.org/rfc/rfc8701.html), after the GREASE extension
to TLS)

For the avoidance of doubt, ActivityPub semantics apply only in cases where 
implementations expect to transfer ActivityStreams 2 documents with the semantics
described in [the ActivityPub specification][AP]. This link relation MUST NOT be
included for other exchanges that an ActivityPub implementation may perform, 
including but not limited to WebFinger queries.

## Why not just rely upon the content type?
The content type alone does not specify semantics. The ActivityStreams 2 syntax
can be used independently of ActivityPub, and non-ActivityPub systems
[such as Cohost](https://cohost.org/jkap/post/1249642-how-i-message-detects)
produce ActivityStreams 2 documents.

Additionally, even though ActivityPub implementations can prevent the creation
of unexpected resources with an ActivityStreams content type, they cannot prevent
other applications running on the same name from permitting the untrusted
creation of such resources. 

It is hoped that a future version of this specification (after widespread deployment
of explicit tagging of messages with explicit semantics) can omit the permission
to determine semantics based upon media type.

## Vulnerabilities (Non-Normative)
The following is a list of known implementation vulnerabilities which would have
been prevented by implementation of this mechanism:

* [CVE-2024-25623](https://github.com/mastodon/mastodon/security/advisories/GHSA-jhrq-qvrm-qr36): 
  Lack of media type verification of Activity Streams objects allows impersonation of remote accounts (Mastodon)
* [GHSA-qqrm-9grj-6v32](https://github.com/misskey-dev/misskey/security/advisories/GHSA-qqrm-9grj-6v32): 
  Lack of media type verification of Activity Streams objects allows impersonation and takeover of remote accounts (Misskey)
* [Strict content type validation](https://github.com/pixelfed/pixelfed/commit/1232cfc8), Pixelfed (Same as previous two), no vulnerability ID assigned
* [add stricter checks during all stages of dereferencing remote AS objects](https://github.com/superseriousbusiness/gotosocial/pull/2639), GoToSocial, No vulnerability ID assigned
* [GHSA-xmw2-875x-rq88](https://github.com/kitsune-soc/kitsune/security/advisories/GHSA-xmw2-875x-rq88):
  Possibility of a fake account on a domain with arbitrary user content (Kitsune)
* [Pleroma Issue 1948](https://git.pleroma.social/pleroma/pleroma/-/issues/1948#note_67278):
  Messages can be spoofed

If all implmenetations strictly required implementation of the link relation check, 
the following vulnerabilities would also be prevented:

* [CVE-2023-36460](https://github.com/mastodon/mastodon/security/advisories/GHSA-9928-3cp5-93fm), 
Arbitrary file creation through media attachments (Mastodon)

(This is not presently proposed because it would break compatibility with existing implementations)

These lists are likely not exhaustive.

## 参考文献
- [ActivityPub][AP] Christine Lemmer Webber, Jessica Tallon et al, 2018
- [Linked Data Platform 1.0][LDP], Steve Speicher, John Arwe, Ashok Malhotra, 2015

## Copyright
CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.


[AP]: https://www.w3.org/TR/activitypub/
[LDP]: https://www.w3.org/TR/ldp/
[RFC2119]: https://www.w3.org/TR/activitystreams-core/#bib-RFC2119
