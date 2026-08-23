---
id: 8ac0d151-25a6-4e41-ad4c-58ac66b3e070
title: Stagecraft IRI Initialization
desc: 'Sequence additive Knop creation, optional FoundingReferentData, a 552-entry press receipt, and any evidence-triggered batch path'
created: 1787439044000
---

## Status

Active coordination plan. This plan never enters [[wd.queues]]; its fireable child tasks do.

## Goals

- Give Stagecraft a Semantic Flow-conformant initialization path for approximately 552 public IRIs in a Waystation press.
- Measure the unchanged singular `knop.create` path before Phase 1 and rerun the identical probe afterward so the append/indexing improvement has a real before/after receipt.
- Migrate `knop.create` onto the shared append/no-op/conflict inventory primitive before adding another Knop-owned artifact to that operation.
- Add optional, locally carried `FoundingReferentData` without inventing a competing inventory writer or broadening KnopMetadata.
- Make settled founding data correctable after publication by creating a later ordinary artifact state, never by rewriting an already published state.
- Measure the real singular-create path at press scale after both changes land.
- Add a batch initializer only when the receipt or a larger committed workload demonstrates that singular create is not acceptable.

## Summary

This plan coordinates an existing correctness migration and a new identifier-initialization capability.

The current `knop.create` implementation validates and reconstructs the whole MeshInventory for every new Knop. The shared `planInventoryAppend` primitive already exists, and [[wa.task.2026.2026-05-17-append-onlyish-inventory]] names later `knop.create` as a remaining whole-document writer. [[wa.task.2026.2026-08-22_1112-founding-referent-data]] adds a third created file and a new inventory subgraph. Implementing the founding artifact on the legacy writer would duplicate inventory-mutation logic and make the later migration harder.

The delivery sequence is therefore:

```text
plan convention
      ↓
baseline 552 singular knop.create
      ↓
append-onlyish knop.create bite
      ↓
identical 552 post-change probe
      ↓
FoundingReferentData contract + runtime
      ↓
552-entry founding-data press receipt
      ↓
batch create task only if evidence requires it
```

SFLO ontology and framework-spec work for founding data may proceed after its contract is ruled while the Weave runtime waits for the `knop.create` append migration.

## Child Tasks

| Phase | Owning artifact | Role |
| --- | --- | --- |
| 0 | [[wd.plans-and-tasks]] | Establish the plan genre, queue boundary, and closure rules used here. |
| 1 | [[wa.task.2026.2026-05-17-append-onlyish-inventory]] | Capture the pre-change 552-create baseline, migrate the later/current `knop.create` MeshInventory path onto `planInventoryAppend`, remove repeated membership scans, and rerun the identical probe. |
| 2 | [[wa.task.2026.2026-08-22_1112-founding-referent-data]] | Add the optional founding artifact and its ordinary support-artifact update/version correction lifecycle across SFLO, the framework contract, Accord, and Weave. |
| 3 | [[wa.task.2026.2026-08-22_1112-founding-referent-data]] | Produce the opt-in 552-entry functional/performance receipt after Phase 2. |
| 4 | Task cut only if Gate G3 selects it | Parse/validate once, plan many Knops, and commit one MeshInventory append for a true batch initialization path. |

Phase 1 is a bounded bite within the broader append-onlyish task. This plan does not require that every remaining inventory writer migrate before founding-data work begins.

## Sequence

### Phase 0 — Convention

Create [[wd.plans-and-tasks]], [[template.plan]], schema support for `wa.plan.*` / `ont.plan.*` / `sf.plan.*`, and this first plan. Record that plans never enter READY and that child tasks retain independent closure.

### Phase 1 — Additive `knop.create`

Before production edits, add one reusable opt-in probe and run it against the unchanged implementation at N=552. Record the Weave commit/tree state, workload parameters, wall-clock time, peak RSS, final MeshInventory bytes, aggregate bytes read/written when observable, and successful Knop count in the archive timing record. The probe itself may be added to make the measurement reproducible; no production create code changes before the baseline receipt is captured.

Move the non-legacy `knop.create` MeshInventory growth path from subject-block reconstruction to `planInventoryAppend`:

- request the settled `_mesh hasKnop D/_knop` fact
- request the new Knop type and current KnopInventory-file link
- request the new KnopInventory `LocatedFile` / `RdfDocument` types
- no-op semantically identical requested facts
- fail before writes on single-valued conflicts
- preserve all existing inventory bytes as an unchanged prefix and preserve unknown facts

