# Candidate Conversations for the FOIS Weave Demo Paper

Search scope: `dependencies/github.com/semantic-flow/sflo-dendron-notes/`, looking for conversations that develop the `DigitalArtifact` model (`DigitalArtifact`, `ArtifactAnchor`/`ArtifactState`, `Slice`, `AbstractFile`, `LocatedFile`, `ArtifactResolutionSpec`, etc.) and that engage with topics from [[potential-references|potential-references.md]] (FRIR/WEMI, Cool URIs, Linked Data, FAIR vocabularies, content negotiation, version IRIs, ACIMOV, Git-based RDF collaboration, DCAT/PROV/PAV).

These conversations are working notes, not citable sources, but they are useful as:

- raw material for the paper's own exposition of `DigitalArtifact`
- evidence of which prior-art comparisons the team has already half-drafted (especially DAO/IAO/PROV/PAV)
- a record of *why* certain modeling choices were made, which can become "Why:" framing in the paper

## Top picks

### [[sflo.conv.2026.2026-01-30-artifact-identity-and-realizability]]

The core conversation. Works through the entire `DigitalArtifact` lattice from first principles: `ArtifactAnchor`/`ArtifactState` (Working/Historical), `AbstractFile`, `LocatedFile`, `ResourcePage`, the "Realizable" question, and the slash-IRI bijection between manifestation and artifact. Includes an explicit WEMI framing:

> "A distribution is expressed in jsonld or TTL, it is manifested in a particular byte sequence, and when it gets a locating IRI it is an Item."

This is effectively the conversation that *produced* the model that [[ar.functional-requirements-for-information-resource-provenance-on-the-web]] and [[ar.works-expressions-manifestations-items-an-ontology]] would ground. Heaviest single source for the "DigitalArtifact and Provenance" section.

Path: `../../../sflo-dendron-notes/sflo.conv.2026.2026-01-30-artifact-identity-and-realizability.md`

### [[sflo.conv.2025-12-28-distribution-file-vs-artifact]]

Earlier pass at the same problem. Artifact vs NonArtifact, explicit Work/Expression/Manifestation/Item discussion, arrives at the "ResourcePages only for NonArtifacts" rule and the AbstractFile/LocatedFile naming. Good companion/precursor to the above for FRIR/WEMI framing — shows the reasoning trail, not just the destination.

Path: `../../../sflo-dendron-notes/sflo.conv.2025-12-28-distribution-file-vs-artifact.md`

### [[sflo.conv.2026.2026-01-29-dao-vs-gpdav-analysis]]

Directly compares the Semantic Flow spine (`FlowArtifact`/`Slice`) against MITRE D3FEND's Digital Artifact Ontology (DAO), the Information Artifact Ontology (IAO), PROV-O, PAV, and DCAT, with concrete mapping suggestions:

- `previousSlice` ↔ `prov:wasRevisionOf`
- `currentSlice` ↔ `pav:hasCurrentVersion`
- `nextSliceSequenceNumber` ↔ `pav:version`

This is the closest thing to an answer for potential-references.md's open follow-up: "Decide whether the paper should cite PAV or PROV-O directly from product/spec notes in addition to FRIR." It's essentially a draft related-work paragraph.

Path: `../../../sflo-dendron-notes/sflo.conv.2026.2026-01-29-dao-vs-gpdav-analysis.md`

### [[sflo.conv.2026.2026-01-23-nomen-designator-discussion]]

Long, dense discussion (100+ hits on dereferenceability/Cool URI/provenance terms) about identifier vs. location, mesh bases, `expectedPublishUrl`, and what "dereferenceable" actually means for Semantic Flow (HTTP semantics vs. local file browsing vs. `urn:uuid:`). Maps strongly to [[ar.w3.cool-uris]] and [[ar.w3.design-issues.linked-data]] — useful for grounding the paper's identifier/location distinction in concrete design tradeoffs the team already worked through.

Path: `../../../sflo-dendron-notes/sflo.conv.2026.2026-01-23-nomen-designator-discussion.md`

### [[sflo.conv.2026.2026-02-01-modular-datasets]]

Works out whether `RdfDocument`/`AbstractFile` should subclass `dcat:Dataset` vs `dcat:Distribution` vs `void:Dataset`, resolving a real conflation in the ontology (artifact stack vs. dataset/distribution layers). Useful for the DCAT-adjacent reserve references and for the "modular ontology" framing ([[ar.modular-ontology-modeling]]).

Path: `../../../sflo-dendron-notes/sflo.conv.2026.2026-02-01-modular-datasets.md`

### [[sflo.conv.2025-12-18-ontology-of-rdf-syntaxes]]

Discusses `dcat:accessURL`/`dcat:downloadURL`, content negotiation, and Linked Data Fragments/HDT in service of designing artifact access properties. Supports the FAIR-vocabulary/content-negotiation citation cluster ([[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]], [[ar.w3.best-practice-recipes-for-publishing-rdf-vocabularies]]).

Path: `../../../sflo-dendron-notes/sflo.conv.2025-12-18-ontology-of-rdf-syntaxes.md`

## Also worth a look

### [[sflo.conv.2026.2026-03-16_1418-picking-up-the-pieces-codex]]

Reframes the artifact lattice as "facets" — `AbstractArtifact`/`Flow`/`State`/`AbstractFile`/`LocatedFile` as granularity levels of one thing (via `narrowerFacet`/`broaderFacet`), and tackles "is every file a DigitalArtifact?" A clean, recent (most up-to-date) statement of the model — good source for a paper figure or for checking that older conversations' terminology hasn't drifted.

Path: `../../../sflo-dendron-notes/sflo.conv.2026.2026-03-16_1418-picking-up-the-pieces-codex.md`

### [[sflo.conv.2026.2026-03-14_0958-existing-solutions-that-could-be-extended-with-weave-codex]]

Architecture comparison vs. Lume/Eleventy/Hugo for static ResourcePage generation. Relevant to [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]] and the static-publication positioning, plus repo/Git workflow decisions echoing [[ar.the-acimov-methodology-agile-and-continuous-integration-for-modular-ontologies-and-vocabularies]].

Path: `../../../sflo-dendron-notes/sflo.conv.2026.2026-03-14_0958-existing-solutions-that-could-be-extended-with-weave-codex.md`

## Suggested next step

The DAO/IAO/PROV/PAV comparison ([[sflo.conv.2026.2026-01-29-dao-vs-gpdav-analysis]]) and the artifact-identity conversation ([[sflo.conv.2026.2026-01-30-artifact-identity-and-realizability]]) are the strongest candidates to mine first when drafting the "DigitalArtifact and Provenance" section — between them they cover both "how the model works" and "how it relates to existing ontologies."
</content>
