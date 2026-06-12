# Semantic Meshes: Stable Identifiers, Immutable Traces, and Dynamic Sense-Finding

## Abstract

Public RDF identifiers have to survive in a world where meanings move. A class IRI, a dataset IRI, a ResourcePage IRI, a release-state IRI, and a Turtle-file IRI are all identifiers, but they respectively name a vocabulary term, a continuing artifact, a page presenting a resource, a settled historical state, and concrete retrievable content. Every identifier has a sense in use; only some identifiers identify DigitalArtifacts. This paper introduces the Semantic Mesh: a static-host-friendly support structure for an identifier surface, defined by the Semantic Flow framework and implemented by Weave. A mesh separates the identified resource from the artifacts that help agents find its sense: ResourcePages, ReferenceCatalogs, source registries, inventories, metadata artifacts, and, when the identified resource is a byte-grounded digital artifact, a layered DigitalArtifact model. DCAT 3 already provides general versioning and dataset-series vocabulary; Semantic Flow profiles that space for static RDF publication by adding named artifact histories, settled historical states, manifestations, located files, resolution coordinates, and explicit operational pointers such as latest-within-a-history and default write-target history. ResourcePages make the distinction between a resource and the returned HTML page visible in RDF without requiring 303 redirects or content negotiation. The result is a framework for stable identifiers, immutable evidence traces, and dynamic sense-finding on commodity static hosting.

**Keywords:** Semantic Web, RDF publication, persistent identifiers, digital artifacts, DCAT, ontology versioning, static sites, dereferenceability

## 1. Introduction

Publishing RDF under public IRIs creates a long-lived obligation. Those IRIs are copied into datasets, cited in papers, embedded in SHACL shapes and SPARQL queries, cached by tools, and eventually encountered by people and agents far removed from the original publishing context. The publisher is not merely promising that a URL will return bytes. The publisher is also helping future readers work out what the identifier is being used to mean.

That obligation is difficult because stability and change are both real. Identifiers need enough persistence for citation, reuse, validation, and caching. At the same time, the senses attached to identifiers are socially maintained and evolve through documentation, practice, revision, deprecation, and dispute. A class IRI may be used more narrowly after a vocabulary revision. A dataset identifier may continue across releases while each release has different bytes. A term identifier may name a domain concept, while the generated page returned from the same path is only a page about that concept. Treating all of these as one kind of "resource" makes publication easy to start and hard to maintain.

Existing practice distributes the needed evidence across systems that do not share a model. Git records commits and file paths, but not public identifier-level publication states. DCAT describes datasets, distributions, version relationships, and dataset series, but does not by itself define a static publication grammar that connects named artifact lineages, settled states, manifestations, located byte files, and presentation pages. Cool URI practice and the httpRange-14 debate explain important distinctions between identifiers and returned documents, but many of the canonical solutions assume server behavior such as 303 redirects or content negotiation. Ontology documentation generators produce useful pages, but those pages are often implicit outputs of build tools rather than first-class RDF resources.

Semantic Flow addresses the missing layer. Its central construct is the Semantic Mesh: a governed identifier surface and the static, inspectable support artifacts that make the surface usable over time. A mesh does not freeze meaning. Instead, it keeps the evidence for sense-finding durable: reference data, source records, generated pages, artifact states, manifestations, retrievable files, digests, and resolution observations. This paper argues that the mesh is the right unit for long-lived RDF publication because it separates three concerns that are often conflated: the identifier and its sense, the thing identified, and the artifacts returned or consulted to understand that thing.

The paper makes four contributions. First, it clarifies the distinction between identifiers that have senses, identifiers that identify DigitalArtifacts, and support artifacts that help establish those senses. Second, it presents the Semantic Flow DigitalArtifact model as a DCAT-compatible refinement for byte-grounded artifact publication histories. Third, it describes the ResourcePage pattern for static dereferenceability without custom server behavior. Fourth, it positions the model relative to DCAT 3, FRIR/WEMI, Cool URIs, static RDF publication tools, repository-native ontology engineering, and the structure-versus-concept distinction.

## 2. Identifiers, Senses, Referents, and Support

