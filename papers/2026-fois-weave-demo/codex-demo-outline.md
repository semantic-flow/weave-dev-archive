# Shared Outline: DigitalArtifact Model and Weave Demo

Status: this is mixed scratch material from before the two-paper split. The canonical FOMI model outline is `../2026-fomi-digitalartifact/outline.md`. The canonical FOIS Demonstrations planning file is `paper-planning-codex.md`, with the concrete demo flow in `demo-script.md`.

Working title:

> Semantic Flow: Byte-Grounded Digital Artifact Histories for Static RDF Publication

Alternative title:

> Weave: Filesystem-Native Semantic Mesh Publication for RDF Artifacts

## Position

This is not only an ontology-publication paper. The demo uses ontology publication because FOIS reviewers care about ontologies and because SFLO is the live example, but the problem is broader: RDF resources often live as files, generated documentation, release artifacts, source registries, examples, SHACL shapes, and term pages. The hard part is keeping the public RDF surface clear about identifiers, bytes, history, provenance, and publication topology.

This is also not a WEMI paper. WEMI is useful as a comparison because it exposes the ambiguity of "the thing", "a version", "a manifestation", and "a copy". Semantic Flow deliberately avoids deciding whether an RDF identifier denotes a work, an expression, a performance, a recording, a conceptual entity, or some other domain object. That modeling belongs to the publisher's domain ontology.

Semantic Flow addresses a narrower operational layer: governed digital artifacts and their concrete bytes. It asks:

- Which public identifier is governed by the mesh?
- Which digital artifact or support artifact is associated with it?
- Which current or historical state is being discussed?
- Which manifestation or located file provides concrete bytes?
- Where did those bytes come from, and what provenance/evidence can be reviewed as RDF?
- Which generated ResourcePage makes this inspectable to humans?

## Thesis

RDF publication needs a byte-grounded digital artifact layer between conceptual RDF identifiers and repository/static-site mechanics. Git can store files and history, and Linked Data conventions can tell us that identifiers should be dereferenceable, but neither alone gives RDF-reviewable artifact history, provenance, identifier-scoped state, or generated pages that explain the relation between abstract identifiers and concrete bytes. Semantic Flow supplies that layer; Weave demonstrates it with a filesystem-native static publication workflow.

## Contribution Claims

- A byte-grounded `DigitalArtifact` framing for RDF publication that separates governed artifact identity, working bytes, historical states, manifestations, located files, and source/provenance records.
- A concrete answer to "why not just Git": Git remains useful substrate, but Semantic Flow provides RDF-visible, identifier-scoped, artifact-level history and provenance.
- A static RDF publication workflow that produces dereferenceable ResourcePages while preserving inspectable metadata about source bytes, histories, and generated support machinery.
- A demonstration over SFLO showing ontology files, SHACL/config artifacts, extracted terms, artifact histories, source registries, and generated ResourcePages.

## Paper Structure

### 1. Introduction

Opening problem:

RDF projects commonly begin as files in repositories and end as public identifiers on the Web. Between those two points, maintainers must manage concrete bytes, source locations, release snapshots, generated documentation, extracted terms, examples, shapes, and provenance. The Web-facing identifier is often treated as if it were interchangeable with a file, a page, a repository path, or a conceptual entity. In practice those are different concerns.

Key move:

The paper should say "RDF publication" first, then say ontology publication is the demo case.

Possible paragraph:

> Weave is a filesystem-oriented reference implementation of Semantic Flow, a model for publishing RDF resources as a SemanticMesh of governed identifiers, digital artifacts, histories, concrete byte manifestations, and generated ResourcePages. This demonstration focuses on ontology publication, but the artifact problem is broader than ontologies: any RDF project that publishes stable identifiers from repository-managed files faces the same tension between conceptual identifiers and concrete bytes.

### 2. Problem: RDF Identifiers, Files, and Publication State Get Conflated

Cover:

- RDF identifiers may denote domain things, ontology terms, datasets, documents, versions, pages, files, or support machinery.
- Cool URI and Linked Data guidance help, but they do not fully specify static-site behavior, artifact histories, source provenance, or repository topology.
- File extensions are useful publication convention: file-like paths tend to identify concrete retrievable bytes; slash/extensionless paths tend to identify discourse-worthy resources. But this is a convention that should be backed by explicit metadata, not an ontology-level law.
- Static publication needs a way to explain both the human page and the byte-bearing artifact without requiring a live server or hidden operational state.

Existing publication tools to mention briefly:

- [[ar.base-platform-for-knowledge-graphs-with-free-software]] is a useful CEUR tool-landscape citation: it describes a free/open-source KG platform assembled from specialized tools for creating, maintaining, serving, viewing, and exploring a modular large-scale KG, and it explicitly mentions pain points in composing a full platform from FOSS pieces.
- [[prdct.pubby]] and [[prdct.lodview]] provide Linked Data presentation/dereferencing layers over SPARQL-capable data, with LodView explicitly inspired by Pubby.
- [[prdct.lode]], [[prdct.pylode]], and [[prdct.widoco]] generate human-readable ontology documentation; Widoco builds on LODE and appears in SEMIC-style tooling recommendations.
- [[ar.harshp.rdf-website-generator]] is a lightweight RDF-driven website generation experiment using RDF metadata and SPARQL to generate static pages.
- [[prdct.walder]] publishes data from knowledge graphs using templates and content negotiation over HTML, RDF, and JSON-LD.
- [[prdct.jekyll-rdf]] and [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]] are the closest citation for static, template-based Linked Data publication and customizable resource pages.

Positioning line:

> These tools establish that RDF and ontology resources can be made human-readable and Web-publishable. Semantic Flow addresses a complementary layer: RDF-visible histories, manifestations, located files, source bindings, and provenance for the concrete digital artifacts behind those pages.

Authoring/exploration shift to mention carefully:

- GUI ontology editors such as [[prdct.protege]] remain useful for inspection, exploration, teaching, and reasoner-backed checks.
- But source-centered authoring is increasingly plausible: RDF/OWL/Turtle files can be edited in an IDE, reviewed in Git, and drafted or refactored with LLM assistance.
- Semantic Flow assumes this source-file workflow rather than forcing authoring through a GUI. Weave then provides generated ResourcePages for exploration, with artifact-history and state/manifestation navigation that ordinary ontology explorers and documentation generators do not usually make central.
- Strong version of the claim for discussion, not abstract: Protégé-like GUI exploration is still valuable, but IDE-native authoring plus generated temporal/artifact browsing changes what the primary authoring surface needs to be.

References likely:

- [[ar.base-platform-for-knowledge-graphs-with-free-software]]
- [[ar.w3.design-issues.linked-data]]
- [[ar.w3.cool-uris]]
- [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]]
- [[ar.w3.best-practice-recipes-for-publishing-rdf-vocabularies]]
- [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]]

### 3. FRIR and Semantic Flow: Touchpoint and Contrast

Purpose:

Use FRIR/WEMI to show that the ambiguity is real, then draw a boundary around what Semantic Flow does not attempt to solve. This should be a full section, not merely a citation aside, because it explains why the DigitalArtifact model exists and what it refuses to model.

Points:

- WEMI tries to distinguish layers such as work, expression, manifestation, and item for cultural/intellectual artifacts.
- That is useful but fuzzy for RDF publication: the "work" might be an ontology, an ontology release, a graph, a graph-canonicalized content state, a Turtle serialization, a generated HTML page, or a domain referent.
- Semantic Flow does not decide those domain-level distinctions.
- Semantic Flow deliberately leaves `frbr:Expression`-like modeling out of the core. A `DigitalArtifact` may be an expression of something more abstract in a publisher's domain model, but Semantic Flow does not require or infer that.
- Different formats, serializations, canonicalizations, or package/build choices are represented as `ArtifactManifestation`s. Whether those manifestations express the same abstract content is outside the core artifact layer.
- FRIR is a key touchpoint for information-resource provenance, but it does not supply Semantic Flow's temporal `HistoricalState`: an explicit published artifact state between the enduring DigitalArtifact and its concrete manifestations/located files.
- Semantic Flow is rooted in concrete digital artifacts and bytes: working bytes, historical states, manifestations, located files, digests, source locators, and observations.
- Publishers can still use WEMI, FRBR, DCAT, IAO, PROV, PAV, or a domain ontology above or beside Semantic Flow when they need richer intellectual/content modeling.

Important wording:

> Semantic Flow is WEMI-aware but not WEMI-shaped.

Better if we want to be sharper:

> Semantic Flow treats WEMI as a warning label: useful distinctions exist, but the publication tool should not force one answer about what the published "work" is.

Figure/table idea: two-column FRIR vs Semantic Flow comparison.

