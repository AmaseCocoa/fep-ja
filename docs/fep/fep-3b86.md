---
slug: "3b86"
authors: Ben Pate \<@benpate@mastodon.social\>
status: DRAFT
dateReceived: 2024-04-19
relatedFeps: FEP-4adb, FEP-888d
trackingIssue: https://codeberg.org/fediverse/fep/issues/302
discussionsTo: https://codeberg.org/fediverse/fep/issues/302
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-3b86.md)から翻訳に協力することができます。

# FEP-3b86: Activity Intents

## Summary
"Activity Intents" extend the capabilities of an ActivityPub server beyond a user's outbox, and enable direct interactions with content on the wider social web. They do this by publishing a machine-readable list of public URLs where users can perform key activities (such as `Follow`, `Like`, or `Announce`)  allowing other websites to initiate remote social interactions without cumbersome copying/pasting of URL strings.

## 1. Requirements
The key words "MUST", "SHOULD", and "MAY" are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119.html).

For the purposes of this document, a "Home Server" is the location where a user is logged in, and that publishes the Actor's Activity Intents via WebFinger.  A "Remote Server" is another location on the Internet that the user wants to interact with, and that will request/receive those intents via WebFinger.

## 2. History
Most centralized social media services have widgets that allow users on the wider Internet to interact with their social services.  These include "like" and "share" buttons that third-party websites embed into their content, and link users back to their corresponding social media account.

This is difficult to achieve in a federated environment.

There have been other attempts to make a "Share on Mastodon" button that performs a similar action on the Fediverse.  But these tasks are implemented differently by different Fediverse servers.  For example, Mastodon uses `/share`, while Hubzilla uses `/rpost`.  Parameters to each application are often different, using variations of `text`, `title`, `url`, and other values.

The lack of a unified standard has led developers to hard-code endpoints for each distinct application.  This is brittle and vulnerable to changes by server authors.  It also pins those URL endpoints, making them difficult for servers to change in the future without breaking an unknown number of "share" buttons out in the wild.

What is needed is a systematic way for each server to announce the endpoint URLs that they support.

