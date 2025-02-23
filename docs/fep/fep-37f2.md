---
slug: "37f2"
authors: bengo <https://mastodon.social/@bengo>
status: DRAFT
dateReceived: 2023-09-28
trackingIssue: https://codeberg.org/fediverse/fep/issues/184
discussionsTo: https://codeberg.org/fediverse/fep/issues/184
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-37f2.md)から翻訳に協力することができます。

# FEP-37f2: a policy for calls for consensus on SWICG group decisions

## Summary

A FEP proposing that W3C Social Web Incubator Community Group [harmonize](https://en.wikipedia.org/wiki/Harmonization_(standards)) its process with other W3C Groups as well as the Fediverse Enhancement Process on socialhub.activitypub.rocks by:
* posting Calls for Consensus on the SWICG mailing list public-swicg@w3.org
* engaging other SWICG fora like socialhub.activitypub.rocks (linked to as "Forum" from the [SWICG Webpage](https://www.w3.org/community/socialcg/))
* having a shared response period

## Introduction

The Social Web Incubation Community Group is missing an explicit decision-making policy, which essentially all other W3C community groups have to ensure asynchronous and healthy consensus mechanisms across timezones and participatory modes.

## Proposal

[W3C SWICG](SWICG) will seek to make decisions through consensus and due process, per the [W3C Process Document, §5.2.1 Consensus][w3c-process].

To afford asynchronous decisions and organizational deliberation, any resolution (including publication decisions) taken in a face-to-face meeting or teleconference will be considered provisional.

A call for consensus (CFC) will be issued for all resolutions via email to [public-swicg@w3.org][swicg-list] ([archives][swicg-list-archives]). The presence of formal resolutions will be indicated by a "CFC" prefix in the subject line of the email. Additional outreach to community venues for more affirmative consent is strongly encouraged. There will be a response period of 14 days. If no sustained objections are raised by the end of the response period, the resolution will be considered to have consensus as a resolution of the Community Group, i.e. a group decision.

All decisions made by the group should be considered resolved unless and until new information becomes available or unless reopened at the discretion of the Chairs or the Director.

This policy is an operational agreement per the [W3C Community and Business Group Process][cg-process].

[SWICG]: https://www.w3.org/community/socialcg/
[w3c-process]: https://www.w3.org/2023/Process-20230612/#Consensus
[swicg-list]: mailto:public-swicg@w3.org
[swicg-list-archives]: https://lists.w3.org/Archives/Public/public-swicg/
[cg-process]: https://www.w3.org/community/about/process/

## Context

### W3C Groups with Similar Decision Policies

These community groups and working groups have similar decision policies with tentative meeting resolutions and confirmation of calls for consensus via email:

* [WebAssembly Community Group Charter](https://webassembly.github.io/cg-charter/#decision)
* [Credentials Community Group Charter](https://www.w3.org/community/credentials/charter/) (see section "Transparency")
* [Web Extensions Community Group Charter](https://github.com/w3c/webextensions/blob/main/charter.md#decision-process)
* [Web of Things Interest Group Charter](https://www.w3.org/2021/12/wot-ig-2021.html#decisions)
* [HTML Working Group Charter](https://www.w3.org/2022/06/html-wg-charter.html#decisions)
* [Web Platform Working Group Charter](https://www.w3.org/2017/08/webplatform-charter.html#decisions)
* [Web Applications Working Group Charter](https://www.w3.org/2022/04/webapps-wg-charter.html#decisions)
* [Media Working Group Charter](https://www.w3.org/2023/06/media-wg-charter.html#decisions)
* [Web Performance Working Group Charter](https://www.w3.org/2021/02/webperf.html#decisions)
* [Service Workers Working Group Charter](https://www.w3.org/2023/01/sw-charter.html#decisions)
* [Verifiable Credentials Working Group Charter](https://www.w3.org/2022/06/verifiable-credentials-wg-charter.html#decisions)
* [JSON-LD Working Group Charter](https://www.w3.org/2018/03/jsonld-wg-charter.html#decisions)
* [WebAssembly Working Group Charter](https://www.w3.org/2020/03/webassembly-wg-charter.html#decisions)
* [Web Authentication Working Group Charter](https://www.w3.org/2022/04/webauthn-wg-charter.html#decisions)
* [Immersive Web Working Group Charter](https://www.w3.org/2022/07/immersive-web-wg-charter.html#decisions)
* [Web Payments Working Group Charter](https://www.w3.org/2017/08/webplatform-charter.html#decisions)
* [Devices and Sensors Working Group Charter](https://www.w3.org/2022/11/das-wg-charter.html#decisions)
* [Distributed Tracing Working Group Charter](https://www.w3.org/2023/05/distributed-tracing-wg-charter.html#decisions)
* [Web Editing Working Group Charter](https://www.w3.org/2023/09/webediting-charter-2023.html#decisions)
* [Internationalization Working Group Charter](https://www.w3.org/International/groups/wg/charter.html#decisions)
* [Publishing Maintenance Working Group Charter](https://www.w3.org/2023/06/pmwg-charter.html#decisions)
* [Solid Community Group Charter](https://www.w3.org/community/solid/charter/) (see section "Decision Policy")
* [Decentralized Identifier Working Group Charter](https://www.w3.org/2020/12/did-wg-charter.html#decisions)

Proposal processes on SWICG Forum with identical response period:

* [FEP-a4ed: The Fediverse Enhancement Proposal Process](https://socialhub.activitypub.rocks/t/fep-a4ed-the-fediverse-enhancement-proposal-process/1171/1)

### W3C Community Group Process

[W3C SWICG][SWICG] is a W3C Community Group (CG).

CGs are described in [their process document][cg-process] as follows (excerpted for concision):
> This document defines W3C Community Groups, where anyone may develop Specifications, hold discussions, develop tests, and so on, with no participation fee. …
> 
> Community Groups that develop specifications do so under policies designed to strike a balance between ease of participation and safety for implementers and patent holders …
> 
> A Community Group may adopt operational agreements&hellip; that establish the group’s scope of work, decision-making processes, communications preferences, and other operations. …
> 
> The following rules govern Community Group operational agreements:
> * They must be publicly documented.
> * They must be fair and must not unreasonably favor or discriminate against any group participant or their employer.
> * They must not conflict with or modify this Community and Business Group Process, the [Community Contributor License Agreement (CLA)][w3c-cg-cla], or the [Final Specification Agreement][w3c-cg-fsa]. …
>
> the Chair determines the means by which the group adopts and modifies operational agreements. The Chair must give actual notice to the participants of any material changes to the agreements. Participants may resign from the group if they do not wish to participate under the new agreements. …
>
> **Note**: W3C encourages groups adopt decision-making policies that promote consensus. …
>
> Each Community Group must have at least one Chair who is responsible for ensuring the group fulfills the requirements of this document as well as the group’s operational agreements.

[w3c-cg-cla]: https://www.w3.org/community/about/process/cla/
[w3c-cg-fsa]: https://www.w3.org/community/about/process/final/

## Related Reading

* [IETF RFC7282 On Consensus and Humming in the IETF][rfc7282]
* Doty, Nick, and Deirdre K. Mulligan. 2013. ["Internet Multistakeholder Processes and Techno-Policy Standards: Initial Reflections on Privacy at the World Wide Web Consortium"][doty-mulligan-2013] Journal on Telecommunications and High Technology Law 11.
* [Harmonization (standards), en.wikipedia.org](https://en.wikipedia.org/wiki/Harmonization_(standards))

[rfc7282]: https://datatracker.ietf.org/doc/html/rfc7282
[doty-mulligan-2013]: http://www.jthtl.org/content/articles/V11I1/JTHTLv11i1_MulliganDoty.PDF

### Editorial Notes

The title of this proposal was generated in line with norms established by [Content addressed vocabulary for extensions][content-addressed-extensions] and [FEP-a4ed: The Fediverse Enhancement Proposal Process][FEP-a4ed].

```bash
⚡ P='a policy for calls for consensus on SWICG group decisions'
⚡ echo "SWIP-$(echo -n "$P" | sha256sum | cut -c-4): $P"
SWIP-37f2: a policy for calls for consensus on group decisions
```

The 'SW' in 'SWIP' stands for 'Social Web'.

[content-addressed-extensions]: https://socialhub.activitypub.rocks/t/content-addressed-vocabulary-for-extensions/539/1
[FEP-a4ed]: https://socialhub.activitypub.rocks/t/fep-a4ed-the-fediverse-enhancement-proposal-process/1171#proposal-title-and-identifier-5

This proposal was initially published at:
* https://socialweb.coop/SWIP/37f2/a-policy-for-calls-for-consensus-on-swicg-group-decisions/

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Proposal have waived all copyright and related or neighboring rights to this work.
