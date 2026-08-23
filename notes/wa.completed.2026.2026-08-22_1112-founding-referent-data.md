---
id: e91c0040-ee44-42d8-b03f-52c0db16dcf3
title: Founding Referent Data
desc: 'Define and implement an optional, locally carried founding RDF slice for a Knop without mixing referent assertions into KnopMetadata or requiring reference-source resolution'
created: 1787422320000
---

## Parent Plan

[[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]] — this task owns Phases 2 and 3. SFLO/framework work may proceed after ruling; Weave runtime implementation waits for the Phase 1 migrated `knop.create` writer.

## Status

Completed 2026-08-23. The capability passed committed-state G2 review and Dave selected the measured singular path at G3; no batch child was owed.

## Goals

- Let `knop.create` accept an optional small RDF document containing assertions about the newly minted Semantic Flow identifier's referent.
- Keep `D/_knop/_meta/meta.ttl` restricted to metadata about the `Knop` support object and keep `D/_knop/_inventory/inventory.ttl` restricted to SF inventory machinery.
- Give client applications a deterministic, local, inventory-discoverable place to read founding referent facts without resolving a `ReferenceSource`, generating pages, or contacting a network resource.
- Support Stagecraft's first-press namespace-registration facts, including a downstream `incarnationOf` predicate linking the new referent IRI to the original referent IRI.
- Keep the founding slice narrow enough that it does not recreate the removed generic `ReferentMetadata` dumping ground.
- Preserve fail-closed initialization: optional founding bytes are validated before writes, never silently overwritten, and applied atomically with the other Knop-create outputs.
- Require founding data to be settled into an ordinary artifact HistoricalState before a Stagecraft press may land, and allow post-publication correction by adding a later state in a new press.
- Prove the contract at the Stagecraft-shaped 552-identifier scale without adding page generation or ambient source retrieval to initialization.

## Summary

The current `knop.create` operation creates only `D/_knop/_meta/meta.ttl` and `D/_knop/_inventory/inventory.ttl`, then registers the new Knop in the current MeshInventory. That is appropriately minimal for SF machinery, but it gives an application no locally carried place for the small amount of referent data known at mint time.

`ReferenceCatalog` does not fill this role. A `ReferenceLink` records the semantic intent and role of a reference, while its `ReferenceSource` identifies separate RDF bytes. The original material may be external or unavailable when a Stagecraft press runs. Current Weave page generation does not fetch remote `targetAccessUrl` sources, and page generation is not part of this initialization use case anyway. Stagecraft needs the founding facts for its own namespace/IRI inventory and client application before any page workflow.

This task introduces an optional Knop-owned `FoundingReferentData` artifact. In a hierarchy-backed serialization it lives at `D/_knop/_founding`, with current Turtle bytes at `D/_knop/_founding/data.ttl`. The Knop inventory discovers it through `hasFoundingReferentData`. The founding document contains assertions about the public referent `D`, never about `D/_knop`.

The first document profile is deliberately flat and base-independent. Every triple has the absolute public referent IRI `D` as its subject. Every named-node term must be an absolute IRI, blank nodes are forbidden in every RDF term position, and objects may be IRIs or literals. This allows direct application assertions such as:

```turtle
@prefix stage: <https://stagecraft.example/vocab/> .

<https://stagecraft.example/waystation/characters/new-npc>
  stage:incarnationOf <https://original.example/characters/42> ;
  stage:origin stage:origin_pressed ;
  stage:byteBearing false ;
  stage:owner <https://stagecraft.example/waystation/campaigns/waystation> ;
  stage:homeGraph <https://stagecraft.example/waystation/graphs/fiction-world> .
```

`incarnationOf` remains downstream vocabulary; this task does not add it to SFLO. SFLO defines the artifact and discovery contract, not Stagecraft's referent predicates.

`knop.create` remains an initialization operation. With founding data it additionally writes the working founding file and registers the artifact/file facts in the Knop inventory. Initialization itself creates no payload, reference catalog, source registry, history, ResourcePage, or network activity. Before a press may land, an explicit support-artifact version operation settles those working bytes into the artifact's first HistoricalState without generating pages. Later corrections update the working founding file and create a later HistoricalState in a new press. Explicit adoption of a prepositioned conventional file is deferred; the first slice accepts bytes in the programmatic request or reads an explicitly supplied source path under normal local-path policy, and the conventional target must not already exist.

## Discussion

### Why Not Arbitrary Referent Triples In KnopMetadata

`KnopMetadata` is currently defined as metadata about a `Knop` as a mesh-managed support object. The public IRI `D` denotes the referent; `D/_knop` denotes the support object. A statement such as:

```turtle
<https://stagecraft.example/waystation/characters/new-npc>
  stage:incarnationOf <https://original.example/characters/42> .
```

is about `D`, not `D/_knop`.

Current Weave shape assertions are open enough that extra triples in `meta.ttl` may parse and the first weave currently snapshots the file bytes. That tolerance is not an endorsed content contract, and generated identifier pages do not treat referent triples in KnopMetadata as a referent fact source. Redefining `KnopMetadata` to contain arbitrary assertions about both `D` and `D/_knop` would undo the machinery/content boundary that motivated removal of `ReferentMetadata` and would couple application-data lifecycle to Knop-metadata lifecycle.

The dedicated founding artifact makes the exception explicit and queryable from inventory while leaving `meta.ttl` single-purpose.

### Why Not A Payload Artifact Or Dataset

This task deliberately narrows the active 2026-04-02 ruling that removed `ReferentMetadata` and directed substantive referent RDF into a payload artifact or separate dataset. That remains the default for broad, mutable, or independently curated description.

