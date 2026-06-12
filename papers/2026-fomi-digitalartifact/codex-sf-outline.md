# Outline: FOMI 2026 DigitalArtifact Paper

Working title:

> Semantic Flow: Byte-Grounded Digital Artifact Histories for Static RDF Publication

## Boundary

This is the model/practice paper, not the Weave demo paper.

The FOMI paper argues that ontology and RDF publication need a byte-grounded DigitalArtifact layer. It should explain the model, its relation to prior work, and the practical ontology-engineering problems it helps make reviewable. Weave appears as the reference implementation and running case.

The FOIS Demonstrations paper argues that Weave demonstrates a concrete filesystem-native workflow for publishing ontology resources as static, inspectable SemanticMeshes. It should include command flow, ResourcePage screenshots, source registries, validation, and implementation limitations.

## Thesis

Repository-native RDF publication needs an explicit model of governed digital artifacts, publication states, concrete manifestations, located files, and source coordinates. Git provides excellent storage and collaboration mechanics, and Linked Data guidance explains dereferenceable identifiers, but neither makes ontology artifact histories, byte manifestations, major revision boundaries, and provenance directly reviewable as RDF. Semantic Flow supplies that publication layer while leaving domain-level work/expression modeling to publishers.

## Contribution Claims

- A byte-grounded `DigitalArtifact` model for RDF and ontology publication.
- A practical distinction between artifact identity, publication history, historical state, manifestation, located file, and source coordinate.
- A WEMI-aware but not WEMI-shaped treatment of digital files: Semantic Flow leaves work/expression/domain interpretation to publisher ontologies.
- A sparse modeling strategy using skip-level properties when the full artifact/state/manifestation/file chain would not add evidence.
- A mechanical ResourcePage pattern for static dereferenceability: commodity hosting may return `index.html`, while RDF distinguishes the identified resource from the page file returned to present it.
- A treatment of major ontology revisions as public contract changes, not merely repository diffs.
- A reference implementation case: Weave demonstrates the model over the SFLO static publication surface.

## Section Plan

### 1. Introduction

Open with the publication problem:

Ontology projects publish public identifiers, but their evidence lives in repository files, generated pages, release artifacts, source registries, examples, SHACL shapes, and static hosting paths. Major revisions complicate this further: term meanings, imports, examples, generated documentation, validation shapes, and deprecated identifiers may all change at once.

Position Semantic Flow:

Semantic Flow models the publication layer between conceptual RDF identifiers and concrete repository/static-site mechanics. It is not an ontology of all intellectual works, and it is not a replacement for Git. It is a byte-grounded artifact layer for governed RDF publication.

### 2. Prior Work and Pressure Points

Use a tight reference spine:

- Linked Data and Cool URIs for dereferenceability and information-resource pressure.
- FAIR vocabulary/ontology publication guidance for metadata, version IRIs, documentation, and serializations.
- FRIR/WEMI/OpenWEMI for information-resource layers and provenance.
- Ontology versioning/change-analysis literature for semantic vs syntactic changes.
- Git-backed RDF collaboration and ACIMOV for repository-native workflows.
- Jekyll RDF and existing publication tools as static publication context.
- Cool URI/httpRange-14 literature, including Tennison's punning proposal, for the distinction between an identified resource, the content returned by dereferencing, and the socially governed sense a URI has in context.

Keep the point clear: Semantic Flow integrates known concerns into a byte-grounded publication model; it does not claim to invent dereferenceability, provenance, versioning, or static RDF publication.

### 3. FRIR/WEMI As Touchpoint and Contrast

Core stance:

> Semantic Flow is WEMI-aware but not WEMI-shaped.

Explain:

- WEMI distinctions expose real ambiguity around work, expression, manifestation, and item.
- FRIR is the closest touchpoint for information-resource provenance.
- Semantic Flow deliberately omits an expression layer from the core model.
- A `DigitalArtifact` may be modeled by a publisher as an expression of something more abstract, but Semantic Flow does not require that.
- Different formats, serializations, packages, or build outputs are `ArtifactManifestation`s.
- `HistoricalState` is the temporal publication-state layer absent from FRIR/WEMI.

Use the two-column comparison table from the shared outline, including the skip-level row.

### 4. DigitalArtifact Model

Define terms on first use:

- `SemanticMesh`: governed namespace and publication surface.
- `Knop`: support object and naming anchor for a public identifier.
- `DigitalArtifact`: mesh-governed digital artifact, not merely a repository path.
- `ArtifactHistory`: lineage of a DigitalArtifact.
- `HistoricalState`: immutable published snapshot in an ArtifactHistory.
- `ArtifactManifestation`: concrete format/package/build/canonicalization variant.
- `LocatedFile`: retrievable byte identity, usually file-like and often extension-bearing.
- `ArtifactResolutionSpec`: relation resource describing source/target coordinates, mode, fallback, digest, and observations.
- `ResourcePage`: generated page that makes identifier support metadata inspectable.