The first distinction is simple but easy to lose: every public identifier has a sense in use, but not every identifier identifies a DigitalArtifact. A term IRI such as `sflo:DigitalArtifact` identifies a class in an ontology. It has a sense established by labels, comments, axioms, examples, use in downstream data, documentation, and revision history. The ontology document that defines the term may itself be a DigitalArtifact. The generated HTML page for the term may be a ResourcePage. The Turtle file containing the ontology may be a LocatedFile. These are related, but they are not the same resource.

Semantic Flow uses the term sense-finding for the activity of reconstructing what an identifier is being used to mean from the evidence available around it. Sense-finding is not the same as deciding what is true. It is closer to scholarly interpretation and software maintenance: following published evidence, checking what changed, locating the authoritative sources, distinguishing stable statements from mutable pointers, and deciding how much specificity a task requires.

A Semantic Mesh supports sense-finding by maintaining artifacts around identifiers. Some identifiers name things outside the digital-artifact model: people, places, abstract concepts, classes, properties, organizations, topics. These identifiers still benefit from mesh support. A ReferenceCatalog can identify curated RDF sources about the referent. A KnopSourceRegistry can record where those sources came from. KnopInventory and KnopMetadata artifacts can describe the mesh-managed support structure itself. A ResourcePageDefinition can define the page generated for human and machine presentation. None of these artifacts is the referent; each is evidence about how the mesh supports the referent's sense.

When the identifier does name a digital artifact, the DigitalArtifact model becomes relevant. A DigitalArtifact is something whose identity can be usefully grounded in byte-bearing manifestations over time: an ontology document, a dataset, a SHACL shapes file, a generated page, a Markdown source file, a PDF, or a bundle. The model does not claim that every concept is a DigitalArtifact. It says that some resources have a byte-grounded artifact identity, and those resources need more structure than "latest file at this URL."

This separation lets Semantic Flow avoid two mistakes. The first is reducing meaning to bytes. The meaning of an ontology class is not identical to the bytes of the Turtle file that currently describes it. The second is floating meaning away from bytes entirely. For digital artifacts, the bytes matter: they are what was validated, cited, rendered, copied, hashed, mirrored, and diffed. Semantic Flow keeps both sides visible.

## 3. Semantic Meshes

A Semantic Mesh is a governed namespace region together with the support artifacts needed to publish, inspect, and maintain its identifier surface. The surface may include term IRIs, ontology IRIs, artifact IRIs, release IRIs, page IRIs, file IRIs, and internal support IRIs. The mesh is not the same thing as a website, although it can be statically hosted as one. It is a structured publication object whose output is ordinary files and ordinary RDF.

The mesh is the important unit because it gives identifier support operational properties that individual RDF files do not. A mesh can be inspected as a set of RDF artifacts and byte artifacts. It can be versioned in Git without making Git the public semantic model. It can be generated locally and served from a directory. It can be moved from one hosting provider to another because the published semantics do not depend on server-specific behavior. It can also be archived or mirrored, because settled states and located files have explicit coordinates.

Within a mesh, a Knop is the per-identifier support object. A Knop is not the thing identified. It is the mesh-managed support anchor for the identifier: the place where reference catalogs, source registries, metadata, inventories, page definitions, and optional payload artifacts can live. This is useful precisely because the referent may be a DigitalArtifact, a domain concept, or something outside the mesh entirely. A mesh can support a person IRI and an ontology-release IRI with the same publication machinery while keeping their referents conceptually distinct.

The point is not to invent another catalog layer for its own sake. The point is to make sense-support explicit in RDF. A ReferenceCatalog says which sources are canonical, supplemental, or deprecated for a given identifier. A source registry says which upstream bytes were extracted, imported, or integrated. Metadata artifacts and inventories tell agents what support artifacts exist and how they are arranged. These support artifacts can themselves have histories when that matters. If the mesh's claim about a source changes, that change can become a citable state rather than an overwritten fact.

This is the sense in which Semantic Meshes provide immutable traces. The trace is not only provenance metadata. The data artifacts themselves are part of the evidence: historical states, manifestations, located files, reference documents, page definitions, inventories, metadata, and resolution observations. A digest-bearing Turtle file from a release is evidence. So is a deprecated reference link preserved in an older catalog state. The model is designed so that evolving sense can be investigated against a trail of settled artifacts.

