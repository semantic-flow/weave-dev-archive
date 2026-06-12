<!--
Draft status: full first draft, 2026-06-11.
Target: FOMI 2026 (JOWO, co-located with FOIS 2026), CEUR-ART one-column, 5-9 pages incl. references.
Alternative title if the draft becomes more artifact-specific:
  "Semantic Flow: Byte-Grounded Digital Artifact Histories for Static RDF Publication"
Citation placeholders ([DCAT3], [CoolURIs], ...) are resolved in the References section stubs;
full bibliographic details to be filled at PDF-assembly time.
This draft intentionally exceeds the FOMI page budget slightly; it is meant to be durable
source material for the submission, the website, and the docs. Trim guidance: §6 and §8.5
compress most easily; Listing 2 can become prose.
-->

# Semantic Meshes: Stable Identifiers, Immutable Evidence, and Dynamic Sense-Finding

## Abstract

Public RDF identifiers carry two obligations that pull in opposite directions. They must remain stable and resolvable for decades, on whatever hosting their publishers can sustain, while the senses their communities attach to them continue to evolve. Existing practice scatters the evidence that connects an identifier to its meaning across version-control history, catalog metadata, generated documentation, and server configuration, leaving no RDF-native way to ask what an identifier was being used to mean at a given time, which bytes backed that use, and how its publication state has changed since. This paper introduces the *Semantic Mesh*: a durable, inspectable, statically hostable support structure for an identifier surface, defined by the Semantic Flow framework. A mesh separates the resource an identifier names from the artifacts that support finding its sense: presentation pages (`ResourcePage`), per-identifier support artifacts (reference catalogs, source registries, inventories, metadata, page definitions), and, for digital artifacts, byte-grounded artifact facets (`DigitalArtifact`, `ArtifactHistory`, `HistoricalState`, `ArtifactManifestation`, `LocatedFile`) with explicit resolution coordinates. Semantic Flow makes the evidence for establishing, inspecting, and revising identifier meaning durable and portable across commodity static hosting. The model is DCAT-compatible — `DigitalArtifact` specializes `dcat:Dataset` and `ArtifactManifestation` specializes `dcat:Distribution` — and WEMI-aware without being WEMI-shaped. We ground the discussion in the publication surface of the Semantic Flow ontology itself and report on Weave, a filesystem-native reference implementation.

**Keywords:** RDF publication, persistent identifiers, digital artifacts, ontology versioning, Linked Data, DCAT, dereferenceability

## 1. Introduction

Anyone who publishes an ontology, a vocabulary, or any other RDF dataset under public IRIs takes on a long-lived commitment. The IRIs will be copied into other people's data, embedded in query templates and validation shapes, cited in papers, and cached in the weights of language models. Long after an IRI has been minted, the identifier is expected to resolve to something useful, and an agent encountering one is expected to be able to work out what it is being used to mean, its sense.

Current practice meets this commitment with an uneasy patchwork. Version-control repositories hold the byte history, but repository history is organized around commits and file paths, not around public identifiers. Catalog vocabularies such as DCAT describe datasets and their distributions, but stop short of modeling an artifact's internal publication lineage or the bytes that back it [DCAT3]. The Linked Data and Cool URIs traditions explain how identifiers should dereference, but their best-known patterns assume server behavior — 303 redirects, content negotiation — that commodity static hosting does not provide [LinkedData] [CoolURIs]. Documentation generators and publication tools produce valuable human-readable surfaces, but those surfaces are not themselves modeled, so the relationship between an identifier and the content returned for it remains implicit in directory conventions and server configuration.

The result is a familiar failure mode: meaning-supporting evidence for a public identifier exists, but it is distributed across systems that share no common model; there is no standardized way to catalog sources and references, and most of it is invisible to RDF tooling. When an ontology undergoes a major revision — a public contract change, not merely a repository diff — there is no standard way to say, in RDF, *this identifier's publication state changed here; the previous state remains citable there; and the bytes for each state are retrievable here, with these digests*.

This paper proposes a model for that missing layer. Its central construct is the **Semantic Mesh**: a governed namespace region that supports an identifier surface with durable, inspectable, statically hostable artifacts. The framework that defines meshes is called **Semantic Flow**; its core vocabulary is the Semantic Flow ontology (preferred namepsace prefix: `sflo`). A filesystem-native reference implementation, **Weave**, demonstrates that the model is operational, but the conceptual contribution of this paper is the mesh, not the tool.

The thesis can be stated compactly. Public RDF identifiers need high availability, static-host-friendly dereferenceability, and stable support for *sense-finding* — the activity by which an agent, human or machine, works out what an identifier is being used to mean from the evidence its publisher has made available. Identifier senses can evolve, and may even become inconsistent with earlier states. The answer is not to pretend senses are immutable, and Semantic Flow does not freeze meaning. Instead, it supplies a coherent framework for establishing, inspecting, and revising meaning: identifier support structures that can be versioned and moved across hosting providers, byte-grounded artifact histories whose settled states are treated as immutable evidence, presentation pages that are graph-visible rather than implicit, and explicit resolution coordinates that distinguish *latest*, *working*, *default*, and *exact* references.

