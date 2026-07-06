---
id: multi-target-payload-advancement-20260705
title: 2026 07 05 Multi Target Payload Advancement
desc: ''
created: 1783741200000
---

## Goals

- Let one full `weave` invocation advance multiple selected payload targets in a single deterministic plan.
- Keep atomicity at the application boundary: the requesting service serializes coherent state and requests the weave; Weave validates and fails closed but does not provide transactional write guarantees.
- Make shared support-artifact progression (especially MeshInventory) merge coherently when several targets advance in one plan, instead of last-write-wins.
- Keep single-target behavior, diagnostics style, and byte-stability guarantees from [[wa.task.2026.2026-07-03_1332-stagecraft-weave-planner-generalization]] unchanged.

## Summary

This is the pre-registered second slice of the planner-generalization epic. Two concrete consumers now shape it: the Stagecraft temporal rung, whose replay advanced three designators via sequential single-target invocations as a workaround, and the planned game+session history mesh, where a service will serialize game and session state together and request one weave for the batch.

The key architectural decision (Dave, 2026-07-05) is that atomicity belongs to the application. Game data is not written to mesh files during gameplay; it is serialized together at the service's or user's request, and the same service requests the weave. Weave therefore does not need locking, rollback, or transactional filesystem semantics. What it owes instead:

- all-or-nothing plan validation: if any requested target is incoherent, refuse the whole plan with a diagnostic naming the target and the missing or conflicting fact
- deterministic planning: same inputs produce the same plan and the same rendered outputs
- coherent shared-artifact rendering: one plan touching one MeshInventory produces one merged progression, not N conflicting rewrites
- safe re-runnability: after a partial failure (e.g., process killed mid-write), re-running the same request must either no-op already-advanced targets or fail with a condition-specific diagnostic, never corrupt state

Earlier exploration in `sflo.conv.2025-11-27-api-and-metadata-flow` (and adjacent bootstrap-cycle notes) captured a direction now considered wrong; do not mine those notes for requirements.

## Discussion

### Why the app-owned atomicity boundary is right

Weave plans then writes files; filesystem writes are not transactional and pretending otherwise would mean building locking and journal machinery for a problem the deployment shape already solves. The service is the only writer, it controls when serialization happens, and it can re-request a weave after a failure. The useful invariant for Weave is therefore idempotent convergence, not isolation.

### Shared support-artifact convergence

This is the real technical work of the slice. Multiple targets in one plan may progress the same MeshInventory, and in nested-Knop cases possibly the same KnopInventory or generated pages. The later-payload read model currently resolves progression per target; the multi-target plan needs a merge step so shared artifacts render once, with all targets' progression facts present and consistent. This is adjacent to [[wa.task.2026.2026-05-17-append-onlyish-inventory]]: appending/no-oping settled facts is exactly the behavior that makes a merged render and re-runnability cheap. Decide how much of that task this slice needs, and record it.

### Cross-target references

Game and session states may reference each other. The application guarantees referential coherence at serialization time; Weave validates the same per-target facts it validates today and does not add cross-target semantic validation in this slice.

### Input snapshot verification (follow-up increment, added 2026-07-06)

One cheap consistency guard is in scope as a second increment: hash the batch's input files (sha256) before content capture begins, capture content, then verify the hashes still hold once capture completes. This proves the plan was computed from one coherent snapshot of the working tree and closes the batch's only real coherence hole — a torn read, where one target's file is read, another target's file is read later, and an input changed in between, silently mixing old and new content in one plan.

Verification failure refuses the whole plan with a condition-specific diagnostic naming the changed file. Because the check completes before any write, refusal means zero writes — no revert, no rollback, no warning taxonomy. This is the same fail-closed whole-plan posture the batch already has, extended from RDF-fact coherence to input-snapshot coherence.

Changes that happen after capture completes are deliberately ignored: the plan derives purely from captured content, so the woven batch stays internally coherent regardless, and the application owns when to serialize and weave again. There is no lost-update warning. A post-write revert mode (`--atomic-only`) remains rejected; it would reintroduce exactly the rollback machinery this task excludes, and pre-capture verification already provides the strictness that matters.

### Candidate selection and the one-candidate limit

The epic already flags the one-candidate limit in `planWeave`. Multi-target selection should stay narrow: only explicitly requested targets advance; untargeted candidates are never silently included. If multiple selected candidates become supported, untargeted behavior must stay deterministic.

## Open Issues