Full explanatory chain:

`Knop -> DigitalArtifact -> ArtifactHistory -> HistoricalState -> ArtifactManifestation -> LocatedFile`

Sparse/skip-level point:

Semantic Flow can use the full chain when it carries evidence, and direct coordinate links when intermediate layers would be ceremonial. Mention `hasManifestation`, `locatedFileForState`, `locatedFileForArtifact`, `hasWorkingLocatedFile`, working coordinates, and resolution targets such as `targetLocatedFile`.

Dereferenceability and ResourcePages:

Semantic Flow also needs a mechanical distinction between a resource and the page returned to present it. Static hosting commonly returns `index.html` when a parent IRI is requested; Semantic Flow should be explicit about that convention instead of hiding it. The identified resource can be linked to the concrete page file with `hasResourcePage`, and the returned `index.html` can declare itself a `ResourcePage`. In the paper's terminology, URIs have socially governed senses in context; the URI does not literally refer to a sense as if the sense were the target object.

The DigitalArtifact stack is content-oriented at each level: `DigitalArtifact`, `HistoricalState`, `ArtifactManifestation`, and `LocatedFile` identify content with increasing specificity and concreteness. Ordinary `LocatedFile`s are terminal byte resources and normally do not need their own ResourcePages. `ResourcePage` is the special `LocatedFile` whose role is to present another resource; if a page itself needs to be cited, versioned, or discussed as an artifact, it can also be given its own governed artifact identity.

### 5. Worked Example: SFLO Metadata Stanza

Use the opening metadata stanza from `semantic-flow-core-ontology.ttl`.

Explain:

- the ontology IRI as the stable artifact-level resource;
- `/ontology/releases/v0.2.0` as a named `HistoricalState`;
- `/ontology/releases/v0.2.0/ttl` as a Turtle manifestation;
- `/ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl` as the located file;
- `owl:versionIRI` as useful GitHub raw/tag metadata, not a replacement for mesh-native artifact coordinates.

This can be Listing 1 plus a small coordinate table.

### 6. Major Ontology Revisions

Treat major revisions as the clearest practice problem:

- A breaking release can change term meanings, generated documentation, validation shapes, examples, dependent meshes, and persistence obligations for prior IRIs.
- Maintainers may need deprecation pages, replacement links, migration guidance, and exact bytes for old and new states.
- Git can show diffs, but Semantic Flow can make the public artifact-state boundary reviewable as RDF.
- Semantic Flow does not solve semantic ontology migration or diffing; it provides the publication coordinates where those analyses can attach.

### 7. Weave As Reference Implementation

Keep this short:

- Weave implements Semantic Flow against ordinary files and static hosting.
- The SFLO public mesh demonstrates ontology resources, release states, manifestations, located files, source registries, extracted terms, and generated ResourcePages.
- The detailed command workflow belongs in the FOIS Demonstrations paper.

### 8. Discussion and Limitations

State plainly:

- Semantic Flow is not WEMI and does not decide what a domain-level work/expression is.
- `DigitalArtifact` is a broad, upper-level Semantic Flow category for governed digital artifacts, but it is not a complete theory of every domain-specific digital object distinction.
- Histories are artifact publication histories, not full semantic-diff histories.
- Static publication cannot replace every server-side content negotiation pattern.
- Weave is pre-1.0 and should be described as a reference implementation.

### 9. Conclusion

Close on:

- ontology publication needs explicit relations between public identifiers and concrete bytes;
- major revisions need reviewable artifact-state boundaries;
- Semantic Flow supplies a byte-grounded model for those publication concerns;
- Weave shows the model can be realized in a filesystem-native static publication workflow.

## Approximate Page Budget

- Introduction: 0.75 page
- Prior work and pressure points: 1 page
- FRIR/WEMI contrast: 1 page
- DigitalArtifact model: 1.5 pages
- SFLO worked example and coordinates: 1 page
- Major revisions: 0.75 page
- Weave implementation note: 0.75 page
- Discussion/conclusion: 0.75 page
- References: 1 page

Total target: 8-9 CEUR pages including references.

## Core References

Use the shared reference pool in `../2026-fois-weave-demo/potential-references.md`.

Likely final ten:

1. [[ar.functional-requirements-for-information-resource-provenance-on-the-web]]
2. [[ar.works-expressions-manifestations-items-an-ontology]]
3. [[ar.w3.design-issues.linked-data]]
4. [[ar.w3.cool-uris]]
5. [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]]
6. [[ar.w3.best-practice-recipes-for-publishing-rdf-vocabularies]]
7. [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]]
8. [[ar.the-acimov-methodology-agile-and-continuous-integration-for-modular-ontologies-and-vocabularies]]
9. [[ar.analysing-multiple-versions-of-an-ontology]]
10. [[ar.decentralized-collaborative-knowledge-management-using-git]]