Founding referent data is a bounded exception with a different role and lifecycle:

- it is accepted atomically with identifier minting rather than integrated as the Knop's primary payload
- it contains facts about exactly one non-byte-bearing public referent and cannot describe a corpus of subjects
- it is a small versioned founding record needed for local identifier registration and application lookup; each settled state is immutable even though a later state may correct it
- it does not make `D` a `PayloadArtifact`, does not imply `hasPayloadArtifact`, and does not claim that the referent itself carries bytes
- it is not the ongoing authoritative application record; broad or mutable facts still belong in payload data or a separate dataset

A separate shared namespace dataset plus canonical `ReferenceLink` remains valid when that dataset is reliably carried and its lookup/performance costs are acceptable. Stagecraft's new 2026-08-22 requirement is narrower: every newly minted identifier needs a self-contained local founding record for the client application even when the original material is unavailable. This supersedes the 2026-08-21 Stagecraft response's statement that Knop-per-high-volume-identifier guidance was a non-ask for that earlier contract round.

Landing this exception requires an explicit dated amendment in [[ont.dev.decision-log]] and corresponding qualifications in [[ont.summary.core]] and [[ont.reference-links]]. Those notes should continue to reject generic referent-description support artifacts while naming this create-time, single-referent exception.

### Artifact And Serialization Shape

The proposed current serialization is:

```turtle
<characters/new-npc/_knop>
  a sflo:Knop ;
  sflo:hasFoundingReferentData <characters/new-npc/_knop/_founding> .

<characters/new-npc/_knop/_founding>
  a sflo:FoundingReferentData,
    sflo:DigitalArtifact,
    sflo:RdfDocument ;
  sflo:hasWorkingLocatedFile <characters/new-npc/_knop/_founding/data.ttl> .

<characters/new-npc/_knop/_founding/data.ttl>
  a sflo:LocatedFile,
    sflo:RdfDocument .
```

`FoundingReferentData` is an artifact-level class, not a class of the referent and not a facet. In the first slice it is a `DigitalArtifact` and `RdfDocument`, but not a `SemanticFlowResource`: the task deliberately creates no founding-data ResourcePage and should not contradict the live expectation that every `SemanticFlowResource` is accompanied by one. A future page-support task may add that classification together with the corresponding page contract. `hasFoundingReferentData` is an explicit Knop slot with range `FoundingReferentData`; SHACL permits at most one IRI value per Knop at the same Warning severity used by other optional Knop slots.

The mutable working `LocatedFile` does not carry a standing digest. A standing digest on that path would make every legitimate correction look like corruption and would conflict with append-onlyish settled facts. When founding data is versioned, the exact snapshot manifestation and/or immutable snapshot `LocatedFile` carry the digest under the ordinary content-digest contract. Validation verifies settled snapshot digests; a changed working file is a pending version candidate, not a digest mismatch.

A settled state follows the ordinary artifact chain:

```turtle
<characters/new-npc/_knop/_founding>
  sflo:hasArtifactHistory <characters/new-npc/_knop/_founding/_history001> ;
  sflo:defaultArtifactHistory <characters/new-npc/_knop/_founding/_history001> .

<characters/new-npc/_knop/_founding/_history001>
  a sflo:ArtifactHistory ;
  sflo:hasHistoricalState <characters/new-npc/_knop/_founding/_history001/_s0001> .

<characters/new-npc/_knop/_founding/_history001/_s0001>
  a sflo:HistoricalState ;
  sflo:hasManifestation <characters/new-npc/_knop/_founding/_history001/_s0001/ttl> .

<characters/new-npc/_knop/_founding/_history001/_s0001/ttl>
  a sflo:ArtifactManifestation,
    sflo:RdfDocument ;
  sflo:hasContentDigest "sha256:..." ;
  sflo:locatedFileForManifestation
    <characters/new-npc/_knop/_founding/_history001/_s0001/ttl/data.ttl> .
```

The exact placement of mutable latest/next progression follows the append-onlyish inventory/metadata split rather than introducing a founding-specific progression model.

### Founding Document Profile

The portable first-slice profile is intentionally smaller than arbitrary RDF:

- the serialization is Turtle
- the document is self-contained and base-independent: `@base` is forbidden, Weave parses without injecting a base IRI, and every parsed named node must be an absolute IRI after prefix expansion
- the graph is non-empty
- every triple's subject is exactly the public referent IRI formed from `meshBase + designatorPath`
- blank nodes are forbidden as subjects and objects; predicates are IRIs under the RDF data model
- objects may be named IRIs or literals
- quoted triples, RDF-star, named graphs, and generalized RDF are outside the first slice
- `rdf:type` and downstream predicates are allowed, but an `rdf:type` object in the SFLO or SFCFG namespace is forbidden
- every predicate in the SFLO or SFCFG namespace is forbidden; the rule is namespace-based rather than dependent on current RDFS domain declarations
- the first Weave profile accepts at most 64 KiB of source bytes and at most 256 parsed triples per founding document
- founding data is refused for the root designator in the first slice; ordinary root `knop.create` without founding data remains supported while root slashless/slash-terminated subject identity is ruled separately

"Reachable from `D`" is not part of the contract. An earlier design idea would have allowed blank-node structures by recursively following object edges from `D`; this task rejects that complexity. If structured founding data becomes necessary, it requires a later explicit profile revision rather than an implicit graph traversal rule.