## 4. Static Dereferenceability and ResourcePages

The Semantic Web inherited a hard question: when an IRI is dereferenced, is the returned document the thing identified, a description of the thing identified, or one representation among several? The httpRange-14 discussion and the Cool URIs guidance gave practical answers, including hash IRIs, 303 redirects, and content negotiation. Those answers remain important, but they sit awkwardly with the most durable hosting available to small projects, research groups, and long-lived open vocabularies: static hosting.

Commodity static hosts are good at serving files. They are not good at project-specific 303 logic, custom content negotiation, or dynamically generated Linked Data front-ends. The behavior we can usually rely on is simpler: if a directory-like path is requested, the host returns that directory's `index.html`; if an extension-bearing file is requested, the host returns that file. Semantic Flow treats this not as a limitation to hide, but as a mechanical fact to model.

A ResourcePage is a modeled located file whose role is to present another resource. When an extensionless or slash-terminated mesh IRI is requested and the host returns `index.html`, the returned page can declare, using embedded RDFa or JSON-LD, that it is a `sflo:ResourcePage` and that the intended resource `sflo:hasResourcePage` this page. The page can link to Turtle, JSON-LD, or other manifestations, but it does not pretend to be those manifestations. The distinction between the identified resource and the page returned to present it is made in RDF, not hidden in HTTP status codes.

This pattern is about the relation between content, presentation, and sense-support. It is not primarily about the DigitalArtifact stack. ResourcePages are orthogonal: a DigitalArtifact can have a ResourcePage, a HistoricalState can have a ResourcePage, a term IRI can have a ResourcePage, and a ResourcePage can itself be treated as a DigitalArtifact if the page's own history matters. Ordinary LocatedFiles usually do not need ResourcePages, because their identifiers already identify concrete retrievable content. A Turtle file IRI identifies the Turtle file; a term IRI may have a page about the term.

The file-extension convention remains a convention. Extension-bearing IRIs generally identify concrete content. Extensionless or directory-like IRIs generally identify higher-level resources. But the RDF assertions are what make the distinction explicit. This gives static hosting a clean publication story: let the host return files, and let the files say what role they play.

## 5. The DigitalArtifact Model

When a resource is a digital artifact, Semantic Flow provides a small facet model for representing artifact identity at the specificity required by a task. The model is DCAT-compatible: `DigitalArtifact` subclasses `dcat:Dataset`, and `ArtifactManifestation` subclasses `dcat:Distribution`. The added value is not versioning in the abstract, which DCAT 3 already supports, but an expected publication grammar around artifacts: named histories, settled states, manifestations, located files, skip-level shortcuts, and resolution coordinates.

`DigitalArtifact` is the artifact-level continuity handle. It identifies the artifact considered across time and representation. For an ontology publication, this might be "the SFLO core ontology" as a continuing digital artifact. It is intentionally not a full theory of works, expressions, or concepts. Whether the artifact realizes an intellectual work or expresses an abstract model is a domain modeling decision outside the core DigitalArtifact stack.

`ArtifactHistory` is a lineage facet within a DigitalArtifact. One artifact can have more than one history: releases, drafts, archived predecessor states, curation states, source extraction states. This is an important difference from treating a series as only an external collection. The history is inside the artifact's identity, because releases and drafts are different ways the same artifact develops. A `defaultArtifactHistory` can identify the lineage that ordinary write operations should extend when no lineage is explicitly chosen.

`HistoricalState` is a settled state within a history. It is the natural target for exact citation: the ontology as released in version 0.2.0, the generated page as published on a given date, the catalog before a source migration. Settled states are quasi-immutable. The practical commitment is not that storage can never fail, but that a settled coordinate is never reused for different content.

`ArtifactManifestation` is a representation variant of an artifact or state: Turtle, JSON-LD, HTML, Markdown, PDF, zipped bundle, canonicalized RDF, or another representation whose bytes can be served. DCAT's `dcat:Distribution` can express the broad dataset/distribution relationship; Semantic Flow narrows that relationship for artifact publication by placing manifestations inside histories and states when that specificity matters.

