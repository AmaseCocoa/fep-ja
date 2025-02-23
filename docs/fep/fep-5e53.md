---
slug: "5e53"
authors: Don Marti <dmarti@zgp.org>
status: DRAFT
dateReceived: 2024-06-09
trackingIssue: https://codeberg.org/fediverse/fep/issues/327
discussionsTo: https://socialhub.activitypub.rocks/t/fep-5e53-opt-out-preference-signals/4323
---

# FEP-5e53: Opt-out Preference Signals
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-5e53.md)から翻訳に協力することができます。

## Summary

Some users have concerns about how their content and/or personal information are
used. For example, some users do not want the content they created
to be used for training generative AI systems, and some users do
not want to have their personal information shared or sold.

Several opt-out preference signals (OOPSs) have been
standardized or proposed in the form of HTTP headers that can apply to a
connection between a user and a central server.
In some jurisdictions, companies that administer web sites are required to
process and act on OOPSs. 

This FEP extends [ActivityPub] to support passing OOPSs along with
the content and user information to which they may apply. This FEP
refers to existing OOPSs that have already been documented, and does not propose new ones.


## Declaring an OOPS

In order to apply an OOPS to an object, an author MAY set the JSON-LD properties `xRobotsTag` and/or
`SPC`.  For example,


```
{
  "@context": ["https://www.w3.org/ns/activitystreams", "https://fep.example/ns/privacyHeaders"],
  "type": "Note",
  "content": "Don't surveil me bro",
  "xRobotsTag": "noai",
  "SPC": 1
}
```

A recipient SHOULD process the content and metadata of the object as if the recipient had received it
over an HTTP connection with the corresponding header.

 * `SPC`: process as if the object has been received in an HTTP connection with the `Sec-GPC` HTTP request header.

 * `xRobotsTag`: process as if the object had been received with an `X-Robots-Tag` HTTP response header
    with a value equal to this property.


## Deployment considerations

The existing Global Privacy Control ([GPC]) standard allows for individuals in an increasing number of
jurisdictions to pass a legally binding opt-out preference signal indicating a commonly
held privacy norm.  However, GPC is implemented as an HTTP request
header, which makes it only works from
client to server. This limits the ability of users to pass a privacy opt-out
in a federated communications medium. A
federated system where a user's information may be passed from
one server to another is currently at a disadvantage in serving
users who wish to opt out, because the opt-out preference signal does not travel with the information
to which it applies.  SPC is intended to have the same effects as GPC, but for
cases where the user does not have a direct HTTP connection to the recipient of some personal 
information. Adding the SPC opt-out to ActivityPub would bring parity with centralized social sites.

Fediverse instances should make SPC configurable by users, and reflect a user's
actual intent to opt out.  Fediverse instances should detect when a user has turned on
GPC or some other privacy opt-out or setting, and offer to apply SPC to that user's 
objects.

Content sharing platforms that operate in a centralized manner
are now frequently promoting their support for signaling opt-outs
to <q>generative AI</q> training, using the [noai] value for the X-Robots-Tag header. See [RobotsMeta] and [RobotExclusion].
In order for federated content sharing systems to provide an
attractive alternative to centralized ones, federated systems will
likely need to also offer an opt out signaling feature.


## Security considerations

This proposal depends on compliant processing of OOPSs by all actors.

Some jurisdictions require privacy OOPSs to be registered in order to have legal effect. In the event this
FEP is accepted, the author intends to pursue registration.


## 実装

None so far.



## Related proposal

[FEP-c118](fep-c118.md) suggests establishing
a content licensing framework. This may be possible by adapting this FEP to include the license header
from [WebLinking].


## 参考文献

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- DeviantArt team, [UPDATE All Deviations Are Opted Out of AI Datasets][noai], 2022
- Google Search Central, [Robots meta tag, data-nosnippet, and X-Robots-Tag specifications][RobotsMeta], undated
- M. Nottingham, [Web Linking][WebLinking], 2010
- Martijn Koster, [A Standard for Robot Exclusion][RobotExclusion], 1994
- Sebastian Zimmeck, Peter Snyder, Justin Brookman, Aram Zucker-Scharff, [Global Privacy Control][GPC], 2024


[ActivityPub]: https://www.w3.org/TR/activitypub/
[GPC]: https://privacycg.github.io/gpc-spec/
[noai]: https://www.deviantart.com/team/journal/UPDATE-All-Deviations-Are-Opted-Out-of-AI-Datasets-934500371
[RobotExclusion]: http://www.robotstxt.org/robotstxt.html
[RobotsMeta]: https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag
[WebLinking]: https://datatracker.ietf.org/doc/html/rfc5988

## 著作権
CC0 1.0 ユニバーサル (CC0 1.0) パブリック ドメイン

法律で認められる範囲において、この Fediverse 拡張提案の著者は、この作品に対するすべての著作権および関連する権利または隣接する権利を放棄しています。
