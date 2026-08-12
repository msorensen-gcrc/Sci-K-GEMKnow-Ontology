# Changelog

All notable changes to the CEBP ontology are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this ontology follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

For ontology versioning, the semantic version components are interpreted as:

- **MAJOR** — breaking changes: removed classes or properties, renamed IRIs without deprecation redirects, semantic shifts that invalidate existing instance data.
- **MINOR** — backwards-compatible additions: new classes, new properties, new annotations on existing terms, new equivalence mappings.
- **PATCH** — corrections that do not change the ontology's meaning: typo fixes in labels and comments, formatting changes, additions of missing `rdfs:label` or `rdfs:comment`, internal namespace cleanup that preserves IRI identity.

The version recorded here must match `owl:versionInfo` in `cebp.ttl`.

---

## Unreleased

No changes yet. For upcoming work see `CLEANUP_PROCEDURE.md`.

---

## [0.6.0] 2026-08-11

Folded in the new relations from Christy Caudill's v21 edit. She worked from the 0.5.0 file in Protégé, so her version reintroduced the namespace and prefix problems that were fixed back in 0.2.0 (the `untitled-ontology-79` base, `dcterms: <dcterms:>`, the `cebp1:`/`cebp2:` split, and external vocabulary terms redeclared as `owl:Class`). None of that was carried over, only the new relations were taken. Note this has only been an update to the ontology and thus has not been transferred to the Neo4j database yet (as outlined in step 9 in CLEANUP-PROCEDURE.md). Until that time the ontology will be out of sync from the database.

### Added

- New object properties: `isConnectedTo` and `relatesTo`. `isConnectedTo` came with a definition from Christy. `relatesTo` has a label but still needs a comment.
- `rdfs:domain` and `rdfs:range` for `affects`, `isFoundIn`, and `isTiedTo`. These three were added in 0.5.0 with no linkages, so this is the first time they identify which classes they connect. They still need comments.
- New class pairings on `informs`: `ConservationStrategy` → `ConservationFramework`, `IndigenousConservationConcept` → `ConservationStrategy`, `IndigenousConservationConcept` → `SocialEcology`, `SocialEcology` → `CommunityBasedMonitoring`, and `Actor` → `ConservationFramework`.
- New class pairings on `isPartOf`: `Environment` → `SocialEcology`, `Species` → `ConservationStrategy`, `Species` → `SocialEcology`, and `Actor` → `SocialEcology`.
- New class pairing on `isRepresentativeOf`: `ConservationManager` → `ParksCanada`.
- SHACL-MAP notes for all of the above. The pairings that came from v21 are marked so they can be told apart from the earlier ones.

### Changed

- The relations Christy wrote as `owl:someValuesFrom` restrictions were converted to `rdfs:domain` and `rdfs:range` with `owl:unionOf`, the same way the earlier restrictions were handled back in 0.4.0.
- Class names in the new relations were converted to singular to match our class schema, e.g. `Resources` → `Resource`, `Roles` → `Role`, `ConservationFrameworks` → `ConservationFramework`, `communityBasedMonitoring` → `CommunityBasedMonitoring`.
- Added the `@en` language code to labels that were missing it, e.g. `isConnectedTo`.

### Fixed

- `owl:versionIRI` was still pointing at 0.4.0 while `owl:versionInfo` said 0.5.0. Both now say 0.6.0.
- `rdfs:Range` on `cebp:hasRole` was capitalized, which is not a real property. Corrected to `rdfs:range`.

---

## [0.5.0] 2026-07-02

Folded in additions from Christy Caudill's June30-2026 edit of the ontology (made independently from the v18 baseline). Reconciled her changes against the current cleaned version so the existing structural fixes and naming conventions were not reintroduced. Note this has only been an update to the ontology and thus has not been transferred to the Neo4j database yet (as outlined in step 9 in CLEANUP-PROCEDURE.md). Until that time the ontology will be out of sync from the database.

### Added

- New classes: `IndigenousConservationConcept`, `CulturalPractice`, `FoodSecurity`, `FoodSecurityConcern`, `SubsistencePractice`, `HistoricalRelationshipWithLand`, `Maligait`, `HolisticFramework`, `CommunityBasedMonitoring`, `GovernmentOfNunavut`, `ParksCanada`, `TrainingForNonIndigenousResearchers`
- New object properties: `affects`, `isFoundIn`, `isTiedTo`. These were added, however, without a domain, range, or comment and still need to be documented.
- `SubsistencePractice` added to the domain, and `HistoricalRelationshipWithLand` added to the range, of `cebp:isPartOf`, along with a SHACL-MAP note recording the pairing.

### Changed

- Class names from Christy's edit were converted to singular to match our class schema, e.g. `HolisticFrameworks` → `HolisticFramework`.
- Descriptions that were pasted directly from source papers (`Maligait`, `HolisticFramework`, `CommunityBasedMonitoring`) were rewritten as proper `rdfs:comment` definitions, with the citation moved to `dcterms:source`.

### Fixed

