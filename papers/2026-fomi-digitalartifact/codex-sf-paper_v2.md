# Semantic Meshes: Stable Identifiers, Pseudo-Immutable Evidence, and Dynamic Sense-Finding

Draft target: FOMI 2026 short paper, CEUR-ART one-column mode.

## Abstract

The Semantic Flow framework seeks to make the Semantic Web more usable and FAIR by addressing long-standing problems in identifier persistence, dereferenceability, artifact versioning, and static publication. This paper introduces the Semantic Mesh: a hierarchical IRI surface with RDF-described support structures and publication conventions. Semantic Meshes support easy IRI minting; can carry arbitrary digital artifacts, with special support for artifact lineages and RDF documents; and provide straightforward dereferenceability through ResourcePages on static hosts that support directory index resolution. At the center of the model is a faceted `DigitalArtifact` stack that reuses DCAT 3 versioning and distribution vocabulary while adding named internal lineages, settled historical states, representation-level manifestations, explicit located files, and resolution coordinates. The model also distinguishes primary identifiers from support identifiers so that ontology terms, datasets, generated pages, files, and support artifacts are not collapsed into one kind of resource. We illustrate the model with the Fantasy Rules ontology fixture and introduce Weave, a command-line implementation for creating and updating Semantic Meshes and generating ResourcePages for mesh-managed IRIs.

**Keywords:** Semantic Web, Linked Data, FAIR, persistent identifiers, digital artifacts, DCAT, ontology publication, static sites, dereferenceability

## 1. Introduction

Public Semantic Web identifiers are promises with long tails. An ontology term IRI may appear in external datasets, validation rules, examples, documentation, papers, cached pages, and software long after its publisher has moved on. The IRI should continue to resolve, and a reader should be able to determine what it is being used to mean. This is not merely a network problem. It is a publication problem: the meaning-supporting evidence around an identifier is scattered across RDF source files, generated documentation, version-control history, catalog metadata, and server behavior.

The Semantic Web has strong traditions for this problem. Cool URIs emphasizes stable identifiers and the need to be unambiguous about whether an identifier denotes a document or another resource [CoolURIs]. Web Architecture warns against URI collisions, where one URI is used to identify more than one resource [WebArch]. DCAT 3 provides a broad vocabulary for datasets, distributions, resource versions, and dataset series [DCAT3]. FAIR vocabulary guidance and publication recipes describe expectations for findable, accessible, interoperable, and reusable vocabularies [FAIRVocab] [Recipes]. Yet a small ontology publisher still faces a practical gap: how should a set of public IRIs be organized so that it can be hosted cheaply as static files, moved between hosts, reviewed in Git, versioned at useful levels of specificity, and inspected by both humans and RDF tools?

Semantic Flow answers this with two linked ideas. The first is the `DigitalArtifact` model: a faceted model for digital objects whose identity is usefully grounded in bytes across time and representation. The second is the `SemanticMesh`: a statically hostable identifier surface plus the support artifacts that make its identifiers dereferenceable, citable, and interpretable. Weave is the filesystem-native command-line implementation.

This paper makes four contributions. First, it presents the faceted `DigitalArtifact` model and its alignment with DCAT 3. Second, it distinguishes primary identifiers from support identifiers, making explicit the difference between the resource a publisher primarily wants to identify and the pages, histories, states, files, catalogs, and metadata used to support that identifier. Third, it describes `ResourcePage`s as a static-host-friendly answer to dereferenceability: when directory index resolution returns `index.html`, embedded RDF can identify the page as a page about the requested resource rather than the resource itself. Fourth, it introduces Semantic Meshes and Weave through the Fantasy Rules ontology fixture, a small ontology publication surface with ontology terms, examples, SHACL, generated pages, and artifact coordinates.

We use **pseudo-immutable** for settled publication artifacts whose underlying storage remains technically mutable, but whose published coordinates are governed by norms, tooling, and access controls that discourage or restrict mutation. The claim is not that bytes cannot be changed. The claim is that settled identifiers should not be reused for different content, and that the publication model should make such reuse visible when it happens.

