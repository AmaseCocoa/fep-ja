---
slug: "07d7"
authors: Jennifer Moore <contact@jenniferplusplus.com>
status: DRAFT
dateReceived: 2023-09-22
trackingIssue: https://codeberg.org/fediverse/fep/issues/171
discussionsTo: https://codeberg.org/fediverse/fep/issues/171
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-07d7.md)から翻訳に協力することができます。
# FEP-07d7: A Custom URL Scheme and Web-Based Protocol Handlers for Linking to ActivityPub Resources

- [Summary](#summary)
- [Motivation](#motivation)
- [Context](#context)
- [Requirements](#requirements)
  - [1. Definitions](#1-definitions)
  - [2. URI Scheme](#2-uri-scheme)
    - [2.1 Intents](#21-intents)
    - [2.2 Example links using the scheme](#22-example-links-using-the-scheme)
  - [3. Protocol Handlers](#3-protocol-handlers)
    - [3.1 Retrieving hyperlinked objects](#31-retrieving-hyperlinked-objects)
    - [3.2 Displaying hyperlinked objects](#32-displaying-hyperlinked-objects)
    - [3.3 Servers](#33-servers)
    - [3.4 Clients](#34-clients)
  - [4. Safety and Security](#4-safety-and-security)
- [References](#references)
- [Copyright](#copyright)


## Summary

This specification addresses sometimes difficult interactions with ActivityPub resources hosted on remote servers. It defines a custom URL scheme which can be used by custom web-based protocol handlers to route hyperlinks to those resources to the user's preferred server. It additionally advises when ActivityPub servers can include these links in HTML views they generate, and how clients and servers can implement those web-based protocol handlers.

## Motivation

When a person follows a link to some ActivityPub powered site, the browser will navigate to the resource on that site. This is often *not* what the person would have prefered when they have an account on a different ActivityPub aware site. In order to interact with the linked resource from their own account, the person must separately open their home server and search for the resource. This is an awkward and sometimes confusing process, especially for people who are not already familiar with the idosyncracies of navigating a federated social network.

Web-based protocol handling is a feature of modern web browsers. This allows a site to register with the user's browser as a handler for custom URI schemes. The browser will send links using that scheme to the registerd handler. This way, it's possible to direct links to the user's preferred server or application. The handler can retrieve the resource and provide familiar and appropriate presentation. Native ActivityPub clients can also take advantage of these URIs to provide similar handling. 

## Context

Similar proposals such as [Fedilinks][FedilinksRef] and [Mastodon Issue 19679][Mastodon19679] have been made, and were even [briefly implemented in Mastodon][MastodonRemove]. There seems to be some concensus that custom protocol handlers have the technical capability to solve the problem of difficult interactions with cross-instance objects. There is also ample prior art for this approach. `mailto:` and `tel:` are common examples from web standards. Zoom uses a custom`zoommtg:` protocol to launch their desktop app, and Apple uses `itms:` to launch iTunes.

It seems the main impediment to early attempts was a perception of poor UX and limited adoption. This seems to be mostly a chicken-and-egg problem. That will always be a problem, until it's not. In this case, it seems it would be helpful to let standards drive implementation, as the opposite hasn't ocurred. This proposal also recommends behavior that is compatible with gradual adoption.

Some [similar proposals][Mastodon14187] have also disussed [emphasizing outcome][Issue1], rather than content. Encapsulating action is an understandable desire, and this proposal attempts to facilitate that. But the protocol aspect is firmly focussed on interaction with ActivityPub objects, without making any assumptions about the design or capabilities of current or future ActivityPub services. The hope is that this gives the resulting implementations better longevity. Documents can outlive software, and people should still be able to interact with them in useful ways. This also keeps the proposal scoped just to ActivityPub concerns, without imposing on other standards.


## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this specification are to be interpreted as described in [RFC-2119].

### 1. Definitions

This proposal refers to ActivityPub objects as viewed and represented in multiple ways from multiple servers. For clarity, these are the terms that will be used to describe those scenarios.

Origin server means the server that has authority for the ActivityPub object. This is the server that hosts and controls access to the object, and will recieve requests for the URI used as the object's ID.

Handler is any software that handles the `web+activitypub:` scheme and protocol. Typically either an ActivityPub server where the person has an account, or a client application they have installed on their device.

ActivityPub object means the JSON-LD document representing an ActivityPub Object, as described in the [ActivityPub][ActivityPubRef] and [ActivityStreams][ActivityStreamsRef] specifications.

HTML representation means an HTML or other document rendered to display the ActivityPub object for human use.

A client is any software that provides a human-friendly presentation of ActivityPub objects, or can interact with an ActivityPub server. For example, this could be a server's web UI or a native mobile app. This software does not necessarily utilize the C2S profile of the ActivityPub spec.

### 2. URI Scheme

When creating hyperlinks to ActivityPub resources, individuals and applications SHOULD include a link using the custom `web+activitypub:` scheme. This scheme can be handled by web-based or native handlers registered with browsers by end-users. Because there's no guarantee that a given browser will have any registered handler for this scheme, these links SHOULD NOT be used in place of a link that refers to the resource by ID or an alternative HTML representation of it. Links using the `web+activitypub:` scheme SHOULD be used as an addition to those more canonical links.

The address provided using the `web+activitypub:` scheme SHOULD be the same as the referenced ActivityPub object's ID. The address MAY instead be for an alternative human-readable address, or for an HTML representation of the object, as normal for the origin server. Applications using or generating these links should be aware that not all clients will be able to dereference those alternative or human readable addresses. The most interoperable choice is to use the object's canonical ActivityPub ID.

The linked address MUST replace the scheme used with `web+activitypub:`. That means the link MUST NOT simply prepend `web+activitypub:` onto a preexisting `https:` or other scheme, as this would violate the generic URI syntax specified by [RFC-3986][RFC-3986]

#### 2.1 Intents

A `web+activitypub:` link MAY encapsulate an intent for an activity to be performed on or with the linked object. For example, a pre-assembled link could be used to directly Follow an actor or to Announce an object. An encapsulated intent MUST be represented by appending a query string to the target URI. It's possible that the target URI already includes a query string. In this case, to encapsulate an intent, the intent MUST append a new query parameter to the existing query string. A `web+activitypub:` link MUST NOT encapsulate more than one intent.

Including an encapsulated intent is OPTIONAL. Responding to the intent is also OPTIONAL. The creator of the intent should keep in mind that the handler MAY support intents, but does not have to. Also keep in mind that the handler MUST provide a mechanism for the user to confirm or decline to perform the intended action. The use or purpose of the `web+activitypub:` link MUST NOT require that the intended activity is actually performed.

The format of the query parameter to encapsulate an intent is `intent=<activity>` where `<activity>` is any of the following activities described by the [Activity Streams vocabulary][ActivityVocabulary]. The activity string SHOULD be all lowercase. It's permissible to use any casing. But, there are many systems in the wild which might intercept or preprocess the URIs in a `web+activitypub:` link, and not all of those will respect the original casing.

* add
* announce
* arrive
* create
* follow
* invite
* like

The `web+activitypub:` link SHOULD NOT encapsulate an intent for any activity not listed above. In addition, the link MUST NOT encapsulate an intent for any of the following activities. Implementers should exercise discretion regarding activities added as extensions to the ActivityPub specification. You should err on the side taking no action where there is doubt about the safety or security of doing so.

* block
* delete
* dislike
* flag
* ignore
* leave
* move
* offer
* remove

For security and privacy, `web+activitypub:` links MUST NOT include a username or password component. They also SHOULD NOT be relative URIs.

#### 2.2 Example links using the scheme

Below are some examples of `web+activitypub:` hyperlinks that reflect some expected uses.

**One-click check in**

A link to enable one-click check in to pick up a shopping order.

`<a hfref="web+activitypub:shopping.example/pickup/12345?intent=arrive">Check in to pick up your order</a>`

**One-click follow**

A link to enable one-click following from another website. The handler must not perform the follow activity unless the user confirms it.

`<a href="web+activitypub:uss-enterprise.example/user/picard?intent=follow>Follow me on the fediverse</a>`

**Link with no intent**

A link to view a blog post published as an ActivityPub Article

`<a href="web+activitypub:my-blog.example/article/write-your-first-fep>Read the article on your home instance</a>`

### 3. Protocol Handlers

#### 3.1 Retrieving hyperlinked objects

ActivityPub applications that can retrieve remote objects and generate an HTML display MAY also implement a web-based protocol handler for the `web+activitypub:` URI scheme. Applications MAY use other available registration mechanisms, such as Android inents. The handler MUST attempt to load the specified URI as an `application/ld+json` document. If the origin does not respond with an ActivityPub object, the handler SHOULD fallback to other methods it uses to locate ActivityPub objects, such as webfinger.

When resolving the given URI, the handler MUST ignore any provided username or password component, and it MUST exclude those components from the requests it makes to retrieve the object. If the given URI includes a query string with an intent parameter, the handler MUST exclude the intent parameter from the requests made to retrieve the object.

The handler MUST NOT attempt to retrieve objects from a relative URI. There's likely no good reason for a `web+activitypub:` link to target a relative URI, and there is some risk it could be used to facilitate scams or phishing attempts by making it appear some third party has access to data they shouldn't.

The handler MUST perform the same sanitization or other safe handling of untrusted URIs as it normally would. For example, there's likely no good reason for a production system to try to load resources from localhost, or using an ip address, rather than a hostname. Doing so could also facilitate phishing or scams.

When retreiving resources identified by a `web+activitypub:` scheme, the handler SHOULD assume the origin uses HTTPS. The handler MAY attempt to use other schemes, such as `did:`. The handler MAY make this determination using any heuristic or algorithm the developers choose.

Other than the above considerations, the handler MUST attempt to resolve the URI exactly as given.

#### 3.2 Displaying hyperlinked objects

After successfully retrieving the referenced object, the handler SHOULD produce an HTML representation of the object and present it for the user. The handler MUST escape and sanitize the content of the object before displaying it, to prevent cross-site scripting attacks.

The handler SHOULD display the object in the normal way for the application, and in the normal context. For example, if the application normally displays replies alongside an object, it SHOULD do so in this case as well.

The handler SHOULD enable the user to interact with the object normally, such as to generate Like or Announce activities referring to it.

When the provided `web+activitypub:` URI includes an encapsulated intent, the handler MAY prompt the user to perform that intended activity. The handler MUST NOT perform that activity without explicit confirmation from the user. The handler MUST NOT perform or offer to perform the intended activity if it is one of the ones listed below. These activities present an elevated risk of disrupting the user's social connections or enlisting them in malicious activities. The handler MAY perform other activities, but SHOULD do so with care. `web+activitypub:` links are usually created by third parties. Be mindful that there is some risk of exposing sensitive information or facilitating harmful behavior when third parties are allowed to direct other people's actions.

The handler MUST NOT perform any of these activities in response to an ecapsulated intent:

* block
* delete
* dislike
* flag
* ignore
* leave
* move
* offer
* remove

#### 3.3 Servers

ActivityPub servers MAY offer users an option to register with their browser as a `web+activitypub:` handler. Servers SHOULD NOT do this unless the user has authenticated with the server. The server SHOULD also offer a similar option to de-register as a handler. Servers SHOULD NOT attempt to register as a handler unless initiated by the user in some way. Servers MUST NOT attempt to automatically register as a handler again after a user has declined to allow it.

#### 3.4 Clients

Clients MAY offer users an option to register as handlers for `web+activitypub:` links. They SHOULD use whatever mechanism is appropriate to the client to do so. For instance, Android apps would likely use the intent system, and Windows apps could set the appropriate registry flags. Clients SHOULD provide a mechanism to de-register as a handler, where possible. Clients MAY defer this capability to the operating system as appropriate.

If a client will retrieve the linked object itself, the client MUST adhere to the same specifications from section 3.1 and 3.2 as any other application would.

### 4. Safety and Security

Because there is no way to know or control which application will ultimately handle any `web+activitypub:` link, creators SHOULD NOT include private or sensitive information in the link. Creators MUST NOT include any type of authenticating token or credential in the link. And handling applications themselves MUST apply all the normal and appropriate access controls when retrieving and displaying resources linked this way.


## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPubRef], 2018
- [Web-based Protocol Handlers] Mozilla Developer Network, [Web-based Protocol Handlers][HandlersRef]
- [HTML Living Standard] WHATWG, [HTML], 2023
- [Fedilinks] Fedilinks Authors, [Fedilinks][FedilinksRef]

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.


[RFC-2119]: https://www.rfc-editor.org/rfc/rfc2119
[ActivityPubRef]: https://www.w3.org/TR/activitypub/
[HandlersRef]: https://developer.mozilla.org/en-US/docs/Web/API/Navigator/registerProtocolHandler/Web-based_protocol_handlers
[HTML]: https://html.spec.whatwg.org/multipage/system-state.html#custom-handlers
[ActivityStreams]: https://www.w3.org/TR/activitystreams-core/
[ActivityVocabulary]: https://www.w3.org/TR/activitystreams-vocabulary
[MastodonRemove]: https://github.com/mastodon/mastodon/pull/8127
[Mastodon14187]: https://github.com/mastodon/mastodon/issues/14187
[Mastodon19679]: https://github.com/mastodon/mastodon/issues/19679
[Issue1]: https://codeberg.org/fediverse/fediverse-ideas/issues/1
[FedilinksRef]: https://fedilinks.org/spec/en/6-The-web-ap-URI
[RFC-3986]: https://www.rfc-editor.org/rfc/rfc3986
