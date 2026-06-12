# Weave: Filesystem-Native Semantic Mesh Publication for Dereferenceable Ontology Resources

Draft target: FOIS 2026 Demonstrations short paper, 5-9 CEUR-ART pages including bibliography.

Paired-paper strategy:

- This paper: "Weave: Filesystem-Native Semantic Mesh Publication for Dereferenceable Ontology Resources." Focus on create/integrate/weave/extract/generate/validate workflow, generated ResourcePages, static hosting, source registries, path policy, and screenshots or live demo.
- Sibling FOMI paper: "Semantic Flow: Byte-Grounded Digital Artifact Histories for Static RDF Publication." Focus on the DigitalArtifact model, FRIR/WEMI contrast, artifact histories, historical states, manifestations, located files, coordinates, sparse/skip-level properties, major ontology revisions, and why Git is necessary but not sufficient as public ontology-publication evidence.
- Do not submit the same paper twice with different emphasis. The FOMI paper is the model/practice paper; this paper is the implementation/demo paper.

Production note: draft here in Markdown or move directly into `.tex`, then assemble the submission PDF in CEUR-ART LaTeX 1-column style. Use Overleaf or install a local TeX toolchain; LibreOffice/ODT is fallback only. The final PDF must include the mandatory generative-AI declaration required by current CEUR templates.

Author metadata:

- Author: TODO
- Affiliation: Spectacular Voyage LLC
- ORCID: 0000-0002-4959-6058
- Email: dave@s
- Presentation preference: onsite

## Abstract pre-draft

Ontology projects often need more than a stable ontology file: they need dereferenceable resource identifiers, human-readable pages, versioned artifacts, provenance for source bytes, and publication workflows that work with ordinary repositories and static hosting. Weave is a filesystem-oriented reference implementation of the Semantic Flow model that turns repository files into an inspectable semantic mesh. In a Weave mesh, public identifiers are backed by Knops, governed digital artifacts, explicit artifact histories, source bindings, and generated ResourcePages. This demonstration shows how Weave can create a publication mesh from ontology and SHACL source files, version payload artifacts into named release states, extract ontology terms into first-class dereferenceable identifiers, record source and extraction provenance, and generate static ResourcePages exposing RDF facts, histories, references, and Semantic Flow metadata. The demo emphasizes a practical authoring workflow: source files remain in natural repository locations, generated publication output remains static-host friendly, and operational path trust is explicit rather than leaked into public RDF. The result is a reproducible workflow for publishing ontology resources as stable, inspectable Web identifiers while preserving a clear boundary between authored source, semantic artifact history, and generated documentation.

## 1. Introduction pre-draft

Ontology publication is usually described as a problem of serializing RDF and serving stable IRIs. In practice, maintainers also face a messier workflow problem. The ontology source may live in a repository with SHACL shapes, examples, tests, release scripts, notes, and generated pages. Public users need dereferenceable identifiers and readable documentation. Downstream systems need stable bytes, version references, and provenance. Maintainers need a publication process that can be replayed without silently copying local machine paths, overwriting unrelated generated files, or turning every source update into an ad hoc website rebuild.

Weave addresses this workflow as a reference implementation of Semantic Flow. Semantic Flow models a public identifier as part of a `SemanticMesh`: a governed namespace region with supporting resources. Each public identifier is anchored by a `Knop`, and the content associated with an identifier is represented as one or more `DigitalArtifact`s with optional `ArtifactHistory`, `HistoricalState`, `ArtifactManifestation`, and `LocatedFile` structure. Weave implements this model against a local filesystem and produces static ResourcePages suitable for ordinary static hosting.

This paper presents Weave as a demonstration of filesystem-native semantic publication. The demonstration starts from ordinary ontology source files, integrates them into a mesh, records release states, extracts ontology terms as dereferenceable identifiers, and generates HTML pages that explain both the domain facts and the Semantic Flow support structure. The goal is not to replace ontology editing tools or RDF stores. The goal is to make the publication surface reproducible, inspectable, and explicit about the relation between authored source, generated pages, artifact history, and provenance.

## 2. Motivation

The motivating use case is an ontology project that wants to publish both source-level artifacts and term-level pages. A conventional repository layout might contain:

- an ontology source file
- SHACL shapes
- example datasets
- release metadata
- generated documentation or website material
- continuous integration scripts