## 3. Activity Intents
In the most basic terms, Activity Intents expand on the common Fediverse use of [WebFinger](https://webfinger.net) in [FEP-4adb](https://w3id.org/fep/4adb) to include mappings between any [Activity Type](https://www.w3.org/TR/activitystreams-vocabulary/#activity-types) and the URL endpoint where that user can perform it. This expands and standardizes the "remote follow" workflow that was used by oStatus protocol, but has not been fully implemented by newer Fediverse applications and no longer has a public specification document.

When generating a WebFinger result for a user account, servers supporting Activity Intents SHOULD respond with one or more intent links in the ["links" property](https://datatracker.ietf.org/doc/html/rfc7033#section-4.4.4). Activity Intent links MUST have `rel` and `href` properties.  All others properties are ignored.

### 3.1. Example
Here is an example response from a WebFinger server which includes three Activity Intents appended to the end of its `links` property.

```json
{
  "subject": "acct:benpate@mastodon.social",
  "aliases": [
    "https://mastodon.social/@benpate",
    "https://mastodon.social/users/benpate"
  ],
  "links": [
    {
      "rel": "http://webfinger.net/rel/profile-page",
      "type": "text/html",
      "href": "https://mastodon.social/@benpate"
    },
    {
      "rel": "self",
      "type": "application/activity+json",
      "href": "https://mastodon.social/users/benpate"
    },
    {
      "rel": "http://ostatus.org/schema/1.0/subscribe",
      "template": "https://mastodon.social/authorize_interaction?uri={uri}"
    },
    {
      "rel": "https://w3id.org/fep/3b86/Follow",
      "href": "https://mastodon.social/authorize_interaction?uri={object}"
    },
    {
      "rel": "https://w3id.org/fep/3b86/Create",
      "href": "https://mastodon.social/share?uri={object}"
    },
    {
      "rel": "https://w3id.org/fep/3b86/Like",
      "template": "https://mastodon.social/intents/like?id={object}"
    }
  ]
}
```

### 3.2. Required Link Properties

**rel**: Activity Intents use the `https://w3id.org/fep/3b86/*` (as described in [FEP-888d](https://w3id.org/fep/8886)) to designate the kind of activity intent, where `*` represents the particular Activity the user intends to perform.  These relations -- such as `https://w3id.org/fep/3b86/Follow`, and `https://w3id.org/fep/3b86/Create` -- are listed in detail below.

**href**: Links also use the standard WebFinger `href` property,  but also include wrapped replace values (as in `{uri}` or `{name}`) to designate parameters to be injected by the caller.  (note: this is in contrast to the non-standard `template` property that was used by oStatus)

Parameter names are chosen to correspond with [Activity Vocabulary properties](https://www.w3.org/TR/activitystreams-vocabulary/#properties) and may differ from parameters used by pre-existing implementations.

To prevent unrecognized properties from corrupting a workflow:
* Remote servers MUST be able to replace all recognized values with the appropriate string.
* Remote servers MUST replace unrecognized values with an empty string.

### 3.3 Template Parameters
In all cases, Activity Intents intentionally use the property names defined in the W3C standard [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary). However, intents must fit into a URL, and must use commonly understood fields.  So, Intent parameters can only use IDs -- URL references to JSON-LD resources available elsewhere on the Internet.

In addition, remote servers MUST [Percent Encode](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1) all values replaced in the href template.

### 3.4 Workflow Actions
Activity Intents MAY include additional query parameters `on-success` and `on-cancel` that allow home servers to return users to their original workflow on the remote server:

**on-success** - If present in the link template, this value identifies the action that the home server SHOULD take once the Activity Intent workflow is complete.  If this parameter is missing, then the resulting page is left up to the home server to choose.

**on-cancel** - If present in the link template, this value identifies the action that the home server SHOULD take if they abort the Activity Intent workflow.  If this parameter is missing, then the resulting page is left up to the home server to choose.

#### Action: (close)
If the value of either `on-success` or `on-cancel` parameter is the string `(close)` then the home server can assume that it is running in a pop-up window, and SHOULD close the current window.

#### Action: Redirecting to a URL
If the value of either `on-success` or `on-cancel` query parameter is a valid URL, then the home server SHOULD redirect clients to the provided URL.  When redirecting to a new URL, the home server MUST use an interstitial page that notified the user that they are being redirected, and displays the URL of the new page to them.

IMPORTANT! [Unvalidated Redirects](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html) are a documented vulnerability that opens the home server up to phishing attacks. 

To avoid a potential security vulnerability when handling `on-success` and `on-cancel` workflows, it is important that the home server: "Force all redirects to first go through a page notifying users that they are going off of your site, with the destination clearly displayed, and have them click a link to confirm." ([owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html#preventing-unvalidated-redirects-and-forwards))

### 3.5 Endpoint Expectations
The user's home server is a trusted environment that manages the user's sign-in status along with the rest of their social inbox and outbox.  When the remote server links to an Activity Intent provided by the home server, the layout, fields, and UI are all determined by the home server

Remote servers MAY open Activity Intent links in many different environments, such as:
1. a full browser window
2. a small pop-up window
3. a mobile app HTML view
4. or other constrained environments.

Home servers SHOULD build their UI with minimal assumptions so that the endpoint will fit well into as many environments as possible.  Home servers may not be aware f the remote server opens the Activity Intent link in a separate popup window, so remote servers SHOULD include `on-success` and `on-cancel` parameters that redirect back to its own pages so that it can close the pop-up itself.

## 4. Intent Definitions

### 4.1. Accept Intent
This intent corresponds to the ActivityStreams [Accept activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-accept) and is defined using the link relation `https://w3id.org/fep/3b86/Accept`.

The Accept intent publishes the API endpoint where the current user can "accept" the designated object.

#### 4.1.1. Parameters
* `{object}` - ID of the object that the user will accept when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.1.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Accept",
	"href": "https://server.org/intents/accept?objectId={object}"
}
```

### 4.2. Add Intent
This intent corresponds to the ActivityStreams [Add activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-add) and is defined using the link relation `https://w3id.org/fep/3b86/Add` .

The Add intent publishes the API endpoint where the current user can add an object to the designated collection..

#### 4.2.1. Parameters
* `{object}` ID of the object that the user will add when they use this workflow.
* `{target}` ID of the collection being added to.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.2.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Add",
	"href": "https://server.org/intents/add?objectId={object}&targetId={target}"
}
```

### 4.3. Announce Intent
This intent corresponds to the ActivityStreams [Announce activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-announce) and is defined using the link relation `https://w3id.org/fep/3b86/Announce`.

The Announce intent publishes the API endpoint where the current user can announce, or "boost" the provided document in their home server.

#### 4.3.1. Parameters
* `{object}` - ID of the document that the user will boost when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.3.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Announce",
	"href": "https://server.org/intents/announce?objectId={object}"
}
```

### 4.4. Arrive Intent
This intent corresponds to the ActivityStreams [Arrive activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-arrive) and is defined using the link relation `https://w3id.org/fep/3b86/Arrive`.

The Arrive intent publishes the API endpoint where the current user can indicate that they have arrived at a particular location from their home server.

#### 4.4.1. Parameters
* `{location}` - ID of the location object where the user will mark as "arrived" when they use this workflow.
 * `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.4.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Arrive",
	"href": "https://server.org/intents/arrive?hotelId={location}"
}
```

### 4.5. Block Intent
This intent corresponds to the ActivityStreams [Block activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-block) and is defined using the link relation  `https://w3id.org/fep/3b86/Block`.

The Block intent publishes the API endpoint where the current user can block the provided object from their home server.

#### 4.5.1. Parameters
* `{object}` - ID of the object (document, user, etc) that the user will block when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.5.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Block",
	"href": "https://server.org/intents/block?userId={object}"
}
```

### 4.6. Create Intent
This intent corresponds to the ActivityStreams [Create activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-create) and is defined using the link relation `https://w3id.org/fep/3b86/Create`.

The Create intent publishes the API endpoint where the current user can create a new post in their own outbox.

This is similar to the existing "share" endpoints supported by several Fediverse apps, where the user can create a new post in their inbox starting with some pre-populated content.

This Intent differs slightly from others in that it does not take IDs/URLs of other objects, but parameters to pre-populate into a new object to be created by the user.  

#### 4.6.1. Parameters
* `{content}` - Text content to pre-populate into the created object.
* `{type}` - (optional) Type of object to create (Note, Article, etc). Home servers can determine whether to use or ignore this value.
* `{name}` - (optional) Name to pre-populate into the created object.
* `{summary}` - (optional) Summary to pre-populate into the created object.
* `{inReplyTo}` - (optional)The ID of the ActivityStreams Document that the user is replying to.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.6.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Create",
	"href": "https://mastodon.social/share?text={content}"
}
```

### 4.7. Delete Intent
This intent corresponds to the ActivityStreams [Delete activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-delete) and is defined using the link relation  `https://w3id.org/fep/3b86/Delete`.

The Delete intent publishes the API endpoint where the current user can initiate a "delete" request.

#### 4.7.1. Parameters
* `{object}` - ID of the object that the user will delete when they use this workflow.
* `{origin}`- (optional) ID of the collection or context that the object will be deleted from.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.7.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Delete",
	"href": "https://server.org/intents/Delete?objectId={object}"
}
```

### 4.8. Dislike Intent
This intent corresponds to the ActivityStreams [Dislike activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-dislike) and is defined using the link relation  `https://w3id.org/fep/3b86/Dislike`.

The Dislike intent publishes the API endpoint where the current user can initiate a "dislike" request.

#### 4.8.1. Parameters
* `{object}` - ID of the document that the user will dislike when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.8.2. Example

```json
{
	"rel": "https://w3id.org/fep/3b86/Dislike",
	"href": "https://server.org/intent/dislike?objectId={object}"
}
```

### 4.9. Flag Intent
This intent corresponds to the ActivityStreams [Flag activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-flag) and is defined using the link relation  `https://w3id.org/fep/3b86/Flag`.

The Flag intent publishes the API endpoint where the current user can initiate a "flag" request, which is used to report inappropriate content.

#### 4.9.1. Parameters
* `{object}` - ID of the object (document, user, etc) that the user will flag when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.9.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Flag",
	"href": "https://server.org/intent/flag?objectId={object}"
}
```

### 4.10. Follow Intent
This intent corresponds to the ActivityStreams [Follow activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-follow) and is defined using the link relation  `https://w3id.org/fep/3b86/Follow`.

