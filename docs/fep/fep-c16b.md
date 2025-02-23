---
slug: "c16b"
authors: ilja <ilja@ilja.space>
status: DRAFT
dateReceived: 2024-08-10
discussionsTo: https://socialhub.activitypub.rocks/t/fep-c16b-formatting-mfm-functions/4448
relatedFeps: FEP-dc88
trackingIssue: https://codeberg.org/fediverse/fep/issues/383
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-c16b.md)から翻訳に協力することができます。
# FEP-c16b: Formatting MFM functions

## Summary

This FEP recommends a method for formatting MFM in ActivityPub post content using HTML with custom classes and [data-* attributes]. Furthermore, this FEP provides a new extension term to indicate that this HTML representation is used.


## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119]. “Fediverse implementation” or "implementation” is to be interpreted as an ActivityPub conformant Client, ActivityPub conformant Server or ActivityPub conformant Federated Server as described in [ActivityPub].


## Acknowledgements

(This section is non-normative.)

The core idea behind this FEP is attributed to Johan150 on the Foundkey issue tracker[1]. Specifically, the proposal for representing the MFM functions in HTML using `span` elements with custom classes and `data-*` attributes.


## History

(This section is non-normative.)

It is common for a Fediverse implementation to allow a markup language as input for text. Federation of this content generally happens by converting this text input to a proper HTML representation that another implementation can easily understand. This HTML representation is federated over ActivityPub using the `content` property of the [ActivityStreams] Object. Meanwhile, the `source` property, which was added by ActivityPub, can optionally be used to provide the original input and input format.

Misskey has been using its own [Markup language For Misskey], also known as MFM. MFM is mostly composed of a combination of HTML, Markdown, Katex, and custom MFM functions of the form `$[name content]`. Properly displaying what these MFM functions intent, generally requires complex CSS or even Javascript. As such, only a watered down HTML representation is provided in the `content`. This representation can strip out so much information that a receiving implementation cannot always properly display what the author meant to convey. The only option for a receiving implementation who wants to display the MFM correctly, is to re-parse the `content` of the `source` property when it's `mediaType` has value `text/x.misskeymarkdown`. This causes not only unnecessary overhead, but also compatibility issues, especially when two implementations are using a different parser.


## MFM Functions

(This section is non-normative.)

An MFM Function consists of a name, optionally one or more attributes who may or may not have a value, and a content. It has the form `$[name.attribute1,attribute2=value content]`.


### Examples

(This section is non-normative.)

```
$[x2 Misskey expands the world of the Fediverse]
$[jelly.speed=2s Misskey expands the world of the Fediverse]
$[spin.x,speed=0.5s Misskey expands the world of the Fediverse]
```


## HTML representation of MFM functions

When representing the MFM function in HTML, a `span` element MUST be used. The `span` element MUST have a class `mfm-name` where `name` is the name of the MFM function. When the MFM function has attributes, the `span` element MUST have a `data-*` attribute `data-mfm-attributename` for each attribute, where `attributename` is the name of the attribute in question. If the attribute of the MFM function has a value, the `data-*` attribute MUST have the same value.


### Examples

(This section is non-normative.)

This turns the previous examples into

```
<span class="mfm-x2">Misskey expands the world of the Fediverse</span>
<span class="mfm-jelly" data-mfm-speed="2s">Misskey expands the world of the Fediverse</span>
<span class="mfm-flip" data-mfm-x data-mfm-speed="0.5s">Misskey expands the world of the Fediverse</span>
```


## Other MFM components

While this FEP focuses on the representation of MFM functions, MFM consist of more than just these MFM functions. The HTML representation in the `content` property MUST be correct and complete such that a receiving implementation can use it to correctly display what the MFM conveys.

HTML and Markdown are generally expressed correctly in the `content` property, and are both widely used on the Fediverse. Therefor these are not considered problematic in the same sense as the MFM functions are.

Katex suffers the same problem of generally not being expressed properly in the `content` property. For properly expressing the Katex input as HTML, [FEP-dc88] SHOULD be used.


## Discovery

(This section is non-normative.)

When compatibility with an MFM capable, but FEP-c16b non-compliant, implementation is desired, the `source` may still need to be federated using `"mediaType": "text/x.misskeymarkdown"`. Meanwhile, the incoming `source` from this implementation may still need to be re-parsed. As such, a discovery mechanism is required to signal to a FEP-c16b compliant implementation that the `content` may be used directly.

For this purpose, a new extension term is proposed, as described in [FEP-888d].


### htmlMfm

To signal that the `content` is FEP-c16b compliant, the implementation MAY use the extension term `htmlMfm` with value `true`. When the `content` is not FEP-c16b compliant, the implementation MUST NOT use the extension term `htmlMfm` with value `true`, but MAY use the extension term `htmlMfm` with value `false`.

* Description: A flag to indicate that the `content` is FEP-c16b compliant.
* URI: <code>https://w3id.org/fep/c16b#htmlMfm</code></li>
* Domain: <code>https://www.w3.org/ns/activitystreams#Object</code></li>
* Range: Boolean</li>


## Example

(This section is non-normative.)

```json
{
	"@context": [
		"https://www.w3.org/ns/activitystreams",
		{
			"htmlMfm": "https://w3id.org/fep/c16b#htmlMfm"
		}
	],
	"content": "<span class=\"mfm-spin\" data-mfm-x data-mfm-speed=\"0.5s\">Misskey expands the world of the Fediverse</span>",
	"source": {
		"content": "$[spin.x,speed=0.5s Misskey expands the world of the Fediverse]",
		"mediaType": "text/x.misskeymarkdown"
	},
	"htmlMfm": true
}
```


## References

- [data-* attributes]: Part of the [HTML Living Standard](https://html.spec.whatwg.org/multipage/dom.html#embedding-custom-non-visible-data-with-the-data-*-attributes)
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119), 1997
- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [1] Johan150, [Federate MFM in content field using HTML](https://akkoma.dev/FoundKeyGang/FoundKey/issues/343#issuecomment-7344), 2023
- [ActivityStreams] James M Snell, Evan Prodromou, [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core), 2017
- [Markup language For Misskey], [MFM](https://misskey-hub.net/en/docs/for-users/features/mfm/)
- [FEP-dc88] Calvin Lee, [FEP-dc88: Formatting Mathematics](https://codeberg.org/ilja/fep/src/branch/main/fep/dc88/fep-dc88.md), 2023
- [FEP-888d] a, [FEP-888d: Using https://w3id.org/fep as a base for FEP-specific namespaces](https://codeberg.org/ilja/fep/src/branch/main/fep/888d/fep-888d.md), 2023


## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