The paper makes four contributions:

1. It motivates and defines the Semantic Mesh as the unit of identifier support, separating the resource an identifier names from the artifacts used to support finding its sense (§2–§3).
2. It presents the `DigitalArtifact` facet stack — `DigitalArtifact`, `ArtifactHistory`, `HistoricalState`, `ArtifactManifestation`, `LocatedFile`, `ResourcePage` — together with sparse-modeling skip-links and explicit resolution coordinates (§4).
3. It shows how `ResourcePage`s make dereferenceability work on commodity static hosting without custom server behavior, and locates this design relative to the httpRange-14 debate (§5).
4. It positions the model against DCAT 3, FRIR/WEMI, the structure-versus-concept distinction, and repository-level versioning, arguing that Semantic Flow refines rather than replaces these (§8).

The Semantic Flow ontology's own publication surface serves as the running example (§7), and Weave is reported as implementation evidence (§9).

## 2. Stable Identifiers, Evolving Senses

### 2.1 Identifiers should be as specific as the reference task requires

Different reference tasks need different grips on their referent. A tutorial that says "the SFLO ontology" is referring to something that persists through releases. A conformance report that was produced against particular bytes needs to cite exactly those bytes, ideally with a digest. A build pipeline needs "the latest release in the release lineage," which is neither of the above: it is a mutable coordinate with a precise resolution rule.

We adopt the principle that *identifiers should be as specific and predictable as the reference task requires*. At the most concrete end, an identifier that denotes a particular arrangement of bytes should keep denoting those bytes: this is the single-referent discipline that makes citation, caching, mirroring, and verification possible. At the more general end, an artifact-level identifier legitimately denotes something that persists through change, and pinning it to bytes would be a category error. Problems arise not from generality or specificity as such, but from systems that offer only one level — or that offer several levels without a consistent way to tell them apart.

### 2.2 Senses evolve; sense-finding support should not

Even a well-run identifier acquires its meaning socially. A class IRI's sense may narrow when a revision tightens its definition; a property may drift as a community uses it in ways the original comment did not anticipate; an concept's specificity may shift as its ecosystem changes. Occasionally an identifier's current sense becomes outright inconsistent with assertions made against an earlier state.

There are two tempting responses, both wrong. One is to declare senses frozen — to insist the identifier means what it meant at minting, which the surrounding usage simply falsifies over time. The other is to give up on stability and let identifiers mean whatever their latest documentation says, hampering the ability to interpret older data. The workable position is in between: let senses evolve, but make the *evidence trail* durable, so that any agent can reconstruct what the identifier was being used to mean at any settled point. When a sense genuinely splits, the publisher can mint new identifiers — preferably two new ones, leaving the old identifier as a documented fork point — and the trail records the split rather than papering over it.

This reframing matters because it changes what the support infrastructure must do. It does not need to adjudicate meaning. It needs to keep settled states settled, keep them retrievable, keep mutable pointers (latest, current, working, deprecated, replaced) clearly distinguished from settled coordinates, and keep the whole structure inspectable in RDF rather than buried in server behavior or repository internals.

### 2.3 The static-publication lesson

A second pressure shapes the design: availability. Identifier infrastructure earns trust over decades, and the most robust hosting available to most publishers is commodity static hosting — object stores, CDN-backed static site services, institutional web space. Dynamic resolution layers add fragility: every redirect service, content-negotiation rule, and triple-store-backed page generator is a component that can be misconfigured, defunded, or attacked. Shared resolver infrastructure concentrates that risk; the 2024 denial-of-service attacks on the Internet Archive, which disrupted services co-hosted with the long-standing `purl.org` resolver, illustrated how a community's identifier plumbing can fail for reasons unrelated to any individual publisher. The lesson is to leave the hard work of availability to the providers who do it at scale, and to demand nothing from the host beyond serving files.

Static site generation also teaches a subtler lesson about *when* computation should happen. If a publication surface is generated at publish time rather than per-request, then everything the surface asserts is itself an artifact: it can be inspected before deployment, diffed, archived, mirrored, and regenerated elsewhere. This supports local-first workflows — a mesh is fully functional on a laptop, served from a directory — and it supports direct authoring of RDF by humans and, increasingly, by LLM-based tools, which work far better against plain files with predictable layouts than against opaque server endpoints.

The design constraint that falls out of §2.1–§2.3 is this: *encode sense-support and artifact coordinates in RDF data that a dumb host can serve, rather than in behavior only a smart server can perform.* The rest of the paper describes a model built to that constraint.

## 3. Semantic Meshes

### 3.1 The mesh as the unit of identifier support

