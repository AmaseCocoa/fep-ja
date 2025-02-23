---
slug: "888d"
authors: a <a@trwnh.com>
status: DRAFT
dateReceived: 2023-04-10
trackingIssue: https://codeberg.org/fediverse/fep/issues/83
discussionsTo: https://socialhub.activitypub.rocks/t/fep-888d-using-w3id-org-fep-as-a-namespace-for-extension-terms-and-for-fep-documents/3098
---
!!! Warning
    このFEPはまだ翻訳されていません。

    [ここ](https://github.com/AmaseCocoa/fep-ja/edit/master/docs/fep/fep-888d.md)から翻訳に協力することができます。

# FEP-888d: Using https://w3id.org/fep as a base for FEP-specific namespaces

## Summary

It is considered best practice in the linked-data ecosystem to have IRIs be HTTPS URIs that resolve to a definition of the term being used, and it is desirable to define such terms in a JSON-LD context file that is referenced by its IRI rather than having the full `@context` object embedded in every single document. ActivityStreams 2.0 and ActivityPub do this with the normative context and namespace provided at `https://www.w3.org/ns/activitystreams`, but this namespace is not generally open to extensions or to experimental terms. This FEP therefore proposes using `https://w3id.org/fep` as a base IRI for the FEP process, allowing sub-namespaces for each FEP.

## Acknowledgements

(This section is non-normative.)

The core idea behind this FEP is attributed to helge on SocialHub [1][1]. Specifically, the proposal to register `fep` at the W3ID service is adopted wholesale, with alterations made to the specifics of implementing the redirect mappings using .htaccess rules. These alterations are intended to allow easier sub-namespace allocation for each FEP.

## Requirements

The key words "MUST", "SHOULD", "MAY" are to be interpreted as described in [RFC-2119][RFC-2119].

## Introduction

(This section is non-normative.)

The Resource Description Framework (RDF), of which JSON-LD is a serialization, uses URIs to identify nodes on a graph, define properties of those nodes, and create relationships between those nodes. Each statement in RDF represents a fact that is constructed by linking a subject to an object with a predicate; for example, in the statement "Alice knows Bob", the subject `Alice` is related to the object `Bob` by the predicate `knows`. To avoid ambiguity, we can specify a URI for what it means to "know" someone. Such a URI represents a named property or named predicate, and it exists within a namespace, often associated with some ontology or vocabulary. [ActivityStreams 2.0][AS2-Core] provides and defines the [Activity Vocabulary][AS2-Vocab] at `https://www.w3.org/ns/activitystreams`, and terms defined within may use either the base IRI `https://www.w3.org/ns/activitystreams#` or the compact IRI `as:`.

For example, we can consider the definition of "Public" addressing within [ActivityPub][ActivityPub], represented by the `Public` magic collection. When the normative ActivityStreams 2.0 context is applied, the IRI for this collection may be equivalently expressed as `Public`, `as:Public`, or `https://www.w3.org/ns/activitystreams#Public`.

Historically, extension terms within early [ActivityPub][ActivityPub] implementations assumed that those extension terms would be readily adopted within the ActivityStreams namespace, but this did not happen. Currently, terms such as `Hashtag`, `manuallyApprovesFollowers`, `movedTo`, and `sensitive` are manually defined with compact IRIs using the `as:` prefix, in effect making it so that any implementation that wishes to understand these properties and types must manually define those terms as well, in the exact same way.

Later extension terms were defined within vendor-specific namespaces such as `http://joinmastodon.org/ns` or `https://joinpeertube.org/ns`. With the increased adoption of ActivityPub by software projects and the diverse needs of such projects, each project will often define its own vendor-specific namespace to contain its own terms. This has led to a multitude of namespaces and extension terms, which must be cherry-picked as needed by JSON-LD consumers wishing to maintain compatibility. Additionally, some of these terms are defined incorrectly within `@context`, leading to the necessity of compatibility hacks on a per-project basis. Even terms defined correctly may overlap with other terms, and proposed new terms must be parented within a vendor namespace, creating the potential for conflict on which vendor should adopt which term.

The aim of this FEP is to provide a vendor-independent namespace under which extension terms can be defined pursuant to the FEP process and the above problems can be reduced.

## Prior art

(This section is non-normative.)

Within the XMPP ecosystem, the core XMPP specification is defined within an RFC, and further functionality is afforded by the "eXtensible" nature of XMPP. The XMPP Standards Foundation (XSF) maintains the process for stewarding new extensions via XMPP Extension Protocols (XEPs). XML namespacing for such extensions is provided by `urn:xmpp:`, as the XML ecosystem generally prefers using URNs rather than using HTTPS URIs. Such URNs are fully location-independent and not vulnerable to DNS expiry, lapsing, or insolvency. Instead, they are assigned within the authority of the XSF. The XSF maintains an XMPP Registrar and allows XEPs to request and define sub-namespaces beneath `urn:xmpp:`. In exchange, these URNs are not generally dereferencable without a resolver that looks up the URN within the XEP database.

Within the RDF and linked-data ecosystems, there is a strong preference for HTTP or HTTPS URIs, as these can usually be dereferenced via the HTTP protocol for additional information about the subject of the URI. In cases where the URI does not resolve, the URI serves as an identifier not much different than a URN, but with its authority derived from DNS domain rather than from some organizational authority. The reliance on DNS domain creates an issue where the primary domain associated with a group or organization might change. If a previously-used domain is reassigned to a different party, then the new party can mint URIs that accidentally or intentionally conflict with previously-assigned URIs.

To mitigate the DNS authority reassignment issue, trusted intermediary services can maintain a "persistent URL" (PURL) service, which allows assigning identifiers on the intermediary domain that will redirect to some other URI. This layer of indirection allows changing the location of the resource by simply changing the redirect's target. W3ID is one such service, operated by the W3C Permanent Identifier Community Group and available at <https://w3id.org>. At the time of writing this FEP, top-level directory names can be claimed by individuals who submit pull requests to the w3id.org repository on GitHub, and .htaccess files allow redirection based on rewrite rules that transform incoming requests to some other target.

## Specification

### Design goals

Broad design goals for the redirect mapping include:

- Content negotiation for JSON-LD consumers. IRIs SHOULD return machine-friendly context documents or term definitions when requested via the `Accept: application/ld+json` HTTP header, and SHOULD otherwise return human-friendly proposal documents or term definitions by default.
- Sub-namespaces for each FEP. Identifiers for each term SHOULD be allocated within the namespace of the FEP that defines them.

At minimum, the following redirects SHOULD resolve as follows:

- `https://w3id.org/fep`
  - `Accept: *` => the FEP repository or current home page
- `https://w3id.org/fep/(:id)`
  - `Accept: application/ld+json` => a specific FEP's context document
  - `Accept: *` => a specific FEP's proposal document

Additionally, the following MAY resolve:

- `https://w3id.org/fep/(:id)/(:term)`
  - `Accept: application/ld+json` => a specific FEP's specific term definition in JSON-LD ontology/schema
  - `Accept: application/rdf+xml` => a specific FEP's specific term definition in RDF/XML ontology/schema
  - `Accept: text/turtle` => a specific FEP's specific term definition in Turtle ontology/schema
  - `Accept: *` => a specific FEP's specific term's folder

### Mapping w3id.org/fep to fediverse/fep on Codeberg

At the time of writing this FEP, the Codeberg repository at `https://codeberg.org/fediverse/fep` is used to host FEP-related files, and can similarly be used to host context documents.

#### Example

(This section is non-normative.)

An example .htaccess file is co-located with this FEP, and reproduced below for convenience:

```perl
RewriteEngine on


# catch root request
RewriteRule ^\/?$ https://codeberg.org/fediverse/fep [R=302,L]



# Catch FEP documents

## By content negotiation

### JSON-LD
RewriteCond %{HTTP_ACCEPT} application/ld\+json
RewriteRule ^([A-Za-z0-9]+)\/?$ https://raw.codeberg.page/fediverse/fep/fep/$1/fep-$1.jsonld [R=302,L]

### RDF+XML
RewriteCond %{HTTP_ACCEPT} application/rdf\+xml
RewriteRule ^([A-Za-z0-9]+)\/?$ https://fediverse.codeberg.page/fep/fep/$1/fep-$1.rdf [R=302,L]

### Turtle
RewriteCond %{HTTP_ACCEPT} text/turtle
RewriteRule ^([A-Za-z0-9]+)\/?$ https://fediverse.codeberg.page/fep/fep/$1/fep-$1.ttl [R=302,L]

## By URL hacking
RewriteRule ^([A-Za-z0-9]+).jsonld$ https://raw.codeberg.page/fediverse/fep/fep/$1/fep-$1.jsonld [R=302,L]
RewriteRule ^([A-Za-z0-9]+).rdf$ https://raw.codeberg.page/fediverse/fep/fep/$1/fep-$1.rdf [R=302,L]
RewriteRule ^([A-Za-z0-9]+).ttl$ https://raw.codeberg.page/fediverse/fep/fep/$1/fep-$1.ttl [R=302,L]

## By default, take you to the FEP document
RewriteRule ^([A-Za-z0-9]+)\/?$ https://codeberg.org/fediverse/fep/src/branch/main/fep/$1/fep-$1.md [R=302,L]



# Catch term definitions/schemas/ontologies

## By content negotiation

### JSON-LD
RewriteCond %{HTTP_ACCEPT} application/ld\+json
RewriteRule ^([A-Za-z0-9]+)\/(.*?)\/?$ https://raw.codeberg.page/fediverse/fep/fep/$1/$2/$2.jsonld [R=302,L]

### RDF+XML
RewriteCond %{HTTP_ACCEPT} application/rdf\+xml
RewriteRule ^([A-Za-z0-9]+)\/(.*?)\/?$ https://fediverse.codeberg.page/fep/fep/$1/$2/$2.rdf [R=302,L]

### Turtle
RewriteCond %{HTTP_ACCEPT} text/turtle
RewriteRule ^([A-Za-z0-9]+)\/(.*?)\/?$ https://fediverse.codeberg.page/fep/fep/$1/$2/$2.ttl [R=302,L]

### test html
RewriteCond %{HTTP_ACCEPT} ^text/html$
RewriteRule ^([A-Za-z0-9]+)\/(.*?)\/?$ https://fediverse.codeberg.page/fep/fep/$1/$2/$2.html [R=302,L]

## By URL hacking
RewriteRule ^([A-Za-z0-9]+)\/(.*?).jsonld$ https://raw.codeberg.page/fediverse/fep/fep/$1/$2/$2.jsonld [R=302,L]
RewriteRule ^([A-Za-z0-9]+)\/(.*?).rdf$ https://fediverse.codeberg.page/fep/fep/$1/$2/$2.rdf [R=302,L]
RewriteRule ^([A-Za-z0-9]+)\/(.*?).ttl$ https://fediverse.codeberg.page/fep/fep/$1/$2/$2.ttl [R=302,L]
RewriteRule ^([A-Za-z0-9]+)\/(.*?).html$ https://fediverse.codeberg.page/fep/fep/$1/$2/$2.html [R=302,L]
RewriteRule ^([A-Za-z0-9]+)\/(.*?).md$ https://fediverse.codeberg.page/fep/fep/$1/$2/README.md [R=302,L]

## By default, just take you to the term's folder
RewriteRule ^([A-Za-z0-9]+)\/(.*?)\/?$ https://codeberg.org/fediverse/fep/src/branch/main/fep/$1/$2 [R=302,L]



# a generic catch-all rule
RewriteRule ^(.*)\/?$  https://codeberg.org/fediverse/fep/raw/branch/main/fep/$1 [R=302,L]
```

### Defining terms associated with an FEP

FEPs that wish to define extension terms within the w3id.org/fep namespace MUST provide a JSON-LD document co-located within their FEP folder with a filename of the form `fep-xxxx.jsonld`, where `xxxx` is the FEP's slug. This document MUST include at least a `@context` key, whose value is a JSON object containing term definitions. Simple term definitions map a `term` key to an IRI value. Expanded term definitions contain `@id` for each term, with `@type` of `@id` if the term links to another node on the graph. Refer to [LD-TERM-DFN][LD-TERM-DFN] for additional guidance on defining terms within JSON-LD.

The document MAY include additional metadata outside of the `@context`, such as schema or ontology definitions. If this is done, then you SHOULD NOT include any prefixes in your `@context` that are only used within the graph, as these can pollute the usage as a context document.

Once the FEP is marked `FINAL`, the context document MAY be cached forever if referenced. FEPs that define extension terms MAY instead define extension terms within a vendor-specific namespace, but generally this SHOULD NOT be done.

#### Example using paths

(This section is non-normative.)

For example, say we wanted to define the following terms within the current FEP-888d:

- `SomeType` is a term for some type
- `exampleA` is a term for a property with some literal value (string, boolean, number)
- `exampleB` is a term for a property that links to another node on the graph (for example, another object)
- `exampleC` is a term for a property that is an ordered list of literal values that are specifically non-negative integers

The context document might look like this, at minimum:

```json
{
	"@context": {
		"SomeType": "https://w3id.org/fep/888d/SomeType",
		"exampleA": "https://w3id.org/fep/888d/exampleA",
		"exampleB": {
			"@id": "https://w3id.org/fep/888d/exampleB",
			"@type": "@id"
		},
		"exampleC": {
			"@id": "https://w3id.org/fep/888d/exampleC",
			"@type": "http://www.w3.org/2001/XMLSchema#nonNegativeInteger",
			"@container": "@list"
		}
	}
}
```

Refer to [LD-TERM-DFN][LD-TERM-DFN] for additional guidance on defining terms within JSON-LD.

A folder within the FEP's directory should be used to provide additional documentation for the term, such as ontology or schema definitions via JSON-LD, RDF/XML, and/or Turtle.

#### Example using fragment identifiers

(This section is non-normative.)

Depending on convenience or preference, the context document might instead look like this:

```json
{
	"@context": {
		"SomeType": "https://w3id.org/fep/888d#SomeType",
		"exampleA": "https://w3id.org/fep/888d#exampleA",
		"exampleB": {
			"@id": "https://w3id.org/fep/888d#exampleB",
			"@type": "@id"
		},
		"exampleC": {
			"@id": "https://w3id.org/fep/888d#exampleC",
			"@type": "http://www.w3.org/2001/XMLSchema#nonNegativeInteger",
			"@container": "@list"
		}
	}
}
```

Refer to [LD-TERM-DFN][LD-TERM-DFN] for additional guidance on defining terms within JSON-LD.

In such a case, the FEP document should include an element with an HTML identifier that exactly matches the term name, so that the IRI fragment resolves properly. In practice, this means one of the following:

- Using a heading with a name that exactly matches the term name. This should be autolinked correctly by most Markdown processors. Be warned that this may cause problems for FEPs that define terms conflicting with common header names, such as `summary`, `acknowledgements`, `requirements`, `references`, `copyright`, and so on, including any headers that the FEP author includes for purposes other than explicitly defining the term.
- Using a heading with a custom attribute containing an ID. Some Markdown processors such as Goldmark will handle cases such as `### h3 {#custom-identifier}` and render `<h3 id="custom-identifier">h3</h3>`. Markdown specifications such as CommonMark currently do not support custom attributes, but some Markdown processors such as Goldmark support custom attributes on headers (but not on arbitrary elements). See [CM-ATTRS][CM-ATTRS] for more discussion of this feature.
- Using an HTML definition list, with `id` attributes exactly matching the term name. HTML within Markdown files is generally rendered as-is, although it may be sanitized, stripped, or disallowed for security purposes. In cases where it is allowed, however, it can be an effective way to express term definitions within an FEP document.

## Defining terms in a machine-readable way

(This section is non-normative.)

If terms are defined within HTML using RDFa, and this HTML is embedded in the Markdown contents of the FEP document, then these term definitions can be used to programmatically generate machine-readable term definitions, schemas, and ontologies in multiple formats. A Python script is provided within the FEP repository at `scripts/make_definitions.py` for convenience.

To make use of the script, it is necessary to structure term definitions in a specific way. We define a "term definition" to be any HTML element that adheres to the following requirements:

- The HTML element has an HTML `id` attribute equal to the shorthand term name.
- The HTML element has an RDFa `resource` attribute equal to the term's full IRI.

Generally, this HTML element should be a `<section>` tag, and it should contain a definition list (`<dl></dl>`) where each term (`<dt></dt>`) and definition (`<dd></dd>`) provide a key-value pair representing a property of that term definition. Examples may be provided within a preformatted code block (`<pre><code></code></pre>`), and multiple examples may be provided. Example code blocks can have a `title` attribute on the `<pre>` tag. It is generally recommended to put any term definitions below a heading called something like "Terms defined".

For all term definitions regardless of type, it is recommended to include the following properties in your definition list:

- `rdfs:label` is a natural language label that can replace the shorthand term. For example, a term that is normally expressed with the shorthand `hasValue` might be labelled `has value` in plain English.
- `rdfs:comment` is a natural language description that gives more detail about what the term means.
- `rdfs:isDefinedBy` is a reference pointing to where the term is defined. Usually this is the current FEP, but meta-FEPs that bundle other FEPs may use this property to refer to the original FEP that defined a term.

For term definitions that define a class (such as `rdfs:Class` or `owl:Class`), it is recommended to include the following properties in your definition list if appropriate:

- `rdfs:seeAlso` is a reference to some other resource that provides more information about the current term. Usually this can be used to link a class to the properties that instances of that class may have.
- `rdfs:subClassOf` is a property that declares the current class to inherit from the object. Logically, if something is an instance of the current class, it is also an instance of the superclass. For example, in [AS2-Vocab][AS2-Vocab], `OrderedCollection` is a subclass of `Collection`, so all `OrderedCollection` objects are also implied to have a type of `Collection` as well, even if not explicitly declared to be one.

For term definitions that define a property (such as `rdf:Property`, `owl:DatatypeProperty`, or `owl:ObjectProperty`), it is recommended to include the following properties in your definition list if appropriate:

- `rdfs:domain` is the Domain of the property. Logically, this implies that if a property `p` has a domain `D`, and a given resource has the property `p`, then that resource has a type of `D`. For example, in [AS2-Vocab][AS2-Vocab] the domain of `actor` is `Activity`, so anything that has an `actor` is implied to be an `Activity` even if not explicitly declared to be one.
- `rdfs:range` is the Range of the property. Logically, this implies that if a property `p` has a range `R`, and a given resource is the value of `p`, then that resource has a type of `R`. For example, in [AS2-Vocab][AS2-Vocab] the range of `replies` is `Collection`, so anything that is the value of `replies` is implied to be a `Collection` even if not explicitly declared to be one.
- In cases where a property is Required, this can be stated by declaring that the `property` of `owl:minCardinality` has a `content` of `1` (with a `datatype` of `xsd:nonNegativeInteger`). In other words, a property with a minimum cardinality of 1 must have at least one value. If the property is not Required, then this statement can be amended to have a `content` of `0`, or in other words, you are stating that this property must have at least 0 values (which allows having no values).
- In cases where a property is Functional, this can be stated by declaring that the `property` of `owl:maxCardinality` has a `content` of `1` (with a `datatype` of `xsd:nonNegativeInteger`). In other words, a property with a maximum cardinality of 1 can have at most one value. If the property is not Functional, then this statement can be omitted.
- `rdfs:seeAlso` is a reference to some other resource that provides more information about the current term. Usually this can be used to link a property to other related terms, such as term definitions that can be used as the value of this property.

If the term definition contains JSON-LD keywords like `@id`, `@vocab`, `@set`, or `@list` within a `<dd>` element whose corresponding `<dt>` element's inner text exactly matches `Range`, then these keywords can be used to automatically generate a context document. Set the attribute `excluded` on the term definition to a non-empty value to exclude it from auto-generated context term definitions.

### Using the python script

Source code for the script is available in `make_definition.py`, currently co-located in the FEP-888d folder.

The script will operate according to the following logic:

#### Options available

- GENERATE_CONTEXT_FROM_TERMDEFS: if true, will automatically generate a context. The generated context can be manipulated by the presence of JSON-LD keywords like `@id`, `@vocab`, `@set`, and `@list`.
- OUTPUT_FORMATS: a list of formats to be used when generating outputs for both singular term definitions as well as FEP-wide schema or ontology files.
- PATH_TO_CONTEXT: a pre-written context document whose context mapping will be injected into the JSON-LD output of the FEP. Defaults to the JSON-LD alternate format of the FEP (`fep-$SLUG.jsonld` co-located within the FEP folder). Failure to read this path as valid JSON will lead to an empty context mapping.

#### Algorithm

1) Render markdown as HTML.
2) Find all HTML elements that have `id` and `resource`. These are considered "term definitions".
3) Initialize an empty graph for the entire FEP.
4) Initialize an empty context mapping for the entire FEP.
4.1) If GENERATE_CONTEXT_FROM_TERMDEFS is set to false, then try to load an existing context mapping from PATH_TO_CONTEXT.
5) Loop over all elements identified as "term definitions".
5.1) Initialize an empty graph for the current term.
5.2) Set the subject `s` equal to the `resource` attribute of the element.
5.3) Set `term_type` equal to the `typeof` attribute of the element.
5.4) Add a statement to the current term graph, of the form `s rdf:type term_type`.
5.5) Loop over all child elements with a `property` attribute. These are considered "property definitions".
5.5.1) Set the predicate `p` equal to the `property` attribute.
5.5.2) Set the object `o` equal to the `resource` attribute. If not present, then set the object `o` equal to the `href` attribute. If not present, then set the object `o` equal to the `content` attribute. If not present, then set the object `o` equal to the inner text of the current element.
5.5.3) Set the language `lang` equal to the `lang` attribute.
5.5.4) Set the datatype `datatype` equal to the `datatype` attribute.
5.5.5) Add a statement to the current term's graph using `s`, `p`, `o`, `lang`, and `datatype`, making sure to expand any CURIEs according to the RDFa initial context.
5.5.5.1) If there is a `datatype` or `lang`, then the object `o` is a Literal.
5.5.5.2) Otherwise, the object `o` is an IRI reference.
5.6) If GENERATE_CONTEXT_FROM_TERMDEFS is set to true and the current element does not have an `excluded` attribute, then extract JSON-LD keywords and automatically generate a context term definition.
5.6.1) Set the shorthand `term_name` equal to the `id` attribute of the element.
5.6.2) Initialize an `options` mapping to keep track of extended term definitions.
5.6.3) Pairwise combine each child `<dt>` element with its corresponding child `<dd>` element.
5.6.4) Find a `<dt>` element whose inner text is exactly "Range". [TODO: something less fragile?]
5.6.5) Check the corresponding `<dd>` element for a `resource` attribute. If there is one, extract this IRI as `type_iri`.
5.6.5.1) If this `type_iri` starts with `xsd`, then insert `@type: type_iri` into the `options` mapping.
5.6.6) Check the text content of the `<dd>` element for a substring `@id`. If found, then insert `@type: @id` into the `options` mapping.
5.6.7) Check the text content of the `<dd>` element for a substring `@vocab`. If found, then insert `@type: @vocab` into the `options` mapping.
5.6.8) Check the text content of the `<dd>` element for a substring `@set`. If found, then insert `@container: @set` into the `options` mapping.
5.6.9) Check the text content of the `<dd>` element for a substring `@list`. If found, then insert `@container: @list` into the `options` mapping.
5.6.10) If the `options` mapping is not empty, then insert `@id: s` into the `options` mapping.
5.6.11) If the `options` mapping is not empty, then map `term_name` to the `options` mapping. Otherwise, map `term_name` to the string `s`. This is considered a "context term definition".
5.6.12) Insert the context term definition into the context mapping.
5.7) Merge the current term graph into the FEP-wide graph.
5.8) If the subject `s` is not a fragment identifier, then for each alternate output format in OUTPUT_FORMATS, serialize the current term graph to that format and write it to a file within a co-located subfolder with the name `term_name`.
5.8.1) If HTML+RDFa is one of the OUTPUT_FORMATS, then the term definition element can be copied as-is.
5.8.2) If Markdown is one of the OUTPUT_FORMATS:
5.8.2.1) Write a first-level heading with the shorthand `term_name` set equal to the `id` attribute of the term definition.
5.8.2.2) Pairwise combine each child `<dt>` element with its corresponding child `<dd>` element.
5.8.2.3) For each pair of `<dt>` and `<dd>` element, write a definition list term and definition list definition.
5.8.2.4) Extract examples by finding all HTML elements that are `<pre>`.
5.8.2.5) If any examples were found, write a second-level heading with the text set to `Examples`.
5.8.2.5) For each example block found:
5.8.2.5.1) If a `title` attribute is present, then write a paragraph with the text set to the value of `title`.
5.8.2.5.2) Write the start of the code block with backticks. If a `lang` attribute is present, then append its value immediately afterward.
5.8.2.5.3) Write the inner text of the example.
5.8.2.5.4) Write the closing backticks for closing the code block.
6) For each alternate output format in OUTPUT_FORMATS, serialize the FEP-wide graph to that format and write it to a file co-located with the name `fep-$SLUG.$FORMAT`. For example, if Turtle output is enabled, then `fep-xxxx.ttl` will be written. For JSON-LD output, use the current context mapping when serializing the FEP-wide graph. For HTML+RDFa or Markdown output, ignore these formats. (Outputting HTML can be done as part of a separate process of rendering the Markdown with a static site generator. Outputting the Markdown makes no sense since it is the source material and should not be overwritten.)
6.3) If GENERATE_CONTEXT_FROM_TERMDEFS is set to true but JSON-LD is not one of the output formats, then initialize and serialize an empty graph, using the generated context term definition.
6.1.1) Remove the graph from the context document.
6.1.2) Save the generated context document as `fep-xxxx.jsonld`.