- CLI shape: repeated `--target designatorPath=...` flags, a comma-separated list, or both? Follow existing targeting syntax conventions.
- Plan ordering: process targets in request order or canonical path order? Either is fine, but it must be deterministic and documented.
- Re-run semantics for already-advanced targets: silent no-op (append-onlyish style) or a specific "already at requested state" diagnostic? This likely depends on how much of the append-onlyish inventory behavior lands here.
- Does a multi-target plan advance MeshInventory once for the batch or once per target? One merged advance seems right, but check what existing fixtures and ResourcePage links assume.
- Does the game+session mesh need anything beyond payload targets (e.g., current-only support artifacts advancing alongside), and does that fall out of the existing read model?
- Snapshot verification: hash only each target's working payload files, or every working-tree file the plan reads as input (current inventories and support artifacts included)? Since the purpose is snapshot coherence rather than payload freshness, all plan-read inputs is the more honest scope — but confirm the read set is enumerable at hash time without restructuring candidate loading.

## Decisions

- Atomicity is the application's responsibility. Weave provides fail-closed whole-plan validation and deterministic output, not transactions, locking, or rollback.
- If any requested target is incoherent, the entire plan is refused; no partial advancement within one invocation.
- Only explicitly requested targets advance; exact targets stay narrow.
- Single-target behavior and diagnostics from the first slice are preserved unchanged.
- No grouped-provenance or "citable batch" vocabulary in this slice; if the game+session mesh proves that need, it becomes its own model task.
- The plan samples the wall clock once per invocation and threads the same instant through every generated page in the batch (the existing injectable `generatedAt`/`now` option in page generation). Page footers are the only clock-bearing outputs; the semantic layer stays ordinal-based and clock-free. A shared footer timestamp is a human-readable batch signal, not queryable provenance.
- Expose the injection seam on the CLI: `--generated-at <iso-8601>` on full `weave` and `weave generate` overrides the wall-clock sample, letting the requesting service stamp pages with event time (e.g., session end) instead of weave-execution time, and making page regeneration reproducible. The value must parse as a valid ISO 8601 instant and is re-serialized canonically (UTC `toISOString()` form) so output format does not depend on the caller's offset spelling; an unparseable value is a CLI error. Omitted means now, sampled once, as decided above. This changes footer display only — it does not create queryable provenance.
- Do not follow the 2025-11-27 API-and-metadata-flow direction.
- CLI syntax uses repeated `--target` flags. That is the existing target convention, already supported by the parser, and avoids inventing a second comma/list layer around a target spec that is itself comma-separated.
- Explicit payload batch planning order is canonical designator-path order, not request order. This matches existing candidate discovery order, keeps output deterministic across equivalent CLI flag ordering, and prevents caller ordering from becoming semantic state.
- MeshInventory advances once per explicit payload batch when batch members share it. The batch state represents the one support-artifact observation after coherent application serialization; recursive and mixed-slice planning keep the existing deterministic sequential behavior.
- This slice uses only the append/no-op/conflict portion of [[wa.task.2026.2026-05-17-append-onlyish-inventory]] that is needed for deterministic support rendering and reruns. It does not land the broader append-onlyish inventory task.
- Re-running an already-applied exact payload batch no-ops already-current payload targets. This supports partial reruns without minting duplicate identical states; a caller that wants a new state must change the payload or request a new explicit state segment.
- Runtime batching is scoped to multiple exact payload targets whose target-scoped planning policies are consistent. Recursive and mixed-slice target sets keep the existing deterministic sequential planner.
- Input snapshot verification is fail-closed and pre-write: sha256 hashes of the batch's input files are taken before content capture and verified after capture completes; any mismatch refuses the whole plan with a diagnostic naming the changed file, before anything is written. Changes after capture are ignored by design — no lost-update warning, no post-write re-checks, no `--atomic-only` revert flag.
- Hash scope for this slice is current working payload files for the requested explicit payload targets. The broader "all plan-read inputs" set is the better long-term snapshot-coherence boundary, but it is not cleanly enumerable before capture without refactoring candidate loading and effective-config/source discovery: support and config reads are discovered inside candidate/config loading, while this increment only needs the torn-read guard for changing working payload content. Current inventories/support artifacts remain covered by whole-plan fact validation and deterministic planning; broad plan-read hashing should be a later loader-read-set refactor if needed.

## Contract Changes

- Full `weave` accepts multiple payload version targets in one invocation and plans them together.
- Whole-plan validation failure names the offending target and fact; other targets are not advanced.
- Shared support artifacts touched by multiple targets render as one merged, deterministic progression.
- Re-running a completed or partially completed request converges instead of corrupting or duplicating state (exact semantics per the open issue above).

## Implementation Evidence 2026-07-06

Implemented explicit multi-target payload batching for `weave` and `weave version`. The CLI surface remains repeated `--target`, for example `weave --target 'designatorPath=game/state' --target 'designatorPath=game/session'`.

