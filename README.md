# CEBP Ontology

This directory holds the formal schema for the CEBP ontology. It is maintained by the Geomatics and Cartographic Research Centre (GCRC) at Carleton University and used by the GEM-Know application to drive search and curation of genomics concepts for the Canada Earth Biogenome Project in concert with our Indigenous partners.

## What's in here

```
ontology/
├── README.md                    This file
├── CHANGELOG.md                 Version history and notable changes
├── CLEANUP-PROCEDURE.md         The active remediation plan
├── STYLE.md                     Authoring conventions (namespaces, labels, properties)
├── cebp.ttl                     The canonical ontology (Turtle format)
└── archive/                     Frozen snapshots of earlier versions
    ├── cebp-v18.ttl
    └── cebp-v19-with-instances.ttl
```

The canonical file is `cebp.ttl`. Everything else is supporting material.

## Status

**The ontology is in active remediation.** The current `cebp.ttl` is derived from `cebp-v19-stable-ontology-standard.ttl` (authored in Protégé) with instance data removed. The instance data will be held within the Neo4j database which will also be referenced from now on as the Knowledge Graph (KG). The `cebp.ttl` file parses as valid Turtle but still has known structural issues including but not limited to: fragmented namespaces, missing documentation on most terms, several standard vocabulary properties misclassified as classes, and proper distinction between object, annotation, and datatype properties. All of these issues are being worked through systematically. See `CLEANUP-PROCEDURE.md` for the active plan and where it sits.

Until that cleanup is complete, treat the ontology as **internal working material**. Don't link to its IRIs from external systems yet; they may be subject to change.

## What the ontology describes

CEBP models the concepts and relationships involved in conservation genomics work. Core classes include:

- **Species** — taxonomic entities under conservation focus
- **ConservationStrategy** — high-level approaches (assisted migration, captive breeding, habitat protection, etc.)
- **ConservationEvaluation** — assessment processes and methodologies
- **GenomicsMethod** — tools and techniques (whole-genome sequencing, eDNA, etc.)
- **Actor** — people, organizations, committees, and Indigenous communities involved in the work
- **Resources** — papers, tools, databases, images
- **GeoLocation** — spatial regions (aligned with Darwin Core and BFO)
- **IndigenousConservationConcepts** — Indigenous knowledge and rights concepts

Object properties (`informs`, `isPartOf`, `isProvidedBy`, etc.) connect these classes into a navigable graph.

## Conventions

These are the short version. `STYLE.md` has the full set with rationale.

- **Canonical namespace:** *(to be decided in Step 2 of the cleanup — placeholder for now)*
- **Separator:** `/` between namespace and local name (no `#`)
- **Display labels:** `skos:prefLabel` on instance nodes, `rdfs:label` on ontology terms
- **Property documentation:** every `owl:ObjectProperty` should carry `rdfs:label`, `rdfs:comment`, `rdfs:domain`, and `rdfs:range`
- **Validation:** `rdfs:domain`/`rdfs:range` are documentation, not enforcement. Real validation lives in `shapes/cebp-shapes.ttl` (SHACL, not yet written) and in the API middleware.
- **External vocabularies:** `dcterms`, `skos`, `foaf`, `dwc`, `BFO`, `IAO`, `schema.org`, `bibo` — used as-is, never redeclared

## How it fits with the rest of the system

```
.ttl file (this directory)  ──►  Neo4j graph database  ──►  Node.js API  ──►  Frontend
        canonical                  operational copy           resolves              renders
                                                              labels at API         labels, not
                                                              boundary              raw keys
```

The TTL file is the source of truth. Neo4j holds a derived operational copy, loaded via [neosemantics (n10s)](https://neo4j.com/labs/neosemantics/) when it's available, or via hand-written Cypher when it isn't. The API loads the ontology at startup to resolve human-readable labels for the UI.

When the ontology changes, the flow is:

1. Edit `cebp.ttl` on a branch
2. Open an MR; CI validates the file parses and meets style requirements
3. Merge to `main`
4. Apply any Neo4j migrations from `migrations/`
5. Restart the API (which reloads the ontology into memory)

## Working with the file

**Parse and validate:**

```bash
# With rdflib (Python)
python -c "from rdflib import Graph; g = Graph(); g.parse('cebp.ttl', format='turtle'); print(f'{len(g)} triples')"

# With Apache Jena
riot --validate cebp.ttl
```

**Browse interactively** with [Protégé](https://protege.stanford.edu/) or [WebProtégé](https://webprotege.stanford.edu/). Note: Protégé's namespace handling has historically introduced regressions in this file (see `CHANGELOG.md` entries for v18 → v19). If you edit in Protégé, expect to clean up the diff before committing.

## Contributing

The maintainer is *Mako Sorensen*. The ontology contributor is *Christy Caudill*. Substantive schema changes need supervisor sign-off; cleanup, refactoring, and documentation changes can be made by the maintainer with notification.

To propose a change:

1. Branch from `main`
2. Edit `cebp.ttl` (or other files in this directory)
3. Update `CHANGELOG.md` with a brief entry
4. Open a merge request; CI will run the validation script
5. Tag the supervisor for review on substantive changes

## References

- Project context and design decisions: *(link to internal context document)*
- Neosemantics (n10s) docs: https://neo4j.com/labs/neosemantics/
- SHACL specification: https://www.w3.org/TR/shacl/
- Darwin Core terms: https://dwc.tdwg.org/terms/
- SKOS reference: https://www.w3.org/TR/skos-reference/
- Basic Formal Ontology (BFO): https://basic-formal-ontology.org/