The Follow intent publishes the API endpoint where the current user can initiate a "follow" request.  This is similar to the [remote follow](https://www.hughrundle.net/how-to-implement-remote-following-for-your-activitypub-project/) workflow defined by oStatus that is still supported at various levels by several Fediverse apps, but is no longer formally documented.

#### 4.10.1. Parameters
* `{object}` - ID of the actor that the user will follow when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.10.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Follow",
	"href": "https://mastodon.social/authorize_interaction?uri={object}"
}
```

### 4.11. Ignore Intent
This intent corresponds to the ActivityStreams [Ignore activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-ignore) and is defined using the link relation  `https://w3id.org/fep/3b86/Ignore`.

The Ignore intent publishes the API endpoint where the current user can initiate an "ignore" request, which is similar to a Block, and is used to ignore or mute various actors and objects online

#### 4.11.1 Parameters
* `{object}` - ID of the object that the user will mark "ignored" when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.11.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Ignore",
	"href": "https://server.org/intents/ignore?objectId={object}"
}
```

### 4.12. Invite Intent
This intent corresponds to the ActivityStreams [Invite activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-invite) and is defined using the link relation  `https://w3id.org/fep/3b86/Invite`.

The Invite intent publishes the API endpoint where the current user can initiate an "invite" request, which is similar to an Offer, and is used to extend an invitation for the object to the target

