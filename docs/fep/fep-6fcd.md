---
slug: "6fcd"
authors: Dmitri Zagidulin <@dmitri@social.coop>
status: DRAFT
dateReceived: 2024-07-11
discussionsTo: https://socialhub.activitypub.rocks/t/fep-6fcd-account-export-container-format/4355
trackingIssue: https://codeberg.org/fediverse/fep/issues/357
---

# FEP-6fcd: Account Export Container Format
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-6fcd.md)から翻訳に協力することができます。
## Summary

This FEP describes a lightweight general purpose account export container format,
with the following properties:

* General purpose, allowing for easy adaptation of existing ActivityPub, social media, and
  cryptographic key material export formats
* Extensible, upgradable, and self-documenting (in the human-readable sense)
* Works with [FEP-7952: Roadmap for Actor and Object Portability](https://codeberg.org/fediverse/fep/pulls/334/files)
* Serves as a concrete serialization of the result of the Export operation described in
  [FEP-9091: Export Actor Service Endpoint](https://codeberg.org/fediverse/fep/pulls/353)

Out of scope:

* Encryption -- handled in a separate layer
* Compression -- handled in a separate layer (how to turn a `.tar` file into a
  `.tar.gz` is well known)

## Inspirations and Prior Art

* (Undocumented) Mastodon Account Export features
* [IndieWeb Blog Archive Format](https://indieweb.org/blog_archive_format)
* WordPress Export Format
* DIF Universal Wallet Backup Containers spec (in progress)

## Overall Concept

1. Serialize export data into files and directories
2. Add a lightweight `manifest.yml` file that describes what's in the files and directories
3. Wrap everything into a `.tar` file.

## Manifest File

### Reserved Properties

* (Required) `ubc-version`: Version of the Account Export Container Format spec
* (Required) `contents`: A listing of files and directories in this .tar file
* (Optional) `meta`: A metadata section describing who this export belongs to, what app
    or service created it, and so on.

### ActivityPub Export Example

Example result of exporting an ActivityPub account:

```bash
$ tar -vtf ap-account-export-2024-06-11.tar

-rw-rw-r-- 0 0   1K Jun 11 15:38 manifest.yml
drwxrwxr-x 0 0 4.0K Jun 11 15:38 activitypub/
drwxrwxr-x 0 0 4.0K Jun 11 15:38 key/
```

Example corresponding `manifest.yml` file:

```yaml
# (Required) Universal Backup Container spec version
ubc-version: 0.1

# (Optional) Metadata section
meta:
  created: 2024-01-01
  createdBy:
    # (Optional) URL to a Controller document, such as an ActivityPub profile using FEP-521a
    # @see https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md
    controller: https://alice-personal-site.example/actor
    # (Optional) The app or service that created this export
    client:
      name: "Example Exporter App"
      url: https://codeberg.example.com/example-export-app

# (Required, but can be empty) Contents section, listing the other files and directories
contents:
  # This file
  manifest.yml:
    url: https://codeberg.org/fediverse/fep/src/branch/main/fep/6fcd/fep-6fcd.md#manifest-file
  # Directory with ActivityPub-relevant exports
  activitypub:
    contents:
      # Serialized ActivityPub Actor profile
      actor.json:
        url: https://www.w3.org/TR/activitypub/#actor-objects
      # ActivityStreams OrderedCollection representing the contents of the actor's Outbox
      outbox.json:
        url: https://www.w3.org/TR/activitystreams-core/#collections
      following_accounts.csv:
        url: https://docs.joinmastodon.org/user/moving/#export
      followers.csv:
        url: https://docs.joinmastodon.org/user/moving/#export
      lists.csv:
        url: https://docs.joinmastodon.org/user/moving/#export
      bookmarks.csv:
        url: https://docs.joinmastodon.org/user/moving/#export
      blocks.csv:
        url: https://docs.joinmastodon.org/user/moving/#export
      mutes.csv:
        url: https://docs.joinmastodon.org/user/moving/#export
      # Directory of object attachments (post images, etc)
      attachments:
        url: https://www.w3.org/TR/activitystreams-vocabulary/#dfn-attachment
        contents:
          # Actor profile avatar
          avatar.jpg:
            url: https://www.w3.org/TR/activitystreams-vocabulary/#dfn-icon
  # 'key' dir, serialized private/public key pairs,
  # such as those declared in a FEP-521a Actor profile
  key:
    url: https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md
    contents:
      key-1234.json:
        url: https://www.w3.org/TR/vc-di-eddsa/#representation-eddsa-rdfc-2022
```

Example exported key file:

```
$ cat key/key-1234.json
```

```json
{
  "@context": ["https://w3id.org/security/multikey/v1"],
  "type": "Multikey",
  "id": "https://alice-personal-site.example/actor#key1234",
  "controller": "https://alice-personal-site.example/actor",
  "publicKeyMultibase": "z6MkrJVnaZkeFzdQyMZu1cgjg7k1pZZ6pvBQ7XJPt4swbTQ2",
  "privateKeyMultibase": "z3u2en7t5LR2WtQH5PfFqMqwVHBeXouLzo6haApm8XHqvjxq"
}
```

### Example Blog Archive Format Export

```bash
$ tar -vtf bar-account-export-2024-06-11.tar

-rw-rw-r-- 0 0   1K Jun 11 15:38 manifest.yml
-rw-rw-r-- 0 0 100K Jun 11 15:38 index.html
-rw-rw-r-- 0 0  50K Jun 11 15:38 feed.json
drwxrwxr-x 0 0 4.0K Jun 11 15:38 uploads/
```

Example corresponding `manifest.yml` file:

```yaml
ubc-version: 0.1

meta:
  created: 2024-01-01

contents:
  # This file
  manifest.yml:
    url: https://codeberg.org/fediverse/fep/src/branch/main/fep/6fcd/fep-6fcd.md#manifest-file
  index.html:
    url: https://indieweb.org/blog_archive_format
  feed.json:
    url: https://indieweb.org/blog_archive_format
  uploads:
    url: https://indieweb.org/blog_archive_format
```

## 参考文献

* [FEP-521a: Representing actor's public keys][FEP-521a]
* Christine Lemmer Webber, Jessica Tallon, [ActivityPub][AP], 2018
* S. Bradner, Key words for use in RFCs to Indicate Requirement Levels, 1997
* Dave Longley, Manu Sporny, [Data Integrity EdDSA Cryptosuites][DI Sigs] v1.0, 2023

[FEP-521a]: https://codeberg.org/fediverse/fep/src/branch/main/fep/521a/fep-521a.md
[AP]: https://www.w3.org/TR/activitypub/
[DI Sigs]: https://w3c.github.io/vc-di-eddsa/#eddsa-jcs-2022

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement
Proposal have waived all copyright and related or neighboring rights to this work.