#### Usage

- Install required python dependencies, for example in a virtual environment:
  - python-markdown
  - beautifulsoup4
  - rdflib
  - markdownify
  - python-frontmatter
- The script should be run in the base directory of the fediverse/fep repo.
- `python make_definitions.py $SLUG` will read term definitions from the contents of `fep/$SLUG/fep-$SLUG.md`, then generate alternate formats for the FEP. If your terms are defined with fragment identifiers (of the form `https://w3id.org/fep/xxxx#term`), then only the FEP itself will be generated in alternate formats. If your terms are defined with absolute identifiers (of the form `https://w3id.org/fep/xxxx/term`), then co-located subdirectories will be created as well, and singular term definitions will be generated in chosen output formats. If a `context.jsonld` document is present in the FEP folder, then it will be copied into the JSON-LD alternate format output. The current default output formats are:
  - .ttl (Turtle)
  - .rdf (RDF/XML)
  - .jsonld (JSON-LD)
  - .html (HTML+RDFa) (singular term definitions only)
  - .md (README.md for the Codeberg repo viewer) (singular term definitions only)
- `python make_definitions.py $SLUG -c` will do everything in the above bullet point, except for copying `context.jsonld`. Instead, it will check for JSON-LD keywords and attempt to auto-generate a context document based on those keywords. If JSON-LD output is disabled, the script will generate only a context document instead of a combined context document and schema or ontology.

