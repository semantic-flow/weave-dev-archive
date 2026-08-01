---
id: stagecraft-driven-sf-20260630
title: 2026 06 30 1108 Stagecraft Driven Semantic Flow Requirements
desc: ''
created: 1782842880000
---

## Goals

- Capture concrete Stagecraft persistence needs before changing Weave or SFLO vocabulary for roleplaying-data use cases.
- Use Stagecraft as a real downstream consumer to prioritize Semantic Flow and Weave engineering work.
- Keep the existing publication/runtime correctness backlog honest while allowing application-driven needs to reorder slices when they expose real blockers.

## Summary

Stagecraft development has begun in earnest, and Semantic Flow is expected to serve as the persisted roleplaying-data solution. This makes Stagecraft the first active application-shaped consumer after the ontology/publication fixtures. The near-term engineering posture should be consumer-driven but cautious: collect concrete workflows, data shapes, mutation patterns, query needs, and publication/inspection expectations before adding vocabulary or CLI/runtime behavior.

The paper sprint is parked for now. The FOMI/FOIS drafts are still useful as conceptual scratch, but they should not drive implementation. Stagecraft should drive by producing real use cases: campaign/world data, characters, scenes, logs, generated artifacts, references, revisions, provenance, and inspection requirements.

## Discussion

Useful pressure from Stagecraft likely differs from ontology publication in several ways:

- application data may change frequently and need practical current-state updates without noisy historical-state churn
- some resources may need exact citation, audit trails, or release-like snapshots, while others may only need current persistence
- generated or assistant-produced content may need provenance, revision, and attribution records
- roleplaying data may include nested or highly connected resources where identifier minting, ResourcePages, and references need to stay ergonomic
- local-first authoring and application runtime persistence may matter more than static public publication at first
- private campaign data raises a stronger boundary between inspectable local mesh state and anything suitable for publication

Stagecraft should not automatically force broad new ontology layers. First check whether existing Semantic Flow concepts already fit:

- `SemanticMesh` for the persisted namespace or workspace
- `Knop` for stable identifiers and support surfaces
- `DigitalArtifact` for files, generated documents, transcripts, bundles, maps, notes, and data exports
- `ArtifactHistory` and `HistoricalState` for snapshots worth citing or auditing
- current-only support artifacts for operational metadata and frequently changing data
- `ReferenceCatalog` and source registries for curated references and provenance
- ResourcePages for local inspection and future static publication

## Open Issues

- Which Stagecraft resources need stable public-style identifiers, and which only need internal IDs?
- What data should be RDF-native versus serialized application payloads carried as `DigitalArtifact`s?
- Which changes need append-onlyish history, and which should remain current-only?
- Does Stagecraft need fast local query/index support beyond the current file-oriented runtime?
- What privacy/publication boundary is required for campaigns, sessions, player data, generated text, and notes?
- Should Stagecraft use whole-repo, sidecar, or application-managed mesh topology for persisted data?

## Decisions

- Treat Stagecraft as a near-term consumer that may reprioritize Weave/Semantic Flow work.
- Do not add Stagecraft-specific vocabulary or runtime features until a concrete workflow demonstrates the need.
- Keep append-onlyish inventory, history-policy coherence, source provenance, and ResourcePage inspection as likely high-value shared substrate.

## Contract Changes

- None yet. This is a requirements-capture task.

## Testing

- No tests required until this produces implementation slices.
- Future slices should include small Stagecraft-shaped fixtures when they introduce reusable behavior.

## Non-Goals

- Do not design a full roleplaying ontology here.
- Do not make Stagecraft private application requirements part of portable Semantic Flow semantics without checking generality.
- Do not let paper-draft terminology override live ontology/framework wording.
- Do not prioritize RDFa/JSON-LD embedding or polished publication features over persistence correctness unless Stagecraft directly needs them.

## Accord Sequencing (2026-07-04)

Stagecraft's fixture-ladder work surfaced a concrete Accord backlog before it surfaced a Weave blocker. The Accord work is sequenced in the `accord` repo notes as follows:

1. `ac.completed.2026.2026-07-04-real-sparql-ask` — natural absence proofs (`FILTER NOT EXISTS`) and boolean/numeric literal ergonomics; may expose an ASK syntax preflight for step 2.
2. `ac.task.2026.2026-04-03-shacl-validation` — separate `accord validate` command executing the shipped SHACL-SPARQL shapes; gates the vocabulary added in later steps. The `shacl-engine` Deno spike can start in parallel with step 1.
3. `ac.completed.2026.2026-07-04-scenario-runner` — `accord check-scenario` with per-step evidence grouping; the biggest Stagecraft workflow win. Parallelizable with step 4 (disjoint modules).
4. `ac.completed.2026.2026-07-04-json-assertions` — JSON assertion vocabulary including first-class absence proofs; wants `accord validate` in place so its new shapes are enforced from day one.
5. `ac.completed.2026.2026-07-04-draft-manifest` — conservative manifest scaffolding from git diff; pure ergonomics, lowest risk, last.

Deferred ideas (profile packs, immutability assertion packs, drift checks, the runner-neutral HTTP Testing Vocab) remain in `ac.product-ideas.runner-neutral-test-spec` with an ownership map. This note stays the Stagecraft-side driver: as rungs land, feed new blockers here and re-check whether any are Weave-general rather than Accord-local.

Status 2026-07-05: Accord steps 1 and 2 have landed (parser-backed ASK profile; standalone `accord validate`). The first Weave-general blocker predicted by this note now exists and is owned by [[wa.task.2026.2026-07-03_1332-stagecraft-weave-planner-generalization]]: the settled second-payload weave shape assertion blocks later-ordinal payload advancement for Stagecraft's temporal rung. That epic runs as its own Weave-side track, parallel to Accord steps 3-5, and takes priority over them when effort is serial because it gates the rung itself rather than ergonomics.

Update later 2026-07-05: Accord steps 3 and 4 and the planner epic's single-target slices have landed, with the real temporal-rung replay verified. A second application workload is now planned: a game+session history mesh, where the application service serializes game and session state to the mesh together at service/user request and requests the weave for the batch. That workload drives [[wa.completed.2026.2026-07-05-multi-target-payload-advancement]], with the boundary decision that atomicity belongs to the application: Weave owes fail-closed whole-plan validation, deterministic merged rendering of shared support artifacts, and safe re-runnability, not transactions or locking.

## Implementation Plan

- [x] Inventory the first Stagecraft persisted-data workflows and identify which ones are mesh, artifact, history, reference, source, or page problems. DONE 2026-08-01: the eleven evidence-backed requirements below classify the exercised workflows (checkpoint batches, later-ordinal advancement, in-process recording, validation, extracted-term lifecycle, transition evidence, domain invariants).
- [ ] Decide the initial Stagecraft mesh topology: whole-repo, sidecar, branch-published, or application-managed local mesh.
- [x] Map a few representative resources to existing Semantic Flow concepts before proposing vocabulary changes. DONE 2026-08-01: the ownership map below ties every evidenced requirement to existing tasks/concepts; no vocabulary change is authorized by current evidence.
- [x] Identify the first Weave blocker that prevents Stagecraft from persisting or inspecting useful roleplaying data. DONE: the extracted-term lifecycle at ~1,700-term nested-source scale ([[wa.task.2026.2026-07-21_1603-extractor-defect-pair]]) — first implementation bite in flight.
- [x] If a blocker is general, create a focused Weave task note and implementation slice. DONE: the blocker was already noted; bite 1 (nested extraction sources without root Knops) carved and fired 2026-08-01. Residual: the binary-payload item in wd.todo still needs its own task note cut.

## Evidence-Backed Stagecraft Persistence Requirements (2026-08-01)

Classification rule: “stated” means directly reported by the Stagecraft consumer or recorded from an exercised Stagecraft fixture. “Inferred” means a Weave generalization from that evidence. Neither category authorizes Stagecraft-specific vocabulary without another concrete workflow.