#### 4.12.1. Parameters
* `{target}` - ID of the actor who will receive the invitation.
* `{object}` - ID of the object (event, group, etc) that the actor will be invited to.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.12.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Invite",
	"href": "https://server.org/intents/invite?actorId={object}&eventId={target}"
}
```

### 4.13. Join Intent
This intent corresponds to the ActivityStreams [Join activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-join) and is defined using the link relation  `https://w3id.org/fep/3b86/Join`.

The Join intent publishes the API endpoint where the current user can initiate a "join" request.

#### 4.13.1. Parameters
* `{object}` - ID of the object that the user will join when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.13.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Join",
	"href": "https://server.org/intents/join?objectId={object}"
}
```

### 4.14. Leave Intent
This intent corresponds to the ActivityStreams [Leave activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-leave) and is defined using the link relation  `https://w3id.org/fep/3b86/Leave`.

The Leave intent publishes the API endpoint where the current user can initiate a "leave" request.

#### 4.14.1. Parameters
* `{object}` - ID of the object that the user will leave when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.14.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Leave",
	"href": "https://server.org/intents/leave?objectId={object}"
}
```

### 4.15. Like Intent
This intent corresponds to the ActivityStreams [Like activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-like) and is defined using the link relation `https://w3id.org/fep/3b86/Like`.

The Like intent publishes the API endpoint where the current user can like the current document.

#### 4.15.1. Parameters
* `{object}` - ID of the object that the user will mark as "liked" when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.15.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Like",
	"href": "https://server.com/intents/like?objectId={object}"
}
```

### 4.16. Listen Intent
This intent corresponds to the ActivityStreams [Listen activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-listen) and is defined using the link relation  `https://w3id.org/fep/3b86/Listen`.