## Example terms defined by this FEP

(This section is non-normative.)

Using the same fictitious terms from the above examples:

- `SomeType` is a term for some type
- `exampleA` is a term for a property with some literal value (string, boolean, number)
- `exampleB` is a term for a property that links to another node on the graph (for example, another object)
- `exampleC` is a term for a property that is an ordered list of literal values that are specifically non-negative integers
- `ExcludedExample` is a term for some class that is useful for ontology but not intended to be used for the context mapping

We can formulate the following machine-readable term definition blocks, which are intended to provide a range of examples of various RDFa syntactical constructs for demonstration purposes. Specifically, the following mechanisms are demonstrated:

- Declaring a `resource` with a `typeof` some class
- Declaring a natural language `property` with a specified `lang`
  - Specifically, the rdfs:label and rdfs:comment properties provide natural language representations and descriptions of the defined term
- Declaring that a `property` has a value that is a `resource`
  - Declaring a `property` where the object value is referred to by `href` instead of `resource` (for example, in an anchor link)
- Declaring multiple values for a given `property` by including multiple tags
- Providing additional schema metadata that wasn't provided in the `@context`, such as signaling that a property is required or functional
  - Declaring that a `property` has literal `content` instead of referring to a resource, and that this content can have its own `datatype` to which it can be coerced