A **Semantic Mesh** is a governed namespace region that supports an identifier surface. By *identifier surface* we mean the set of public IRIs a publisher exposes under one or more base IRIs: ontology and term IRIs, dataset IRIs, release IRIs, file IRIs, page IRIs. The mesh is the support structure behind that surface: a coherent collection of RDF documents, byte artifacts, and generated pages, organized by Semantic Flow conventions, that together let an agent dereference the identifiers, retrieve their backing bytes, and inspect the evidence for their senses.

The mesh — not the framework, and not any particular tool — is the interesting unit, because it is the thing that has the desirable operational properties:

- it can be **inspected**: every layer of support is either an RDF document or a byte artifact described by one;
- it can be **versioned**: meshes live comfortably in version control, and their settled states are explicit resources;
- it can be **statically hosted**: a mesh's published form is a directory tree that any web server or object store can serve;
- it can be **moved**: because nothing in the mesh depends on host-specific behavior, the same tree can be re-hosted, mirrored, or archived without losing its semantics.

In Weave, a mesh is backed by a filesystem region, but the mesh concept is framework-level: a mesh could in principle be materialized from an object store or held in memory.

### 3.2 Knops: per-identifier support objects

Within a mesh, each supported identifier gets a **Knop**: a mesh-managed support object and naming anchor associated one-to-one with that identifier. The Knop carries the mesh-relative *designator path* from which the identifier IRI is formed in mesh context, and it hosts whatever support artifacts the identifier needs — possibly including a primary *payload* artifact when the identifier names mesh-managed content.

The crucial move is the separation the Knop enforces: **the thing referenced is not the artifacts used to support finding its sense.** An identifier in a mesh may name an ontology, a dataset, a person, a place, or an abstract concept. The Knop is never that thing; it is the mesh's support object *for* it. This separation is what allows the same support machinery to serve identifiers whose referents are digital artifacts (where the mesh may govern the artifact's actual bytes) and identifiers whose referents are not (where the mesh can still curate evidence about them).

### 3.3 Support artifacts for sense-finding

Whether or not an identifier's referent is a digital artifact, a Knop can host a small family of support artifacts, each an RDF document with a narrow job:

- **`ReferenceCatalog`** — a catalog of curated *reference links* about the identified resource. Each `ReferenceLink` binds a source of RDF reference data to a role from a small controlled vocabulary (*canonical*, *supplemental*, *deprecated*) and can carry verification stamps (`verifiedAt`, `verifiedBy`). A reference catalog is deliberately narrow: it holds reference-link relators, not arbitrary descriptive RDF.
- **`KnopSourceRegistry`** — a registry of *source bindings*: records of where the Knop's data came from. Extraction sources record the RDF document bytes from which a managed resource was first grounded; import sources record bytes actively copied into the mesh; integration sources register bytes where they already live. A source registry is provenance about source bytes and resolution, not operational configuration.
- **`KnopInventory`** — the current managed surface of the Knop: what support artifacts exist and how they are arranged.
- **`KnopMetadata`** — metadata about the Knop and its support structure as mesh-managed objects.
- **`ResourcePageDefinition`** — the authored definition of the identifier's presentation page: its content regions and the source bindings each region draws from (§5).

Consider an identifier that names a person — clearly not a digital artifact. Its Knop can still carry a reference catalog pointing at the canonical RDF descriptions of that person (with one marked deprecated after a data-source migration), a source registry recording which upstream document each description was drawn from and when, and a page definition that composes a human-readable profile. None of this fixes who the person *is*; all of it helps an agent establish what the identifier is being used to denote, and how confidently.

A point worth stressing: in this model, **evidence is not only provenance metadata**. The settled data artifacts themselves — historical states, manifestations, located files, reference sources, inventories, metadata states — are evidence. A digest-bearing Turtle file from a settled release supports sense-finding at least as strongly as a provenance triple asserting that a release occurred.

## 4. The DigitalArtifact Facet Stack

When an identifier's referent *is* a digital artifact — an ontology document, a dataset, a SHACL shapes file, a generated page — Semantic Flow offers a layered model of that artifact's identity, history, and bytes. The layers are *facets* of one artifact's identity, not independent things, and the model is deliberately sparse-friendly: publishers materialize a layer when it carries evidence and skip it when it would be ceremonial.

### 4.1 The six layers

**`DigitalArtifact`** is the governed artifact-level continuity handle: the resource that persists across releases and revisions. It subclasses `dcat:Dataset` (§8.1). A `DigitalArtifact` is not a complete theory of works, editions, or intellectual products; it is the artifact considered as the governing resource across its states, manifestations, and retrievable files.

**`ArtifactHistory`** is an explicit lineage resource *inside* a `DigitalArtifact`: a diachronic facet that groups published states and carries lineage-level operational metadata. One artifact can have several histories — a release lineage, a draft lineage, an archive of a predecessor's states, an editorial-curation lineage, a source lineage tracking upstream extraction. Two distinguished pointers govern how histories are selected. `defaultArtifactHistory` names the lineage that write operations should extend when no history is explicitly selected. `currentArtifactHistory` is better treated as a compatibility or current-lineage pointer — "the lineage consumers should regard as current" — not as a write target. The two often coincide, and data may assert both toward the same history; but when the default history and the latest-updated or current history diverge, well-behaved operations *require explicit history selection* rather than guessing. This rule keeps a common failure — accidentally extending the wrong lineage — from being silent.