The Listen intent publishes the API endpoint where the current user can initiate a "listen" request.

#### 4.16.1. Parameters
* `{object}` - ID of the object that the user will mark as "listened" to when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.16.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Listen",
	"href": "https://server.org/intents/listen?objectId={object}"
}
```

### 4.17. Move Intent
This intent corresponds to the ActivityStreams [Move activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-move) and is defined using the link relation  `https://w3id.org/fep/3b86/Move`.

The Move intent publishes the API endpoint where the current user can initiate a "move" request.

#### 4.17.1. Parameters
* `{object}` - ID of the object that the user will move when they use this workflow.
* `{target}` - ID of the collection that the object will be moved to.
* `{origin}` - (optional) ID of the collection that the object will be moved from.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.17.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Move",
	"href": "https://server.org/intents/move?objectId={object}&destId={target}"
}
```

### 4.18. Offer Intent
This intent corresponds to the ActivityStreams [Offer activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-offer) and is defined using the link relation  `https://w3id.org/fep/3b86/Offer`.

The Offer intent publishes the API endpoint where the current user can initiate an "offer" request.

#### 4.18.1. Parameters
* `{object}` - ID of the object that the user will offer when they use this workflow.
* `{target}` - ID of the actor that will receive the offer.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.18.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Offer",
	"href": "https://server.org/intents/offer?objectId={object}&to={target}"
}
```

### 4.19. Question Intent
This intent corresponds to the ActivityStreams [Question activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-question) and is defined using the link relation  `https://w3id.org/fep/3b86/Question`.

The Question intent publishes the API endpoint where the current user can initiate an "question" workflow.

#### 4.19.1. Parameters
* `{name}` - The "name" property to pre-populate into the question the user will ask when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.19.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Question",
	"href": "https://server.org/intents/question?name={name}"
}
```

### 4.20. Read Intent
This intent corresponds to the ActivityStreams [Read activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-read) and is defined using the link relation  `https://w3id.org/fep/3b86/Read`.

The Read intent publishes the API endpoint where the current user can initiate an "read" request.

#### 4.20.1. Parameters
* `{object}` - ID of the object that the user will mark as "read" when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.20.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Read",
	"href": "https://server.org/intents/object?objectId={object}"
}
```

### 4.21. Reject Intent
This intent corresponds to the ActivityStreams [Reject activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-reject) and is defined using the link relation  `https://w3id.org/fep/3b86/Reject`.

The Reject intent publishes the API endpoint where the current user can initiate an "reject" request.

#### 4.21.1. Parameters
* `{object}` - ID of the object that the user will reject when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.21.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Reject",
	"href": "https://server.org/intents/reject?offerId={object}"
}
```

### 4.22. Remove Intent
This intent corresponds to the ActivityStreams [Remove activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-remove) and is defined using the link relation  `https://w3id.org/fep/3b86/Remove`.

The Remove intent publishes the API endpoint where the current user can initiate a "remove" request.

#### 4.22.1. Parameters
* `{object}` - ID of the object that the user will remove when they use this workflow.
* `{target}` - (optional) The ID of the collection that the object will be removed from.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.22.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Reject",
	"href": "https://server.org/intents/reject?objectId={object}"
}
```

### 4.23. TentativeAccept Intent
This intent corresponds to the ActivityStreams [TentativeAccept activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-tentativeaccept), which itself is a specialization of the [Accept activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-tentativeaccept) and is defined using the link relation  `https://w3id.org/fep/3b86/TentativeAccept`.

The TentativeAccept intent publishes the API endpoint where the current user can initiate an "tentative accept" request, indicating that acceptance of the original offer is tentative.