Core coverage proves two explicit first-payload targets sharing one MeshInventory produce one `_mesh/_inventory/_history001/_s0003` state and one current MeshInventory update containing both targets' facts.

Integration coverage uses the Stagecraft temporal-rung shape with three later-ordinal payload targets: `projections/contracts/inn-ambush-contract-context`, `projections/contracts/inn-ambush-contract-shapes`, and `world/states/inn-ambush-plan-b-state`. The batch run is byte-identical to the equivalent sequential single-target runs, two independent batch runs are byte-identical to each other with a fixed generated timestamp, and prior `_history001/_sNNNN` payload snapshots remain byte-identical. Re-running the completed batch no-ops with no file changes. A malformed batch with one target missing `sflo:latestHistoricalState` refuses the whole plan and leaves the workspace byte-identical to the preflight snapshot.

Added `--generated-at <iso-8601>` to full `weave` and `weave generate`. The CLI validates the value as an ISO 8601 instant with an explicit offset, canonicalizes it to UTC `toISOString()` form, and threads it through the existing page-generation `now` seam as the invocation's single clock sample. When the flag is explicit, timestamp-only page differences are written so pages converge to the pinned instant; omitted keeps the existing sample-once-now and timestamp-only skip behavior.

Added explicit payload-batch input snapshot verification with the scoped current-working-payload-file hash set. Tests mutate a working payload after the initial hash and before capture completes, proving fail-closed whole-plan refusal with no Weave writes after the simulated concurrent mutation and a diagnostic naming the file. Tests also mutate a working payload after verified capture and prove the batch output files remain byte-identical to an unmutated run.

## Testing

- Multi-target plan with independent targets: all advance in one invocation; RDF outputs match the equivalent sequential single-target runs byte-for-byte, and generated pages match under the existing timestamp-only-normalized page comparison (sequential runs necessarily stamp different footer times; the batch stamps one).
- Shared-inventory case: two targets under one MeshInventory produce one merged progression with both targets' facts.
- One incoherent target among coherent ones: whole plan refused, diagnostic names the target and fact, no files written.
- Determinism: identical inputs produce byte-identical plans and outputs across runs.
- Byte-stability: prior `_history*/_s*` payload states remain untouched (extend the existing regression).
- Re-run coverage: repeat an already-applied multi-target request and assert the decided no-op/diagnostic behavior.
- Snapshot verification: mutate an input file inside the capture window (between initial hashing and capture completion — find or inject the seam) and assert the whole plan is refused with zero writes and a diagnostic naming the changed file.
- Snapshot verification: mutate a working file after capture completes and assert the batch runs to completion with output byte-identical to the unmutated run and no diagnostic.
- `--generated-at`: a provided instant appears canonically in every generated page of the invocation; two runs with the same `--generated-at` and inputs are byte-identical including pages; an unparseable value fails as a CLI error before planning; omitted keeps the single-sample default.
- Run `deno task test` and `deno task lint`.

## Non-Goals

- Transactional filesystem writes, locking, journaling, or rollback.
- An `--atomic-only` post-write revert flag and any post-write hash re-checking or lost-update warnings (see the input snapshot verification discussion).
- Cross-target semantic reference validation.
- Grouped provenance / batch-citation vocabulary.
- Advancing untargeted pending candidates as a side effect.
- Game- or session-specific vocabulary or runtime branches.

## Implementation Plan

- [x] Decide the multi-target CLI syntax and plan ordering; record both here and in user docs.
- [x] Extend candidate selection and `planWeave` to accept multiple explicit targets, keeping untargeted behavior deterministic.
- [x] Add the shared support-artifact merge step to the later-payload read model path, deciding how much append-onlyish inventory behavior this requires.
- [x] Define and implement re-run semantics for already-advanced targets.
- [x] Add the test coverage listed above, including the sequential-equivalence check against the temporal-rung replay shape.
- [x] Update [[wd.decision-log]] (app-owned atomicity boundary) and tick the epic's multi-target follow-up in [[wa.task.2026.2026-07-03_1332-stagecraft-weave-planner-generalization]].
- [x] Snapshot verification: hash the batch's input files (sha256) before content capture and verify after capture completes; mismatch refuses the whole plan pre-write with a diagnostic naming the changed file.
- [x] Snapshot verification: decide the hash scope (working payload files vs all plan-read inputs) and record it here.
- [x] Snapshot verification: add the capture-window mutation test and the post-capture mutation test, then update user docs.
- [x] Add `--generated-at <iso-8601>` to full `weave` and `weave generate`: validate and canonicalize the instant, thread it through the existing `generatedAt` seam as the invocation's single sample, add the tests listed above, and document it in user docs.
