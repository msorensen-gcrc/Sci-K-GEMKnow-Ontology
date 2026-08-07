# Ontology Cleanup Procedure
Active remediation plan for cebp.ttl. The starting point is the schema-only file derived from cebp-v19-stable-ontology-standard.ttl with all instance data removed.

## High-Level Summary of Goals
**Step 1** — Verify the file parses: Ensures the base file is syntactically valid before making any edits.

Example: Catching baseline typos like a broken rightsHolder IRI or local owl:sameAs redefinitions early.

**Step 2** — Fix misclassified declarations & redefinitions: Cleans up structural errors where standard vocabularies were misused.

Example: Stripping duplicate owl:Class designations from external properties and removing our invalid local override of owl:sameAs.

**Step 3** — Add ontology metadata: Establishes proper versioning, tracking, and attribution directly within the file.

Example: Injecting standard fields like owl:versionIRI, dcterms:license, and an updated modification date.

**Step 4** — Lock the canonical namespace: Solidifies a single, permanent IRI format for the project to prevent future namespace fragmentation.

Example: Formally choosing the / terminator, binding it to the cebp: prefix, and documenting the decision in STYLE.md.

**Step 5** — Collapse all CEBP IRIs into the canonical namespace: Standardizes all local identifiers across the entire file.

Example: Running a script to migrate the six different legacy variations of the CEBP namespace into our newly chosen uniform format.

**Step 6** — Document classes: Ensures every class is human-readable and structurally sound.

Example: Adding clear rdfs:label and rdfs:comment tags to classes and resolving dangling parent classes like BCO or Producers.

**Step 7** — Document and constrain object properties: Defines exactly how properties should behave and what they connect.

Example: Appending explicit rdfs:domain and rdfs:range restrictions to properties and phasing out custom properties like cebp:isA in favor of standard rdf:type.

**Step 8** — Formalize the metadata crosswalk: Integrates external mapping documentation directly into the graph.

Example: Translating an offline mapping spreadsheet into native owl:equivalentClass and skos:exactMatch statements so the TTL becomes the sole source of truth.

**Step 9** — Reconcile with Neo4j: Syncs our structural ontology changes with the live graph database.

Example: Identifying mismatches between the TTL file and Neo4j, then running Cypher scripts to update old node URIs to the new canonical format.

**Step 10** — Backfill skos:prefLabel on instance data: Ensures all live data elements look clean and readable in the user interface.

Example: Formatting raw strings and fixing acronyms (like DNA, RNA, NCBI) or external ENVO identifiers so they render nicely on screen.

**Step 11** — Add deprecation redirects for renamed terms: Prevents breaking changes for external systems or historical data queries when terms are renamed.

Example: Creating a backward-compatible deprecation stub pointing IndiginousCommunities to the newly corrected IndigenousCommunity class.

**Step 12** — Wire the ontology into the application: Uses the clean ontology data to dynamically drive the application's backend and frontend.

Example: Parsing the TTL at API startup so the UI dashboard automatically pulls and displays human-readable property labels instead of raw database keys.

**Step 13** — Set up validation and CI: Builds automated guardrails to ensure future updates don't break the ontology.

Example: Creating a GitLab CI job that automatically scans any incoming Merge Request to guarantee the modified TTL file parses perfectly and includes all mandatory documentation.