## 2. The Faceted DigitalArtifact Model

The most concrete Semantic Flow contribution is a model for digital artifacts. A `DigitalArtifact` is a resource with byte-grounded artifact identity across time and representation. Examples include an ontology document, a SHACL shapes file, a generated HTML page, a Markdown source file, a PDF, a bundle, or an RDF dataset. A `DigitalArtifact` is not a theory of works, expressions, or concepts. It is a way to represent a digital object as a continuing artifact with histories, settled states, representations, retrievable files, and resolution coordinates.

The main chain is:

```text
DigitalArtifact -> ArtifactHistory -> HistoricalState -> ArtifactManifestation -> LocatedFile
```

`DigitalArtifact` is the continuing artifact-level resource. For an ontology publication, it may identify the ontology document considered across releases and serializations. `ArtifactHistory` is a named lineage inside that artifact: release history, draft history, archived predecessor history, source-import history, or editorial curation history. `HistoricalState` is a settled state within one such history. It is the natural target for exact citation: the Fantasy Rules ontology as released in a given state, the generated page as published at a given point, or the reference catalog before a source migration. `ArtifactManifestation` is a representation or intended byte-pattern identity for an artifact or state: Turtle, JSON-LD, HTML, Markdown, PDF, a zipped bundle, or a canonical RDF serialization. `LocatedFile` is a retrievable location for bytes: a mesh path, static URL, repository-backed raw file, or other file-like access point.

The distinction between `ArtifactManifestation` and `LocatedFile` is important. A manifestation identifies the representation or intended bytes; a located file identifies where bytes can be retrieved. The same Turtle manifestation may be available from a static site, a raw GitHub URL, and an archival mirror. Conversely, a location may need digest or observation evidence before an application trusts it as the intended manifestation. DCAT's distribution/download pattern is broad enough to carry much of this, but Semantic Flow separates representation identity from retrieval location because publication workflows frequently need to reason about them independently.

The model is deliberately sparse-friendly. The full chain should be used when each layer carries evidence. A small support artifact may instead link directly from `DigitalArtifact` to `LocatedFile`; a state may link directly to a file; a working file may be identified by a local path or access URL. These skip-level links are not alternate metaphysics. They are coordinates of convenience for cases where the omitted layers would be ceremonial.

### 2.1 Change, Lineage, and Coordinates

Digital artifacts change in more than one way. An ontology may have formal releases, working drafts, imported upstream source states, generated documentation states, and curated reference-catalog states. These should not all compete for one global "current" pointer. Semantic Flow uses `ArtifactHistory` to make lineages explicit. A release history can have a latest release state; a draft history can have a latest draft state; a source-import history can preserve upstream extraction states. A `defaultArtifactHistory` identifies the lineage that ordinary versioning operations should extend when no history is explicitly selected.

Resolution coordinates make common reference tasks explicit. Exact coordinates target a specific state, manifestation, located file, repository commit, or digest. Latest coordinates resolve within a named `ArtifactHistory`, such as "latest release state." Working coordinates refer to mutable authoring surfaces and should not be confused with settled publication evidence. Default-history coordinates identify where new states should be written. Semantic Flow avoids making "current" a core coordinate because current is contextual: current release, current working file, and current published draft are different policies over different histories.

The model also clarifies pseudo-immutability. A `HistoricalState` or settled `LocatedFile` may be hosted on mutable infrastructure, but the coordinate is governed as if it were append-only: do not replace the bytes behind a settled coordinate with different content. If a correction is necessary, mint a new state or a new located file and preserve the trail.

### 2.2 DCAT Alignment

Semantic Flow should be read as a DCAT-compatible profile for static artifact publication, not as a replacement for DCAT. DCAT 3 already provides resource versioning through `dcat:hasVersion`, `dcat:isVersionOf`, `dcat:previousVersion`, and `dcat:hasCurrentVersion`, and it provides `dcat:DatasetSeries` for collections of separately published datasets. Semantic Flow reuses that layer while adding a more specific artifact publication grammar.

