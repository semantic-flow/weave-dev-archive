# Codex Outline: FOMI 2026 Semantic Mesh / DigitalArtifact Paper

Working title:

> Semantic Meshes: Stable Identifiers, Immutable Data, and Dynamic Sense-Finding

More precise variant:

> Semantic Meshes: Stable Identifiers, Immutable Evidence, and Dynamic Sense-Finding

Literal fallback title:

> Semantic Flow: Byte-Grounded Digital Artifact Histories for Static RDF Publication

## Scope

- FOMI model/practice paper
- Semantic Meshes first
- Semantic Flow as the framework/model behind them
- Weave as reference implementation and running example
- FOIS Demonstrations paper reserved for:
  - command workflow
  - screenshots
  - static site generator comparison
  - Weave implementation details
  - interactive resource browsing

## Thesis

- Public RDF identifiers need:
  - high availability and no server-side fixes
  - stable sense-finding support
  - governed artifact-level continuity handles
  - reviewable byte coordinates
  - explicit histories
  - dereferenceable presentation pages
- Identifier senses can evolve, and may even become inconsistent with earlier states.
- Semantic Flow does not freeze meaning; it supplies a coherent framework for establishing, inspecting, and revising meaning.
- Identifiers for digital artifacts and their facets should be as specific and predictable as the reference task requires.
- A `DigitalArtifact` IRI can serve as a governed artifact-level continuity handle.
- Histories, states, manifestations, located files, and per-layer ResourcePages are facets of that artifact identity in the broader modeling sense.
- These layers make artifact identity inspectable at chosen levels of specificity.
- For identifiers that are not themselves `DigitalArtifact`s, Knop support artifacts still help establish sense:
  - `ReferenceCatalog`
  - `KnopSourceRegistry`
  - `KnopInventory`
  - `KnopMetadata`
  - `ResourcePageDefinition`
- Primary data and settled historical states should be persistent or quasi-immutable.
- Evolving metadata artifacts can point at latest, working, current, deprecated, replaced, or exact coordinates.
- Metadata can itself have states.
- Meta-metadata can have states if somebody mints an IRI for it.
- Semantic Flow contributes a DCAT-compatible publication layer.
- Semantic Flow does not replace:
  - DCAT catalogs
  - Git storage and collaboration
  - WEMI/domain work-expression modeling
  - downstream ontology diff, compatibility analysis, or migration tooling

## Contribution Claims

- DCAT-compatible byte-grounded artifact model.
- `DigitalArtifact` as governed artifact identity, subclassing `dcat:Dataset`.
- `ArtifactManifestation` as state-situated representation, subclassing `dcat:Distribution`.
- `HistoricalState` as explicit immutable temporal facet.
- `ArtifactHistory` as explicit internal lineage support for one artifact.
- `LocatedFile` as first-class retrievable byte identity.
- `ResourcePage` as special located HTML file for static dereferenceability.
- `ArtifactResolutionSpec` as reviewable coordinate relator.
- Sparse modeling via consistency-preserving skip-level properties.
- Expected navigation grammar:
  - artifact
  - history
  - state
  - manifestation
  - located file
  - resource page
  - source coordinate

## 1. Introduction

- Opening pressure:
  - ontology publication exposes public IRIs
  - evidence lives in files, generated pages, release directories, source registries, shapes, examples, repository commits, and static hosting paths
  - major revisions create public contract changes, not only diffs
- Current gap:
  - Git tracks repository bytes
  - DCAT describes cataloged data resources and distributions
  - Cool URI guidance explains dereferenceability patterns
  - existing ontology publication tools generate useful pages
  - missing: one RDF-native publication grammar for identity, state, representation, file, page, and coordinate
- Main claim:
  - Semantic Flow supplies the missing publication layer for byte-grounded digital artifact histories.

## 2. Prior Work Spine

- DCAT 3:
  - broad `dcat:Dataset`
  - `dcat:Distribution` as representation
  - `dcat:DatasetSeries`
  - versioning terms
  - `dcat:landingPage`
  - `dcat:accessURL`
  - `dcat:downloadURL`
  - `spdx:checksum`
- FRIR:
  - information-resource provenance
  - closest conceptual touchpoint
  - no explicit Semantic Flow-style `HistoricalState`