### Stated Or Directly Evidenced

1. **Checkpoint batch persistence.** A Stagecraft service serializes game and session payloads together at a service/user checkpoint and requests one explicit multi-target weave. Weave must validate the entire plan and captured payload snapshot before writes, order targets deterministically, merge shared support artifacts once, and converge safely on rerun. The application owns single-writer serialization and transactional business semantics; Weave does not promise filesystem transactions. Evidence: this note’s Accord Sequencing update and [[wa.completed.2026.2026-07-05-multi-target-payload-advancement]]. Status: landed for the evidenced shape.

2. **Later-ordinal history advancement with current-only support.** Existing payload histories must advance beyond `_s0002` while coherent current-only KnopInventory/KnopMetadata support remains valid. Prior historical payload files remain byte-identical; the operation adds the requested new state and advances only the owned inventory/progression/page outputs. Evidence: [[wa.task.2026.2026-07-03_1332-stagecraft-weave-planner-generalization]] and [[ac.product-ideas.runner-neutral-test-spec]] temporal addition 4. Status: the Stagecraft temporal-rung case landed; broader inventory semantics remain with [[wa.task.2026.2026-05-17-append-onlyish-inventory]].

3. **In-process payload persistence for product paths.** Product code must be able to record caller-supplied payload bytes through a stable API without spawning the CLI or requiring caller-managed temporary files. The mesh remains file-backed. Evidence: [[wa.task.2026.2026-07-21_1322-programmatic-version-api]]. Status: landed for UTF-8 text/RDF payloads.