| Question | FRIR / WEMI | Semantic Flow |
|---|---|---|
| Primary concern | Provenance for Web information resources across resource/content/bytes/copies | Publication and resolution of governed digital artifacts and their concrete bytes |
| Abstract work/content modeling | Includes work/resource and expression/content distinctions | Leaves work/expression/domain interpretation to the publisher's RDF model |
| Expression layer | Explicitly models expression-like content independent of serialization | Intentionally absent from the core artifact layer |
| Concrete byte variant | Manifestation captures a specific bit pattern | `ArtifactManifestation` captures a concrete format/package/build/canonicalization variant |
| Located copy | Item represents a specific copy/location/transaction | `LocatedFile` identifies retrievable bytes, usually file-like and often extension-bearing |
| Temporal publication state | Not a central layer in the model | `HistoricalState` is explicit: an immutable published state in an `ArtifactHistory` |
| Layer materialization | Often read as a conceptual stack to be interpreted for each domain object | Layers are optional: skip-level properties allow sparse publication when state, manifestation, or file nodes are not useful |
| Source coordinates | Can support provenance, but does not center repository/ref/path resolution | `ArtifactResolutionSpec` and repository locators make source, target, state, manifestation, file, digest, and working coordinates reviewable |
| Tooling role | Conceptual/provenance model for information resources | Operational publication framework with a filesystem-native implementation in Weave |
| Boundary | May invite modeling the work/expression nature of the thing | Refuses that modeling commitment; focuses on concrete digital artifact evidence |

Caption idea:

> FRIR gives Semantic Flow its nearest prior-art touchpoint for distinguishing information-resource layers, but Semantic Flow moves the center of gravity toward byte-grounded artifact publication. It omits expression-level commitments and adds explicit temporal states and coordinate-bearing resolution specs.

References likely:

- [[ar.functional-requirements-for-information-resource-provenance-on-the-web]]
- [[ar.works-expressions-manifestations-items-an-ontology]]

### 4. The Semantic Flow DigitalArtifact Model

Explain every local term as it appears.

Core terms:

- `SemanticMesh`: a governed namespace and filesystem/static publication surface.
- `Knop`: the support object and naming anchor for a public identifier.
- `DigitalArtifact`: a mesh-governed digital artifact, not merely a repository file path.
- `ArtifactHistory`: an explicit lineage for a DigitalArtifact.
- `HistoricalState`: an immutable published snapshot in an ArtifactHistory.
- `ArtifactManifestation`: a concrete variant such as Turtle, JSON-LD, Markdown, PDF, zipped/unzipped, or a canonicalization/build choice.
- `LocatedFile`: a retrievable bytes identity, typically file-like and often extension-bearing.
- `ArtifactResolutionSpec`: a relator describing requested byte coordinates, target artifact/history/state/manifestation/file, expected digest, fallback, and observations.
- `ResourcePage`: a generated HTML page that makes an identifier and its support metadata inspectable.

Stress:

- This model is operational and byte-grounded.
- It does not claim that `DigitalArtifact` is the same as a domain object, a WEMI Work, or an ontology term.
- It does not claim that `DigitalArtifact` is a `frbr:Expression`; expression-level modeling is left to the publisher's ontology.
- `ArtifactManifestation` covers concrete format/package/build variants, while `HistoricalState` captures temporal publication state.
- Histories are artifact publication histories, not necessarily semantic/version-theory histories of the domain truth.

Sparse/skip-level modeling:

Semantic Flow should not require every mesh to materialize every artifact layer. The full explanatory chain is useful:

`Knop -> DigitalArtifact -> ArtifactHistory -> HistoricalState -> ArtifactManifestation -> LocatedFile`

But the ontology also includes skip-level properties for cases where extra nodes would not add evidence:

- a `DigitalArtifact` may link directly to an `ArtifactManifestation` when the publisher wants a concrete variant without naming a separate state;
- a `HistoricalState` may use `locatedFileForState` as a shortcut to retrievable bytes when a separate manifestation resource is unnecessary;
- a `DigitalArtifact` may use `locatedFileForArtifact` when the mesh needs a sparse file shortcut without materializing state or manifestation resources;
- a `DigitalArtifact` may use `hasWorkingLocatedFile`, `workingLocalRelativePath`, or `workingAccessUrl` for current working bytes that are useful to tooling but are not immutable historical states;
- an `ArtifactResolutionSpec` may target the most specific coordinate available, from `targetArtifact` through `targetArtifactHistory`, `targetHistoricalState`, `targetManifestation`, and `targetLocatedFile`.

This is an important contrast with WEMI-shaped modeling. Semantic Flow offers progressive disclosure of artifact structure: publishers can expose the full chain when it carries publication evidence, and use direct coordinate links when the intermediate layers would be ceremonial. When both shortcut and fuller links are present, the shortcut should be consistent with the fuller path it abbreviates.

Figure idea:

`Knop -> DigitalArtifact -> ArtifactHistory -> HistoricalState -> ArtifactManifestation -> LocatedFile`

With side links:

- `ArtifactResolutionSpec` points into the chain
- `ResourcePage` presents the resource plus generated support metadata
- source registry records provenance/evidence

Running example: opening metadata stanza from `semantic-flow-core-ontology.ttl`.

Use this stanza as Listing 1, or as a figure/table, because it demonstrates the model on the ontology itself. This is abbreviated from the source and uses an `sflo:` prefix for paper readability:

```turtle
@prefix sflo: <https://semantic-flow.github.io/sflo/ontology/> .

<https://semantic-flow.github.io/sflo/ontology> a owl:Ontology ;
  dcterms:hasVersion <https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0> ;
  owl:versionIRI <https://raw.githubusercontent.com/semantic-flow/sflo/refs/tags/v0.2.0/semantic-flow-core-ontology.ttl> ;
  owl:versionInfo "0.2.0" .

<https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0>
  a owl:Ontology, sflo:HistoricalState ;
  dcterms:isVersionOf <https://semantic-flow.github.io/sflo/ontology> ;
  schema:contentUrl <https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl> ;
  sflo:hasManifestation <https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl> .

<https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl>
  dcat:downloadURL <https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl> .
```

What to explain:

- `<https://semantic-flow.github.io/sflo/ontology>` is the public ontology identifier and the stable artifact-level resource.
- `<https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0>` is a named historical state of that artifact, not merely a Git tag and not merely a file.
- `<https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl>` is a Turtle manifestation of that historical state.
- `<https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl>` is the located file: concrete retrievable bytes.
- The `owl:versionIRI` points to a GitHub raw tagged file, which is useful source/version metadata, but the Semantic Flow publication graph still exposes the mesh-native state, manifestation, and located-file coordinates.

### 4a. Coordinates As First-Class Publication Evidence

Coordinates should be explained in depth because they are where Semantic Flow differs from "just make a page" publication tools.

Definition:

> A coordinate is a reviewable description of where bytes are requested, published, or observed. Coordinates may identify a public mesh resource, a historical state, a manifestation, a located file, a repository/ref/path tuple, a digest, or a mutable working source under explicit policy.

Coordinate families to explain:

- Identifier coordinate: the public Semantic Flow IRI, e.g. `https://semantic-flow.github.io/sflo/ontology`.
- State coordinate: the release-state IRI, e.g. `/ontology/releases/v0.2.0`.
- Manifestation coordinate: the packaged variant, e.g. `/ontology/releases/v0.2.0/ttl`.
- Located-file coordinate: the retrievable byte endpoint, e.g. `/ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl`.
- Repository coordinate: repository URL plus ref/commit plus repository-relative path; this avoids persisting host-local checkout paths.
- Digest coordinate/evidence: content digest used to prove or check byte identity.
- Working coordinate: mutable source coordinates that require explicit resolution mode and operational policy; they are useful but not immutable identity by themselves.

Important distinction:

- A coordinate is not automatically permission to fetch or trust bytes.
- A coordinate is not always exact. A `LocatedFile`, `HistoricalState`, commit, manifestation, or digest can be exact; a working source, branch ref, or latest-state request is intentionally policy-mediated unless paired with commit/digest/state evidence.
- Coordinates are the connective tissue between Git, static hosting, ontology metadata, and ResourcePages.

Possible table:

| Coordinate kind | Example | What it fixes | What remains policy-dependent |
|---|---|---|---|
| Artifact IRI | `/ontology` | public artifact identity | current/latest state choice |
| Historical state IRI | `/ontology/releases/v0.2.0` | published state | manifestation choice |
| Manifestation IRI | `/ontology/releases/v0.2.0/ttl` | packaged variant | exact located copy if several exist |
| Located file IRI | `/ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl` | retrievable bytes endpoint | network/cache/trust behavior |
| Repository locator | repo URL + tag/commit + path | source coordinates | local checkout mapping/fetch policy |
| Digest | `sha256:...` | byte identity check | how bytes are obtained |

### 5. Why Not Just Git?

Tone:

Git is useful and Weave should happily use it. The point is that Git is repo-level infrastructure, not an RDF publication model.

Git gives:

- commits
- branches
- diffs
- repository-level history
- file contents

Git does not natively give:

- public RDF metadata about artifact state
- identifier-scoped history independent of repository topology
- explicit source/provenance records on the generated public site
- reviewable links between ontology terms, source artifacts, pages, histories, and byte manifestations
- a distinction between mutable working bytes, published historical states, and exact located files
- static ResourcePages that explain the publication state to humans and machines

Claim:

> Git is a storage and collaboration substrate. Semantic Flow is a semantic publication layer over governed artifacts and bytes.

Major ontology revisions:

