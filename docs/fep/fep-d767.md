---
slug: "d767"
authors: Lynn Foster <lynnfoster@social.coop>
status: WITHDRAWN
dateReceived: 2023-04-02
dateWithdrawn: 2024-02-03
trackingIssue: https://codeberg.org/fediverse/fep/issues/76
discussionsTo: https://codeberg.org/fediverse/fep/issues/76
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-d767.md)から翻訳に協力することができます。
# FEP-d767: Extend ActivityPub with Valueflows

## Summary

A standard method to extend ActivityPub/ActivityStream with Valueflows vocabulary, to enable varied economic networking activity in the fediverse.

## History

Valueflows was started in 2015 by a group of developers who met around the concept of an "open app ecosystem".  The goal was to define one of the vocabularies needed to enable modular apps and components to communicate in a standard way, using distributed architecture.  By extension, the goal was for people and organizations to more easily work together to coordinate the creation, distribution, and exchange of economic resources.

Valueflows got a first version out in 2017, and started a period of learning from [implementations](https://www.valueflo.ws/appendix/usedfor/).

Valueflows met the fediverse through Mayel de Borniol and Ivan Minutillo, and work was begun in 2017 to integrate Valueflows into the software that became Bonfire.

Valueflows would like to get to a stable v1.0 this year.  The core is stable, but there are several features around the edges that are either not yet implemented or not yet tested enough.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this specification are to be interpreted as described in [RFC-2119].

Fediverse software that supports economic activity MAY implement the Valueflows extension.

## Vocabulary Integration

*Note this is not at all finalized, will be seeking feedback in SocialHub.*

The following general patterns MUST be followed to integrate Valueflows objects into ActivityPub messages.

- Any Valueflows construct (a class or a more complex bundle of related classes) will be an Object in the AP message.
- The activities Create, Update, and Delete will be used.
- Other possibilities to be investigated:  Add, Remove, Accept, Reject.
- Only stable parts of the Valueflows vocabulary will be addressed
-

The general idea is that AP/AS will be used basically for messaging.  Valueflows does have some concepts that could be considered activities, but they don't fit nicely into the current list.

Note: The overlap between the Agent portion of the Valueflows model and the AP actors will not be addressed at this time, due to the complexity of mapping to the actor model, as well as the in-process suggestions for AP groups, organizations, communities, etc. Until it is explicitly addressed, when there is overlap, any Person, Group, Organization actor with agency can be used as a Valueflows Agent.  In addition, Valueflows will include Agents that are not AP actors, and these can be handled with the standard Create, Update, Delete.

Examples (*loosely documented atm, just for discussion*):
```
{
  "@context": {
    "https://www.w3.org/ns/activitystreams",
    "vf": "https://w3id.org/valueflows/",
  },
  "summary": "Sally created a project plan.",
  "type": "Create",
  "actor": {
    "type": "Person",
    "name": "Sally"
  },
  "object": {
    "type": "vf:Plan",
    "name": "Define AP-VF Extension",
    "content": "Collaborate on defining the VF extension to AP/AS. Propose as a FEP (Fediverse Enhancement Proposal)." # or use skos:note per vf, different meanings?
  }
}

{
  "@context": {
    "https://www.w3.org/ns/activitystreams",
    "vf": "https://w3id.org/valueflows/",
  },
  "summary": "Sally created a process.",
  "type": "Create",
  "actor": {
    "type": "Person",
    "name": "Sally"
  },
  "object": {
    "type": "vf:Process",
    "vf:name": "Draft AP-VF examples",
    "content": "Collaborate on examples to get discussion going on the VF extension to AP/AS.",
    "vf:inScopeOf": {
      "type": "Organization",
      "vf:name": "HUMANs", # could use target for this?
    },
    "vf:plannedWithin": {
      "type": "Plan",
      "vf:name": "Define AP-VF Extension", # this would be an id?
    },
    "vf:outputs": {
      "object": {
        "type": "vf:Commitment",
        "vf:action": "produce",
        "vf:conformsTo": {
          "type": "ResourceSpecification",
          "vf:name": "Vocabulary Spec"
        },
        "vf:due": "2023-05-08T10:30:00-5:00",
        "vf:provider": {
          "type": "Organization", # vf? as?
          "vf:name": "HUMANs"
        },
        "vf:receiver": {
          "type": "Organization",
          "vf:name": "SocialHub"
        }
      }
    }
  }
}

{
  "@context": {
    "https://www.w3.org/ns/activitystreams",
    "vf": "https://w3id.org/valueflows/",
    "om2": "http://www.ontology-of-units-of-measure.org/resource/om-2/"
  },
  "summary": "Lynn created a commitment.",
  "type": "Create",
  "actor": {
    "type": "Person",
    "name": "Lynn"
  },
  "object": {
    "type": "vf:Commitment",
    "vf:inputOf": {
      "type": "vf:Process",
      "vf:name": "Draft AP-VF examples"
    }
    "vf:action": "work",
    "vf:conformsTo": {
      "type": "ResourceSpecification",
      "vf:name": "Vocabulary Work"
    },
    "vf:effortQuantity": {
      "type": "om:Measure",
      "om2:hasNumericalValue": 10,
      "om2:hasUnit": "hr"
    }
    "vf:due": "2023-05-01T10:30:00-5:00",
    "vf:provider": {
      "type": "Person", # vf? as?
      "vf:name": "Lynn"
    },
    "vf:receiver": {
      "type": "Organization",
      "vf:name": "HUMANs"
    },
    "content": "First draft of examples, submit for feedback." # skos:note?
  }
}
```


## Implementations

- [Bonfire](https://bonfirenetworks.org/)
- [Oceco - Communecter](https://www.communecter.org/#) (not yet)

## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub](https://www.w3.org/TR/activitypub/), 2018
- [ActivityStream] James M Snell, Evan Prodromou, [ActivityStream](https://www.w3.org/TR/activitystreams-vocabulary/), 2017
- [Valueflows] Lynn Foster, elf Pavlik, Bob Haugen, [Valueflows](https://valueflo.ws), 2023


## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication 

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
