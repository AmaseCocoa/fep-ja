---
slug: "cb76"
authors: a <a@trwnh.com>
status: DRAFT
dateReceived: 2022-11-29
trackingIssue: https://codeberg.org/fediverse/fep/issues/41
discussionsTo: https://codeberg.org/fediverse/fep/issues/41
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-cb76.md)から翻訳に協力することができます。
# FEP-cb76: Content Addressed Vocabulary

## Summary

JSON-LD context definitions typically live at some URI which gets used as a namespace. It is generally expected that the URI is long-lived, and often the context document is retrievable from that URI, but sometimes these links break due to technical errors, expired domains, and other such issues. This FEP proposes adopting a solution proposed by [CAV] for any extension terms defined within other FEPs, as well as optionally for standard vocabulary.

## 1. Defining FEP extension terms

Each extension term MUST have a specification text.

This specification is then used to calculate a SHA256 hash, which can be used as a URN within `@context` in lieu of a namespaced property.

### Example 1: Specifying a new extension term

Say we want to define a new ActivityStreams Vocabulary [AS2V] term, which will be a sub-type of Activity called `Message`, with the following specification text:

```text
Indicates that the actor is sending a direct message.
```

We obtain the SHA256 hash of this specification text:

```bash
$ echo -n "Indicates that the actor is sending a direct message." | sha256sum

bab53e61faa0ddecce6991df4c26259b6c2e1b880cef12225033590fcaad1aaa  -
```

We can now use this hash to extend ActivityPub [AP] with a `Message` activity:

```json
{
"@context": [
  "https://www.w3.org/ns/activitystreams",
  {
    "Message": "urn:sha256:bab53e61faa0ddecce6991df4c26259b6c2e1b880cef12225033590fcaad1aaa"
  }
],
"actor": "https://social.example/~alice",
"type": "Message",
"to": "https://bob.example.com",
"content": "hi friend"
}
```

## 2. Storing FEP extension terms within the FEP Git repository

(This section is non-normative.)

A directory named `context/` should be created to store content addressed vocabulary extensions associated with FEPs. FEPs that propose vocabulary extensions MUST create a text file called `fep-abcd_property.txt` within this directory, where `abcd` is the identifier of the associated FEP, `property` is the recommended shorthand name of the property, and the contents of the text file are the specification text. FEP authors SHOULD try to use unique property names within the existing FEP vocabulary, unless an FEP is intended to supersede or replace an older FEP.

A `context.jsonld` file SHOULD be generated any time a new vocabulary term is finalized, with its contents being a definition of every finalized vocabulary term.

### Example 2: Standardizing an FEP with a new extension term

In Example 1, we defined a `Message` activity type to be used for ActivityPub [AP] direct messaging. Say we now want to draft an FEP for this definition.

We follow the FEP process as defined in FEP-a4ed to title and identify our FEP proposal. Applying this process to our proposal of `A Message activity for direct messaging` yields the identifier `FEP-0ac6`. Combined with our specification text of `Indicates that the actor is sending a direct message`, this yields the following text file `fep-0ac6_Message.txt`:

```text
Indicates that the actor is sending a direct message
```

The text file is then stored at `context/fep-0ac6_Message.txt`, and when it is finalized, its SHA256 hash is added to `context/context.jsonld`:

```json
{
  "@context": {
    "Message": "urn:sha256:bab53e61faa0ddecce6991df4c26259b6c2e1b880cef12225033590fcaad1aaa"
  }
}
```

## References

- [AS2V] James M Snell, Evan Prodromou, [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary), 2017
- [AP] Christine Lemmer-Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub), 2018
- [CAV] Christine Lemmer-Webber, [Content Addressed Vocabulary](https://dustycloud.org/blog/content-addressed-vocabulary/), 2020

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication 

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