- Excluding a term definition from the automatically generated context document, using the `excluded` attribute



<section id="SomeType" resource="https://w3id.org/fep/888d/SomeType" typeof="rdfs:Class">
<h3>SomeType</h3>
<dl>
<dt>URI</dt>
<dd><code>https://w3id.org/fep/888d/SomeType</code></dd>
<dt>Label</dt>
<dd property="rdfs:label" lang="en">Some Type</dd>
<dt>Comment</dt>
<dd property="rdfs:comment" lang="en">Some type.</dd>
<dt>Subclass of</dt>
<dd><a property="rdfs:subClassOf" href="https://www.w3.org/ns/activitystreams#Object">Object</dd>
<dt>See also</dt>
<dd><a property="rdfs:seeAlso" href="https://w3id.org/fep/888d/exampleA">exampleA</a> | <a property="rdfs:seeAlso" href="https://w3id.org/fep/888d/exampleB">exampleB</a> | <a property="rdfs:seeAlso" href="https://w3id.org/fep/888d/exampleC">exampleC</a></dd>
<dt>Is defined by</dt>
<dd><a property="rdfs:isDefinedBy" href="https://w3id.org/fep/888d">FEP-888d</a></dd>
</dl>
<pre title="Example of a SomeType with properties exampleA, exampleB, exampleC" lang="json">
<code>
{
  "@context": "https://w3id.org/fep/888d",
  "@type": "SomeType",
  "exampleA": true,
  "exampleB": "https://example.com/some-object",
  "exampleC": [1, 1]
}
</code>
</pre>
</section>