The subject restriction is enforced per founding document by the operation/runtime that has file boundaries. It cannot be reliably expressed merely by validating a union of mesh graphs with ordinary SHACL, because the union loses which triples came from `data.ttl`. SFLO SHACL enforces the owner slot and artifact typing; the Semantic Flow behavior spec and Weave enforce document-local content constraints. Weave detects the forbidden `@base` directive from N3 lexer tokens, not a raw substring scan, and compares the parsed subject lexically to the normalized planner value `new URL(designatorPath, meshBase).href`; equivalent but differently spelled IRIs are deliberately refused.

### Lifecycle

Founding referent data records assertions accepted with identifier initialization. It is not automatically the current authoritative description of a mutable application entity.

The first slice uses the ordinary working-versus-settled artifact lifecycle:

- `knop.create` may create one working founding document from explicitly supplied bytes
- an existing working target is never silently overwritten
- a press must version the working founding document into an initial immutable HistoricalState before commit/publication
- changing working founding bytes after a settled state is valid pending work, not corruption
- a correction creates a later HistoricalState and a new press; it never rewrites an already published state or commit
- exact historical state IRIs continue to expose the original and corrected records independently

This lifecycle is deliberately different from a general payload or mutable referent-description dataset. A later reference or payload may coexist with founding data. `FoundingReferentData` does not imply `hasPayloadArtifact`, and it does not replace `ReferenceCatalog`, `ReferenceLink`, `ReferenceSource`, or `ExtractionSource`.

The repair window has two distinct parts:

- **before press landing:** reset-and-replay or replace the uncommitted working founding file, then settle the corrected initial state
- **after press commit/publication:** the landed press is immutable; reset-and-replay is no longer repair. Update working founding data, create a later HistoricalState, and publish a new press that advances the current/latest view while preserving the original exact state

The current public programmatic version API supports exact payload targets only, and candidate loading recognizes only specific support-artifact weave slices. This task therefore adds a narrow FoundingReferentData arm to `weave version` rather than merely declaring the artifact versioned in ontology prose or broadening every version target.

The ruled CLI surface is:

- `weave version <D> --artifact-role founding-referent-data` versions the current working founding bytes into the next HistoricalState without page generation
- `weave version <D> --artifact-role founding-referent-data --source <path>` validates and admission-captures the supplied bytes, plans the working-file update and next HistoricalState together, then commits both as one composed operation

`--source` is valid only with the founding-data artifact role in this slice. The composed path follows ADMIT → LOAD → PLAN-from-overlay → WRITE → RESULT: it copies and validates source bytes before planning, plans against an in-memory overlay, includes both the working-file replacement and historical snapshot in one preflighted plan, and performs no write unless the complete plan is green. A planning refusal leaves the prior working file and history unchanged.

The programmatic equivalent is a narrow `versionFoundingReferentData` bytes-in operation with optional bytes: omitted bytes version the current working file; supplied bytes are admission-copied and update-plus-version atomically. It reuses the version planner/execution primitives but does not widen `versionPayloads`, whose stable contract remains payload-only. No standalone founding-data update command ships in the first slice.

### Initialization Input And Atomicity

The core/programmatic request gains optional founding Turtle bytes as `Uint8Array`, not as a JavaScript string. The core planner validates well-formed UTF-8 and the RDF/profile while retaining the original bytes, then plans the founding artifact, exact working file bytes, and inventory facts together with the ordinary Knop files.

The binary byte path is required end to end:

- the request and resolved request carry `Uint8Array`
- the plan uses `PlannedBinaryFile` for founding bytes
- the runtime reads CLI source bytes with `Deno.readFile` and writes the conventional target with `Deno.writeFile` plus create-new semantics
- the later version step computes SHA-256 over exactly the snapshot bytes written, using one extracted shared digest helper rather than adding a fifth operation-local implementation
- a fatal UTF-8 decode is used only for Turtle parsing; decoding must not replace, normalize, or define the digested bytes

Absent optional input preserves current `knop.create` output byte-for-byte. Supplied bytes are fully preflighted before any write. The conventional founding target, metadata target, and inventory target must all be absent. The runtime provides in-process failure atomicity for both founding and ordinary no-founding create: stage new bytes under the workspace, retain the prior MeshInventory bytes, commit created files with create-new semantics, and replace MeshInventory atomically where supported. On a caught mid-commit failure it removes only files created by this operation, removes newly created directories only when empty, and restores the prior MeshInventory bytes. A rollback failure raises a distinct diagnostic naming safe recovery paths without content. Cross-file crash atomicity is not claimed; validation detects a crash-interrupted surface.

The exact CLI spelling is secondary to the programmatic contract. The first local surface is `weave knop create <D> --founding-data <path>`. The source path resolves from the command working directory, not from `--mesh-root`, and must pass `OperationalLocalPathPolicy` using the existing `workingLocalRelativePath` locator kind, matching integrate's caller-supplied local-source precedent without adding host-policy vocabulary. Path traversal and a disallowed external path fail closed. Source and target absolute paths are resolved and compared before generic source/target existence checks so a collision receives one deterministic diagnostic. No adoption mode ships in the first slice.

The create result reports the founding artifact IRI and working located-file path along with ordinary created and updated paths. The version result reports the new history/state/manifestation/snapshot IRIs and digest. Logs must not echo the founding Turtle contents.

Parser and profile failures use fixed, content-free diagnostics. Runtime and audit logs must not interpolate N3 messages, terms, literals, or source lines; tests use a sentinel string to prove it never reaches either logger.

### Stagecraft Namespace Inventory

Stagecraft can mint a reference-style identifier by providing one flat founding document per `knop.create`. The client discovers the document from the Knop inventory and reads the local bytes directly. It does not need a `ReferenceLink` merely to recover the founding record and does not need page generation.