| Concern | DCAT 3 | Semantic Flow refinement |
|---|---|---|
| Continuing resource | `dcat:Dataset` / `dcat:Resource` | `DigitalArtifact` as the byte-grounded artifact-level resource |
| Versioned resource | `dcat:hasVersion`, `dcat:isVersionOf`, `dcat:previousVersion` | `HistoricalState` as a settled state within a named `ArtifactHistory` |
| Distribution | `dcat:Distribution` | `ArtifactManifestation` as representation or intended byte-pattern identity |
| Download/access | `dcat:downloadURL`, access properties | `LocatedFile` as a modeled retrievable byte location |
| Series | `dcat:DatasetSeries` | `ArtifactHistory` as an internal lineage facet of one artifact |
| Landing/presentation page | `dcat:landingPage` | `ResourcePage` as a modeled located file that presents another resource |
| Operational resolution | out of scope for DCAT core | `ArtifactResolutionSpec` and observations for exact, latest, default, and working coordinates |

The most distinctive addition is not "versions exist." DCAT already says that. Semantic Flow's addition is that versions can be organized into named internal histories, connected to representation-level manifestations and located files, and used by static publication tools without hiding semantics in repository conventions or server configuration.

## 3. Identifiers, Referents, and ResourcePages

Digital artifacts are only part of the identifier story. A Semantic Mesh may mint identifiers for ontology documents, ontology terms, examples, generated pages, metadata files, source registries, histories, states, and located files. These identifiers are all IRIs, but they do not all name the same kind of thing. A class IRI names a vocabulary term. An ontology IRI may identify a continuing DigitalArtifact. A release-state IRI names a settled state. A Turtle-file IRI names retrievable content. A ResourcePage IRI names a page presenting another resource.

We use **primary identifier** for an IRI minted for the resource a publisher primarily wants to name for external use: an ontology, ontology term, dataset, document, person, organization, concept, or other referent. We use **support identifier** for an IRI minted for a mesh-managed support resource: a ResourcePage, ReferenceCatalog, KnopMetadata artifact, ArtifactHistory, HistoricalState, ArtifactManifestation, LocatedFile, or similar resource. This is a role distinction in a mesh context, not an eternal class distinction. A support artifact can itself become a primary subject when someone wants to talk about it directly.

### 3.1 Be Unambiguous

The Cool URIs guidance says to be unambiguous: do not confuse the IRI of a thing with the IRI of a document about the thing [CoolURIs]. Web Architecture similarly says that a URI identifies one resource, and calls the failure case a URI collision [WebArch]. Semantic Flow operationalizes this discipline by separating primary identifiers from support identifiers and by making pages, states, manifestations, and located files explicit.

For example, in the Fantasy Rules ontology, the IRI for `fant:wisdom` names the Wisdom ability, while `.../ontology/wisdom/index.html` names the generated HTML page presenting that ability. The ontology document that contains the definition is a different resource again, and the concrete Turtle file containing bytes for a release state is yet another. These distinctions may feel fussy until a downstream application needs to cite exactly which bytes it consumed, or until a term page is regenerated with different prose while the term itself remains the same.

### 3.2 ResourcePages and Embedded RDF

A `ResourcePage` is a `LocatedFile` whose role is to present another resource. It is the page returned for human and machine use when a directory-like IRI is requested from a static host that supports directory index resolution. In Apache this behavior is conventionally controlled by `DirectoryIndex`; on GitHub Pages and similar hosts, it is the ordinary behavior that `/path/` returns `/path/index.html`.

A ResourcePage should contain machine-readable RDF, via RDFa and/or embedded JSON-LD, that distinguishes the requested IRI's referent from the HTML file returned to present it. At minimum, a generated ResourcePage should carry a page identity graph:

```turtle
<presented-resource>
  sflo:hasResourcePage <page-iri> .

<page-iri>
  a sflo:ResourcePage, schema:WebPage ;
  schema:about <presented-resource> .
```

The question "which RDF should be embedded?" is answered by `ResourcePageDefinition`. A page definition specifies which page regions, sources, panels, and generated facts belong in the page. This prevents "ResourcePage" from meaning "dump all known triples into HTML." The page RDF is governed publication content. It can be selected, reviewed, regenerated, versioned, and traced to sources.

Listing 1 shows the minimal RDFa pattern for a Fantasy Rules term page.

**Listing 1. Minimal RDFa identity graph for a ResourcePage.**

```html
<!doctype html>
<html lang="en"
  prefix="
    sflo: https://semantic-flow.github.io/sflo/ontology/
    fant: https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/
    rdfs: http://www.w3.org/2000/01/rdf-schema#
    schema: https://schema.org/
    skos: http://www.w3.org/2004/02/skos/core#
  ">
<head>
  <meta charset="utf-8" />
  <title>Wisdom</title>
</head>
<body
  resource="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom/index.html"
  typeof="sflo:ResourcePage schema:WebPage">
  <link
    property="schema:about"
    href="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom" />
  <main>
    <section
      resource="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom"
      typeof="fant:Ability">
      <link
        property="sflo:hasResourcePage"
        href="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom/index.html" />
      <p property="rdfs:label">Wisdom</p>
      <p property="skos:definition">Perceptiveness and mental fortitude.</p>
    </section>
  </main>
</body>
</html>
```

### 3.3 From Cool URIs to Cooler IRIs

Semantic Meshes extend the Cool URIs tradition toward what we might call "Cooler IRIs": identifiers that are not only stable and dereferenceable, but also statically hostable, artifact-aware, and supported by inspectable RDF evidence. A Cooler IRI can return a useful HTML page through directory index resolution; the page can say whether it is a page, a file, or a presentation of some other resource; the page can link to exact state, manifestation, and located-file coordinates; and the mesh can preserve the support evidence needed to interpret the identifier over time.

Fragment identifiers still have a role in this story, but they should not be asked to do all the work. Hash IRIs remain useful when a publisher wants identifiers whose fragment resolution is contained in a single retrieved representation. Semantic Flow also supports slash/directory-style identifiers because they compose well with static hosting, per-resource pages, and generated support artifacts. When fragment identifiers are used in a ResourcePage, they should resolve to meaningful HTML anchors and be backed by the page's embedded RDF where appropriate.

## 4. Semantic Meshes

A `SemanticMesh` is a governed identifier surface plus the RDF-described support structure needed to publish, inspect, and maintain that surface. It is not merely a website, although it can be hosted as one. It is a publication object whose output is ordinary files and ordinary RDF.

A mesh usually has a canonical base IRI, one or more primary identifiers, Knop-managed support for those identifiers, generated ResourcePages, support artifacts such as inventories and metadata, and optional payload DigitalArtifacts. Because the output is static files, a mesh can be served from GitHub Pages, institutional web space, an object store, or a CDN-backed static host. Because the semantics are in RDF files and embedded page data, the mesh can be reviewed in Git and moved between hosts without depending on a custom server.

Within a mesh, a **Knop** is the support object for one primary identifier or governed designator. The Knop is not the referent. It is the support anchor that can own metadata, inventory, a reference catalog, a source registry, a page definition, a generated ResourcePage, and an optional payload artifact. This pattern lets the same publication machinery support different referents without collapsing them: ontology terms, ontology documents, concepts, examples, and support artifacts can all have explicit support.

Several support artifacts matter for sense-finding. A `ReferenceCatalog` records curated RDF reference sources for a resource. A `KnopSourceRegistry` records source bindings: where source bytes were extracted, imported, or integrated. A `KnopInventory` says what support artifacts exist. `KnopMetadata` describes the support structure. A `ResourcePageDefinition` controls generated page composition and embedded RDF. These support artifacts can themselves be DigitalArtifacts when their own states matter.

