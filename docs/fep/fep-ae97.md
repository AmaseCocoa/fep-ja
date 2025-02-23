---
slug: "ae97"
authors: silverpill <@silverpill@mitra.social>
status: DRAFT
dateReceived: 2023-08-14
trackingIssue: https://codeberg.org/fediverse/fep/issues/148
discussionsTo: https://socialhub.activitypub.rocks/t/fep-ae97-client-side-activity-signing/3502
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-ae97.md)から翻訳に協力することができます。
# FEP-ae97: Client-side activity signing

## Summary

Existing Fediverse servers manage signing keys on behalf of their users. This proposal describes a new kind of [ActivityPub][ActivityPub] client that lets users sign activities with their own keys, and a server that can distribute client-signed activities to other servers.

## History

[Initial version](https://codeberg.org/fediverse/fep/src/commit/fc9c65daca267be9f91761ed854eac9e829222a2/fep/ae97/fep-ae97.md) of this proposal relied on linking of cryptographic identities to actor objects via [FEP-c390] identity proofs. That mechanism was superseded by [FEP-ef61] which achieves full data portability.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119][RFC-2119].

## Registering an actor

Client creates portable actor objects according to [FEP-ef61] and stores them.

Before registering a portable actor on the server, the client MUST generate an RSA key and attach it to actor object via `publicKey` property, and also add it to the `assertionMethod` array as described in [FEP-521a]. Then it MUST add the server URL to the `gateways` array of the actor object.

To register the actor, the client MUST send an HTTP POST request to the gateway endpoint at `/.well-known/apgateway` path. The body of the request MUST be an actor object. The request MUST have an `X-Rsa-Secret-Key` header containing the previously generated RSA secret key in `secretKeyMultibase` format described in section *2.3.1.2 Multikey* of [Data Integrity](https://www.w3.org/TR/vc-data-integrity/#multikey) specification.

If the server accepts registration request, its response MUST have `201 Created` status code.

## Sending activities

Client submits signed [FEP-ef61] activities to actor's outbox. Contrary to what ActivityPub specification prescribes in section [6. Client to Server Interactions](https://www.w3.org/TR/activitypub/#client-to-server-interactions), the server MUST NOT overwrite the ID of activity. Instead of assigning a new ID, the server MUST verify that provided ID has not been used before. If activity ID is in compatible format (HTTP(S) URI), the server MUST check that its [origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin) is the same as the server's origin. If the server accepts activity, its response MUST have `202 Accepted` status code.

If activity contains a wrapped object (as in `Create` and `Update` activities), it MUST be a portable object created according to [FEP-ef61]. The server MUST validate object IDs in the same way it validates activity IDs.

The server MUST deliver activities to their indended audiences without altering them.

## Receiving activities

Client receives activities by polling the actor's inbox.

Requests to inbox endpoint MUST have an HTTP signature created using the RSA secret key generated during registration.

## Implementations

- [fep-ae97-client](https://codeberg.org/silverpill/fep-ae97-client) (client)
- Mitra (server)

## References

- Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119], 1997
- silverpill, [FEP-c390: Identity Proofs][FEP-c390], 2022
- silverpill, [FEP-ef61: Portable Objects][FEP-ef61], 2023
- silverpill, [FEP-521a: Representing actor's public keys][FEP-521a], 2023
- Dave Longley, Manu Sporny, [Verifiable Credential Data Integrity 1.0][DataIntegrity], 2023

[ActivityPub]: https://www.w3.org/TR/activitypub/
[RFC-2119]: https://tools.ietf.org/html/rfc2119.html
[FEP-c390]: https://codeberg.org/fediverse/fep/src/branch/main/fep/c390/fep-c390.md
[FEP-ef61]: https://codeberg.org/fediverse/fep/src/branch/main/fep/ef61/fep-ef61.md
[FEP-521a]: https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md
[DataIntegrity]: https://www.w3.org/TR/vc-data-integrity/

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
