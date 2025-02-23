---
slug: "db0e"
authors: Gregory Klyushnikov <activitypub@grishka.me>
status: DRAFT
dateReceived: 2024-05-03
relatedFeps: FEP-400e
trackingIssue: https://codeberg.org/fediverse/fep/issues/313
discussionsTo: https://codeberg.org/fediverse/fep/issues/313
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-db0e.md)から翻訳に協力することができます。
# FEP-db0e: Authentication mechanism for non-public groups

## Summary

This proposal addresses the problem of authenticating access to the content of non-public groups. It is mostly intended to supplement [FEP-400e].

Only the server that hosts the `Group` actor knows for sure who can and can not access the content in the group. However, due to each object being hosted on the server of the actor that created it, it is not ordinarily possible for those other servers to restrict access to that object only to those actors who have the permission to see it.

This FEP defines an authentication mechanism, "actor tokens", that allows an actor to issue tokens that serve as a temporary proof of group membership for other servers.

## Requirements

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this specification are to be interpreted as described in [RFC-2119].

## Fetching content from the server that hosts the group

To fetch an object from **the server that hosts the group** (including the `Group` actor itself for private groups), the requesting server MUST sign the GET request with an HTTP signature using the key of **any** of its actors. It is RECOMMENDED to use a server-wide service actor for this purpose, e.g. `/activitypub/serviceActor` in Smithereen. The rationale for this is that most ActivityPub servers only fetch and store a single copy of each remote object for all users to whom it may concern, and are responsible themselves for enforcing the visibility rules, if any, either way.

## Using actor tokens

The process of fetching an object from **other** server involves an **actor token**. An actor token is a cryptographically signed temporary proof of membership in a group. Since it would be impractical to provide a revocation mechanism, an actor token has a limited validity time in order to account for cases when someone has left a group or was removed from it.

### Structure of the actor token

An actor token is a JSON object with the following REQUIRED fields:

* `issuer`: ID of the actor that generated this token
* `actor`: ID of the actor that the token is issued to (and must be presented with a valid HTTP signature of)
* `issuedAt`: timestamp when the token was generated, ISO-8601 instant (same format as ActivityPub timestamps)
* `validUntil`: timestamp when the token expires, ISO-8601 instant
* `signatures`: array of signature objects, currently with only one possible, and REQUIRED, element defined:
  * `algorithm`: must be the string `rsa-sha256`
  * `keyId`: key ID, same as in HTTP signatures (e.g. `https://example.com/groups/1#main-key`)
  * `signature`: the RSA-SHA256 signature itself encoded as base64, see below for details

### The `sm:actorToken` endpoint

Actors that are capable of issuing actor tokens have a `sm:actorToken` endpoint (where `sm` is an alias to JSON-LD namespace `http://smithereen.software/ns#`) in their `endpoints` object. This endpoint accepts signed GET requests and returns actor tokens.

### Making use of the actor token

To use an actor token when fetching an object, pass it as `Authorization: ActivityPubActorToken {...}` HTTP header.

### Generation of the source string for signature

1. Iterate over the keys in the actor token JSON object, skipping `signature`, and transform them into the format `key: value`. Add these strings to an array.
2. Sort the resulting array lexicographically.
3. Join the strings with newline character (`\n`, U+000A).
4. Convert the resulting string to a UTF-8 byte array.

### Generation of the actor token

1. Verify that the requesting actor, as per HTTP signature, has access to the group (there are members with the same domain). If it does not, you MUST return a 403 error and stop.
2. Create a JSON object with the fields above (except `signature`). It is RECOMMENDED that the validity period is 30 minutes, and it MUST NOT exceed 2 hours.
3. Generate a signature source string as above, sign it, and wrap the signature into an object with `signature`, `algorithm`, and `keyId` fields.
4. Add the object as a single element in the `signatures` array.
5. Return the resulting JSON object to the client.

### Verification of the actor token

1. Check that the HTTP signature is valid, and that `actor` in the token object matches the actor ID from `keyId` in the HTTP signature. Otherwise, you MUST return a 403 and stop.
2. In the `signatures` array, find an object that has `algorithm` set to `rsa-sha256` to get the `signature` value. If there isn't any, you MUST return a 403 and stop.
3. Check the validity time: `issuedAt` MUST be in the past, `validUntil` MUST be in the future, and the difference between them MUST NOT exceed 2 hours. It is RECOMMENDED to apply some margin to these checks to account for imprecisely set clocks. Smithereen uses 5 minutes.
4. Generate the signature source string as above and verify the signature.
5. Check that the object the requester is accessing is, in fact, part of a collection owned by `issuer`.
6. If all of the above checks pass, return the requested object. Otherwise, return a 403.

### Example of an actor token object

```json
{
	"issuer":"https://friends.grishka.me/groups/75",
	"actor":"https://activitypub.academy/actor",
	"issuedAt":"2024-05-03T14:02:18.680404311Z",
	"validUntil":"2024-05-03T14:32:18.680404311Z",
	"signatures":[
		{
			"algorithm":"rsa-sha256",
			"keyId":"https://friends.grishka.me/groups/75#main-key",
			"signature":"w+W1nNV+XBvXi8sDEUZB7muWSSnv1mEE4tNZJqF5LeoxAstBMiBZi8dtHF+v+vXKVPWBAdZUKLS5CttmgZ4tvnvZAfsBztCjYLyiolVQ71IO2Jxlu00Xo9FDoSTRZ61tXdfWufuzs5lRjG3t+S1t1lLllBFmvPLg6BwmdEPvlZvPYnTJzwNY0ljOjickPqfyvdzIslmdYX6dPC0Ayyi028ZmR2SN1Vooc9vnUQ7GMPrlAZtmXgjCVGw5X/cKlAVvGECxRjJnkKEKiLp3lv/SM1UUhP3VRpBSFhXnRX/1QhTUaFV1MhrfDFgWGPg8ypIf6O/M52+iSpJyIOGepmjmow=="
		}
	]
}
```

## Implementations

* [Smithereen](https://github.com/grishka/Smithereen)

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- Gregory Klyushnikov, [Publicly-appendable ActivityPub collections][FEP-400e], 2021
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119.html)

[ActivityPub]: https://www.w3.org/TR/activitypub/
[FEP-400e]: https://w3id.org/fep/400e

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