Semantic Flow distinguishes **integrate** from **import**. Integration binds available source bytes to a target designator and payload artifact while leaving the source where it is, such as an ontology file in a source branch or adjacent directory. Import copies a working file into the mesh or publication tree so the copy becomes governed local working content. This distinction matters for repository topologies. A sidecar mesh may live in `docs/` while source files remain in `ontology/`; a branch-published mesh may publish generated pages on `gh-pages` while source files remain on `main`; a whole-repository mesh may use the repository root as the publication surface. The artifact model is the same in each case; topology affects operational configuration and provenance.

## 5. Worked Example: The Fantasy Rules Ontology Surface

The Fantasy Rules fixture is a small SRD-inspired ontology used to test Semantic Flow publication workflows. Its source ontology defines terms such as `fant:Character`, `fant:Ability`, `fant:AbilityScore`, `fant:Alignment`, `fant:wisdom`, `fant:strength`, and `fant:hasAbilityScore`. This makes it a better explanatory example than the Semantic Flow ontology itself: readers can understand the domain without first understanding the framework.

The source ontology is a primary DigitalArtifact. It has an ontology-level IRI such as:

```turtle
<https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology>
  a owl:Ontology, sflo:DigitalArtifact ;
  dcterms:title "Fantasy Rules Ontology" ;
  dcterms:description "A small SRD-inspired ontology fixture for Semantic Flow branch publication." ;
  dcterms:source <https://www.dndbeyond.com/srd> .
```

Its term IRIs are primary identifiers too, but they are not DigitalArtifacts. The IRI `https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom` identifies the Wisdom ability. Its sense is established by RDF facts in the ontology, generated pages, source provenance, and future publication states. The generated ResourcePage at `.../ontology/wisdom/index.html` is a support resource presenting the term. The support directory `.../ontology/wisdom/_knop/` contains the Knop support surface, including inventory, sources, and metadata in the generated sidecar fixture.

After a release-oriented weave, the ontology artifact can expose publication coordinates such as:

```turtle
<ontology>
  a owl:Ontology, sflo:DigitalArtifact ;
  dcat:hasVersion <ontology/releases/v0.0.2> ;
  sflo:defaultArtifactHistory <ontology/releases> ;
  sflo:hasResourcePage <ontology/index.html> .

<ontology/releases>
  a sflo:ArtifactHistory ;
  sflo:latestHistoricalState <ontology/releases/v0.0.2> .

<ontology/releases/v0.0.2>
  a owl:Ontology, sflo:HistoricalState ;
  dcat:isVersionOf <ontology> ;
  sflo:hasManifestation <ontology/releases/v0.0.2/ttl> .

<ontology/releases/v0.0.2/ttl>
  a sflo:ArtifactManifestation ;
  sflo:locatedFileForManifestation
    <ontology/releases/v0.0.2/ttl/fantasy-rules-ontology.ttl> .

<ontology/releases/v0.0.2/ttl/fantasy-rules-ontology.ttl>
  a sflo:LocatedFile ;
  sflo:hasContentDigest "sha256:..." .
```

This compact graph illustrates the paper's distinctions. The ontology IRI identifies a continuing DigitalArtifact. The release IRI identifies a settled state, reached through DCAT versioning and bounded by a Semantic Flow history. The `/ttl` IRI identifies the Turtle manifestation, not a particular host location. The `.ttl` IRI identifies the retrievable file. The term IRI for `wisdom` identifies a vocabulary resource whose ResourcePage presents it but is not it.

The same fixture demonstrates mesh topology. In a sidecar form, generated pages and support artifacts live under a publishable `docs/` directory while source files remain in `ontology/`, `shacl/`, and `examples/`. In a branch-published form, the source lane and publication lane can diverge: source files remain on a source branch, while generated ResourcePages and support artifacts are published from a publication branch. Both topologies preserve the same model-level distinctions.