At Waystation press 100 scale, the acceptance corpus contains approximately 552 new Knops with small founding documents. Initialization must perform no HTTP requests, repository resolution, reference-source resolution, page rendering, history materialization, or source-data copying beyond the explicitly supplied founding bytes. The later press-settlement phase versions the founding artifacts but still performs no page generation or external resolution.

The first task does not redesign singular `knop.create` as a batch API. Before this task's Weave runtime lands, Phase 1 of [[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]] migrates create onto `planInventoryAppend` and removes the repeated per-Knop quad scans. Repeated singular creates will still parse and write a growing MeshInventory, so aggregate byte work may remain quadratic even after the cubic comparison defect is removed. The receipt measures the migrated implementation rather than treating the legacy renderer's cost as permanent.

A small representative multi-create regression runs in ordinary CI. Phase 1 of the parent plan records like-for-like N=552 measurements before and after the `knop.create` append/indexing improvement. The full Stagecraft-shaped founding probe then measures migrated create plus founding initialization and first-state settlement through an explicit scale-test task/environment gate, not ordinary CI. Its Linux receipt records wall-clock time and peak RSS using an external process harness; no unstable timing threshold enters ordinary CI. If the migrated singular workflow is materially costly, carve a batch-initialization/version task before recommending larger presses rather than hiding the result behind a timeout increase.

### Future Page Generation

Future page generation may treat founding data as a locally available fallback fact source when payload, extraction, or canonical reference bytes are unavailable. Future remote `ReferenceSource` resolution needs explicit access policy, bounded retrieval, caching, digest verification, and clear source precedence.

None of that is part of this task. Initialization must not invoke page generation or source resolution, and ordinary `generate` behavior must not change merely because the vocabulary exists.

Ordinary `weave` behavior must nevertheless preserve the founding-data inventory subgraph. Current preservation recognizes only source registries and reference catalogs and would otherwise orphan `data.ttl` on first weave. This task must extend preservation so `hasFoundingReferentData`, artifact/file typing, the working-file relation, history/state/manifestation membership, snapshot relations, and snapshot digests survive every existing weave slice. It does not add a founding-data page or use founding facts as identifier-page input.

## Open Issues

No modeling or first-slice operation-surface issue remains open after the Stagecraft ruling. The following later decisions have explicit triggers rather than blocking ontology work:

- Add explicit retraction/removal only if correction-by-later-state is insufficient; this task does not rewrite or delete an already settled state.
- Add structured RDF only when a real founding record requires it; the first profile forbids blank nodes and named graphs.
- Add batch initialization if the 552-entry receipt shows material cost or before a substantially larger consumer workload.
- Add `SemanticFlowResource` typing, founding-data ResourcePages, or identifier-page fallback only in a page-generation task that rules source precedence.
- Add root-designator founding data only after the root canonical-IRI spelling is settled.

## Decisions

- Do not broaden `KnopMetadata` to carry arbitrary referent assertions.
- Add a distinct optional Knop-owned founding-data artifact and explicit inventory slot.
- Narrow the active 2026-04-02 referent-data ruling explicitly; do not leave live ontology guidance contradictory.
- Use `D/_knop/_founding` and `D/_knop/_founding/data.ttl` as the hierarchy-backed conventional serialization.
- Put application assertions about `D` in the founding document; do not assert `incarnationOf` or other referent predicates on `D/_knop`.
- Keep `incarnationOf` and the Stagecraft classification axes in downstream vocabulary.
- Require exactly `D` as every triple subject in the first profile.
- Forbid `@base`, parse without an injected base, and require every named node to resolve from absolute IRI syntax or an absolute prefix expansion.
- Forbid blank nodes everywhere in the first profile; do not use a reachability rule.
- Allow IRI and literal objects and arbitrary downstream predicates within the bounded profile.
- Accept initial working founding bytes as `Uint8Array`; do not put a standing digest on the mutable working file.
- Require an explicit no-page version step before press landing and use ordinary ArtifactHistory/HistoricalState/ArtifactManifestation structure for initial and corrected settled states.
- Put exact-byte digests on immutable snapshot manifestations/files and verify those settled claims during validation.
- Keep founding data distinct from payload, curated references, source provenance, configuration, and page composition.
- Require explicit optional input and create-new writing; do not implement adoption in the first slice.
- Apply ordinary local-path policy to CLI source input and fixed content-free diagnostics to parser/profile failures.
- Preserve the complete founding artifact, history, state, manifestation, snapshot, and digest subgraph across ordinary weave operations without adding page behavior.
- Refuse founding data for the root designator in the first slice while preserving ordinary root Knop creation.
- Keep create-time attachment rather than adding a later `knop add-founding-data` operation: the founding slice is part of the identifier-minting transaction and the client must not observe an initialized Knop without it.
- Treat reset-and-replay as a pre-landing repair only. After publication, correct founding data through a later HistoricalState and new press; never rewrite the landed state.
- Add a narrow FoundingReferentData arm to `weave version`, with optional `--source` for one composed update-plus-version operation; add equivalent `versionFoundingReferentData` bytes-in behavior and keep `versionPayloads` payload-only.
- Do not add a standalone founding-data update command in the first slice.
- Keep singular `knop.create` in the first slice and make the 552-entry receipt the batch trigger.
- Make initialization perform no network access, reference resolution, history materialization, weaving, generation, or publication.
- Treat future page use, remote retrieval, merging, precedence, retraction, and supersession as later work; correction-by-later-state is in scope.

