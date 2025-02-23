---
slug: "8c3f"
authors: Diogo Peralta Cordeiro <mail@diogo.site>, Phablulo Joel <phablulo@gmail.com>
status: WITHDRAWN
dateReceived: 2022-01-18
dateWithdrawn: 2023-10-27
trackingIssue: https://codeberg.org/fediverse/fep/issues/3
discussionsTo: https://codeberg.org/fediverse/fep/issues/3
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-8c3f.md)から翻訳に協力することができます。
# FEP-8c3f: Web Monetization


## Summary

Web Monetization federation via [ActivityPub].


## History

The ability to transfer money has been a long-standing omission from the web platform. As a result, the web suffers from a flood of advertising and corrupt business models. Web Monetization provides an open, native, efficient, and automatic way to compensate creators, pay for API calls, and support crucial web infrastructure.

[Web Monetization] is being proposed as a W3C standard at [the Web Platform Incubator Community Group](https://discourse.wicg.io/t/proposal-web-monetization-a-new-revenue-model-for-the-web/3785).


## Requirements

In GNU social this is [implemented on a plugin](https://code.undefinedhackers.net/GNUsocial/gnu-social/src/branch/v3/plugins/WebMonetization/WebMonetization.php#L243) using an extra property `gs:webmonetizationWallet` on the actor object.


### Example

```json
{
  "type": "Person",
  "streams": [],
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    {
      "gs": "https://www.gnu.org/software/social/ns#"
    },
    {
      "webmonetizationWallet": {
        "@id": "gs:webmonetizationWallet",
        "@type": "@id"
      }
    }
  ],
  "id": "https://social.gnusocial.rocks/actor/42",
  "inbox": "https://social.gnusocial.rocks/actor/42/inbox.json",
  "outbox": "https://social.gnusocial.rocks/actor/42/outbox.json",
  "following": "https://instance.gnusocial.test/actor/42/subscriptions",
  "followers": "https://instance.gnusocial.test/actor/42/subscribers",
  "preferredUsername": "alice",
  "name": "Alyssa P.Hacker",
  "url": "https://social.gnusocial.rocks/@alice",
  "webmonetizationWallet": "$wallet.example.com/alice"
}
```

## About the value of `gs:webmonetizationWallet`

That string is the same as the example one in [Web Monetization specification](https://webmonetization.org/specification.html#web-monetization-meta-tag)
and it consists on a [payment pointer](https://paymentpointers.org/#syntax/).

> Payment Pointers start with a `$` character to distinguish them from other identifiers and make it obvious that they are related to payments. To convert a Payment Pointer to a URL the $ is replaced with the standard prefix of a secure URL, `https://`.


## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [Web Monetization] Adrian Hope-Bailie, Ben Sharafian, Nick Dudfield, [Web Monetization](https://webmonetization.org/), 2021


## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
