---
slug: "612d"
authors: Helge <helge.krueger@gmail.com>
status: DRAFT
dateReceived: 2023-04-18
trackingIssue: https://codeberg.org/fediverse/fep/issues/89
discussionsTo: https://codeberg.org/fediverse/fep/issues/89
---
# FEP-612d: Identifying ActivityPub Objects through DNS
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-612d.md)から翻訳に協力することができます。

## Summary

In [ActivityPub], objects are identified through their id, which is a dereferenciable URI. For this, one adds a TXT record to DNS with name `_apobjid` and value corresponding to the URI of the ActivityPub object. If a domain name is then passed to a FediVerse application, it can then perform the DNS lookup, and resolve it to the ActivityPub object.

## Discussion

This FEP is inspired by [BlueSky], and it gets the FediVerse half of the way there. It allows one to look up the ActivityPub Actor from an associated domain name, it does not address the issue of diplaying the domain name instead of the FediVerse handle. One first step would be to include the domain name in [alsoKnownAs](https://www.w3.org/TR/did-core/#dfn-alsoknownas). Then use some property of the actor to specify, which object to display.

## Other Mechanisms

Two other mechanisms to identify ActivityPub objects beside their id are currently in use:

### Webfinger

The FediVerse handle `username@domain.tld` is commonly used as an identifier for users on the FediVerse. This handle can be resolved by performing a Webfinger lookup via

```http
GET https://domain.tld/.well-known/webfinger?resource=acct:username@domain.tld
```

the activity pub object is then contained in the link with type `application/activity+json`.

### HTML Link header

The second method, that is less commonly used, is to provide a HTML link header of the form

```html
<link
    href="https://activty.pub/object/id"
    rel="alternate"
    type="application/activity+json"
/>
```

A FediVerse object receiving this HTML as a response to a lookup, can then parse it and use the link to lookup the corresponding object.

### Content Negotiation

On a server supporting content negotiation, a request with Accept header "application/activity+json" will be either served or redirected to the corresponding ActivityPub object. The author does not a method to identify where the ActivityPub object is, as it is being directly served.

## Example of using DNS to identify an ActivityPub object

The following shows an example configuration for the domain `mymath.rocks` correspond to the actor with FediVerse handle `@helge@mymath.rocks`, i.e. the author,

```bash
$ dig _apobjid.mymath.rocks

;; ANSWER SECTION:
_apobjid.mymath.rocks.	7200	IN	TXT	"https://mymath.rocks/endpoints/SYn3cl_N4HAPfPHgo2x37XunLEmhV9LnxCggcYwyec0"
```

The corresponding object being

```json
{
    "@context": ["https://www.w3.org/ns/activitystreams","https://w3id.org/security/v1"],
    "id":"https://mymath.rocks/endpoints/SYn3cl_N4HAPfPHgo2x37XunLEmhV9LnxCggcYwyec0",
    "name":"Helge",
    "preferredUsername":"helge",
    "summary":"<p>I like Math, cows, and wrote <a href=\"https://codeberg.org/bovine/bovine/\">bovine</a>.</p>",
    "type":"Person", ...
}
```

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [BlueSky] Jay Graber [Domain Names as Handles in Bluesky](https://blueskyweb.xyz/blog/3-6-2023-domain-names-as-handles-in-bluesky), 2023

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