- WEMI / OpenWEMI:
  - useful warning about layered ambiguity
  - not Semantic Flow's model
  - Semantic Flow leaves work/expression interpretation to publisher ontologies
- Cool URIs / httpRange-14 / Tennison:
  - resource/content distinction
  - punning and URI sense
  - Semantic Flow stance: URI has a socially governed sense in context; URI does not literally refer to a sense
- Structure versus concept:
  - Cagle and Shannon distinguish structural ontology from conceptual/taxonomic layers
  - useful warning against collapsing concept meaning into formal structure
  - Semantic Meshes separate support structure from the domain concepts whose senses are being established
  - DigitalArtifact facets describe artifact structure
  - ReferenceCatalogs, source registries, inventories, metadata, and ResourcePages support sense-finding
- FAIR vocabulary guidance and RDF publication recipes:
  - metadata
  - version IRIs
  - documentation
  - serializations
  - persistence
- Static RDF and ontology publication tools:
  - Jekyll RDF
  - Widoco
  - LODE / pyLODE
  - Pubby
  - LodView
  - Walder
  - Harshp
  - useful publication surfaces, not artifact-history coordinate models
- Repository-native ontology work:
  - Git as excellent substrate
  - ACIMOV / continuous integration
  - collaborative knowledge management using Git
  - missing reviewable RDF coordinates for artifact identity and historical state

## 3. DCAT 3 Comparison

- DCAT 3 baseline:
  - `dcat:Dataset`: broad conceptual data resource
  - includes text, images, sound, video, RDF, and other data forms
  - `dcat:Distribution`: specific representation of a dataset
  - distributions may vary by format, serialization, schema, profile, language, resolution, packaging, fidelity
  - provider decides whether resources are same dataset, different datasets, or dataset series
- Semantic Flow alignment:
  - `DigitalArtifact rdfs:subClassOf dcat:Dataset`
  - `ArtifactManifestation rdfs:subClassOf dcat:Distribution`
  - `hasManifestation rdfs:subPropertyOf dcat:distribution`
  - `locatedFileForManifestation rdfs:subPropertyOf dcat:downloadURL`
- Semantic Flow differences:
  - `DigitalArtifact`: governed artifact identity, not only catalog entry
  - `HistoricalState`: immutable artifact-state facet, absent from DCAT's core pattern
  - `ArtifactManifestation`: DCAT distribution constrained by Semantic Flow's state/artifact stack
  - `LocatedFile`: retrievable bytes as resource, not only URL value
  - `ResourcePage`: concrete static page file, not only landing page
  - `ArtifactResolutionSpec`: coordinate relator for latest, working, exact, fallback, digest, and source
- What Semantic Flow replaces in practice:
  - bare `dcat:distribution` with `hasManifestation` for artifact/state representation links
  - bare `dcat:downloadURL` with `locatedFileForManifestation` when the downloadable thing is a modeled file resource
  - generic `dcat:landingPage` with `hasResourcePage` when the page itself is a modeled static `ResourcePage`
  - `dcat:DatasetSeries` for internal artifact lineage; use `ArtifactHistory` instead
  - ad hoc "latest" links with explicit `defaultArtifactHistory`, `currentArtifactHistory`, `latestHistoricalState`, and resolution modes
- What Semantic Flow does not replace from DCAT:
  - catalog-level aggregation
  - broad discovery metadata
  - external interoperability vocabulary
  - DCAT export or crosswalks
- `dcat:DatasetSeries` contrast:
  - series is itself a dataset
  - collection of separately published datasets
  - outside/around the artifact
  - not the internal publication lineage of one `DigitalArtifact`
- `ArtifactHistory` contrast:
  - inside a `DigitalArtifact`
  - multiple lineages for same artifact
  - typed as `DigitalArtifactFacet` and `SemanticFlowResource`
  - facet of the artifact's diachronic identity
  - not a `DigitalArtifact`
  - not a container superclass
  - drafts
  - releases
  - editorial curation
  - extracted/source histories
  - local working versus published histories
  - default write-target history pointer
  - latest state pointer as mutable metadata, not historical-state content
  - release versus non-release separation lives here

## 4. Core Model

