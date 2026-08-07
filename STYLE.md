# Ontology Style Guide

This document provides authoring conventions for `cebp.ttl`. This is a living document where some decisions have been noted as TBD with their benefits and tradeoffs. These may then be resolved in subsequent revisions with supervisor and team sign-off.

Decisions in this document take precedence over the patterns currently visible in the file, since the file itself is being edited and cleaned. Cleanup work brings the file into alignment with this guide, not the other way around.

---

## Canonical namespace

**Status:** TBD. To be resolved in Step 4 of `CLEANUP-PROCEDURE.md`.

### Examples of Candidates

| IRI | Trait |
|---|---|
| `https://gcrc.carleton.ca/ontologies/cebp/` | Institutional anchor; requires Carleton IT to host content at this URL eventually |
| `https://w3id.org/cebp/ontology/` | Permanent identifier service; redirects to wherever the ontology is hosted; requires a small PR to the w3id.org repository |
| `http://gcrc.carleton.ca/ontologies/2023/GCRC_CEBP_Genome_Architecture/` | Current value in v19 and in current v0.1.0; long and dated path |

### Constraints on the decision

- The namespace IRI must be resolvable, or must be in a domain that will eventually serve a resolvable definition. RDF best practice is that an IRI used as an identifier should also dereference to a description of the resource it names.
- Whatever is chosen, it is committed to once. Subsequent namespace changes are MAJOR-version breaking changes and require deprecation redirects from the old IRIs.

The current six legacy namespace shapes in `cebp.ttl` (`#`, `/`, `/#`, `//`, `/cebp#`, `/cebp:`) all collapse into the chosen canonical form during Step 5 of the cleanup procedure.

---

## Separator

**Decision:** `/` (slash) between the namespace and local name.

Example: `https://example.org/ontology/Species`, never `https://example.org/ontology#Species`.

### Rationale