The Semantic Flow ontology itself is also published through these ideas. That reflexive use is useful for dogfooding, but Fantasy Rules is the clearer worked example: it shows term pages, source artifacts, release coordinates, and support identifiers without requiring the reader to already know Semantic Flow vocabulary.

## 6. Implementation: Weave

Weave is the filesystem-native command-line implementation of Semantic Flow. It creates and updates Semantic Meshes, integrates source artifacts, records source bindings, appends historical states, validates mesh structure, and generates ResourcePages.

The core operations correspond to the model. `mesh.create` establishes the mesh support surface. `knop.create` creates support for one designator. `integrate` binds source bytes to a target designator without copying them into the mesh as the normal source of truth. `import` copies bytes into the governed mesh when that is the intended lifecycle. `weave` orchestrates versioning, validation, and generation for already governed targets. `extract` creates support surfaces for terms already mentioned in governed RDF artifacts.

Because a mesh is ordinary files, Weave workflows are local-first. A user can author RDF in an IDE, run validation, generate pages, inspect diffs, and publish to a static host. Release states and generated pages are reviewable as files, not hidden behind a running service. This matters in the age of LLM-assisted authoring: modern tools are increasingly effective at editing plain RDF, Markdown, and configuration files when the repository structure is predictable and validation feedback is explicit.

## 7. Related Work

Cool URIs and the broader httpRange-14 discussion provide the immediate dereferenceability background [CoolURIs] [Tennison2012]. Semantic Flow does not reject those patterns. It offers a static-hosting-friendly pattern in which directory index resolution returns a ResourcePage that states its relationship to the requested resource in RDF.

DCAT 3 is the core catalog alignment [DCAT3]. Semantic Flow reuses DCAT's dataset, distribution, and versioning concepts, while adding static artifact publication structure: named internal histories, historical states as settled version resources, manifestations as distribution specializations, located files, ResourcePages, and resolution specs. FRIR and WEMI provide an important contrast around information resources, works, expressions, manifestations, items, provenance, and electronic resources [FRIR] [OpenWEMI]. Semantic Flow is WEMI-aware but not WEMI-shaped: it leaves work/expression modeling to the domain and concentrates on byte-grounded digital artifacts and publication states.

FAIR vocabulary guidance and RDF publication recipes identify the expectations Semantic Flow tries to make easier to satisfy [FAIRVocab] [Recipes]. Existing publication tools such as Pubby, LodView, Widoco, LODE, pyLODE, Jekyll RDF, Walder, and Harshp demonstrate the demand for human- and machine-readable RDF publication surfaces [Pubby] [LodView] [Widoco] [LODE] [pyLODE] [JekyllRDF] [Walder] [Harshp]. Semantic Flow's distinction is not that it generates prettier documentation, but that it models the pages, support artifacts, histories, states, manifestations, and files as part of the publication graph.

Repository-native ontology engineering and CI/CD methods, including ACIMOV and related Git-based knowledge management work, show that RDF authoring can be practiced in ordinary software-development environments [ACIMOV] [GitKM]. Semantic Flow builds on that reality rather than replacing it. Git is excellent at preserving development history, but public RDF consumers need identifier-level coordinates: which artifact, which lineage, which state, which manifestation, which located file, which digest. Semantic Flow supplies those coordinates in RDF.

Cagle and Shannon's distinction between structure and concept also helps locate the proposal [CagleShannon2026]. Semantic Flow is mostly structural scaffolding for sense-finding. It does not own the concepts expressed by a domain ontology. It keeps the evidence around identifiers durable enough that concept work can be reviewed, revised, and interpreted over time.

## 8. Limitations

Semantic Flow is not a general theory of meaning or intellectual works. It does not decide when two artifacts express the same work, when an ontology term's sense has changed enough to require a new IRI, or which domain model should be used for a class, person, organization, or concept. It provides structures for evidence, coordinates, and publication discipline.

