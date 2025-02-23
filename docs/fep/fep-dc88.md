---
slug: "dc88"
authors: Calvin Lee <pounce@integraldoma.in>
status: DRAFT
dateReceived: 2023-09-12
trackingIssue: https://codeberg.org/fediverse/fep/issues/161
discussionsTo: https://codeberg.org/fediverse/fep/issues/161
---
# FEP-dc88: Formatting Mathematics
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-dc88.md)から翻訳に協力することができます。


## Summary

This FEP recommends a method for formatting mathematics in ActivityPub
post content in [MathML Core]. Furthermore, this FEP describes how to
sanitize and convert such mathematics to plain text, if an
implementation does not wish to support mathematical formatting.


## Requirements

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”,
“SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this
specification are to be interpreted as described in [RFC-2119]. “The
implementation” is to be interpreted as an ActivityPub conformant
Client, ActivityPub conformant Server or ActivityPub conformant
Federated Server as described in [ActivityPub] which wishes to produce
or consume mathematically formatted content.


## History

Written mathematics depends very heavily on a system of notation which
has been continuously evolving over thousands of years. Despite its
ubiquity, mathematical notation is anything but constant. Mathematicians
rely heavily on complicated typesetting systems such as [LaTeX] to
layout text in their desired fashion.

Several fediverse instances, such as [Mathstodon] have emerged to host
discussion related to mathematics. This is to no small part due to the
difficulty of conveying and formatting mathematical text, and many
provide their own typesetting implementation based on [LaTeX]. However,
rendering TeX-like formats is expensive and fraught with issues due to
security and implementation-specific quirks. This has resulted in
multiple instances with incompatible TeX-like implementations.

In recent years, [MathML Core] has become standardized by all major web
browsers, and offers an alternative method to communicate mathematics
between differing fediverse implementations that is compatible with the
ActivityPub standard.


## Formatting Mathematics

The implementation MAY produce mathematical formatting in the
`summary` or `content` properties of [ActivityStreams] objects, as
defined in [Activity Vocabulary] if the `mediaType` is `text/HTML` (the
default). This formatting MUST be placed within one or more top-level
`<math>` elements, hereon referred to as 'a math element'.

A math element MUST contain one `<semantics>` child element, and no
other children. The `<semantics>` element MUST contain a [MathML Core]
expression as its first child, and at least one `<annotation>` element.
The `encoding` property of this `<annotation>` element SHOULD be
`"application/x-tex"`, but MAY be `"text/plain"`, and MUST contain
a plain-text description of the mathematics—preferably in the authored
format. The implementation MAY include additional `<annotation>` or
`<annotation-xml>` elements with other semantic information.

All elements contained within a math element MUST be **MathML Core
Elements** as defined in [MathML Core], excluding those contained within
`<annotation>` elements.


## Sanitizing Mathematically Formatted Text

The implementation SHOULD sanitize incoming mathematical formatting
before displaying it to a user. There are two methods by which an
implementation may sanitize incoming mathematical formatting.


### Sanitizing a math element

The implementation MAY sanitize a math element before displaying it to
a user. The implementation SHOULD remove all non **MathML Core
Elements** as defined in [MathML Core]. The implementation MAY remove
any attribute which does not contain semantic information (see [Semantic
Attributes]). The implementation SHALL NOT remove any [Semantic
Attributes] or **MathML core Elements** and instead should replace
a math element with text.


### Replacing a math element with text

The implementation MAY remove a math element completely, and replace it
with text within the `<annotation>` element with encoding
`"application/x-tex"` as described in [Formatting Mathematics] and
SHOULD fall back to a `"text/plain"` annotation. If a math element is
not formatted as described in [Formatting Mathematics], then the
implementation MUST remove it completely.

The implementation MAY surround the text from the `<annotation>` element
with a pair of delimiters. For example, if a math element has the
attribute `display="block"`, it may choose the delimiters `$$` and `$$`,
and if `displaystyle="inline"` it may choose `$` and `$` to match the
TeX typesetting system. 

## 例
```json
{"@context": ["https://www.w3.org/ns/activitystreams", {"@language": "en"}],
 "type": "Note",
 "id": "http://postparty.example/p/2415",
 "content": "I have a truly marvelous proof that
             <math>
              <semantics>
                <mrow>
                  <msup><mi>x</mi><mi>n</mi></msup>
                  <mo>+</mo>
                  <msup><mi>y</mi><mi>n</mi></msup>
                  <mo>≠</mo>
                  <msup><mi>z</mi><mi>n</mi></msup>
                </mrow>
                <annotation encoding=\"application/x-tex\">x^n+y^n\\ne z^n</annotation>
              </semantics>
             </math>
           which this note is too small to contain!",
  "source": {
    "content": "I have a truly marvelous proof that \\(x^n+y^n\\ne z^n\\) which this note is too small to contain!",
    "mediaType": "text/markdown+math"}}
```

This object's source content represents a valid sanitization of its
`content` field.


## Semantic Attributes

| Element   | Attributes                    | Values           |
| -------   | ------------                  | ---------------- |
| all       | `mathvariant`                 | `normal`         |
| all       | `displaystyle`, `scriptlevel` | all              |
| `<math>`  | `display`                     | all              |
| `<mfrac>` | `linethickness`               | `0`, `1`         |
| `<mspace>`| `width`, `height`, `depth`    | all              |
| `<mo>`    | `form` `stretchy`, `symmetric`, `largeop`, `movablelimits`, `lspace`, `rspace`, `minsize`, | all              |


## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [ActivityStreams] James M Snell, Evan Prodromou, [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core), 2017
- [Activity Vocabulary] James M Snell, Evan Prodromou, [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/), 2017
- [LaTeX] LaTeX authors, [The LaTeX Project](https://www.latex-project.org)
- [MathML Core] David Carlisle, Frédéric Wang, [MathML Core W3C Candidate Reccomendation Snapshot](https://w3c.github.io/mathml-core/), 2023
- [Mathstodon] Mathstodon Admins, [About Mathstodon](https://mathstodon.xyz/about), retrieved 2023
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119), 1997


## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication 

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