<section id="exampleA" resource="https://w3id.org/fep/888d/exampleA" typeof="owl:DatatypeProperty">
<h3>exampleA</h3>
<dl>
<dt>URI</dt>
<dd><code>https://w3id.org/fep/888d/exampleA</code></dd>
<dt>Label</dt>
<dd property="rdfs:label" lang="en">has example relation A with the literal value</dd>
<dt>Comment</dt>
<dd property="rdfs:comment" lang="en">A property with some literal value (string, boolean, number)</dd>
<dt>Domain</dt>
<dd><a property="rdfs:domain" href="https://w3id.org/fep/888d/SomeType">SomeType</a></dd>
<dt>Range</dt>
<dd property="rdfs:range" resource="rdfs:Literal">A literal value</a></dd>
<dt>Required</dt>
<dd property="owl:minCardinality" content="1" datatype="xsd:nonNegativeInteger">Yes</dd>
<dt>Functional</dt>
<dd property="owl:maxCardinality" content="1" datatype="xsd:nonNegativeInteger">Yes</dd>
<dt>Is defined by</dt>
<dd><a property="rdfs:isDefinedBy" href="https://w3id.org/fep/888d">FEP-888d</a></dd>
</dl>
<pre title="Example of a SomeType with properties exampleA, exampleB, exampleC" lang="json">
<code>
{
  "@context": "https://w3id.org/fep/888d",
  "@type": "SomeType",
  "exampleA": true,
  "exampleB": "https://example.com/some-object",
  "exampleC": [1, 1]
}
</code>
</pre>
</section>