**`HistoricalState`** is a settled snapshot facet, normally organized within an `ArtifactHistory`. Settled states are the model's quasi-immutable evidence: once published, a state's content does not change, and convenience pointers that do change (such as a history's `latestHistoricalState`) must not be asserted *into* the settled state's own data. States chain via `previousHistoricalState` (a specialization of `prov:wasRevisionOf`), giving each lineage an inspectable revision order. A `HistoricalState` is the natural citation target for "the ontology as released on 2026-05-24."

**`ArtifactManifestation`** is a concrete variant of an artifact or state: Turtle versus JSON-LD, Markdown versus PDF, zipped versus unzipped, one canonicalization profile versus another. It subclasses `dcat:Distribution` (§8.1) and is typically linked from a `HistoricalState` via `hasManifestation`, though sparse data may link it directly from the artifact.

**`LocatedFile`** is a retrievable byte identity — typically an extension-bearing IRI from which exactly those bytes can be fetched. Located files are where digests live (`hasContentDigest`, with algorithm-prefixed values such as `"sha256:…"`), making them the unit of verification, caching, and mirroring. `locatedFileForManifestation` relates a manifestation to the files serving its bytes, specializing `dcat:downloadURL` for the case where the downloadable thing is itself modeled.

**`ResourcePage`** is the special `LocatedFile` whose role is to *present another resource*: an HTML page, conventionally at a reserved `index.html` location, linked from the presented resource via `hasResourcePage`. Ordinary located files identify content directly — they are terminal byte resources and usually need no pages of their own. The page/content distinction is developed in §5.

### 4.2 Sparse modeling and skip-level links

The full chain — artifact → history → state → manifestation → located file — earns its keep when each layer carries evidence: multiple lineages, citable states, format variants, digest-bearing files. But a small supporting artifact, such as a Knop's inventory, may need none of that, and forcing five resources into existence for every file would make the model unusable.

Semantic Flow therefore provides consistency-preserving skip-level links: `locatedFileForArtifact` (artifact directly to file), `locatedFileForState` (state directly to file), `hasManifestation` asserted directly from an artifact, and working-surface shortcuts such as `hasWorkingLocatedFile`, `workingLocalRelativePath`, and `workingAccessUrl`. Two commitments keep sparseness honest. First, skip-links are *coordinates of convenience, not alternate metaphysics*: a `locatedFileForState` assertion means exactly what the corresponding full chain would mean. Second, when both a skip-link and the full chain are asserted, they must agree — the shortcut SHOULD be consistent with `hasManifestation`/`locatedFileForManifestation` composition, and mesh tooling can check this.

### 4.3 Resolution coordinates

The remaining piece is a vocabulary for *requesting* bytes. An **`ArtifactResolutionSpec`** is a relator resource — an instance of the familiar n-ary relation pattern [NAry] — that identifies target bytes for some application concern. A spec can name its target at any level of the stack (`targetArtifact`, `targetArtifactHistory`, `targetHistoricalState`, `targetManifestation`, `targetLocatedFile`), or by an unmanaged local path, an access URL, or a version-control coordinate (`RepositorySourceLocator`, recording repository identity, ref or commit, and repository-relative path). It can carry an expected digest (`expectsContentDigest`), a fallback spec, and a resolution mode.

The modes encode the latest/working/exact distinctions from §2.1 explicitly. Exact coordinates — a specific state, located file, manifestation, commit, or digest — are exact by default. A targeted `ArtifactHistory` resolves to the latest state *in that history*. The `working` mode resolves mutable working bytes under operational policy and is intentionally not an immutable identity unless paired with commit, state, file, or digest evidence. And when a resolution event is worth remembering, an `ArtifactResolutionObservation` records what was actually resolved — observed digest, timestamp, observer — as appendable evidence distinct from the request.

Together the stack and the coordinates yield a small navigation grammar with recognizable idioms. A *citation* path runs artifact → release history → exact state → manifestation → located file → digest. An *update* path runs artifact → default history → new state, with explicit history selection required if default and current lineages have diverged. A *sparse* path runs artifact → located file. A *working* path runs artifact → working file or path. Each idiom is plain RDF, so each is reviewable: a pull request that changes a mesh's coordinates is a readable diff over these assertions.

### 4.4 Why ontologies are the motivating case

Nothing in the stack is ontology-specific, but ontologies are the RDF data for which mesh representation pays off most visibly. Their term IRIs are maximally public and maximally long-lived; their releases function as contracts for downstream data, shapes, and tooling; their senses demonstrably drift across revisions; and their publication already involves exactly the artifact types meshes govern — serialized documents in several formats, generated documentation, validation shapes, examples, registries. An ontology's publication surface is a digital artifact ecosystem in miniature, which is why this paper's running example (§7) is an ontology's surface — in fact, SFLO's own.

## 5. ResourcePages and Static Dereferenceability

### 5.1 The problem, twenty years on

The httpRange-14 question — does an IRI identify a document, a thing, or something in between? — produced the 303-redirect and hash-IRI disciplines documented in Cool URIs [CoolURIs]. Both work; both also assume capabilities or conventions that sit awkwardly with the static-hosting constraint of §2.3. Commodity static hosts do not issue 303s, do not negotiate content, and have exactly one widespread relevant behavior: when a directory-like path is requested, they return that directory's `index.html`.

On the question of what IRIs identify, our position is deliberately modest. IRIs have *socially governed senses in context*: a community's usage, anchored by the publisher's evidence, determines what an identifier is being used to mean. We do not say that an IRI literally refers to its sense, as if the sense were the target object — though analyses in that direction, such as Tennison's punning treatment of httpRange-14 [Tennison2012], usefully highlight that the document retrieved from an IRI and the thing the IRI is used to denote come apart routinely and harmlessly. What matters operationally is not settling the metaphysics but making the distinction *visible in the graph* wherever it matters.

### 5.2 The ResourcePage pattern

Semantic Flow's answer leans on the one behavior static hosts reliably provide. When an agent dereferences a mesh identifier whose IRI is directory-like, the host returns an `index.html`. The mesh makes that mechanically-returned page semantically honest:

1. the returned page is itself a modeled resource — a `ResourcePage`, which is a `LocatedFile` and a `schema:WebPage`;
2. the identified resource is linked to it explicitly: `<resource> sflo:hasResourcePage <…/index.html>`;
3. the page carries RDF (embedded JSON-LD or RDFa, plus links to RDF manifestations) so that both humans and machines receive the assertion *this is a page presenting the resource, not the resource*.

The distinction between an identifier, its referent, and the bytes you happen to receive is thus carried by RDF assertions served from static files, rather than by HTTP status-code choreography. Mesh layout conventions support the pattern mechanically — `index.html` is reserved for resource pages; extension-bearing IRIs generally identify concrete retrievable content while extensionless or slash-terminated paths generally identify higher-level resources — but these file-extension and slash conventions are *mechanical hints, not metaphysical truths*. The RDF assertions are authoritative; the conventions just make the common case predictable.

Two boundary clarifications. First, ordinary located files do not get resource pages: a Turtle file's IRI identifies its bytes directly, and wrapping every file in a presentation layer would reintroduce the ambiguity the model removes. The `ResourcePage` is precisely the located file whose *role* is presentation of another resource. Second, a resource page can itself be promoted to a governed `DigitalArtifact` — with its own history, states, and manifestations — when page identity or page evolution matters, for instance when a deprecation notice's wording is itself part of the public record. This is the metadata-can-have-states principle (§6) applied to presentation.

What this pattern does *not* claim is a solution to content negotiation in general. A static host will not select a representation by `Accept` header; an agent that wants Turtle must follow the mesh's coordinates — from the page or from the resource's RDF — to the manifestation it wants. The trade is deliberate: predictable, link-followable coordinates on infrastructure that is hard to break, instead of per-request negotiation on infrastructure that must be kept alive. Prior static- and server-based publication tools — Jekyll RDF's template-generated per-resource pages [JekyllRDF], documentation generators such as Widoco, LODE, and pyLODE [Widoco] [LODE] [pyLODE], server-side Linked Data front-ends such as Pubby and LodView [Pubby] [LodView], declarative servers such as Walder [Walder], and hybrid human/machine page designs such as Harshp [Harshp] — demonstrate the demand for dereferenceable surfaces; FAIR vocabulary guidance codifies the expectations [FAIRVocab] [Recipes]. Semantic Flow's addition is not another page generator but the modeling move: the page, the files, and their relations to the identified resource become first-class, statically servable RDF.

## 6. Immutable Evidence, Mutable Pointers, and Metadata States

The model's treatment of change can now be stated as a small set of commitments.

**Primary data and settled states are quasi-immutable.** Once a `HistoricalState` is published, its content is settled; once a `LocatedFile` for a settled manifestation is published, its bytes are fixed and digest-verifiable. We say *quasi*-immutable deliberately: no publisher can promise bytes will survive every hosting accident, and meshes are designed to be mirrored and re-hosted precisely because durability is an ongoing practice, not a metaphysical property. The commitment is that settled coordinates are never *reused* for different content.

**Mutable pointers are segregated and labeled.** Latest, working, default, current, deprecated, and replaced are all legitimate references — but they are mutable coordinates, and the model keeps them out of settled evidence. `latestHistoricalState` lives on the history resource, never inside a settled snapshot; working-surface locators (`hasWorkingLocatedFile`, `workingLocalRelativePath`, `workingAccessUrl`) are explicitly non-settled; resolution specs name their mode rather than leaving it implicit. An agent can always tell whether a reference will keep meaning the same bytes.

**Metadata can evolve — and can itself have states.** A Knop's metadata or reference catalog is a `DigitalArtifact` like any other, so it can be given a history and settled states when its evolution is part of the public record: *what did this mesh claim about its sources as of March?* is then an answerable question with citable evidence. And because the move is uniform, metadata about metadata can also be made explicit, simply by minting an identifier for it. There is no infinite-regress obligation here, only an option: the practical rule throughout is *materialize a layer when it carries evidence; skip it when it is ceremonial*.

This is the sense of the paper's title. The identifiers are stable; the evidence is immutable (in the qualified, operational sense above); and sense-finding is dynamic — conducted over a durable trail rather than against a frozen decree.

## 7. Worked Example: The SFLO Publication Surface

The Semantic Flow core ontology is published as a mesh, so its own opening metadata stanza exercises the model. Listing 1 reproduces the stanza from the published Turtle, lightly abridged.

**Listing 1.** The SFLO core ontology's publication stanza (abridged).

```turtle
@base <https://semantic-flow.github.io/sflo/> .
@prefix sflo: <https://semantic-flow.github.io/sflo/ontology/> .

<ontology> a owl:Ontology ;
  dcterms:title "Semantic Flow Core Ontology" ;
  dcat:hasVersion <ontology/releases/v0.2.0> ;
  dcterms:modified "2026-05-19"^^xsd:date ;
  owl:versionIRI <https://raw.githubusercontent.com/semantic-flow/sflo/
                  refs/tags/v0.2.0/semantic-flow-core-ontology.ttl> ;
  owl:versionInfo "0.2.0" .

<ontology/releases/v0.2.0> a owl:Ontology, sflo:HistoricalState ;
  dcat:isVersionOf <ontology> ;
  dcterms:issued "2026-05-24"^^xsd:date ;
  owl:versionInfo "0.2.0" ;
  sflo:hasManifestation <ontology/releases/v0.2.0/ttl> ;
  schema:contentUrl <ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl> .

<ontology/releases/v0.2.0/ttl>
  dcat:downloadURL <ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl> .
```

Reading the stanza through the model: `<ontology>` is the artifact-level identifier — the ontology as a persisting `DigitalArtifact`. `<ontology/releases/v0.2.0>` is a settled `HistoricalState` in the release lineage; it is *also* typed `owl:Ontology`, because for OWL tooling the release is an ontology version, while for mesh navigation it is a state coordinate — an instructive case of one IRI legitimately serving two vocabularies' purposes. `<…/v0.2.0/ttl>` is the Turtle `ArtifactManifestation`, and the `.ttl` IRI under it is the `LocatedFile`: the exact bytes, fetchable, mirrorable, digestable. Note also the `owl:versionIRI`: it points at a tag-pinned raw file in the source repository — a repository coordinate of exactly the kind `RepositorySourceLocator` models, giving the release a second, host-independent byte anchor.

The stanza is deliberately sparse — it links state to manifestation to file without materializing every layer's full description inline. A fuller mesh rendering of the same surface adds the lineage and presentation layers; Listing 2 sketches it (illustrative IRIs).

**Listing 2.** Fuller mesh rendering (illustrative).

```turtle
<ontology> sflo:defaultArtifactHistory <ontology/releases> ;
  sflo:hasResourcePage <ontology/index.html> .

<ontology/releases> a sflo:ArtifactHistory ;
  sflo:hasHistoricalState <ontology/releases/v0.2.0> ;
  sflo:latestHistoricalState <ontology/releases/v0.2.0> .

<ontology/releases/v0.2.0>
  sflo:hasResourcePage <ontology/releases/v0.2.0/index.html> .

<ontology/releases/v0.2.0/ttl> a sflo:ArtifactManifestation ;
  sflo:locatedFileForManifestation
    <ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl> .

<ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl>
  a sflo:LocatedFile ;
  sflo:hasContentDigest "sha256:…" .
```

Table 1 summarizes the coordinates this surface exposes and their mutability expectations.

**Table 1.** Publication coordinates in the SFLO surface.

| Coordinate | Class / role | Example IRI (abbreviated) | Mutability | Job |
|---|---|---|---|---|
| Ontology identifier | `DigitalArtifact` (and `owl:Ontology`) | `…/ontology` | stable identifier; evolving referent | artifact-level continuity handle |
| Release lineage | `ArtifactHistory` | `…/ontology/releases` | append-only | groups release states; default write target |
| Release state | `HistoricalState` | `…/releases/v0.2.0` | settled | exact citation; revision chain anchor |
| Turtle manifestation | `ArtifactManifestation` | `…/v0.2.0/ttl` | settled | format-specific representation |
| Located file | `LocatedFile` | `…/ttl/semantic-flow-core-ontology.ttl` | settled bytes | retrieval, digest verification, mirroring |
| Resource page | `ResourcePage` | `…/v0.2.0/index.html` | regenerable | human/machine presentation of the state |
| Repository pin | repository coordinate | `raw.githubusercontent.com/...:v0.2.0/...` | settled (tag-pinned) | host-independent byte anchor |
| Working surface | working coordinates | repo checkout path | mutable | authoring; explicitly non-settled |

Every row is dereferenceable from a static host, every settled row stays settled, and every relation among rows is asserted in RDF. When v0.3.0 ships, the artifact identifier and lineage persist, a new state and its manifestations appear beside the old ones, mutable pointers move, and the v0.2.0 coordinates remain exactly as citable as the day they were issued. If a term's definition changes meaningfully in that release, nothing in the mesh hides it — the mesh is what lets anyone *see* it, state against state.

## 8. Relation to Existing Models and Practice

### 8.1 DCAT 3

Semantic Flow is DCAT-compatible by construction, and should be read as refining DCAT for byte-grounded publication coordinates rather than replacing it. The alignments are direct subclass/subproperty relations: `DigitalArtifact ⊑ dcat:Dataset`; `ArtifactManifestation ⊑ dcat:Distribution`; `hasManifestation ⊑ dcat:distribution`; and `locatedFileForManifestation ⊑ dcat:downloadURL` for the case where the downloadable file is itself modeled as a resource. Any DCAT consumer can therefore read a mesh's artifacts as datasets with distributions; the mesh adds the layers DCAT's core pattern leaves implicit — settled states, internal lineages, file-level byte identities, modeled pages, and mode-explicit resolution coordinates.

The instructive contrast is `dcat:DatasetSeries`. A dataset series is itself a dataset: a collection *around* separately published datasets, suited to, say, an annual statistical series. It is not the right model for the internal lineage of one artifact: an ontology's release history is not a collection of sibling ontologies but a facet of a single artifact's diachronic identity — and one artifact may need *several* such lineages (releases, drafts, archives, curation, source). `ArtifactHistory` sits *inside* a `DigitalArtifact` to do precisely that job. The two constructs are complementary: a publisher could expose a dataset series whose members are artifact-level datasets, each internally lineaged.

### 8.2 FRIR and WEMI

FRIR — Functional Requirements for Information Resource Provenance on the Web — is the closest conceptual touchpoint [FRIR]. It extends the FRBR/WEMI tradition to electronic information resources, distinguishing roughly work-, content-, byte-, and copy-level identities and integrating with PROV. Semantic Flow shares FRIR's conviction that the resource, its content, its bytes, and its retrievable copies must not be collapsed; `LocatedFile` and `ArtifactManifestation` occupy recognizably similar ground to FRIR's lower layers. OpenWEMI's relaxation of FRBR's rigidities is similarly congenial [OpenWEMI].

But Semantic Flow is *WEMI-aware, not WEMI-shaped*, and it does not implement WEMI. The model deliberately omits a work/expression layer: whether a given `DigitalArtifact` is an expression of some more abstract work — whether two artifacts realize "the same" vocabulary — is a publisher or domain modeling decision, made in the publisher's own ontology if at all. Semantic Flow's distinctive addition relative to FRIR/WEMI is the explicit temporal publication layer: `ArtifactHistory` and `HistoricalState` give artifact publication a first-class diachronic structure that the work/expression tradition, focused on synchronic identity levels, does not supply.

### 8.3 Structure and concept

Cagle and Shannon's "Structure vs. Concept" distinguishes the structural, application-facing layer of knowledge representation from the conceptual and taxonomic layer where shared meaning is curated, observing that structural representations vary by application while conceptual anchors tend to be more stable and differently governed; as they put it, "an ontology tells a reasoner what is true. A taxonomy tells a retrieval system what is near." [CagleShannon2026]. The distinction locates Semantic Flow precisely: meshes are almost entirely *structure in the service of sense-finding*. The facet stack, the coordinates, the pages, the registries — all of it is structural scaffolding around identifiers. The concepts themselves — what a domain class means, how a community's taxonomy organizes nearness — remain the publisher's and the domain's responsibility. A mesh does not try to own or freeze concepts; it gives whoever does curate them a durable place to stand.

### 8.4 Repositories and version control

Git is an excellent storage and collaboration substrate for meshes, and repository-native ontology engineering is an established practice line, from Git-based RDF dataset collaboration [GitKM] to CI/CD-oriented methodologies such as ACIMOV [ACIMOV]. Semantic Flow assumes that substrate rather than competing with it. The gap it addresses is that repository-level history is not identifier-level history: commits and diffs do not, by themselves, expose public RDF coordinates for artifact identity, historical state, manifestation, located file, or page, and they encode no distinction between default, current, working, and exact targeting. A major ontology revision is a *public contract change* — affecting term senses, dependent data, shapes, documentation, and migration paths — and contract changes deserve public, RDF-visible state boundaries, not just a tag in a repository consumers may never see [OntoVersions]. Equally, Semantic Flow does not replace ontology diffing, compatibility analysis, or migration tooling; it provides the settled coordinates and evidence to which such analyses can attach their results.

## 9. Implementation Evidence: Weave

Weave is the filesystem-native reference implementation of Semantic Flow. It materializes meshes as directory trees; its central *weave* operation settles new historical states into the selected artifact history (enforcing explicit history selection when default and current lineages diverge), regenerates inventories and resource pages from `ResourcePageDefinition`s, records source bindings and resolution observations, and validates mesh structure against SHACL shapes. The published output is a plain static site: every identifier in the mesh's surface dereferences correctly from any host that can serve files. Because meshes are plain files with predictable layouts, Weave workflows run local-first and sit naturally alongside IDE- and LLM-assisted RDF authoring. Weave's workflow and demonstration are reported separately; for this paper its role is evidence that the model in §3–§6 is operational rather than aspirational.

## 10. Limitations

The model's restraint is also its boundary, and several limitations should be plain. Semantic Flow is not a theory of intellectual works: it deliberately offers no account of when two artifacts express the same work, and publishers who need one must bring their own. It is not a catalog: DCAT-level aggregation and discovery metadata remain DCAT's job. It is not a semantic-diff or migration framework: it marks the state boundaries that make such analyses attachable, but performs none of them. Static publication cannot reproduce every dynamic server behavior — header-driven content negotiation chief among them — and the pattern of §5 trades that capability away knowingly. Quasi-immutability is an operational discipline, not a guarantee; meshes mitigate, rather than abolish, hosting mortality. The layered vocabulary imposes a real learning cost on adopters, which sparse modeling only partly offsets. And to date the model's evaluation rests on one reference implementation and one family of meshes; broader adoption evidence, profiles for multi-publisher governance, and quantitative comparison with existing publication pipelines are future work.

## 11. Conclusion

Public RDF identifiers do not need their meanings frozen; they need their evidence kept. This paper has argued that the right unit for keeping it is the Semantic Mesh: a durable, inspectable, statically hostable support structure for an identifier surface, in which the thing referenced is systematically separated from the artifacts that support finding its sense. The `DigitalArtifact` facet stack makes artifact identity inspectable at exactly the specificity a reference task requires — artifact, lineage, settled state, manifestation, located bytes — while Knop support artifacts extend sense-finding to identifiers whose referents are not artifacts at all. `ResourcePage`s make dereferenceability a property of plain files plus RDF assertions rather than of server behavior. The model refines DCAT rather than replacing it, learns from FRIR and WEMI without adopting their shape, and supplies what repository history alone cannot: public, identifier-level coordinates for publication states, with mutable pointers clearly fenced off from immutable evidence. Senses will keep evolving; meshes are built so that, whenever anyone needs to ask what an identifier meant, the trail is still there.

## Declaration on Generative AI

During the preparation of this work, the author(s) used LLM-based writing and coding assistants for drafting and editing support, working from author-supplied outlines, terminology, and source material. All content was reviewed and edited by the author(s), who take full responsibility for the publication's content. *(Finalize wording per venue requirements.)*

## Acknowledgements

*(TODO.)*

## References

*(Placeholders; resolve full bibliographic details at assembly time.)*

- **[LinkedData]** T. Berners-Lee. *Linked Data*. W3C Design Issues note.
- **[CoolURIs]** L. Sauermann, R. Cyganiak (eds.). *Cool URIs for the Semantic Web*. W3C Interest Group Note.
- **[Tennison2012]** J. Tennison. *Using "Punning" to Answer httpRange-14*. Blog post.
- **[DCAT3]** W3C. *Data Catalog Vocabulary (DCAT) — Version 3*. W3C Recommendation.
- **[FRIR]** *Functional Requirements for Information Resource Provenance on the Web*.
- **[OpenWEMI]** *Works, Expressions, Manifestations, Items: An Ontology* (OpenWEMI).
- **[CagleShannon2026]** K. Cagle, C. Shannon. *Structure vs. Concept*.
- **[FAIRVocab]** *Best Practices for Implementing FAIR Vocabularies and Ontologies on the Web*.
- **[Recipes]** *Best Practice Recipes for Publishing RDF Vocabularies*. W3C Working Group Note.
- **[JekyllRDF]** *Jekyll RDF: Template-Based Linked Data Publication with Minimized Effort and Maximum Scalability*.
- **[ACIMOV]** *The ACIMOV Methodology: Agile and Continuous Integration for Modular Ontologies and Vocabularies*.
- **[OntoVersions]** *Analysing Multiple Versions of an Ontology*.
- **[GitKM]** *Decentralized Collaborative Knowledge Management Using Git*.
- **[NAry]** *Defining N-ary Relations on the Semantic Web*. W3C Working Group Note.
- **[Widoco]** WIDOCO documentation generator. *(cite if tool mentions are kept)*
- **[LODE]** LODE: Live OWL Documentation Environment. *(cite if kept)*
- **[pyLODE]** pyLODE documentation generator. *(cite if kept)*
- **[Pubby]** Pubby Linked Data front-end. *(cite if kept)*
- **[LodView]** LodView IRI dereferencer. *(cite if kept)*
- **[Walder]** Walder declarative Linked Data server. *(cite if kept)*
- **[Harshp]** Harshp personal web-space Linked Data publication. *(cite if kept)*