Semantic Flow also does not replace DCAT catalogs, ontology diffing, migration tooling, SHACL validation, or repository history. It gives those tools public coordinates to attach to. A migration report can compare two HistoricalStates; a compatibility note can attach to a release history; a validation report can cite the exact located file and digest it checked.

Static publication has tradeoffs. A static host does not perform arbitrary header-driven content negotiation. Semantic Flow trades that behavior for explicit links and page-embedded RDF. A client that wants Turtle follows the ResourcePage or RDF graph to the manifestation and located file it needs. This is less magical than content negotiation, but easier to host, mirror, archive, and inspect.

Finally, the vocabulary adds learning cost. The full chain is useful when it carries evidence, but it should not be required ceremonially. Sparse modeling and skip-level properties are essential to keeping the model practical.

## 9. Conclusion

Semantic Web identifiers need stable support, not frozen meanings. Their senses evolve through use, documentation, revision, deprecation, and disagreement. Semantic Flow responds by making the publication evidence around identifiers durable, inspectable, and statically hostable.

The faceted `DigitalArtifact` model gives byte-grounded resources a DCAT-compatible navigation grammar from artifact to history, state, manifestation, and located file. The primary/support identifier distinction prevents ontology terms, pages, files, and metadata artifacts from being collapsed into one generic resource kind. ResourcePages make directory-index-based static hosting useful for dereferenceable IRIs while preserving the distinction between a resource and the page presenting it. Semantic Meshes bring these pieces together as portable identifier-support structures that can be edited, reviewed, generated, and hosted as ordinary files.

In that combination, Semantic Flow aims at Cooler IRIs: identifiers that are stable and dereferenceable, but also artifact-aware, source-aware, statically hostable, and supported by pseudo-immutable evidence traces.

## Generative AI Declaration

During the preparation of this work, the author used Codex and other LLM-based writing and coding assistants to plan the work, gather references from notes, search past conversations for decisions and context, review for accuracy and clarity, and generate draft prose. The author reviewed and edited the material and takes responsibility for the publication's content. Final wording should follow venue requirements.

## References

- **[ACIMOV]** *The ACIMOV Methodology: Agile and Continuous Integration for Modular Ontologies and Vocabularies*.
- **[CagleShannon2026]** K. Cagle and C. Shannon. *Structure vs. Concept*.
- **[CoolURIs]** L. Sauermann and R. Cyganiak, editors. *Cool URIs for the Semantic Web*. W3C Interest Group Note.
- **[DCAT3]** W3C. *Data Catalog Vocabulary (DCAT) - Version 3*. W3C Recommendation.
- **[FAIRVocab]** *Best Practices for Implementing FAIR Vocabularies and Ontologies on the Web*.
- **[FRIR]** *Functional Requirements for Information Resource Provenance on the Web*.
- **[GitKM]** *Decentralized Collaborative Knowledge Management Using Git*.
- **[Harshp]** Harshp personal web-space Linked Data publication.
- **[JekyllRDF]** *Jekyll RDF: Template-Based Linked Data Publication with Minimized Effort and Maximum Scalability*.
- **[LinkedData]** T. Berners-Lee. *Linked Data*. W3C Design Issues note.
- **[LODE]** LODE: Live OWL Documentation Environment.
- **[LodView]** LodView IRI dereferencer.
- **[OpenWEMI]** *Works, Expressions, Manifestations, Items: An Ontology*.
- **[Pubby]** Pubby Linked Data front-end.
- **[pyLODE]** pyLODE documentation generator.
- **[Recipes]** *Best Practice Recipes for Publishing RDF Vocabularies*. W3C Working Group Note.
- **[Tennison2012]** J. Tennison. *Using "Punning" to Answer httpRange-14*. Blog post.
- **[Walder]** Walder declarative Linked Data server.
- **[WebArch]** W3C. *Architecture of the World Wide Web, Volume One*.
- **[Widoco]** Widoco ontology documentation generator.