## Contract Changes

### SFLO Core Ontology

- Add `FoundingReferentData` as an artifact-level subclass of `DigitalArtifact` and `RdfDocument`; do not classify it as `SemanticFlowResource` until its page contract exists.
- Add `hasFoundingReferentData` from `Knop` to `FoundingReferentData` with wording that distinguishes founding assertions about the associated public referent from metadata about the Knop.
- Add the new explicit slot to the core summary and artifact-role guidance.
- Add a dated [[ont.dev.decision-log]] entry that explicitly narrows the active 2026-04-02 `ReferentMetadata` removal decision for this bounded create-time artifact.
- Amend [[ont.summary.core]] and [[ont.reference-links]] so their payload/dataset default remains intact while naming the founding-data exception.
- Do not add Stagecraft predicates such as `incarnationOf`, origin, byte-bearing classification, owner, or graph.
- Do not revive `ReferentMetadata`, `hasReferentMetadata`, or a generic arbitrary-description slot.

### SFLO SHACL

- Allow at most one `hasFoundingReferentData` IRI per Knop and require it to be typed `FoundingReferentData`, at Warning severity matching other optional Knop slots.
- Reuse the existing `DigitalArtifact` maximum-one-working-file shape; add only the founding-artifact/file structural checks not already enforced and require the working file to be typed `LocatedFile` and `RdfDocument` when present.
- Retain open-world/downstream vocabulary support; document-local subject and blank-node restrictions are enforced at operation boundaries rather than pretending a union-graph SHACL target can recover file membership.
- Add positive and negative isolated fixtures for the owner/slot/artifact structural contract.

### Semantic Flow Framework

- Add a behavior spec for optional founding referent data during `knop.create`.
- Define the absolute public-referent subject rule, forbidden-base/no-blank-node rule, exact working-byte preservation, snapshot digest behavior, create-new refusal matrix, local-source policy, pre-landing settlement requirement, post-publication correction flow, no-network/no-page boundary, and coexistence with later payload/reference/source artifacts.
- Update [[sf.spec.2026-04-03-knop-create]] to point to the expanded optional input without turning founding data into a mandatory Knop component.
- Update the glossary/core mental model to distinguish Knop metadata, founding referent data, payload data, reference data, and source provenance.
- Add three carried Accord/fixture transitions rather than relying only on TypeScript tests: founding-created with working bytes and no history, founding-versioned with immutable state 1 and no pages, and founding-corrected with immutable state 1 plus later state 2 and no rewrite of state 1.

### Weave

- Extend core and runtime `knop.create` request/plan/result models with optional `Uint8Array` founding-data input; use `PlannedBinaryFile` and binary runtime writes for the founding file.
- Parse Turtle without an injected base and validate lexer-level forbidden `@base`, absolute named nodes, exact lexical subject, term-kind, root refusal, size, triple-count, forbidden SFLO/SFCFG predicates, and forbidden SFLO/SFCFG `rdf:type` objects before any writes.
- Preserve supplied working bytes exactly; the version path computes canonical SHA-256 over the exact snapshot bytes written using an extracted shared helper.
- Render the optional slot, artifact, working located-file, and RDF-document facts during create without a working-file digest.
- Create the conventional founding file with create-new semantics; refuse any pre-existing founding, metadata, or inventory target.
- Apply `OperationalLocalPathPolicy` to CLI source reads, resolve the source from command working directory, and forbid a source/target identity collision.
- Apply the complete create plan atomically or roll back every mutation on execution failure.
- Keep logs and result descriptions content-free while reporting safe paths, IRIs, counts, and digest; sanitize parser/profile failures before logging.
- Add a FoundingReferentData-specific support-artifact selector to `weave version`; omitted source versions current working bytes, while `--source` plans working update plus version from admission-copied bytes in one operation. Add equivalent `versionFoundingReferentData` programmatic behavior and keep `versionPayloads` payload-only.
- Version initial and corrected founding bytes through ordinary ArtifactHistory/HistoricalState/ArtifactManifestation/LocatedFile structure without generating pages, and append settled facts through the shared inventory planner.
- Add public validation finding `unsettled-founding-referent-data` when publication/press validation sees working bytes with no matching latest settled state. A working change remains valid pending state during ordinary authoring validation.
- Use existing/public `content-digest-mismatch` behavior for corrupted immutable snapshot bytes, not for legitimate working-file changes. Whole-mesh validation checks every registered founding artifact, while target-scoped validation checks only selected designators.
- Extend carried Knop-inventory preservation explicitly for the founding slot and complete artifact/file/history/state/manifestation/snapshot/digest subgraph; current preservation handles only source registries and reference catalogs and would otherwise orphan the file.
- Exercise the same preservation helper through `knop add-reference` so adding a reference cannot orphan founding data.
- Add a `toFoundingReferentDataPath` helper following existing designator support-path helpers.
- Do not add page models, page renderers, remote resolution, reference creation, source registries, or payload facts.
- Update [[wu.cli-reference.knop.create]], version/update CLI guidance, [[wd.codebase-overview]], and relevant API documentation with the contract. Documentation must state the true repair window: reset-and-replay is available only before a press lands; after commit/publication, correction requires a later HistoricalState and new press.

## Testing

In `sflo`:

- Assert the new class hierarchy, slot domain/range, labels/comments, and SHACL cardinality/type constraints.
- Execute positive and negative SHACL fixtures through the pinned PySHACL, JavaScript, and Jena runners and compare normalized receipts.
- Assert that no `ReferentMetadata` compatibility alias or Stagecraft-specific predicate enters core.
- Run `deno task ci`.