- Full chain:
  - `SemanticMesh`
  - `Knop`
  - `DigitalArtifact`
  - `ArtifactHistory`
  - `HistoricalState`
  - `ArtifactManifestation`
  - `LocatedFile`
  - `ResourcePage`
  - `ArtifactResolutionSpec`
- `SemanticMesh`:
  - governed namespace region
  - static publication surface
  - filesystem-backed in Weave, but framework-level concept
- `Knop`:
  - naming anchor
  - support object for an identifier
  - place for payload, metadata, inventory, reference catalog, source registry, page definition
  - sense-finding support even when the identified resource is not itself a `DigitalArtifact`
- `DigitalArtifact`:
  - governed artifact-level continuity handle
  - "digital artifact" in the sense of byte-realizable governed content
  - identity clarified by its histories, states, manifestations, located files, pages, and metadata
  - not a complete theory of books, works, expressions, performances, or products
- `ArtifactHistory`:
  - lineage resource
  - diachronic facet in prose
  - typed as `DigitalArtifactFacet` and `SemanticFlowResource`
  - draft/release/curation separation
  - release history versus archive/draft/curation history
  - named histories such as `releases`
  - ordinals
  - `defaultArtifactHistory` as default write target
  - `currentArtifactHistory` as compatibility/current-lineage pointer
  - if default and latest/current history differ, write operations require explicit history selection
  - `latestHistoricalState`
  - optional resource page
- `HistoricalState`:
  - immutable publication-state facet
  - revision chain
  - major/minor/release semantics can attach here
  - coordinates for exact citation
- `ArtifactManifestation`:
  - representation of a state or sparse artifact
  - Turtle
  - JSON-LD
  - RDF/XML
  - Markdown
  - PDF
  - canonicalized graph
  - zipped package
  - build output
- `LocatedFile`:
  - retrievable byte identity
  - typically file-like and extension-bearing
  - digest-bearing
  - mirrorable
  - concrete enough for download, cache, citation, and verification
- `ResourcePage`:
  - special `LocatedFile`
  - HTML page
  - conventionally `index.html`
  - presents another resource
  - can become its own governed artifact if page identity, page version, or page history matters
- Support artifacts for sense-finding:
  - `ReferenceCatalog`: curated RDF reference links about the identified resource
  - `KnopSourceRegistry`: source bindings and extraction provenance for grounded identifier support
  - `KnopInventory`: current managed surface for the Knop and its support artifacts
  - `KnopMetadata`: metadata about the Knop and support structure
  - `ResourcePageDefinition`: authored page-composition policy for the identifier page
  - these artifacts do not freeze meaning, but make sense-establishing evidence inspectable and revisable
- `ArtifactResolutionSpec`:
  - target artifact
  - target history
  - target state
  - target manifestation
  - target located file
  - target local path
  - target access URL
  - repository source coordinate
  - expected digest
  - fallback
  - observation
  - resolution mode

## 5. Expected Navigation Grammar

- Common exact path:
  - artifact -> current history -> latest state -> manifestation -> located file
- Common citation path:
  - artifact -> release history -> exact state -> exact manifestation -> exact located file -> digest
- Common update path:
  - artifact -> default/current history -> next/new historical state
  - if default history and latest-updated history diverge, require explicit history selection
- Common sparse path:
  - artifact -> located file
- Common state shortcut:
  - historical state -> located file
- Common working path:
  - artifact -> working located file
  - artifact -> working local relative path
  - artifact -> working access URL
- Coordinate modes:
  - working
  - latest state
  - exact
- Skip-level properties:
  - not alternate metaphysics
  - convenience coordinates
  - should remain consistent with the full chain when both are asserted
- Why grammar matters:
  - reviewable metadata
  - reproducible publication
  - durable citation
  - release/non-release lineage separation
  - static hosting
  - LLM/IDE authoring workflows
  - separation of mutable pointers from immutable states

## 6. URI Sense and Resource Pages

- Problem:
  - dereferencing a public IRI returns bytes
  - the public IRI may identify an ontology term, artifact, concept, page, file, or other resource
  - static hosts often return `index.html` for parent path requests
  - no dynamic host headers
  - no required server-side content negotiation
  - no required 303 redirect machinery
- Semantic Flow stance:
  - URIs have senses in social and graph contexts
  - URIs do not literally refer to senses
  - RDF should state what the identifier denotes or presents when distinction matters