- The W3C [Cool URIs for the Semantic Web](https://www.w3.org/TR/cooluris/) note distinguishes "hash URIs" (`#`) and "slash URIs" also known as "303 URIs" (`/`) as two acceptable patterns. In section 4.4 of the document W3C writes in defence of 303 URIs "A client interested only in #product123 will inadvertently load the data for all other resources as well, because they are in the same file. 303 URIs, on the other hand, are very flexible because the redirection target can be configured separately for each resource. There could be one describing document for each resource, or one large document for all of them, or any combination in between. It is also possible to change the policy later on."
- Furthermore, most of the IRIs in v19 already use `/` as the namespace terminator. Standardizing on `/` requires the fewest rewrites.
- Protégé seems to generate some terms with the `#` symbol resulting in `/#LocalName` URIs (507 of them in v19) result from Protégé appending `#` to a base URI that ends in `/`. Treating `/` as the canonical terminator eliminates this class of error rather than working around it.

---

## Local-name casing

**Decision:**

- **Classes:** `CamelCase` (also called `UpperCamelCase` or `PascalCase`). Example: `ConservationStrategy`, `IndigenousCommunity`, `GenomicsMethod`.
- **Object and datatype properties:** `camelCase` (also called `lowerCamelCase`). Example: `isPartOf`, `hasLocation`, `informs`.
- **Annotation properties:** `camelCase`. Example: `layDescription`, `exampleTextSnippet`.
- **Instance individuals:** `camelCase` if they are ever included in the ontology file. Instances normally live only in Neo4j; if they are merged into the TTL (for instance, for export or publication), they follow the same `camelCase` rule as properties to distinguish them from classes at a glance.

### Rationale

This convention is standard practice across published ontologies (OWL primer examples, FOAF, Schema.org, Darwin Core) and provides a visual cue for the role of each term: an IRI ending in `/Species` is clearly a class, an IRI ending in `/isPartOf` is clearly a property.

### Exceptions

- Acronyms in class names are written as their canonical capitalized form: `DNASequence`, `RNATranscript`, not `DnaSequence` / `RnaTranscript`. This may produce locally-jarring forms (`eDNASample`) which are nonetheless preferable to forms that obscure the acronym.
- Numeric or coded identifiers from external vocabularies retain their source form (`ENVO_00000358`), since rewriting them breaks the link to the source ontology.

---

## Annotations on every term

**Decision:**

Every locally-defined class, object property, and datatype property — that is, every term in the `cebp:` namespace — carries at minimum:

- `rdfs:label` — short human-readable name suitable for display in a UI
- `rdfs:comment` — one- to three-sentence definition suitable for a developer or curator reading the ontology directly

For locally-defined object and datatype properties, additionally:

- `rdfs:domain` — the class (or `owl:unionOf` of classes) to which the property applies as subject
- `rdfs:range` — the class (or `owl:unionOf` of classes; or a datatype IRI for datatype properties) that the property accepts as object

### Scope of these requirements

These requirements apply only to terms **defined in the `cebp:` namespace**. They do not apply to external vocabulary terms referenced from the ontology (`dcterms:isPartOf`, `skos:broader`, `rdfs:label`, etc.). External terms are referenced as-is from their source vocabulary, which has made its own choices about domain and range — typically omitting them deliberately so the term stays reusable across contexts.

### Narrow exception: local parent properties

Where a local property exists primarily to serve as the parent of typed subproperties (per the "Property design" section), and is not itself intended to appear directly on instance data, `rdfs:domain` and `rdfs:range` may be omitted. In that case the `rdfs:comment` must explicitly state the property's role as a parent and list (or characterize) its known subproperties. Such cases should be rare: where a standard vocabulary parent (`dcterms:isPartOf`, `skos:broader`, etc.) is available, it is preferred over a local parent.

### Semantics of domain and range

`rdfs:domain` and `rdfs:range` are treated as documentation for human readers and as inputs to UI generation, not as enforcement. RDFS interprets multiple domain or range statements as conjunction (AND), so any property that applies to multiple disjoint classes uses `owl:unionOf` rather than multiple `rdfs:domain` triples.

Enforcement of cardinality, type constraints, and required properties is delegated to SHACL shapes in `shapes/cebp-shapes.ttl` (not yet written).

---

## Display labels on instances

**Decision:** instance nodes in Neo4j use `skos:prefLabel` as the canonical display label. Alternative labels (translations, abbreviations, acronyms) use `skos:altLabel`.

`rdfs:label` may also be present on instance nodes as a fallback for tools that don't recognize SKOS, but it is not the primary display field.

This applies to instance data in Neo4j. Ontology terms (classes, properties) use `rdfs:label`, since `skos:prefLabel` is intended for SKOS concepts (instances), not OWL terms (schema).

---

## Language tags

**Status:** TBD. To be resolved when language-tagging conventions are agreed on.

Pending the decision, current practice is mixed: some `rdfs:label` strings in v19 carry `@en`, most carry no tag. After resolution, this section will specify:

- Whether all English-language strings must be tagged `@en` explicitly
- The handling of Indigenous-language labels (e.g. Inuktitut syllabic representations) and their corresponding language tags
- Whether the ontology itself carries multilingual labels or only English with translations living in instance data

---

## Use of external vocabularies

**Decision:** external vocabulary terms are referenced as-is, never redeclared locally.

The following vocabularies are in use:

| Prefix | IRI | Purpose |
|---|---|---|
| `owl:` | `http://www.w3.org/2002/07/owl#` | OWL primitives |
| `rdf:` | `http://www.w3.org/1999/02/22-rdf-syntax-ns#` | RDF core |
| `rdfs:` | `http://www.w3.org/2000/01/rdf-schema#` | RDFS annotations and class hierarchy |
| `skos:` | `http://www.w3.org/2004/02/skos/core#` | Concept labels, alignments |
| `dcterms:` | `http://purl.org/dc/terms/` | Dublin Core metadata (preferred over `dc:` and `terms:`) |
| `foaf:` | `http://xmlns.com/foaf/0.1/` | Agents, persons, organizations |
| `dwc:` | `http://rs.tdwg.org/dwc/terms/` | Darwin Core (biodiversity) |
| `bibo:` | `http://purl.org/ontology/bibo/` | Bibliographic ontology |
| `schema:` | `http://schema.org/` | Schema.org |
| `bfo:` | `http://purl.obolibrary.org/obo/bfo.owl#` | Basic Formal Ontology |
| `iao:` | `http://purl.obolibrary.org/obo/iao.owl#` | Information Artifact Ontology |
| `envo:` | `http://purl.obolibrary.org/obo/envo.owl#` | Environment Ontology |
| `xsd:` | `http://www.w3.org/2001/XMLSchema#` | XML Schema

### Anti-pattern

A term such as `rdfs:label`, `dcterms:title`, or `skos:note` must never appear as the subject of an `owl:Class`, `owl:ObjectProperty`, `owl:DatatypeProperty`, or `owl:AnnotationProperty` declaration in `cebp.ttl`. Because these terms are already defined by their source vocabulary redeclaring them locally creates conflicts in assertions and is the regression that the cleanup procedure's Step 2 corrects.

### Prefix duplication

Each external vocabulary is bound to exactly one prefix. While writing this doc the v19 file carried both `dcterms:` and `terms:` bound to `http://purl.org/dc/terms/`, which we will be eliminating under this convention. The canonical prefix for `http://purl.org/dc/terms/` is `dcterms:`.

---

## Property design

**Decision:** prefer polymorphic properties over hyper specific subproperties.

When the same general relationship applies to multiple class pairs (for example, `isPartOf` applies to species-in-genus, strategy-in-framework, location-in-region), the ontology declares a general parent property that is either aligned with a standard vocabulary (`dcterms:isPartOf`, `skos:broader`, etc.) or one that is specifically designed for the GEM-KNOW ontology (`cebp:isAKeyword`, `cebp:makesAssessmentOf`, etc.).

### Rationale

A fully polymorphic property allows for a more flexible property that can apply to more use cases. Furthermore, validation can still occur with the use of SHACL shapes by specifying the domain and range of each shape (however, this is still under development). Extensive subproperties would result in proliferation of many more properties which would be harder to maintain, is non-standard, and harder to implement.

There is a case to be made for more specific subproperties that are governed under broader properties. For example it does allow for more explicit validation, however, for the purposes of this application we don't see this being a particularly pressing issue. If in the future this presents as a necessity, discussions may be had about altering the property format.

---

## Class expressions

**Decision:** use plain `rdfs:subClassOf` for the class hierarchy. Avoid `owl:Restriction`, `owl:someValuesFrom`, and `owl:allValuesFrom` as the primary expression of relationships.

OWL restrictions are appropriate when reasoning-time inference is genuinely intended (for example, asserting that every member of `RNAVirus` must have some `rna:Genome`). They are inappropriate as a substitute for plain property declarations: they generate blank nodes in Neo4j, are harder to read in the source TTL, and OWL's open-world semantics means they do not validate instance data the way SHACL does.

Existing `owl:Restriction` declarations in `cebp.ttl` (25 in v19) are audited during cleanup and retained only where reasoning-time inference is the intended effect. Otherwise, they are converted to plain object property declarations with `rdfs:domain` and `rdfs:range`.

### Equivalence statements

`owl:equivalentClass` and `skos:exactMatch` are appropriate and encouraged for crosswalk alignments to external vocabularies. 

---

## Versioning

**Decision:** Semantic Versioning (`MAJOR.MINOR.PATCH`) as recorded in `CHANGELOG.md`.

The current version is mirrored as `owl:versionInfo` in `cebp.ttl`. `owl:versionIRI` follows the pattern `<canonical-namespace>/X.Y.Z`.

See `CHANGELOG.md` (and the worked examples in `CHANGELOG_examples.md`) for the mapping between change types and version components.

---

## Authoring workflow

**Status:** TBD. Affects whether direct edits to `cebp.ttl` are produced through Protégé, by hand, or through an alternative tool such as WebProtégé or VocBench.

### Considerations

- Protégé has demonstrably introduced regressions on every recent revision (six namespace shapes in v19, external vocabulary terms misclassified as classes, `owl:sameAs` redefined locally).
- Hand-editing TTL produces cleaner diffs and avoids the Protégé bug surface but requires direct RDF/OWL fluency from all editors.
- WebProtégé and VocBench are alternative ontology editors that may have different bug profiles and better support for collaborative editing.

This section will record the chosen workflow once the decision is made. The decision affects who can contribute and how supervisor contributions are integrated.

---

## References

External standards and conventions cited in this document are listed with full citations in `README.md` under "References."