In the Semantic Flow Framework:

- Specify examples for no founding data, supplied founding data, malformed Turtle, forbidden `@base`, relative IRI, wrong subject, `D#fragment`, blank-node subject, blank-node object, empty graph, excessive size/triples, pre-existing target, refused root founding data, and an `incarnationOf` IRI object.
- Specify that initialization creates no payload, reference, source, history, or pages and performs no network access.
- Add three Accord manifests and carried fixture transitions. The create transition covers the optional slot, exact working founding file, machinery-only `meta.ttl`, and absence of pages/histories/references/sources. The versioned transition adds immutable state 1, exact snapshot digest, and no pages. The corrected transition preserves state 1 byte-for-byte, adds state 2 from corrected working bytes, advances current/latest selection, and still creates no founding-data page.

In Weave:

- Unit-test core planning with no founding input and prove existing output remains unchanged.
- Unit-test exact working-byte preservation during create and exact snapshot/digest computation during version for ordinary UTF-8, BOM-prefixed, and CRLF Turtle.
- Reject malformed or non-UTF-8 Turtle, lexer-token `@base`, relative IRIs, any subject other than normalized lexical `D`, equivalent differently spelled `D`, `D#fragment`, all blank-node occurrences, quoted/generalized RDF, empty/comment-only graphs, excessive source bytes, excessive triples, every SFLO/SFCFG predicate, SFLO/SFCFG `rdf:type` objects such as `sflo:PayloadArtifact`, and founding input on the root designator.
- Accept language-tagged literals, typed literals, absolute IRI objects, and prefixed names whose prefix expands to an absolute IRI.
- Integration-test create-new input, including no partial writes on preflight failure, rollback after an injected mid-write failure, and no overwrite of existing bytes.
- Test CLI path resolution from command working directory, allowed and denied local-path policy, workspace escape, and source/target collision.
- Assert create carries the optional slot/artifact/working-file facts without a standing working digest and `meta.ttl` remains machinery-only.
- Assert client discovery requires only MeshInventory -> KnopInventory -> `hasFoundingReferentData` -> working located file.
- Stub `globalThis.fetch`, run the initialization tests without network permission, and prove no network path is exercised without widening `ExecuteKnopCreateOptions` merely to test a negative.
- Assert zero page files and zero history files are created by initialization.
- Version initial founding data; assert state 1 snapshot bytes/digest, progression, and inventory facts are correct and no page is generated.
- Update working founding data and version a correction; assert state 1 remains byte-identical, state 2 carries corrected bytes/digest, and the latest/current view advances.
- Assert publication/press validation reports `unsettled-founding-referent-data` before the initial version and after a working correction that has not yet been versioned, while ordinary authoring validation treats the working change as pending rather than corrupt.
- Corrupt an immutable founding snapshot and assert `content-digest-mismatch` follows whole-mesh versus target-scoped selection without network access.
- Round-trip the versioned founding artifact through ordinary `weave`; assert its complete support/history subgraph remains present. Do not require or generate a founding-artifact page.
- Add a create-with-founding → `knop add-reference` regression proving the shared preservation helper retains the complete founding subgraph and exact file.
- Feed a sentinel source token through every parser/profile failure path and assert it appears in neither operational nor audit logs.
- Run a small representative multi-create regression in ordinary CI. Run the full Stagecraft-shaped 552-entry probe through an explicit scale task/environment gate over migrated create, founding initialization, and first-state settlement; verify every snapshot digest and no external reads/page writes. Retain the Phase 1 legacy/migrated no-founding before/after numbers so founding overhead is isolated.
- Run focused create/validation/inventory tests followed by `deno task fmt` and `deno task ci`.

## Non-Goals

- Arbitrary user metadata in `D/_knop/_meta/meta.ttl`.
- Reintroducing generic `ReferentMetadata` or `ReferentDescription`.
- Defining Stagecraft's `incarnationOf` or four-axis namespace vocabulary in SFLO.
- Treating founding data as the primary payload of `D` or inferring `byteBearing=true` from its existence.
- Rewriting or deleting an already settled founding state. Corrections append later states and publish a new press.
- Adopting a prepositioned conventional founding file in the first slice.
- Adding a general inline-RDF field to `ReferenceLink` or `ReferenceSource`.
- Fetching remote `ReferenceSource`, `targetAccessUrl`, repository, or linked-object bytes.
- Founding-data page generation, page-source precedence, fallback rendering, external-reference caching, retraction, deployment, or git operations. Founding-data history/version correction is in scope.
- A mandatory founding artifact on every Knop.
- Founding data on the root Knop before root canonical-IRI spelling is settled.
- A multi-item `knop.create` API in the first slice; the 552-entry receipt is the trigger for a separate batch task if necessary.
- Blank-node reachability or any structured blank-node subgraph allowance.

## Implementation Plan