- Static pattern:
  - parent IRI requested
  - commodity host serves `index.html`
  - returned page declares itself a `ResourcePage`
  - identified resource links to that page with `hasResourcePage`
  - embedded RDFa or JSON-LD carries the assertion
- Content/page distinction:
  - `LocatedFile`s identify content directly
  - ordinary located files are terminal byte resources
  - ordinary located files normally do not need resource pages
  - `ResourcePage` is the located file whose role is presentation of another resource
- File extension convention:
  - extension-bearing IRIs usually identify concrete retrievable content
  - extensionless or slash paths often identify higher-level resources
  - convention, not universal law
  - RDF assertions remain authoritative
- ResourcePage as artifact:
  - page can have its own IRI
  - page can be a `DigitalArtifact`
  - page can have historical states
  - page can have manifestations and located files
  - optional, not forced

## 7. Structure, Concept, and Sense-Finding

- Starting distinction:
  - structures and concepts serve different jobs
  - structure can be precise, operational, and application-dependent
  - concepts and identifier senses can evolve socially and editorially
  - retrieval, reasoning, validation, and human interpretation need different kinds of support
- Semantic Flow stance:
  - SFLO is not primarily a domain ontology that says what is true in the world
  - SFLO describes how to capture, publish, support, and revise semantic resources predictably
  - "everything is a concept" in the broad sense that every identifier needs sense-finding support
  - but not everything is a `DigitalArtifact`
- What gets structural treatment:
  - `DigitalArtifact` stack
  - histories
  - states
  - manifestations
  - located files
  - ResourcePages
  - source bindings
  - inventories
- What remains publisher/domain responsibility:
  - conceptual interpretation
  - class membership in a domain ontology
  - SKOS concept schemes
  - social meaning of an identifier
  - semantic compatibility claims
- Claim:
  - Semantic Meshes keep structure and concept from collapsing into each other.
  - Mesh structure provides durable, inspectable evidence for dynamic sense-finding.

## 8. Digital Artifacts, Metadata, and Meta-Metadata

- Primary data:
  - governed artifact-level continuity
  - persistent historical states
  - quasi-immutable located files
  - digests for verification
- Metadata:
  - can evolve
  - can carry latest/current pointers
  - can describe histories, pages, manifestations, located files, source coordinates
  - can itself be a `DigitalArtifact`
- Metadata states:
  - exact metadata state citation
  - changing metadata without rewriting old primary data
  - current metadata view as mutable coordinate
- Meta-metadata:
  - possible by minting an IRI for the metadata artifact
  - no infinite-stack obligation
  - explicit only when useful
- Practical rule:
  - materialize layers when they carry evidence
  - skip layers when they are ceremonial

## 9. FRIR/WEMI Contrast

- FRIR:
  - useful information-resource provenance lens
  - Web resource concerns
  - provenance requirements
  - expression-like distinctions
  - no native `HistoricalState` role as Semantic Flow uses it
- WEMI/OpenWEMI:
  - work/expression/manifestation/item
  - useful for library and cultural object cases
  - physical-library bias
  - intentionally fuzzy at boundaries
- Semantic Flow:
  - not WEMI
  - not an expression model
  - rooted in concrete bytes
  - leaves "Beethoven's 9th", draft, performance, recording, edition, product, or work modeling to domain ontologies
  - models governed byte-realizable artifacts and their publication coordinates
- Possible table:
  - FRIR/WEMI layer
  - DCAT layer
  - Semantic Flow layer
  - what SFLO accepts
  - what SFLO leaves to publisher

## 10. Major Ontology Revisions

- Why major revisions are hard:
  - term meanings change
  - term IRIs persist
  - deprecations need pages
  - replacements need links
  - examples change
  - SHACL profiles change
  - generated documentation changes
  - serialized RDF changes
  - dependent meshes may need migration
- Git helps:
  - diffs
  - branches
  - tags
  - commits
  - collaboration
- Git is not enough:
  - repo-level history, not identifier-level public history
  - hard-to-review publication metadata
  - no RDF-native distinction among artifact/state/manifestation/file/page
  - no mesh-level latest/exact coordinate grammar