<section id="exampleB" resource="https://w3id.org/fep/888d/exampleB" typeof="owl:ObjectProperty">
<h3>exampleB</h3>
<dl>
<dt>URI</dt>
<dd><code>https://w3id.org/fep/888d/exampleB</code></dd>
<dt>Label</dt>
<dd property="rdfs:label" lang="en">has example relation B with</dd>
<dt>Comment</dt>
<dd property="rdfs:comment" lang="en">A property that links to another node on the graph (for example, another object)</dd>
<dt>Domain</dt>
<dd><a property="rdfs:domain" href="https://w3id.org/fep/888d/SomeType">SomeType</a></dd>
<dt>Range</dt>
<dd property="rdfs:range" resource="rdfs:Resource">An object (@id)</dd>
<dt>Required</dt>
<dd property="owl:minCardinality" content="0" datatype="xsd:nonNegativeInteger">No</dd>
<dt>Functional</dt>
<dd>No</dd>
<dt>Is defined by</dt>
<dd><a property="rdfs:isDefinedBy" href="https://w3id.org/fep/888d">FEP-888d</a></dd>
</dl>
<pre title="Example of a SomeType with properties exampleA, exampleB, exampleC" lang="json">
<code>
{
  "@context": "https://w3id.org/fep/888d",
  "@type": "SomeType",
  "exampleA": true,
  "exampleB": "https://example.com/some-object",
  "exampleC": [1, 1]
}
</code>
</pre>
</section>