- Did not carry over `Provincal`, a misspelled duplicate of the already-existing `ProvincialGovernment`.
- Did not carry over the `hasRole`/`someValuesFrom` restriction that had `IndigenousConservationConcept` as a subclass of `Actor`. A body of concepts being a subclass of an actor doesn't make much sense and is therefore a category error. So, `IndigenousConservationConcept` was kept as a standalone class with no parent for now until this is discussed further.

---

## [0.4.0] 2026-06-04

Structural cleanup version release. This release addresses issue 6 and 7 in the CLEANUP-PROCEDURE.md document. Note this has only been an update to the ontology and thus has not been transferred to the Neo4j database yet (as outlined in step 9 in CLEANUP-PROCEDURE.md). Until that time the ontology will be out of sync from the database.

### Added

- `rdfs:label` and `rdfs:comment` to every object property and class. Although, the majority of classes still need to have a comment written to describe the definition of the class and how it is expected to be used.
- `rdfs:domain` and `rdfs:range` for all of the object properties that previously had `owl:Restriction` relationships to identify linkages between classes.
- SHACL mappings as comments in the ttl file to some of the object properties that have multiple use cases.
- Language code `@en` for all of the labels and comments to identify their language.

### Changed

- Altered some of the names of Classes that are plural. This does not fit in with our class schema.
- Replaced `owl:equivalentClass` with `skos:exactMatch`. This way parsers will not try and lump in any restrictions the already established ontologies have with our ontology.

---

## [0.2.0] 2026-05-26

Structural cleanup version release. This release addresses issues 2, 3, 4, 5 in the CLEANUP-PROCEDURE.md document. Note this has only been an update to the ontology and thus has not been transferred to the Neo4j database yet (as outlined in step 9 in CLEANUP-PROCEDURE.md). Until that time the ontology will be out of sync from the database.

### Added

- Full `owl:Ontology` metadata block: `owl:versionIRI`, `owl:versionInfo`,
  `dcterms:title`, `dcterms:description`, `dcterms:creator`,
  `dcterms:license`, `dcterms:created`, `dcterms:modified`
- `@prefix cebp:` declaration binding the canonical CEBP namespace
- `@prefix obo:` declaration for OBO Foundry terms

### Changed

- All of the CEBP namespace IRIs have been changed to the canonical `http://gcrc.carleton.ca/ontologies/2023/GCRC_CEBP_Genome_Architecture/` form. All of the old forms (`#`, `/#`, `//`, `/cebp#`, `/cebp:`, and bare `/`) have been removed. This is would be a change for anyone who previously used the earlier IRIs.

### Fixed

- Removed ~20 external-vocabulary terms (`rdfs:label`, `dc:description`,
  `dcterms:title`, `skos:note`, `dcat:relation`, `dc11:keyword`, and others)
  that were incorrectly declared as `owl:Class` or `owl:AnnotationProperty`
- Removed imported BFO/IAO editorial annotation stubs (`IAO_0000114`,
  `IAO_0000116`, `IAO_0000424`, `IAO_0000600`, `IAO_0000601`, `IAO_0000602`,
  `IAO_0010000`, `BFO_0000179`, `BFO_0000180`, `BCO_0000060`) and the
  `BFO_0000006` documentation block; BFO alignment preserved on
  `cebp:SpatialRegion` via `owl:equivalentClass`
- Removed `<http://ihtsdo.org/snomedct/organism#LANGUAGECODE>` stub
- Removed local redeclaration of `owl:sameAs` as `owl:ObjectProperty`
- Temporarily comented out incorrect `owl:sameAs` literal on `cebp:ConservationEvaluation`
- Replaced `<http://www.w3.org/2000/10/swap/pim/contact#Person>` with
  `foaf:Person` on `cebp:CommitteeMember` and `cebp:KnowledgeHolder`
- Replaced non-standard `rdfs:publication` with `dcterms:source`
- Fixed spelling: `cebp:NorthernIndigneousCommunities` →
  `cebp:NorthernIndigenousCommunities`
- Fixed broken `ns0:` Darwin Core prefix → canonical `dwc:`
- Renamed `cebp:useCaseText` (from `cebp:useCase-text`, hyphen invalid
  in Turtle prefixed names)
- Removed duplicate `dc:` / `terms:` prefix declarations for
  `http://purl.org/dc/terms/`

---

## [0.1.0] 2026-05-20

Initial version release. Base import from Christy Caudill's v19 that removed instance data from the ontology (this will be kept within the knowledge graph). The ontology is in active development and remediation please see `CLEANUP_PROCEDURE.md` for the plan of what is getting fixed and the `README.md` for the current known issues.

### Added

- `ontology/` directory under version control
- `cebp.ttl` — schema-only ontology derived from `cebp-v19-stable-ontology-standard.ttl` with all 400 `owl:NamedIndividual` declarations and 2 orphan `owl:Axiom` reifications removed
- `README.md` — directory overview and current status
- `STYLE.md` — initial authoring conventions
- `CLEANUP_PROCEDURE.md` — active remediation plan
- `archive/cebp-v18.ttl` — frozen snapshot of v18 for reference
- `archive/cebp-v19-with-instances.ttl` — frozen snapshot of v19 before instance removal

### Removed

- 400 `owl:NamedIndividual` declarations (instance data; relocated to Neo4j)
- 2 `owl:Axiom` reifications that annotated removed individuals