#### 4.23.1. Parameters
* `{object}` - ID of the object that the user will tentatively accept when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.23.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/TentativeAccept",
	"href": "https://server.org/intents/tentativeAccept?objectId={object}"
}
```

### 4.24. TentativeReject Intent
This intent corresponds to the ActivityStreams [TentativeReject activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-tentativereject), which itself is a specialization of the [Reject activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-tentativereject) and is defined using the link relation  `https://w3id.org/fep/3b86/TentativeReject`.

The TentativeReject intent publishes the API endpoint where the current user can initiate an "tentative reject" request, indicating that rejection of the original offer is tentative.

#### 4.24.1. Parameters
* `{object}` - ID of the object that the user will tentatively reject when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.24.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/TentativeReject",
	"href": "https://server.org/intents/tentativeReject?objectId={object}"
}
```

### 4.25. Travel Intent
This intent corresponds to the ActivityStreams [Travel activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-travel) and is defined using the link relation  `https://w3id.org/fep/3b86/Travel`.

The Travel intent publishes the API endpoint where the user can initiate a "travel" request.

#### 4.25.1. Parameters
* `{target}` - (optional) The ID of the location that the actor will travel to.
* `{origin}` - (optional) The ID of the location that the actor will travel from.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.25.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Travel",
	"href": "https://server.org/intents/travel?from={origin}&to={target}"
}
```

### 4.26. Undo Intent
This intent corresponds to the ActivityStreams [Undo activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-undo) and is defined using the link relation  `https://w3id.org/fep/3b86/Undo`.

The Undo intent publishes the API endpoint where the current user can initiate a "travel" request.

#### 4.26.1. Parameters
* `{object}` - ID of the activity that the actor will undo.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect users if they abort the workflow.

#### 4.26.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Undo",
	"href": "https://server.org/intents/undo?activityId={object}"
}
```

### 4.27. Update Intent
This intent corresponds to the ActivityStreams [Update activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-update) and is defined using the link relation  `https://w3id.org/fep/3b86/Update`.

The Update intent publishes the API endpoint where the user can initiate an "update" request.

#### 4.27.1. Parameters
* `{object}` - ID of the object that the actor will update when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.27.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/Update",
	"href": "https://server.org/intents/update?objectId={object}"
}
```

### 4.28. View Intent
This intent corresponds to the ActivityStreams [View activity](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-view) and is defined using the link relation  `https://w3id.org/fep/3b86/Update`.

The View intent publishes the API endpoint where the current user can initiate an "update" request.

#### 4.28.1. Parameters
* `{object}` - ID of the object that the actor will mark as "viewed" when they use this workflow.
* `{on-success}` - (optional) URL to redirect the user to after the workflow completes.
* `{on-cancel}` - (optional) URL to redirect the user to if they abort the workflow.

#### 4.28.2. Example
```json
{
	"rel": "https://w3id.org/fep/3b86/View",
	"href": "https://server.org/intents/view?objectId={object}"
}
```

## 5.0. Security Considerations 

### 5.1 CSRF issues
It is important to reiterate some key security practices to prevent [Cross Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) vulnerabilities.

* Remote Servers MUST only send `GET` requests to Home Servers.
* Home Servers MUST NOT change data based on GET requests.
* Home Servers SHOULD protect these published endpoints by generating CSRF tokens and including them with every POST request.  This validates that the request originated on the user's home server, and was initiated by the user. See [OWASP Related Controls](https://owasp.org/www-community/attacks/csrf#related-controls) for in-depth discussion.

### 5.2 Open/Unvalidated Redirects
Activity Intents includes the ability for home servers to redirect browsers to new URLs based on the `on-success` and `on-cancel` query parameters.  If improperly implemented, this has the potential to expose an [Unvalidated Redirect](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html) vulnerability, which can open the home server up to phishing attack. 