<section id="exampleC" resource="https://w3id.org/fep/888d/exampleC" typeof="owl:DatatypeProperty">
<h3>exampleC</h3>
<dl>
<dt>URI</dt>
<dd><code>https://w3id.org/fep/888d/exampleC</code></dd>
<dt>Label</dt>
<dd property="rdfs:label" lang="en">has an exampleC value of</dd>
<dt>Comment</dt>
<dd property="rdfs:comment" lang="en">A property that is an ordered list of literal values that are specifically non-negative integers</dd>
<dt>Domain</dt>
<dd><a property="rdfs:domain" href="https://w3id.org/fep/888d/SomeType">SomeType</a></dd>
<dt>Range</dt>
<dd property="rdfs:range" resource="xsd:nonNegativeInteger"> A list of non-negative integers (@list)</dd>
<dt>Required</dt>
<dd property="owl:minCardinality" content="0" datatype="xsd:nonNegativeInteger">No</dd>
<dt>Functional</dt>
<dd property="owl:maxCardinality" content="1" datatype="xsd:nonNegativeInteger">Yes</dd>
<dt>Is defined by</dt>
<dd><a property="rdfs:isDefinedBy" href="https://w3id.org/fep/888d">FEP-888d</a></dd>
</dl>
<pre title="Example of a SomeType with properties exampleA, exampleB, exampleC" lang="json">
<code>
{
  "@context": "https://w3id.org/fep/888d",
  "@type": "SomeType",
  "exampleA": true,
  "exampleB": "https://example.com/some-object",
  "exampleC": [1, 1]
}
</code>
</pre>
</section>