The public site, however, should present a stable semantic surface:

- dereferenceable IRIs for the ontology, SHACL artifacts, examples, and extracted terms
- pages for humans to inspect classes, properties, histories, references, and source information
- release-state bytes that can be cited independently from current working files
- provenance that says where integrated or imported source bytes came from
- generated output that does not expose a maintainer's local checkout path

Weave treats these as publication invariants rather than as incidental website features. This is especially important for static hosting, where a generated site may need to carry enough semantic evidence to be useful without a live backend.

## 3. Semantic Flow Concepts Used In The Demo

The demonstration uses the following Semantic Flow concepts.

`SemanticMesh`: the governed namespace and support surface. In a filesystem-backed mesh this is usually a root directory plus `_mesh` support artifacts.

`Knop`: the mesh-managed support object and naming anchor for a public Semantic Flow identifier. For identifier `D`, the corresponding support surface is conventionally `D/_knop`.

`DigitalArtifact`: an artifact governed by the mesh, such as an ontology file, SHACL file, imported Markdown file, reference catalog, source registry, or page definition.

`ArtifactHistory` and `HistoricalState`: explicit lineage resources for versioned artifact states. Release publication can use named histories such as `releases` and named states such as `v0.2.0`.

`ArtifactResolutionSpec`: a generic relator used by source bindings, imports, extraction sources, reference sources, and page sources to describe requested source coordinates and resolution policy.

`ResourcePage`: a generated HTML page for a Semantic Flow resource. ResourcePages combine document metadata, RDF-derived panels, history/source panels, references, and optional authored regions.

These concepts allow Weave to represent a publication as a graph of identifiers, artifacts, histories, source bindings, and generated pages rather than as a collection of unrelated files.

## 4. Demonstration Workflow

The core demonstration is a branch or sidecar ontology publication workflow.

Step 1: create the mesh.

Weave creates the mesh support surface, records the mesh base IRI, initializes mesh metadata and inventory, and applies a publication profile such as GitHub Pages when requested.

Step 2: integrate ontology source artifacts.

The maintainer integrates an ontology file, SHACL file, or example dataset from its ordinary source location. Integration leaves the source bytes where they are and records a governed payload artifact plus source binding metadata. This is distinct from import, where bytes are intentionally copied into a governed working file.

Step 3: weave release states.

The maintainer selects a release history and state, then runs the composed weave operation. Weave validates the selected targets, versions payload bytes into historical states, and generates current and historical ResourcePages. For release artifacts, the demo uses explicit segments such as `releases/v0.2.0/ttl` rather than relying only on ordinal state names.

Step 4: extract ontology terms.

Once source artifacts are woven, Weave can extract mesh-scoped RDF terms mentioned in the source graph. Each extracted term receives its own Knop support surface and can become a dereferenceable identifier with a generated page. Extraction provenance records which source artifact grounded the extracted identifier.

Step 5: generate ResourcePages.

Generated pages expose the identifier, RDF facts, children, references, raw source panels, artifact histories, and Semantic Flow metadata. Custom page definitions can add authored Markdown regions when a resource needs human-curated explanation in addition to generated panels.

Step 6: validate publication output.

Publication validation checks static-host controls and path leakage concerns. The public RDF should not contain host-local paths such as `/home/...`, temporary checkout roots, or machine-specific file URLs.

## 5. What The Audience Will See

The live demo should show:

- a normal source repository layout containing ontology, SHACL, and example files
- a generated mesh/publication root containing `_mesh`, identifier paths, artifact histories, source registries, and ResourcePages
- a ResourcePage for an ontology artifact showing title, current facts, release history, source information, and generated Semantic Flow metadata
- a ResourcePage for an extracted term showing RDF facts from the source artifact and extraction-source provenance
- a source registry Turtle file showing source coordinates without leaking the local checkout path
- a repeated validation or generation command to show the workflow is replayable

If time permits, the Alice Bio fixture can be used as a compact vignette for custom identifier pages: authored Markdown, imported content, references, and generated panels appear together in the same ResourcePage shell.

## 6. Implementation Status

Weave is currently a Deno CLI/runtime implementation. The daemon and browser application surfaces are future-facing and are not the focus of this demonstration.