`LocatedFile` is the byte-level retrieval resource. It is where access URLs, file paths, and content digests become explicit. A located file can be served, hashed, cached, mirrored, and compared. Located files are also the place where Semantic Flow's byte-grounded discipline becomes strongest: if a located file is part of a settled state, the identifier should continue to identify the same bytes.

The full chain is artifact -> history -> state -> manifestation -> located file. It should be used when every layer carries evidence. The model also supports sparseness. A small support artifact can link directly from artifact to located file. A state can link directly to a file. A working file can be identified by a local path or working access URL. These are shortcuts, not a competing ontology: they are coordinates of convenience for cases where the omitted layers would be ceremonial.

Resolution coordinates make the expected operations explicit. Exact references target a state, manifestation, located file, repository commit, or digest. Latest references resolve within a named ArtifactHistory, such as "latest release state" or "latest draft state." Working references target mutable authoring surfaces and should not be mistaken for settled publication evidence. Default history references identify where new states should be written when no history is explicitly selected. Semantic Flow should avoid treating "current" as a primitive coordinate, because current is contextual: current release, current working file, and current published draft are different policies over different histories.

## 6. Change, Revision, and Major Ontology Releases

The DigitalArtifact model is motivated by ordinary publication, but it becomes especially useful when an ontology changes in a way that affects downstream users. A major ontology revision is not just a repository diff. It is a public contract change. It may affect term senses, validation shapes, examples, documentation, importers, migrations, and applications that rely on older assumptions.

Git is excellent at preserving development history, but repository history is organized around commits and files. Consumers of a public RDF identifier usually need a different set of coordinates: which artifact is this, which lineage is relevant, which settled state was cited, which manifestation was consumed, which located file supplied the bytes, and which digest was observed. Semantic Flow exposes those coordinates in RDF so that diff tools, migration tools, compatibility notes, and human reviews have public places to attach their results.

This does not replace semantic diffing or migration frameworks. It gives them targets. A compatibility report can point at two HistoricalStates. A migration guide can be linked from the release history. A build observation can record which resolution spec was used and which digest was actually observed. A deprecated source can remain visible in an older ReferenceCatalog state. The mesh preserves the publication trail even when the conceptual interpretation continues to develop.

The same principle applies to metadata. Reference catalogs, source registries, inventories, page definitions, and metadata files can themselves be DigitalArtifacts when their evolution matters. There is no infinite-regress requirement: model the layer when it carries useful evidence, and skip it when it does not. But the option matters. A mature mesh can answer questions such as "what did this identifier's page claim in the last release?", "which source was canonical before the migration?", and "which bytes did this generated page come from?"

## 7. Worked Example: The SFLO Ontology Surface

The Semantic Flow core ontology is a useful example because it is both the vocabulary being described and an artifact published through the model. The opening Turtle stanza already exposes several coordinates:

```turtle
<https://semantic-flow.github.io/sflo/ontology> a owl:Ontology ;
  dcterms:title "Semantic Flow Core Ontology" ;
  dcat:hasVersion <https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0> ;
  owl:versionIRI <https://raw.githubusercontent.com/semantic-flow/sflo/refs/tags/v0.2.0/semantic-flow-core-ontology.ttl> ;
  owl:versionInfo "0.2.0" .

<https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0>
  a owl:Ontology, sflo:HistoricalState ;
  dcat:isVersionOf <https://semantic-flow.github.io/sflo/ontology> ;
  sflo:hasManifestation <https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl> ;
  schema:contentUrl <https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl> .

<https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl>
  dcat:downloadURL <https://semantic-flow.github.io/sflo/ontology/releases/v0.2.0/ttl/semantic-flow-core-ontology.ttl> .
```

The Semantic Flow reading is layered but sparse. The ontology IRI is the continuing ontology resource and can be modeled as a DigitalArtifact. The release IRI is a HistoricalState, also typed as `owl:Ontology` for OWL tooling. The `/ttl` IRI is the Turtle manifestation. The `.ttl` IRI is the located file returned as concrete content. The `owl:versionIRI` supplies a tag-pinned repository coordinate, giving the release a second byte anchor outside the static site.