<section id="ExcludedExample" resource="https://w3id.org/fep/888d#ExcludedExample" typeof="rdfs:Class" excluded="1">
<h3>ExcludedExample</h3>
<dl>
<dt>URI</dt>
<dd><code>https://w3id.org/fep/888d#ExcludedExample</code></dd>
<dt>Label</dt>
<dd property="rdfs:label" lang="en">Excluded Example</dd>
<dt>Comment</dt>
<dd property="rdfs:comment" lang="en">A class that is useful for ontology but not intended to be used for the context mapping</dd>
<dt>Subclass of</dt>
<dd><a property="rdfs:subClassOf" href="https://www.w3.org/ns/activitystreams#Object">Object</dd>
<dt>Is defined by</dt>
<dd><a property="rdfs:isDefinedBy" href="https://w3id.org/fep/888d">FEP-888d</a></dd>
</dl>
</section>



## References

- [ActivityPub] Christine Lemmer Webber, Jessica Tallon, [ActivityPub][ActivityPub], 2018
- [AS2-Core] James M Snell, Evan Prodromou, [Activity Streams 2.0][AS2-Core], 2017
- [AS2-Vocab] James M Snell, Evan Prodromou, [Activity Vocabulary][AS2-Vocab], 2017
- [CM-ATTRS] mb21, [Consistent attribute syntax][CM-ATTRS], 2014
- [LD-TERM-DFN] Gregg Kellogg, Pierre-Antoine Champin, Dave Longley, [JSON-LD 1.1 - Section 9.15.1 "Expanded term definition"][LD-TERM-DFN], 2020
- [RFC-2119] S. Bradner, [Key words for use in RFCs to Indicate Requirement Levels][RFC-2119]
- [1] helge, [FEP-2e40: The FEP Vocabulary Extension Process][1], 2023

[ActivityPub]: https://www.w3.org/TR/activitypub

[AS2-Core]: https://www.w3.org/TR/activitystreams-core

[AS2-Vocab]: https://www.w3.org/TR/activitystreams-vocabulary

[CM-ATTRS]: https://talk.commonmark.org/t/consistent-attribute-syntax/272/

[LD-TERM-DFN]: https://www.w3.org/TR/json-ld/#expanded-term-definition

[RFC-2119]: https://tools.ietf.org/html/rfc2119.html

[1]: https://socialhub.activitypub.rocks/t/fep-2e40-the-fep-vocabulary-extension-process/2972

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication 

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