To avoid a potential security vulnerability when handling `on-success` and `on-cancel` workflows, it is important that the home server: "Force all redirects to first go through a page notifying users that they are going off of your site, with the destination clearly displayed, and have them click a link to confirm." ([owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html#preventing-unvalidated-redirects-and-forwards))

There is another good description of this issue [OAuth 2.0 Security Best Current Practice § 4.11. Open Redirection](https://www.ietf.org/archive/id/draft-ietf-oauth-security-topics-27.html#name-open-redirection)

## 6.0. Remote Servers: The Rest of the Equation
This FEP provides the prerequisite information required for a "Home Server" publish Activity Intents for its Actors.  It does not specify how "Remote Servers" will use this information - i.e., how they implement "share" and "like" buttons in their content.

### 6.1. Remote Server Example
Here is a brief example of the workflow as implemented by a remote server:

1. A user visits a remote server (i.e., any address on the Internet where they do not have an ActivityPub account)
2. The remote server displays buttons such as "like" or "share" for a particular document or object.
3. When the user clicks on one of these links, the remote server checks to see if the user is already recognized.  This does not necessarily mean logged in, or authenticated, just whether or not the user has entered their home server credentials before.  True authentication on the remote server is not necessary for Activity Intents to work.
	1. If the user is not recognized, then they are prompted to enter their Fediverse ID (such as @benpate@mastodon.social).  The remote server can save this information for use in the future (preferably in a private data store, such as `localStorage`)
	2. If the user is recognized, this means they have already completed step 3.1, and can continue to step 4.
4. The remote server uses a standard WebFinger query to look up the user's Fediverse ID and searches for Activity Intents supported by the user's home server.
	1. If none are found, the remote server MAY try to substitute fallback values for known server types.
	2. If none are found (and no fallbacks substituted) the remote server SHOULD inform the user that their account is incompatible with the selected feature and halt the workflow.
5. The remote server replaces values into the designated href template and forwards the user to the assigned page on their home server.  This initiates the Activity Intent workflow on their home server.
6. When the user completes the workflow, the home server SHOULD use URL in the `on-success` parameter to redirect the user back to the correct page on the remote server.
	1. Similarly, if the user cancels the workflow, the home server SHOULD use the URL in the `on-cancel` parameter to redirect the user back to the correct page on the remote server.

### 6.2. Fallbacks for Unpublished Links
Remote servers MAY also account for applications that do not publish Activity Intents, but whose endpoints are still well known.  In this case, remote servers SHOULD use Activity Intents links if they are present, then fall back to older links (such as the oStatus `/authorize_interaction` endpoint) if they are present, then fall back to hard-coded values (such as the Mastodon `/share` endpoint) if no other values exist.

## 7.0. Implementations
* [Emissary](https://emissary.dev) will support this FEP once its syntax is finalized.
* Add your name to this list and win a cookie 🍪

## References
* [Activity Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary/#activity-types)
* [Twitter: Web Intents](https://developer.twitter.com/en/docs/twitter-for-websites/web-intents/overview)
* [Facebook: Share Button](https://developers.facebook.com/docs/plugins/share-button/)
* [Remote Follows](https://www.hughrundle.net/how-to-implement-remote-following-for-your-activitypub-project/) - Tutorial on the original oStatus protocol
* [Cross Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) - Wikipedia
* [Cross Site Request Forgery](https://owasp.org/www-community/attacks/csrf) - OWASP
* [FEP-888d](https://w3id.org/fep/8886) - FEP-specific namespaces
* [IETF RFC 7033](https://datatracker.ietf.org/doc/html/rfc7033) - WebFinger
* [IETF RFC 3896](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1) - Uniform Resource Identifier Generic Syntax
* [IEFT RFC 2119](https://tools.ietf.org/html/rfc2119.html) - Key words for use in RFCs to Indicate Requirement Levels
* [Unvalidated Redirects and Forwards Cheat Sheet - owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html#preventing-unvalidated-redirects-and-forwards))
* [OAuth 2.0 Security Best Current Practice](https://www.ietf.org/archive/id/draft-ietf-oauth-security-topics-27.html)

## Copyright
CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