A fuller mesh surface would add an ArtifactHistory for releases, a `defaultArtifactHistory` pointer from the ontology artifact to that history, `latestHistoricalState` on the history, digests on the located file, and ResourcePages for the ontology, release state, and possibly the manifestation. Those ResourcePages would not replace the artifact coordinates. They would present them.

The same surface can also support ontology term IRIs. A term such as `sflo:LocatedFile` has a sense established by the ontology state, examples, labels, comments, and future revisions. It is not itself the ontology artifact. Its ResourcePage can present its definition and links to relevant source artifacts. Its sense-finding trail depends on the ontology's DigitalArtifact history, but the term's referent is the class, not the Turtle file, the HTML page, or the ontology as a whole. This is the conceptual separation the mesh is designed to preserve.

## 8. Relation to Prior Work

DCAT 3 provides the most important base vocabulary. Its broad treatment of datasets, distributions, resource versioning, and dataset series is useful, and Semantic Flow should reuse it rather than compete with it. DCAT's `dcat:hasVersion`, `dcat:previousVersion`, and `dcat:hasCurrentVersion` already support resource life-cycle versioning; `dcat:DatasetSeries` supports collections of separately published datasets that share characteristics. Semantic Flow's contribution is a profile-like refinement for static artifact publication. `DigitalArtifact` specializes `dcat:Dataset`; `ArtifactManifestation` specializes `dcat:Distribution`; `locatedFileForManifestation` specializes the download relationship when the downloadable file is also modeled. Semantic Flow adds `ArtifactHistory`, `HistoricalState`, `LocatedFile`, ResourcePages, skip-level links, and resolution specs so that a dataset/distribution/version description can become an artifact publication surface with named internal lineages and byte-facing coordinates.

FRIR and WEMI are important contrasts. They distinguish work-like, expression-like, manifestation-like, and item-like concerns around information resources. Semantic Flow agrees that content, representation, bytes, and retrieval locations should not be collapsed. But Semantic Flow is WEMI-aware rather than WEMI-shaped. It does not try to decide whether a digital artifact is an expression of a more abstract work. It leaves that to the publisher's domain model. Its distinctive contribution is the temporal publication layer: histories and states for byte-grounded artifacts.

Cool URIs, Linked Data practice, and the httpRange-14 discussion frame the dereferenceability problem. Semantic Flow's answer is deliberately pragmatic. It does not require a custom server to perform a metaphysical distinction. Instead, it uses static files and RDF assertions: the returned `index.html` is a ResourcePage, and the page says which resource it presents.

Existing RDF publication tools demonstrate the demand for usable publication surfaces. Pubby, LodView, LODE, pyLODE, Widoco, Walder, Harshp, and Jekyll RDF all help publish or present RDF in different ways. Semantic Flow's distinction is not that it generates nicer pages. It models the page, the represented resource, the byte files, and their publication coordinates as RDF resources that survive the generator.

Repository-native ontology engineering and CI/CD methods show that RDF authoring can live comfortably in Git and IDEs. In an era of LLM-assisted authoring, this matters more, not less: plain files with predictable layouts are a strong substrate for review, validation, generation, and repair. Semantic Flow does not replace Git, Protege, diff tools, or migration tools. It supplies the public artifact coordinates those tools can consume and produce.

Finally, the structure-versus-concept distinction helps locate the proposal. Semantic Flow is mostly structural scaffolding for sense-finding. It does not claim to own the concepts expressed in a domain ontology. DigitalArtifacts have structure. Concepts have evolving social and logical roles. A mesh keeps the structural evidence durable so that concept work can remain inspectable rather than implicit.

## 9. Implementation Evidence: Weave

Weave is the filesystem-native reference implementation of Semantic Flow. It materializes meshes as directory trees, settles new HistoricalStates into selected ArtifactHistories, writes or regenerates support artifacts, builds ResourcePages from page definitions, records source and resolution observations, and validates mesh structure with SHACL. The result is a static site that can be served by any host capable of returning files.