- Semantic Flow role:
  - public artifact-state boundary
  - exact release coordinates
  - old states remain citeable
  - new metadata can point to deprecated/replaced/current
  - analysis tools can attach semantic diffs and migration notes to states

## 11. Worked Example: SFLO Metadata Stanza

- Source:
  - `semantic-flow-core-ontology.ttl`
  - opening metadata stanza
- Show as listing:
  - ontology/artifact IRI
  - version IRI
  - release state IRI
  - Turtle manifestation IRI
  - located file IRI
  - content URL
  - resource page link if present or proposed
- Explain coordinates:
  - artifact-level identifier
  - history lineage
  - exact historical state
  - manifestation format
  - located byte resource
  - digest/checksum where available
  - page returned for human exploration
- Possible table:
  - coordinate
  - class
  - example IRI
  - mutability expectation
  - why it exists

## 12. Weave Note

- Keep short in FOMI paper.
- Weave:
  - reference implementation
  - filesystem-native
  - static-hosting-friendly
  - publishes Semantic Meshes
  - generates ResourcePages
  - records source registries
  - validates with SHACL
  - supports IDE/LLM-era authoring
- Detailed demonstration belongs in FOIS demo paper.

## 13. Figures and Tables

- Figure 1:
  - DCAT/SFLO refinement ladder
  - `dcat:Dataset` / `DigitalArtifact`
  - `HistoricalState`
  - `dcat:Distribution` / `ArtifactManifestation`
  - `LocatedFile`
  - `ResourcePage`
- Figure 2:
  - static dereferenceability graph
  - resource IRI
  - served `index.html`
  - `ResourcePage`
  - `hasResourcePage`
  - embedded RDFa/JSON-LD
- Table 1:
  - DCAT 3 term
  - Semantic Flow term
  - relationship
  - added constraint or commitment
- Table 2:
  - FRIR/WEMI versus SFLO
  - expression omitted
  - manifestation comparison
  - temporal state comparison
  - byte grounding comparison
- Listing 1:
  - SFLO opening metadata stanza
- Table 3:
  - SFLO coordinate examples
  - latest
  - exact state
  - exact located file
  - working file
  - metadata state

## 14. Limitations

- Not a universal theory of intellectual works.
- Not WEMI.
- Not a replacement for DCAT cataloging.
- Not a replacement for Git.
- Not an ontology semantic-diff algorithm.
- Not a complete migration framework.
- Static publication cannot express every dynamic server behavior.
- File-extension conventions are mechanical hints, not metaphysical rules.
- Layering can be sparse.
- Reviewers need vocabulary explained before use.

## 15. Conclusion

- DCAT gives broad datasets and distributions.
- Semantic Flow refines that pattern for byte-grounded publication.
- `HistoricalState` and `ArtifactHistory` make artifact histories explicit.
- `LocatedFile` makes retrievable bytes first-class.
- `ResourcePage` makes static dereferenceability graph-visible.
- Coordinates make latest, working, exact, and historical references reviewable.
- Weave demonstrates feasibility, but Semantic Flow is the framework-level contribution.

## Page Budget

- Introduction: 0.75 page
- Prior work spine: 1 page
- DCAT comparison: 1 page
- Core model and navigation grammar: 1.5 pages
- URI sense and ResourcePages: 1 page
- Histories, metadata, major revisions: 1 page
- Structure, concept, and sense-finding: 0.75 page
- SFLO metadata example: 1 page
- Discussion, limitations, conclusion: 0.75 page
- References: 1 page

## Core Reference Pool

- W3C DCAT 3
- Kurt Cagle and Chloe Shannon, "Structure vs. Concept"
- [[ar.functional-requirements-for-information-resource-provenance-on-the-web]]
- [[ar.works-expressions-manifestations-items-an-ontology]]
- Jeni Tennison, "Using 'Punning' to Answer httpRange-14"
- [[ar.w3.design-issues.linked-data]]
- [[ar.w3.cool-uris]]
- [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]]
- [[ar.w3.best-practice-recipes-for-publishing-rdf-vocabularies]]
- [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]]
- [[ar.the-acimov-methodology-agile-and-continuous-integration-for-modular-ontologies-and-vocabularies]]
- [[ar.analysing-multiple-versions-of-an-ontology]]
- [[ar.decentralized-collaborative-knowledge-management-using-git]]