The current shape assertion also loops over every existing Knop and rescans all quads for each membership check. Phase 1 builds membership/type indexes from the one parsed graph so a singular create is linear in current inventory size rather than quadratic in quad comparisons. Repeating N singular creates may still produce quadratic aggregate parse/write bytes; this phase does not claim batch complexity.

`planInventoryAppend` currently returns a complete output string whose old bytes are a stable prefix. Phase 1 guarantees semantic append-onlyish behavior and byte preservation; it does not require an OS-level append syscall. The runtime may atomically replace the complete file.

Required Phase 1 evidence:

- checked-in reusable scale probe plus a pre-change N=552 receipt
- fail-on-old test preserving an unknown predicate/comment byte-for-byte
- exact semantic duplicate no-op
- named single-valued conflict refusal
- no `replaceSubjectBlock` / `upsertSubjectBlockAfter` use in the migrated `knop.create` path
- indexed membership test or instrumentation proving one membership pass rather than one quad scan per existing Knop
- existing no-founding `knop.create` fixture and root behavior remain green
- an after run using the identical N=552 probe, host, runtime flags, and workload, with an explicit before/after table and explanation of any non-comparable environmental change

### Phase 2 — Founding Referent Data

Execute [[wa.task.2026.2026-08-22_1112-founding-referent-data]]. Its Weave inventory rendering must request settled facts through the migrated append planner rather than reintroducing subject-block rendering.

Initialization creates the validated working founding document. Before a press may land, `weave version <D> --artifact-role founding-referent-data` settles it into its first HistoricalState without page generation. A later correction uses the same narrow version arm with `--source <path>` to plan the working update and next state together, then lands a new press; no already published state is rewritten. Programmatic `versionFoundingReferentData` provides equivalent optional-bytes behavior, while `versionPayloads` remains payload-only.

The SFLO and Semantic Flow Framework portions may be reviewed or implemented in parallel with Phase 1 after the founding-data contract is ruled. The Weave runtime merge waits for Phase 1 or rebases onto it before landing.

### Phase 3 — Press Receipt

Run the task's explicit 552-entry scale probe through real `executeKnopCreate` calls with small valid founding documents. Record:

- wall-clock time
- peak RSS
- MeshInventory bytes read and written
- semantic comparison counts if instrumentation is retained
- created file counts and digest verification results
- proof of no network access and no initialization page/history output

Compare the receipt with a no-founding 552-create baseline so the artifact cost is distinguishable from singular MeshInventory growth.

The no-founding comparator is the post-Phase-1 measurement, not the legacy pre-change measurement. Keep all three numbers: legacy create, migrated create, and migrated create with founding data.

### Phase 4 — Conditional Batch Path

Do not pre-create this task. If Gate G3 selects batching, cut a new `wa.task.*` note whose first slice:

- accepts a multi-item create request
- validates designator uniqueness within the request and against current inventory
- parses and validates MeshInventory once
- plans every Knop support/founding file before writes
- requests all settled MeshInventory facts in one append plan
- commits the batch failure-atomically
- reports per-item and aggregate results without page generation

The batch task must reuse the Phase 1 append semantics and Phase 2 founding-document validator rather than fork either implementation.

## Gates

### G0 — Convention Ready

Passes when the plan convention, template, schemas, and queue boundary are documented and this note conforms to them.

### G1 — `knop.create` Append Bite Landed

Passes when the reproducible pre-change baseline, migrated implementation, identical post-change receipt, and Phase 1 correctness evidence are recorded and green on `main`. This gates the Weave implementation portion of founding data but not SFLO/framework contract work.

Current status: **OPEN after review.** [[wa.review.2026-08-22_1711-stagecraft-phase1-claude]] found one blocking compact-renderer plan/output mismatch and four major disposition items. The working implementation and receipts do not pass G1 until the blocker is fixed, majors are fixed or explicitly owned as residuals, and follow-up gates are green.

Final-review update: [[wa.review.2026-08-22_2032-stagecraft-phase1-final-claude]] verified the original blocker/majors closed and returned GO WITH CHANGES. G1 remains open for the carried-blank-node proof defect, missing fallback tests, stateful E2E parser, and uncommitted governing notes. The suffix-only proof optimization is required before Phase 3 measurement; no batch task is justified by the current receipts.

Suffix-proof update: the carried-blank-node defect, fallback/failure coverage, and E2E parser isolation are fixed with 861 green tests. The N=552 observation improved from 3.22 s to 2.06 s while retaining self-contained directives. Governing notes are staged for commit; G1 awaits one final review of the committed result.

