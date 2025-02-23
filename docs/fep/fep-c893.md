---
slug: "c893"
authors: AvidSeeker <avidseeker7@protonmail.com>
status: DRAFT
dateReceived: 2024-07-15
discussionsTo: https://socialhub.activitypub.rocks/t/fep-c893-doap/4363
title: DOAP
trackingIssue: https://codeberg.org/fediverse/fep/issues/372
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-c893.md)から翻訳に協力することができます。

# FEP-c893: DOAP

## Summary

This proposal introduces a standardized method for describing Fediverse projects
using the Description of a Project (DOAP) format. The proposal outlines the
creation of `doap.jsonld` file that includes details about implemented
federation protocols and supported Fediverse Enhancement Proposals (FEPs). This
makes it easier for developers and users to understand the capabilities and
compatibility of various Fediverse projects.

### Example `doap.jsonld` File

Here is an example structure for the `doap.jsonld` file:

```json
{
  "@context": {
    "doap": "http://usefulinc.com/ns/doap#",
    "foaf": "http://xmlns.com/foaf/0.1/",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "doap:description": {
      "@id": "doap:description",
      "@container": "@language"
    },
    "doap:shortdesc": {
      "@id": "doap:shortdesc",
      "@container": "@language"
    }
  },
  "@type": "doap:Project",
  "doap:name": "ExampleProject",
  "doap:homepage": "https://example.org",
  "doap:description": {
    "en": "ExampleProject is a Fediverse client/server that supports multiple protocols and enhancements.",
    "es": "ExampleProject es un cliente/servidor de Fediverse que admite múltiples protocolos y mejoras."
  },
  "doap:shortdesc": {
    "en": "Tools and vocabulary for describing community-based software projects.",
    "es": "Vocabulario y herramientas para describir proyectos de software comunitarios."
  },
  "doap:created": "2022-01-01",
  "doap:logo": "https://example.org/logo.png",
  "doap:screenshots": [
    "https://example.org/screenshot1.png",
    "https://example.org/screenshot2.png"
  ],
  "doap:category": [
    "http://software.freshmeat.net/browse/1020/",
    "http://osdir.com/Downloads+index-req-viewsdownload-sid-201.phtml"
  ],
  "doap:repository": {
    "@type": "doap:GitRepository",
    "doap:browse": "https://github.com/example/exampleproject/",
    "doap:location": "https://github.com/example/exampleproject.git"
  },
  "doap:release": {
    "@type": "doap:Version",
    "doap:created": "2024-07-15",
    "doap:name": "v1.0.0",
    "doap:revision": "1.0.0"
  },
  "doap:maintainer": [
    {
      "@type": "foaf:Person",
      "foaf:name": "John Doe",
      "foaf:homepage": "https://github.com/johndoe"
    },
    {
      "@type": "foaf:Person",
      "foaf:name": "Jane Smith",
      "foaf:homepage": "https://github.com/janesmith"
    }
  ],
  "doap:implements": [
    "https://activitypub.rocks/specification",
    "https://diasporafoundation.org"
  ],
  "doap:supportedFEPs": [
    "https://fediverse.org/fep-0001",
    "https://fediverse.org/fep-0002"
  ]
}
```

## References

- [Description of a Project (DOAP) Specification](http://usefulinc.com/ns/doap#)
- [XEP-0453: DOAP Usage in XMPP](https://xmpp.org/extensions/xep-0453.html)

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement
Proposal have waived all copyright and related or neighboring rights to this
work.