- [x] Amend the active ontology decision and guidance notes so the bounded exception is explicit before adding vocabulary.
- [x] Add the SFLO ontology and SHACL contract plus executed cross-engine positive/negative fixtures.
- [x] Rule the exact working-update and support-artifact version CLI/API surfaces while retaining ordinary artifact history semantics and the payload-only `versionPayloads` contract: narrow `weave version --artifact-role founding-referent-data [--source]` plus `versionFoundingReferentData`.
- [x] Add the portable behavior spec, update the current `knop.create`/version specs, and add founding-created/founding-versioned/founding-corrected Accord fixture transitions.
- [x] Extract one shared exact-byte SHA-256 helper for immutable founding snapshots rather than adding another operation-local implementation.
- [x] Add failing Weave core tests for binary optional planning, the absolute flat-document profile, exact bytes/digest, root refusal, and unchanged no-input output.
- [x] Implement `Uint8Array` founding-data planning, conventional-path helpers, binary writes, and inventory rendering.
- [x] Implement CLI local-path policy and atomic create plus composed update/version execution with fixed, content-free failure diagnostics.
- [x] Implement FoundingReferentData support-artifact version planning for initial and corrected states without page generation.
- [x] Extend validation with unsettled-working and immutable-snapshot-digest findings and extend inventory preservation with version/correction → weave round trips.
- [x] Add integration, CLI/API, correction, no-network, initialization/version-no-page, rollback, log-redaction, and 552-entry functional coverage plus the separate Linux scale receipt.
- [x] Update [[wu.cli-reference.knop.create]], [[wd.codebase-overview]], API documentation, [[wd.todo]], and [[wd.queues]] when the task is ruled and admitted. The task was executed by direct Phase 2 instruction and never occupied READY, so no queue mutation was owed.
- [x] Run per-repository focused checks, then each affected repository's full CI and SFLO cross-engine conformance comparison.
- [x] Provide one reasonably detailed semantic commit message per affected repository.

## Implementation Receipt

### Committed Slices

- SFLO `cf7e79a5`: ontology, SHACL, decision amendment, summary/reference guidance, guardrails, and 14-case cross-engine suite.
- Semantic Flow Framework `e6d8bdd` plus executable-ASK follow-up `f391813`: portable behavior, payload-only version boundary, glossary, and the created/versioned/corrected Accord sequence.
- Weave `1b2f080` plus backlog disposition `54f7f7c` and final-review hardening `06da19c`: binary initialization, flat profile, append-planned discovery/state facts, atomic create/correction, public CLI/API, validation, preservation, inline Accord replay, documentation, tests, scale probes, fail-closed registered-inventory validation, stable public error mapping, directory-safe rollback, test-seam export hygiene, and source-policy coverage.
- Contract review: [[wa.review.2026-08-23_0007-stagecraft-phase2-contract-claude]] returned GO WITH CHANGES. Its manifest findings were fixed or explicitly ruled; inline replay materialization landed before founding runtime code.
- Final implementation review: [[wa.review.2026-08-23_0939-stagecraft-phase2-final-claude]] records the initial GO, the five bounded hardening fixes, and the follow-up committed-state GO with no blocker or major.

### Gate Receipts

- SFLO full CI: 33 tests and release validation green. PySHACL 0.40.0, shacl-engine 1.1.2, and Apache Jena SHACL 6.2.0 produced identical normalized results across 14 cases at `cf7e79a5`.
- Framework: all three new manifests and the scenario index are Accord-SHACL conformant.
- Executed Accord transitions against an isolated temporary Git fixture: founding-created 16 pass / 0 fail / 0 error; founding-versioned 9/0/0; founding-corrected 11/0/0. The real mesh fixture repository and refs were not modified.
- Weave focused profile/create/version/validation/preservation/CLI/fixture tests green; the final hardening focus passed 19/19.
- Weave full `deno task ci` passed format, lint, check, 887/887 coverage tests, and LCOV generation at `06da19c`.
- `git diff --check` passed before each local commit.

### N=552 Receipt

Commands ran on Linux 7.0.0-29-generic x86_64 with Deno 2.9.2 under `/usr/bin/time -v`. Both tasks intentionally omit `--allow-net`; the founding run also reported zero socket messages.

| Observation | Wall | Probe total | Create loop | Settlement | Peak RSS | Created / updated | MeshInventory read / write | Final inventory |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Phase 1 suffix-proof comparator | 2.06 s | 1,982.976 ms | 1,919.791 ms | — | 222,432 KiB | 1,104 / 552 | not instrumented | 220,817 B |
| Phase 2 current no-founding | 2.59 s | 2,463.535 ms | 2,392.355 ms | — | 230,216 KiB | 1,104 / 552 | 61,145,040 / 61,364,736 B | 220,817 B |
| Phase 2 founding + state 1 | 4.62 s | 4,507.340 ms | 2,141.542 ms | 2,300.591 ms | 246,144 KiB | 2,208 / 1,104 | 61,145,040 / 61,364,736 B | 220,817 B |

The current founding workflow added 2.03 s wall and 15,928 KiB peak RSS over the same-session no-founding observation while verifying 552/552 snapshot digests and creating zero founding pages. The create-loop difference runs in the opposite direction and is ordinary single-observation noise; founding overhead is therefore represented by total and settlement time, not by subtracting the two create-loop numbers.

The Phase 2 task adds the required atomic create path and byte accounting, and these runs did not pin the Phase 1 `DENO_DIR`; the 2.06 s historical row is retained but is not treated as an exact same-environment ratio. Semantic comparison instrumentation is not retained. Singular create still parses and physically replaces a growing MeshInventory, so aggregate byte work remains quadratic.

### Gate Disposition

- G2 passed after [[wa.review.2026-08-23_0939-stagecraft-phase2-final-claude]] returned final committed-state GO with no blocker or major.
- G3 is not ruled. Stagecraft supplied no wall-clock or peak-memory budget, and one descriptive N=552 receipt does not justify inventing a batch task. Dave must select singular accepted or batch required against the consumer budget.

## Deep Review R0 — Disposition

[[wa.review.2026-08-22_1129-founding-referent-data-claude]] reviewed the first draft read-only with Claude Opus at maximum effort and returned NO-GO. The review's four blockers and every major finding were folded as follows.

### Blocking