### G2 — Founding Capability Landed

Passes when [[wa.task.2026.2026-08-22_1112-founding-referent-data]] meets its ontology, behavior, runtime, Accord, validation, preservation, and documentation exit criteria.

### G3 — Batch Decision

After the 552 receipt, Dave rules one of:

- **singular accepted:** record the receipt and close this plan without a batch task
- **batch required:** cut the Phase 4 child task with an explicit performance/memory target derived from the receipt and Stagecraft's press budget

No batch target is invented before this evidence. Before running the receipt, Stagecraft should state the operational budget it cares about; otherwise the receipt remains descriptive and Dave makes the product judgment directly.

## Decisions

- Use one formal `plan` genre; "epic" is an informal size label.
- This plan is coordination-only and never enters READY.
- Phase 1 is a bounded child bite of the existing append-onlyish task, not a requirement to finish that whole migration.
- The FoundingReferentData Weave implementation lands on the migrated create writer, not the legacy subject-block renderer.
- Phase 1 records like-for-like N=552 measurements before and after the `knop.create` migration.
- A landed Stagecraft press is not resettable. Reset-and-replay is only a pre-landing repair window; post-publication founding corrections create later artifact states and a new press.
- Founding working update and correction are a narrow arm of `weave version`, optionally taking source bytes; no standalone founding update command ships in slice one.
- SFLO/framework founding-contract work may proceed in parallel after ruling; Weave runtime ordering remains strict.
- Measure singular create after the append migration and founding capability both land.
- Do not create speculative batch work; Gate G3 owns that branch.

## Open Issues

- What wall-clock and peak-memory budget does Stagecraft require for a 552-entry press on its representative host? Record it before Gate G3 if available.
- Should Phase 1 be carved into its own child task note before implementation, or remain a clearly bounded bite and receipt inside [[wa.task.2026.2026-05-17-append-onlyish-inventory]]? Lean: keep it in the existing task unless queue/review independence requires a carve.

## Testing And Receipts

- Phase 1 append/no-op/conflict and unknown-byte-preservation tests.
- Phase 1 reproducible N=552 before/after measurement receipt.
- Existing `knop.create` unit, integration, CLI, root, and Accord regression suite.
- Phase 2 founding-created, founding-versioned, and founding-corrected Accord transitions and task-level tests.
- Founding-data preservation through ordinary weave and `knop add-reference`.
- Phase 3 opt-in 552-entry receipt plus no-founding baseline.
- If selected, Phase 4 batch/sequential semantic-equivalence and failure-atomicity tests.

## Non-Goals

- Completing every append-onlyish inventory writer before founding data.
- Treating the plan as an executable implementation task or queue item.
- Duplicating child-task ontology/API/test contracts here.
- Choosing a batch architecture before the singular receipt.
- Page generation, external reference fetching, rewriting settled founding states, or root founding data beyond the child task's stated scope.
- OS-level append as a requirement; semantic append/no-op/conflict behavior and byte-stable preservation are the contract.

## Exit Criteria

- The plan convention is documented and mechanically recognized by Dendron schemas.
- `knop.create` uses the shared append planner and indexed membership validation on `main`.
- The like-for-like pre/post Phase 1 receipt is recorded.
- FoundingReferentData is delivered or explicitly cancelled with its consequences recorded.
- A published founding record can be corrected by a later HistoricalState/new press without rewriting the original state.
- The 552-entry receipt and no-founding baseline are recorded.
- Gate G3 is ruled, with either no batch task owed or a concrete batch child task created and completed/cancelled.
- No actionable work remains only in this plan.
- Backlog, roadmap, decision log, child links, and durable guidance reflect the final outcome.

## Plan Checklist

- [x] Establish [[wd.plans-and-tasks]], [[template.plan]], schema support, and the queue boundary.
- [ ] Capture the legacy N=552 baseline, deliver the `knop.create` append-planner/indexed-read-model bite, and record the identical post-change receipt under [[wa.task.2026.2026-05-17-append-onlyish-inventory]].
- [ ] Deliver [[wa.task.2026.2026-08-22_1112-founding-referent-data]] on the migrated writer.
- [ ] Record the 552-entry and no-founding baseline receipts.
- [ ] Rule Gate G3 and, only if selected, cut and deliver the batch child task.
- [ ] Reconcile durable docs/backlog/decision receipts and close this plan under [[wd.plans-and-tasks]].