This point matters most during major version revisions. A breaking ontology release is not merely a large Git diff: it can change term meanings, import closure, generated documentation, validation shapes, examples, dependent meshes, and the persistence obligations of previously published IRIs. Maintainers may need to keep deprecated terms dereferenceable, explain replacements, publish migration guidance, and distinguish "this file changed" from "this public artifact has a new historical state with different downstream consequences." Semantic Flow does not solve semantic ontology migration or diffing by itself, but it gives the revision a reviewable publication surface: states, manifestations, located files, source coordinates, and ResourcePages can make the major-version boundary inspectable.

References likely:

- [[ar.decentralized-collaborative-knowledge-management-using-git]]
- [[ar.distributed-collaboration-on-rdf-datasets-using-git]] if needed
- [[ar.analysing-multiple-versions-of-an-ontology]]
- [[ar.the-acimov-methodology-agile-and-continuous-integration-for-modular-ontologies-and-vocabularies]]

### 6. Weave Demonstration: Static RDF Publication

Use SFLO as primary demo.

Workflow:

1. Create a mesh with a public base IRI.
2. Integrate source RDF artifacts from ordinary repository locations.
3. Assign/reuse artifact histories and named historical states.
4. Weave current bytes into historical states and manifestations.
5. Extract ontology terms into first-class Knop-managed identifiers.
6. Generate ResourcePages.
7. Validate the publication surface, including absence of host-local path leakage.

Show:

- SFLO root page
- ontology artifact page
- term page such as `SemanticMesh`
- source registry
- branch vs sidecar topology examples if there is room

References likely:

- [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]]
- [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]]

### 7. Discussion and Limitations

State plainly:

- Weave is pre-1.0.
- The demo is a CLI/runtime and static page generator, not a mature daemon/browser app.
- Semantic Flow does not solve all WEMI-like intellectual-artifact modeling.
- Semantic Flow histories are digital artifact histories, not necessarily ontology semantic-diff histories.
- Remote fetching and trust policy are intentionally constrained.
- Static sites cannot perform full server-side content negotiation; Weave compensates with explicit pages, links, and concrete byte resources.

Useful line:

> The strength of the model is partly in what it refuses to decide: it keeps byte-level publication evidence explicit while leaving domain-level interpretation to the RDF vocabularies chosen by the publisher.

### 8. Conclusion

Close on:

- RDF publication needs inspectable relationships between public identifiers and concrete bytes.
- Semantic Flow provides a byte-grounded artifact layer.
- Weave demonstrates it on static ontology publication, but the pattern applies to RDF publication more generally.

## Approximate Page Budget

- Introduction: 0.75 page
- Problem/background: 1 page
- FRIR contrast and DigitalArtifact model: 1.75 pages
- Why not Git: 0.75 page
- Weave implementation/demo: 2 pages
- Discussion/limitations/conclusion: 1 page
- References: 1 page

Total target: 8-9 CEUR pages including references.

## Ten-Reference Core

Use roughly ten, probably:

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

Reserve:

- [[ar.base-platform-for-knowledge-graphs-with-free-software]] if we need a CEUR citation for the existing free/open-source KG tooling landscape.
- [[ar.defining-n-ary-relations-on-the-semantic-web]] if `ArtifactResolutionSpec` needs a relator/n-ary citation.
- [[ar.modular-ontology-modeling]] if the paper leans more into applied ontology methodology.
- [[ar.semiceu.semic-s-semantic-modelling-reference-architecture-and-tool-recommendations]] if the tool architecture angle needs practice support.

## Phrases To Avoid

- "Semantic Flow implements WEMI."
- "WEMI grounds the DigitalArtifact model."
- "Ontology publication breaks down..."
- "Git is insufficient" without immediately saying what Git is good for.
- "File extension means..." as a universal ontology rule.
- "Protégé is obsolete" or "Protégé is only an explorer" as a flat claim.

## Phrases To Prefer

- "RDF publication"
- "WEMI-aware but not WEMI-shaped"
- "byte-grounded digital artifact layer"
- "Git as substrate, Semantic Flow as publication layer"
- "file-like paths conventionally identify concrete retrievable bytes"
- "domain-level interpretation remains the publisher's modeling responsibility"
- "FRIR as touchpoint and contrast, not foundation"
- "HistoricalState supplies the temporal artifact-state layer absent from FRIR/WEMI"
- "full artifact chain when it carries evidence, sparse coordinate link when it does not"
- "consistency-preserving skip-level properties"
- "GUI ontology editors remain useful for exploration; IDE-native authoring is increasingly viable"
- "generated ResourcePages provide temporal/artifact-state browsing over published RDF resources"