4. **Exact-byte binary persistence.** ***RECLASSIFIED 2026-08-01 (Dave's challenge: "was there a stagecraft use I missed?" — no).*** This does not belong under "stated or directly evidenced" and has been demoted; it is kept here with the honest trail rather than renumbered. The claim "the press moment includes non-text artifacts" is an Open Issue line written by the Stagecraft flagship PM seat in [[wa.task.2026.2026-07-21_1322-programmatic-version-api]], in the same note that explicitly deferred press-side consumption to the Stagecraft press-completeness lane. **Neither consumer feedback round (2026-07-28, 2026-07-29) mentions binary payloads at all** — the only "binary" in those rounds is the compiled weave CLI, a different sense entirely. What IS evidenced is a Weave-side code defect found by our own r1 review, not a consumer ask: later payload advancement decodes `currentPayloadTurtle` through a text `PlannedFile` while the loader retains `currentPayloadBytes`, plus later renderers hard-code `sflo:RdfDocument` onto binary payloads. v1 refuses binary at API admission, so the live exposure is narrow: an existing binary working payload advanced through file-backed CLI `version`. Status: [[wa.task.2026.2026-08-01_1411-binary-payload-advancement]] is cut and PARKED — its trigger is a real binary payload hitting later advancement, or the Stagecraft press lane actually landing, whichever comes first.

5. **Non-mutating plan inspection.** A caller must be able to forecast payload outcomes and created/updated paths through the normal admit/load/plan checks while writing nothing. Evidence: [[wd.consumer-feedback.0.5.1.2026-07-28_0849]] §2. Status: delivered as `versionPayloads({ dryRun: true })` under [[wd.consumer-feedback.0.5.1]].

6. **Actionable writer-coordination and recovery contract.** Callers need a usable means to satisfy the single-writer precondition and a defined recovery path after partial writes. Stagecraft explicitly accepted documented advisory locking; the later game/session service design centralizes writing in the application. Conservative recovery is restore from VCS/snapshot, verify, validate, and retry; programmatic errors distinguish completed creates from completed updates. Evidence: [[wd.consumer-feedback.0.5.1.2026-07-28_0849]] §§3–4, [[wd.consumer-feedback.0.5.1]], and [[wa.completed.2026.2026-07-05-multi-target-payload-advancement]]. Status: documented; no Weave lock/transaction mechanism is currently required.

7. **Structured read-only validation.** Stagecraft needs non-mutating mesh validation through both the supported CLI path and a stable programmatic result carrying severity, code, message, and path/designator attribution. V1 coverage is planner/preflight plus publication readiness, not exhaustive per-file integrity. Evidence: [[wd.consumer-feedback.0.5.1.2026-07-28_0849]] §8 and [[wa.completed.2026.2026-07-29_1219-programmatic-validate-mesh-api]]. Status: released in v0.6.0.

8. **Whole-mesh validation at the observed pending-heavy scale.** Untargeted validation must complete for the reported roughly 6,900-file / 1,700-pending-term corpus; targeted-only validation is not the intended pattern. The broader product intent is the roughly `10^4`-file range. Evidence: [[wd.consumer-feedback.0.5.1.2026-07-29_1213]] §3 and [[wa.completed.2026.2026-07-29_1220-whole-mesh-validate-bounded-memory]]. Status: the reported OOM was reproduced and fixed at roughly 554 MiB peak; the exact broader `10^4` and versioned-policy residuals are not yet separately accepted.

9. **Viable extracted-term publication lifecycle at corpus scale.** Extraction remains non-publication-bearing. The supported extract→weave→generate sequence must work for Stagecraft’s roughly 1,700-term nested-source mesh so weave owns governed `sflo:hasResourcePage` materialization and the downstream synthesis workaround can retire. Evidence: [[wd.consumer-feedback.0.5.1.2026-07-29_1213]] §2, [[wa.task.2026.2026-07-21_1603-extractor-defect-pair]], and [[wd.consumer-feedback.0.5.1.reply]]. Status: active as the first Kim queue item.

10. **Reproducible transition evidence.** Each persistence rung needs grouped path expectations and semantic assertions, explicit proof that old `_history*/_s*/...` payloads remain unchanged, and clear reporting of unexpected drift. Evidence: [[ac.product-ideas.runner-neutral-test-spec]] temporal additions 1, 2, 4, and 5. Status: the triggering rung is covered; broader Weave adoption maps to the Accord-integration `wd.todo` item and [[wa.task.2026.2026-05-16_1625-manifest-completeness-check]].

11. **Application-domain persisted-artifact invariants.** Exercised Stagecraft contracts include resolvable evidence pointers, absence of participant-aim leakage, conditional `recommendedActantIntent`, and negative branches with no committed events, state mutations, or authority-produced resources. Evidence: [[ac.product-ideas.runner-neutral-test-spec]] and [[ac.completed.2026.2026-07-04-json-assertions]]. Ownership: Stagecraft behavior and fixtures, with Accord as proof tooling; no Weave vocabulary/runtime task should be created unless one of these exposes a reusable substrate gap.

### Inferred Shared Requirements

- Stagecraft payloads require stable designator identity and addressable HistoricalState segments because the exercised workflows target exact designators and exact later ordinals. This does not prove that all roleplaying resources need public identifiers.
- The exercised byte-stability and rerun requirements support the general append/no-op/fail-on-conflict inventory rule in [[wa.task.2026.2026-05-17-append-onlyish-inventory]].
- Stagecraft must be able to persist and validate locally without publishing. Its current refusal to run `weave publish` supports that operation boundary, but does not yet settle privacy classes, mesh topology, public-ID policy, or later publication mechanics.

### Still Unevidenced — Do Not Promote To Requirements

- A campaign/world/character/scene ontology or any other Stagecraft-specific vocabulary.
- Which application structures should be RDF-native versus serialized payloads.
- A whole-repo, sidecar, branch-published, or application-managed mesh topology decision.
- Fast local query/index support beyond current validation and file-backed operation.
- A detailed privacy/publication classification for campaign, session, player, generated-text, and note data.
- Grouped-provenance or citable-batch vocabulary.
- Weave-level cross-target semantic validation; the application currently owns referential coherence.
- A requirement that extraction itself emit ResourcePage claims.

Release tags, machine-readable executable version information, changelog quality, and package-version relationships are real consumer asks, but they are delivery/packaging requirements rather than Stagecraft persistence requirements.