For this paper, Weave is evidence that the model is operational. It also illustrates why the model is practical. A mesh can be inspected locally before publication. Changes to identifier support are ordinary file diffs. Release states can be reviewed in pull requests. Static output can be mirrored. Authoring can happen in an IDE with RDF validation and LLM assistance, while publication remains grounded in explicit RDF and byte artifacts.

## 10. Limitations

Semantic Flow does not solve every problem in identifier governance. It does not provide a theory of intellectual works. It does not decide when two artifacts express the same concept or when an ontology term's meaning has changed enough to require a new IRI. It does not replace DCAT catalogs, ontology diffing, migration tooling, SHACL validation, or domain modeling. It does not reproduce every dynamic-server feature; in particular, it trades header-driven content negotiation for explicit, link-followable static coordinates.

The model also adds vocabulary. That cost is only justified when the publication surface is important enough to need durable evidence. Sparse modeling and skip-level properties reduce the ceremony, but adopters still need to understand the main distinction: identifier sense-support is general, while the DigitalArtifact stack is for resources with byte-grounded artifact identity.

## 11. Conclusion

Public RDF identifiers need stable support, not frozen meanings. Their senses evolve through use, documentation, revision, and disagreement. Semantic Flow responds by making the evidence for sense-finding durable and inspectable. A Semantic Mesh separates identifiers and referents from the pages, catalogs, registries, metadata artifacts, states, manifestations, and files that support them. For identifiers that name digital artifacts, the DigitalArtifact model supplies a DCAT-compatible navigation grammar from artifact to history, state, manifestation, and located bytes. For identifiers that name concepts or other non-artifact resources, mesh support artifacts still provide a coherent trail.

The central discipline is to put the publication structure in RDF and files that static hosts can serve. ResourcePages make the returned page/resource distinction explicit. Artifact histories and historical states make revision boundaries citable. Located files and digests make byte evidence inspectable. Reference catalogs and source registries keep sense-support visible. In that combination, Semantic Meshes offer a way to build a quasi-immutable semantic web that still leaves room for dynamic sense-finding.

## Declaration on Generative AI

During the preparation of this work, the author(s) used LLM-based writing and coding assistants for drafting, critique, and editing support, working from author-supplied outlines, terminology, source material, and ontology files. All content was reviewed and edited by the author(s), who take responsibility for the publication's content. Final wording should follow venue requirements.

## References

- **[LinkedData]** T. Berners-Lee. *Linked Data*. W3C Design Issues note.
- **[CoolURIs]** L. Sauermann and R. Cyganiak, editors. *Cool URIs for the Semantic Web*. W3C Interest Group Note.
- **[Tennison2012]** J. Tennison. *Using "Punning" to Answer httpRange-14*. Blog post.
- **[DCAT3]** W3C. *Data Catalog Vocabulary (DCAT) - Version 3*. W3C Recommendation.
- **[FRIR]** *Functional Requirements for Information Resource Provenance on the Web*.
- **[OpenWEMI]** *Works, Expressions, Manifestations, Items: An Ontology*.
- **[CagleShannon2026]** K. Cagle and C. Shannon. *Structure vs. Concept*.
- **[FAIRVocab]** *Best Practices for Implementing FAIR Vocabularies and Ontologies on the Web*.
- **[Recipes]** *Best Practice Recipes for Publishing RDF Vocabularies*. W3C Working Group Note.
- **[JekyllRDF]** *Jekyll RDF: Template-Based Linked Data Publication with Minimized Effort and Maximum Scalability*.
- **[ACIMOV]** *The ACIMOV Methodology: Agile and Continuous Integration for Modular Ontologies and Vocabularies*.
- **[OntoVersions]** *Analysing Multiple Versions of an Ontology*.
- **[GitKM]** *Decentralized Collaborative Knowledge Management Using Git*.
- **[NAry]** *Defining N-ary Relations on the Semantic Web*. W3C Working Group Note.
- **[Widoco]** Widoco ontology documentation generator.
- **[LODE]** LODE: Live OWL Documentation Environment.
- **[pyLODE]** pyLODE documentation generator.
- **[Pubby]** Pubby Linked Data front-end.
- **[LodView]** LodView IRI dereferencer.
- **[Walder]** Walder declarative Linked Data server.
- **[Harshp]** Harshp personal web-space Linked Data publication.