- **B1 accepted.** The profile is now base-independent: `@base` is forbidden, parsing injects no base, every named node must be absolute after prefix expansion, and the example uses the absolute public `D`.
- **B2 accepted.** Inventory preservation is now an explicit deliverable with a create → weave round-trip proving the full founding subgraph survives and the file remains byte-identical.
- **B3 accepted.** The request/plan/runtime contract now carries `Uint8Array` through `PlannedBinaryFile` and binary writes; BOM and CRLF receipts prove exact bytes and digests.
- **B4 accepted.** The task now explicitly narrows the active 2026-04-02 decision, explains why the slice is not a payload, and requires updates to the ontology decision log, core summary, and reference guidance.

### Major

- **M1 accepted with a first-consumer recovery ruling.** The surface remains create-only; Stagecraft resets and replays its disposable press worktree after an accepted-data mistake. Non-disposable adoption requires a future correction/removal operation.
- **M2 accepted.** Adoption, bounds, vocabulary, and `SemanticFlowResource` status are resolved rather than left contradictory: no adoption, 64 KiB/256 triples, the proposed names, and `SemanticFlowResource` with acknowledged deferred page support.
- **M3 accepted.** Active mesh validation must verify the founding file's standing digest.
- **M4 accepted.** CLI source paths resolve from command CWD under `OperationalLocalPathPolicy`; programmatic callers provide bytes.
- **M5 accepted.** The task now requires an Accord manifest and carried fixture transition.
- **M6 accepted.** Ordinary CI gets a functional 552-entry smoke; a separate Linux receipt records wall-clock and peak RSS without an unstable gate.
- **M7 accepted.** Parser/profile diagnostics are fixed and content-free, with a sentinel leak test over both loggers.
- **M8 accepted.** Founding input is refused for the root designator in slice one; ordinary root Knop creation is unchanged.

### Advisory And Nits

- Create-time attachment is retained instead of a later add command because the client must not observe a successfully initialized Knop missing its founding slice.
- The SHACL work reuses the global working-file cardinality shape and assigns the new optional slot Warning severity.
- The task states the expected quadratic singular-create shape, requires atomic rollback beyond preflight, acknowledges temporary `SemanticFlowResource` page incompleteness, corrects CI commands, extracts a shared digest helper, reconciles the new Stagecraft ask with the prior non-ask, adds the path helper, expands negative tests, and names backlog/queue follow-through.

The post-r0 disposition draft proceeded to the second read-only review below; implementation remained unstarted.

## Deep Review R1 — Disposition

[[wa.review.2026-08-22_1148-founding-referent-data-claude-r1]] verified all four r0 blockers closed and returned GO WITH CHANGES. Its five remaining majors and advisories were folded without redesign:

- **M1 accepted.** The profile now rejects every SFLO/SFCFG predicate and every SFLO/SFCFG `rdf:type` object, with explicit negative tests for payload typing and working-file predicates.
- **M2 accepted by removing the conflict.** `FoundingReferentData` is only `DigitalArtifact` plus `RdfDocument` in slice one; `SemanticFlowResource` typing waits for a page contract. This supersedes r0's temporary-incompleteness disposition.
- **M3 accepted and made explicit.** Hand edits produce a validation-blocking `content-digest-mismatch`; reset-and-replay is the only first-slice repair and the warning is required ship-blocking CLI documentation.
- **M4 accepted.** Accord coverage is a carried create/woven pair, and the woven sibling proves preservation and no founding page/history.
- **M5 accepted.** The task now predicts cubic comparison work and quadratic byte work. Ordinary CI uses a small representative regression; the full 552-entry run is an explicit scale task with a Linux receipt.
- **Advisories accepted.** The task names the public finding code and validation selection, uses no-network permission plus a `globalThis.fetch` stub, defines in-process rollback and its crash boundary, adds `knop add-reference` preservation coverage, requires lexer-token `@base` detection and lexical subject comparison, defines collision-check precedence, and reuses the existing `workingLocalRelativePath` policy kind.

The resulting task has no open blocker and is ready for human ruling/queue admission. Implementation remains unstarted.

## Stagecraft Feedback R2 — Disposition

Stagecraft identified two gaps after r1:

- **Repair window accepted.** "Disposable worktree" was too broad. Reset-and-replay ends when the press is committed/published. The lifecycle now requires initial settlement before landing and correction through a later HistoricalState/new press afterward.
- **Unmeasured cost accepted.** The parent plan and append-onlyish task now require an unchanged N=552 baseline before the `knop.create` migration and an identical after run. The founding receipt runs after migration and retains all three comparisons: legacy create, migrated create, and migrated create plus founding initialization/settlement.

This feedback supersedes the r0/r1 create-only/digest-on-working-file dispositions. Mutable working founding data has no standing digest; immutable snapshots do. The current artifact model supports that history, but Weave's public version path is payload-only, so R2 reopened the exact FoundingReferentData update/version surface as an implementation-blocking ruling; R3 below resolves it.

## Dave Ruling R3 — Narrow Version Surface

Dave accepted the narrow update surface on `weave version`:

- no standalone founding-data update command in slice one
- explicit `--artifact-role founding-referent-data`
- no `--source` means version the current working founding file
- `--source <path>` means admission-copy/validate bytes and commit working update plus the next state as one composed plan
- programmatic `versionFoundingReferentData` provides equivalent optional-bytes behavior
- `versionPayloads` remains payload-only

This ruling closes the R2 operation-surface blocker. Implementation remains sequenced after the parent plan's Phase 1 `knop.create` append migration and before the founding-data scale receipt.