Implemented demo-relevant capabilities include:

- mesh creation with portable mesh configuration and publication profile support
- integration and import boundaries for source bytes
- payload versioning with explicit artifact histories and historical states
- extraction of mesh-scoped RDF terms into Knop-managed identifiers
- source registries for integration, import, and extraction provenance
- ResourcePage generation for identifiers, payloads, support artifacts, histories, states, and mesh support resources
- custom page definitions with authored Markdown regions and generated-panel selection
- runtime artifact resolution for working, latest-state, exact-state, and bounded page-source fallback cases
- config resolution across application defaults, mesh config, inherited Knop config, Knop-local config, and command overrides

Current limitations:

- Weave is pre-1.0 and the on-disk vocabulary is still being refined.
- The daemon and web client are not yet the mature user-facing surface.
- Remote current-byte fetching is intentionally not a default runtime behavior.
- Append-only inventory semantics and some publication link-policy cleanup remain active work; the demo should avoid overstating full release automation idempotence.

## 7. Significance For Applied Ontology

Semantic Flow contributes a concrete publication model for RDF projects that want stable identifiers and inspectable artifacts without adopting a heavyweight server-side publishing platform. The model keeps source files in natural repository layouts while producing a public semantic mesh that carries generated pages, versioned artifact states, and provenance. Weave contributes the filesystem-native reference implementation used in this demonstration.

The most important design choice is separating topology from semantics. A sidecar mesh, branch-published site, whole-repository mesh, or separate source/publication worktree can use the same Semantic Flow artifact model. The operational topology affects configuration and provenance; it does not require a different semantic model.

This separation is useful for ontology maintainers because repository organization tends to evolve. Source trees change, publication branches are rebuilt, and static hosts impose their own conventions. The semantic publication surface should still present stable identifiers, source relationships, and historical states.

## 8. Demo Artifacts

Candidate public/demo artifacts:

- Weave repository: https://github.com/semantic-flow/weave
- Semantic Flow ontology repository: https://github.com/semantic-flow/sflo
- generated SFLO publication mesh: https://semantic-flow.github.io/sflo/
- SFLO ontology resource page: https://semantic-flow.github.io/sflo/ontology
- SFLO config ontology resource page: https://semantic-flow.github.io/sflo/config
- Fantasy Rules sidecar mesh: https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/
- Fantasy Rules branch-published mesh: https://semantic-flow.github.io/mesh-branch-fantasy-rules/
- Alice Bio fixture mesh: TODO public URL or local demo path
- demo video, if SEMANTiCS variant is attempted: TODO URL

## 9. Submission Variant Notes

FOIS Demonstrations:

- Best fit for the separate Weave implementation demo paper, not for the DigitalArtifact model paper.
- Emphasize tools and methods for creating, maintaining, integrating, publishing, evaluating, and implementing ontologies.
- No video requirement found in the call, though a demo script and screenshots are useful.

SEMANTiCS Posters and Demos:

- 5 pages including references.
- Demo video expected for demos.
- In-person presentation required.
- Must not be a duplicate of another submitted paper.

FOMI:

- Probably the best fit for the DigitalArtifact model.
- Needs a distinct practice/lessons framing, e.g., "Byte-grounded digital artifact histories for repository-native ontology publication."
- Emphasize practical ontology engineering problems: major version revisions, deprecation and replacement, import closure changes, generated documentation, source/provenance review, static RDF publication, and IDE/LLM-assisted authoring.
- Keep Weave as implementation evidence, not the headline contribution.

Separate Weave demo paper:

- Best fit for FOIS Demonstrations or a later demo venue.
- Should be narrower and more operational than the FOMI paper.
- Show the complete workflow and generated site surfaces rather than re-arguing the DigitalArtifact model in depth.

FOIS Ontology Showcase:

- Better as an SFLO ontology paper than a Weave tool paper.
- Do not attempt unless the SFLO ontology narrative is independently coherent and fast to write.

## References To Add

- Semantic Flow ontology and documentation, public URL.
- Weave repository and release/version URL.
- FOIS 2026 Demonstrations call.
- CEUR-ART style and CEUR GenAI policy.
- W3C Linked Data or Cool URIs note, if useful.
- GitHub Pages or static hosting reference only if needed.
- RDF, OWL, SHACL specifications as needed.
