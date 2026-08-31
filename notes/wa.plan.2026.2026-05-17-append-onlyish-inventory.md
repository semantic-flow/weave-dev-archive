---
id: gx515t52uvyddqh14agkrhu
title: Append-Only Inventory Migration
desc: ''
updated: 1779080314278
created: 1779079677519
---

## Status

Active. Reclassified from a legacy oversized task to a coordination plan on 2026-08-31. Add-reference, extracted/current-only MeshInventory, current-only PageDefinition, and current-shape extract children are complete. Two versioned writers wait on rulings; the next independent operation-specific writer can proceed without weakening either ruling boundary.

## Origin

[[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]] — owns the bounded Phase 1 `knop.create` append-planner/indexed-read-model bite. The plan does not wait for every remaining writer in this broader task.

## Goals

- Make inventory writes append-onlyish: normal operations append new settled facts, no-op when those facts already exist, and fail closed on conflicting settled facts.
- Stop treating "graph-preserving rewrite" as the target abstraction. A rewrite that preserves more triples is still a rewrite; inventory mutation should be fact-level append/no-op/fail unless an explicit repair, regeneration, or retraction mode is active.
- Keep current/progression pointers out of inventory working files. Inventory should describe settled membership and artifact structure; metadata/progression surfaces should say what is current, latest, or next.
- Make automated release reruns boring: `weave`, `weave version`, `weave generate`, and composed branch-published release operations should not churn inventory files when source bytes, config, target history, and generated-page policy inputs have not changed.
- Preserve dereferenceability and local static-host usefulness without making generated ResourcePage facts a reason to rewrite settled inventory.

## Summary

Inventory is a ledger, not a freshly rendered report.

The current implementation is already halfway there for MeshInventory: `_mesh/_inventory/inventory.ttl` keeps stable history and state membership while `_mesh/_meta/meta.ttl` owns MeshInventory current/latest/next progression. The remaining work is to make that the general rule for MeshInventory, KnopInventory, payload histories, support-artifact histories, source-registry links, reference-catalog links, and generated ResourcePage facts.

The desired write primitive is:

- append missing settled triples or subject blocks
- leave existing matching triples byte-stable
- reject conflicting settled triples unless an explicit repair/regeneration/retraction mode was requested
- never silently remove unknown or older facts from inventory

This task supersedes the older TODO wording about "subject-level canonical rewrites with graph-preserving updates". The replacement principle is simpler and stricter: normal inventory operations do not rewrite existing inventory facts at all.

### Remaining-writer audit — 2026-08-31

Audit substrate: Weave `main` at `6643ae1`.

Delivered substrate:

- `planInventoryAppend`, consistency-by-construction prepared input, and exact suffix rendering are shared production code;
- current-only ReferenceCatalog weave appends through the planner;
- routine generation no longer deletes settled ResourcePage facts;
- first/later `knop.create` and FoundingReferentData settlement append through the planner;
- import's existing-payload source-registry insertion uses the planner;
- first `knop add-reference` ReferenceCatalog registration preserves the carried KnopInventory prefix and appends through the planner.
- homogeneous and versioned-sequential extracted MeshInventory growth plus shared current-only first-payload/extracted page claims append through the planner and emit no new blank nodes.

Remaining mutation paths, in execution order rather than file order:

1. `src/core/weave/mesh_inventory_renderers.ts` still uses subject-block replacement across first Knop, first payload, batched payload, and extracted-term paths. The batched extracted renderer explicitly filters target subject blocks before reconstruction.
2. current-only ResourcePageDefinition weave still replaces its subject block in `knop_inventory_renderers.ts`; versioned KnopInventory/payload/support renderers remain whole-document producers.
3. extract and integrate still build updated MeshInventory and KnopInventory documents through operation-specific append strings or canonical renderers rather than the shared planner.
4. `mesh_support_pages.ts` retains a separate block-mutation implementation for initial support-page and versioned MeshInventory planning.
5. mutable current/latest/next facts still require the storage-ownership ruling in Open Issues before the final inventory/metadata split.

## Child Tasks

- [[wa.completed.2026.2026-08-31_0106-add-reference-inventory-append]] — preserve the current KnopInventory as the exact prefix and append only planner-approved ReferenceCatalog facts.
- [[wa.completed.2026.2026-08-31_0845-batched-extracted-mesh-inventory-append]] — migrate the highest-risk MeshInventory weave renderer using a bounded owned-fact request rather than whole desired-output Turtle.
- [[wa.completed.2026.2026-08-31_1026-versioned-sequential-extracted-mesh-inventory-append]] — migrate the remaining versioned extracted-term MeshInventory renderer before current-mode extracted planner work revisits the same seam.
- [[wa.completed.2026.2026-08-31_1050-current-only-payload-like-mesh-inventory-append]] — migrate the shared current-only first-payload/extracted page-claim renderer to exact append/no-op semantics.
- [[wa.task.2026.2026-08-31_1111-versioned-first-knop-mesh-inventory-append]] — blocked on how to remove legacy inventory-owned progression without disguising repair as append-only normal operation.
- [[wa.completed.2026.2026-08-31_1127-current-only-page-definition-inventory-append]] — migrate current-only ResourcePageDefinition page claims while both MeshInventory rulings are open.
- [[wa.completed.2026.2026-08-31_1147-current-shape-extract-mesh-inventory-append]] — migrated extract's current-shape MeshInventory append while retaining the separate legacy renderer pending later disposition.
- Remaining KnopInventory/PageDefinition migration — cut after the MeshInventory seam is stable so shared progression and page-fact behavior are not duplicated.
- Extract/integrate and mesh-support migration — cut after the core weave writers prove the shared pattern.
- Progression-storage and fixture/documentation closure — cut only after the plan-level ownership rulings are resolved.

## Sequence

1. Deliver the `knop add-reference` data-loss fix independently.
2. Migrate MeshInventory weave paths in small behavior-preserving children, starting with batched extracted-term weave before current-mode extracted-term planner work touches the same seam.
3. Migrate remaining KnopInventory/PageDefinition and operation-specific extract/integrate writers. File-disjoint children may run in parallel only after the shared requested-fact/render pattern is proven.
4. Resolve progression-document ownership and repair/retraction policy before moving mutable facts.
5. Regenerate deliberately affected fixtures, update durable guidance/release notes, and close the plan only after a whole-repository writer audit finds no unowned normal-operation rewrite.

## Gates

- G1 — CLOSED 2026-08-31. `knop add-reference` preserves arbitrary carried facts/comments as an exact prefix, appends only missing planned facts, and refuses single-valued conflicts before writes. Weave PR #50 merged as `b8678d0` after green GitHub gates and a no-action CodeRabbit review.
- G2 — every migrated writer has fail-on-old append/no-op/conflict or carried-fact preservation evidence plus focused integration coverage.
- G3 — Dave rules the exact Knop-local progression document and whether append-onlyish is initially a Weave invariant or portable Semantic Flow contract before the storage split.
- G4 — fixture regeneration is deliberate and exact; no compatibility shim is added for stale pre-v1 shapes.
- G5 — full gates and a final source audit prove normal inventory paths no longer delete or canonical-rewrite settled facts.

### Stagecraft Phase 1 Before/After Receipt

[[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]] requires the bounded `knop.create` migration to be measured before and after rather than justified only by asymptotic inspection.

Before changing production create code:

- add a reusable opt-in N-create probe that calls the real `executeKnopCreate` path
- run N=552 on the unchanged implementation
- record commit/tree state, Deno version, broad host description, wall-clock time, peak RSS, final MeshInventory size, and successful create count
- save the observation in `timings/weave-performance.csv` with a workload label that can be repeated exactly

After the append-planner/indexed-membership change, rerun the identical command and record a before/after table in this task. Do not change N, fixture shape, runtime flags, temp-storage class, or measurement wrapper between runs. If an environmental difference is unavoidable, name it and do not present the ratio as like-for-like.

The baseline probe may be added before measurement; production create behavior may not change until the baseline is captured. The full N=552 probe remains outside ordinary CI. A small representative regression belongs in CI so the probe path itself does not rot.

### Stagecraft Phase 1 baseline receipt — 2026-08-22

The unchanged production `knop.create` implementation was measured after adding only `scripts/probe-knop-create-scale.ts` and `tests/integration/knop_create_scale_probe_test.ts`. `git rev-parse HEAD` returned `279f86b26fc94340a00bc3c9a38a38e1506624e8`; `git status --short -- src scripts tests` listed only those two untracked probe files, and `git diff -- src scripts tests` was empty because neither new file was tracked. No production create file had been edited.

- Runtime/host: Deno 2.9.2, V8 14.9.207.2-rusty, TypeScript 6.0.3; Linux 7.0.0-29-generic x86_64; Intel Core Ultra 7 265KF, 20 logical CPUs; `/tmp` on tmpfs.
- Fixed cache/environment: `DENO_DIR=/tmp/weave-stagecraft-phase1-deno-dir`; the warm-up populated that cache and the measured run reused it.
- Warm-up command: `env DENO_DIR=/tmp/weave-stagecraft-phase1-deno-dir deno run --allow-read --allow-write --allow-env scripts/probe-knop-create-scale.ts --count 3` — 3/3 successful creates, 1,934 final MeshInventory bytes.
- Measured command: `/usr/bin/time -v env DENO_DIR=/tmp/weave-stagecraft-phase1-deno-dir deno run --allow-read --allow-write --allow-env scripts/probe-knop-create-scale.ts --count 552`.
- Result: exit 0; 552/552 successful creates; 1,104 created files; 552 updated-file writes; 150,713 final MeshInventory bytes; 1,623.733 ms probe total and 1,557.654 ms inside the create loop; 1.71 s `/usr/bin/time` wall clock; 222,048 KiB maximum RSS.
- `/usr/bin/time -v` reported 48 filesystem input units and 0 filesystem output units on tmpfs. Aggregate MeshInventory bytes read/written were not instrumented, so no stronger byte-I/O claim is made.

This is the durable baseline gate. Production create edits may begin only after this receipt and its matching `stagecraft-phase1-knop-create-scale-legacy-n552` CSV row are present.

### Stagecraft Phase 1 single-run observations — 2026-08-22

The migrated paths were run with the identical wrapper, command, count, Deno cache, temp-storage class, fresh-mesh setup, and host as the baseline. `git rev-parse HEAD` remained `279f86b26fc94340a00bc3c9a38a38e1506624e8`; each after state was the same commit plus its recorded Phase 1 working tree. No known environmental condition changed. Each arm is one observation with no variance estimate, so the values are descriptive and do not support a precise regression percentage or causal performance claim.

| Metric | Legacy baseline | First append/index after | Post-review exact-union | Suffix-proof final |
| --- | ---: | ---: | ---: | ---: |
| Successful creates | 552 | 552 | 552 | 552 |
| Created files | 1,104 | 1,104 | 1,104 | 1,104 |
| Updated-file writes | 552 | 552 | 552 | 552 |
| `/usr/bin/time` wall clock | 1.71 s | 1.82 s | 3.22 s | 2.06 s |
| Probe total | 1,623.733 ms | 1,736.860 ms | 3,130.525 ms | 1,982.976 ms |
| Create loop | 1,557.654 ms | 1,672.435 ms | 3,066.352 ms | 1,919.791 ms |
| Maximum RSS | 222,048 KiB | 230,076 KiB | 230,072 KiB | 222,432 KiB |
| Final MeshInventory | 150,713 bytes | 154,577 bytes | 220,817 bytes | 220,817 bytes |

Exact first-after command: `/usr/bin/time -v env DENO_DIR=/tmp/weave-stagecraft-phase1-deno-dir deno run --allow-read --allow-write --allow-env scripts/probe-knop-create-scale.ts --count 552`. It exited 0; `/usr/bin/time -v` reported 0 filesystem input and 0 filesystem output units on tmpfs. Aggregate MeshInventory bytes read/written remain uninstrumented.

Exact post-review command: `/usr/bin/time -v env DENO_DIR=/tmp/weave-stagecraft-phase1-deno-dir deno run --allow-read --allow-write --allow-env scripts/probe-knop-create-scale.ts --count 552`. It exited 0; `/usr/bin/time -v` reported 0 filesystem input and 0 filesystem output units on tmpfs. The probe now excludes its final inventory `stat` from `createElapsedMs`. Aggregate MeshInventory bytes read/written remain uninstrumented.

Exact suffix-proof command: `/usr/bin/time -v env DENO_DIR=/tmp/weave-stagecraft-phase1-deno-dir deno run --allow-read --allow-write --allow-env scripts/probe-knop-create-scale.ts --count 552`. It exited 0; `/usr/bin/time -v` reported 0 filesystem input and 0 filesystem output units on tmpfs. Aggregate MeshInventory bytes read/written remain uninstrumented.

The observations do not demonstrate a wall-clock improvement over legacy at N=552. Suffix-only proof recovers most of the review-safe full-candidate reparse cost and returns peak RSS to the baseline range, while preserving the exact prefix and adversarial fail-closed behavior. Repeated singular creates still parse the growing inventory and physically rewrite the complete file, and self-contained base/prefix directives remain in each append chunk, retaining the 220,817-byte final inventory. Each arm remains one recorded observation in the durable table; the final Claude review's temporary repeated runs independently corroborate the direction without turning these rows into a statistical benchmark. Further directive amortization, pass reduction, and physical append remain deferred; do not claim a speedup.

### Stagecraft Phase 1 implementation and gate receipt — 2026-08-22

Fail-on-old was captured before implementation: the legacy path failed `planKnopCreate preserves carried MeshInventory bytes and appends only missing facts` because it rewrote the carried prefix, and failed `planKnopCreate names conflicting carried and requested working inventory locators` because it silently replaced the carried locator instead of throwing.

The later/current create path now requests `_mesh hasKnop D/_knop`, the new Knop type and working-inventory link, and the inventory file `LocatedFile` / `RdfDocument` types through `planInventoryAppend`. Only `hasWorkingKnopInventoryFile` is treated as single-valued. The legacy first-Knop renderer remains for its bootstrap shape. The carried-shape reader builds one `KnopCreateInventoryIndex` over the parsed graph; the append planner consumes the same pre-parsed quads, and the scale-index test proves that 552 membership lookups do not re-consume the quad iterable. `splitTurtleBlocks`, `replaceSubjectBlock`, and `upsertSubjectBlockAfter` are absent from `src/core/knop/create.ts`.

### Stagecraft Phase 1 Claude review — 2026-08-22

[[wa.review.2026-08-22_1711-stagecraft-phase1-claude]] returned **GO WITH CHANGES**. Gate G1 remains open.

- **Blocking:** the bespoke compact suffix renderer can emit undefined `sflo:` prefixes or resolve mesh-relative appended terms against an unrelated carried `@base`, so written RDF may disagree with the append plan. Fix and add the two reproduced adversarial cases before merge.
- **Major:** name the legacy first-Knop template deletion as a residual and scope Gate G1 to the later/current path; restore an exact RDF acceptance oracle rather than indefinite containment; test/document the pre-parsed planner invariant; and consolidate compact suffix rendering or use planner output.
- **Advisory:** rule duplicate/graph semantics, add runtime no-write-on-conflict coverage, keep single-run timing claims modest, reconcile docs, and address or re-board probe/newline nits.

Do not mark Phase 1 or parent-plan Gate G1 complete from the pre-review receipt. Implementation follow-up and review disposition remain owed.

### Claude review follow-up scope — ruled 2026-08-22

Take now:

- B1 compact-renderer plan/output equivalence and adversarial base/prefix coverage
- M1 first-Knop settled-fact preservation, because it is live data loss on every fresh mesh
- M2 exact RDF equality with deliberately carried facts, while boarding fixture regeneration explicitly
- M3 a consistency-by-construction prepared-current-inventory input plus direct planner tests
- M4 one fact-preserving shared append renderer driven by `plan.appendTurtle`, not a second hand reconstruction
- exact duplicate consistency for named-node and literal objects, default-graph enforcement, runtime zero-write conflict coverage, conservative single-run timing wording, probe cleanup/help/timing fixes, and current developer/backlog documentation

Defer but name:

- further reduction of the remaining linear planner/index passes unless the final repeated measurement demonstrates value
- OS-level physical append; semantic append with a byte-stable prefix remains the contract
- unconditional runtime write avoidance for the public-create no-op, because public create refuses an already registered Knop and the no-op is currently helper-only
- a source-text guard forbidding future imports of block helpers; behavioral exactness tests are the durable guard
- broad fixture-ladder regeneration beyond the minimum acceptance correction needed for this slice

The trailing-newline residual must either be fixed by the shared renderer or remain explicitly boarded with evidence after the follow-up.

### Stagecraft Phase 1 review follow-up disposition — 2026-08-22

Gate G1 remains **OPEN**. The ruled follow-up is implemented and green, but a final read-only review is requested before the planning seat marks the gate complete or allows the Weave FoundingReferentData runtime slice to proceed.

Fail-on-current evidence was captured before production follow-up edits:

- `deno test --allow-read --allow-write --allow-env --allow-run=git src/core/knop/create_test.ts --filter='/(no sflo prefix|carried base differs|complete Alice first-Knop)/'` — 0 passed, 3 failed. The no-prefix case produced undefined `sflo:`, the different-base case denoted `elsewhere.example` appended resources, and the first-Knop case failed exact-prefix preservation.
- `deno test --allow-read --allow-write --allow-env --allow-run=git,deno tests/e2e/knop_create_cli_test.ts --filter=alice-bio` — 0 passed, 1 failed because the legacy first-Knop template did not preserve the carried Alice config graph required by the restored exact oracle.

Review findings:

- **B1 — FIXED.** `renderInventoryAppendPlan` parses only `plan.appendTurtle`, renders one self-contained compact chunk with its own `@base` and `sflo:` declaration, parses the complete candidate, and checks exact RDF-set equality against prepared current quads union every `plan.missing` key. A compact mismatch falls back only to the planner's absolute-IRI output after the same exact check; otherwise it fails closed. Both Claude adversarial cases and a literal/datatype/language coverage-drift case pass.
- **M1 — FIXED.** First and later Knop creation now share the same append/no-op/conflict path. The fixed first-Knop MeshInventory template was deleted. The complete carried Alice `a.03` MeshInventory—including `_mesh/_config`, histories, pages, comments, prefixes, and unknown facts—remains the exact output prefix.
- **M2 — FIXED WITH NAMED FIXTURE RESIDUAL.** Alice and sidecar E2E MeshInventory checks are exact RDF equality against the old expected target graph union the explicitly selected carried `_mesh/_config` graph; indefinite containment is gone. The stale Alice `a.04` and sidecar `a.10` fixture branches, plus affected downstream rungs, still need deliberate broad-ladder regeneration and are boarded in [[wd.codebase-overview]] and [[wd.todo]]. No fixture topology changed here.
- **M3 — FIXED.** `prepareCurrentInventory` is the only producer of the opaque `PreparedCurrentInventory`; its byte string and frozen parsed quads cannot be supplied independently through the public planner input type. `planKnopCreate` feeds the same prepared quads to `KnopCreateInventoryIndex` and `planInventoryAppend`. The raw-Turtle convenience arm remains for other callers. Direct prepared append/no-op/conflict tests pass, and preparation rejects non-default-graph facts.
- **M4 — FIXED.** The create-specific fact reconstruction and ReferenceCatalog post-processing compactor are gone. Both consumers use `renderInventoryAppendPlan`, whose compact output is derived generically from parsed `plan.appendTurtle` and checked exactly before return.

Advisories taken:

- Exact duplicate named-node and literal objects now both dedupe by RDF term identity before singleton checks; datatype and language remain part of literal identity. Create-level and index-level duplicate tests pass.
- Prepared current inventories reject named graphs, so the index and planner share default-graph semantics.
- Runtime conflict coverage proves zero created directories/files and byte-identical MeshInventory after refusal.
- Timing text now treats all three N=552 arms as single observations and makes no precise regression or causal claim.
- The probe uses shared test-temp setup plus `try/finally`, excludes final `stat` time from `createElapsedMs`, and exposes tested `-h` / `--help` output.
- Shared appended output ends in one newline; the ReferenceCatalog append and generic renderer tests prove no `\n\n` EOF. Semantic no-op deliberately retains the input's original trailing bytes.
- The create append helper is private rather than an unenforced exported `@internal`; the test-only `indexedQuadCount` property was removed while iterable-consumption instrumentation remains.
- [[wd.codebase-overview]] and [[wd.todo]] describe prepared append input, shared exact rendering, both migrated create paths, final-review status, and fixture residuals.

Retained residuals:

- The planner/index still perform multiple linear passes, and the exact renderer adds a complete-output parse per append. The post-review receipt demonstrates no reason to claim a speedup; further pass reduction is deferred until owned by measured work.
- Runtime still reads/parses and physically replaces the complete growing MeshInventory for every singular create. Semantic byte-prefix preservation is not an OS-level append, so repeated singular creates retain quadratic aggregate parse/write bytes.
- Public create refuses already registered Knops, so unconditional runtime write avoidance for helper-only semantic no-op remains deferred.
- Broad fixture-ladder regeneration and a source-text import guard against future block helpers remain deferred as ruled. Behavioral exactness tests are the current durable guard.

Final validation receipts:

- Focused: `deno test --allow-read --allow-write --allow-env --allow-run=git src/core/weave/inventory_append_planner_test.ts src/core/weave/knop_inventory_renderers_test.ts src/core/knop/create_inventory_index_test.ts src/core/knop/create_test.ts tests/integration/knop_create_test.ts tests/integration/knop_create_scale_probe_test.ts` — 35 passed, 0 failed.
- E2E/Accord: `deno test --allow-read --allow-write --allow-env --allow-run=git,deno tests/e2e/knop_create_cli_test.ts` — 4 passed, 0 failed.
- Probe help: `deno run --allow-read --allow-write --allow-env scripts/probe-knop-create-scale.ts --help` — exit 0 with usage, count, preserve, and help options.
- Full gates: `deno task fmt` — 257 files checked; `deno task lint` — 256 files checked; `deno task check` — all modules checked; `deno task ci` — 857 passed, 0 failed, LCOV generated. Coverage emitted only the known deleted-temporary-source skip messages.
- Whitespace: `git diff --check` in Weave and `git -C dependencies/github.com/semantic-flow/weave-dev-archive diff --check` — exit 0.

Post-review N=552 receipt: `/usr/bin/time -v env DENO_DIR=/tmp/weave-stagecraft-phase1-deno-dir deno run --allow-read --allow-write --allow-env scripts/probe-knop-create-scale.ts --count 552` — exit 0; 552/552 successful creates; 1,104 created files; 552 updated-file writes; 220,817 final MeshInventory bytes; 3,130.525 ms probe total; 3,066.352 ms create loop; 3.22 s wall clock; 230,072 KiB maximum RSS; 0 reported filesystem input/output units on tmpfs. The host, runtime, cache, wrapper, flags, temp-storage class, and workload match the earlier observations; aggregate MeshInventory bytes read/written remain uninstrumented.

**Review request:** perform one final read-only review against B1, M1–M4, the advisory dispositions above, and the new exact E2E/measurement receipts. Do not mark Gate G1 complete before that review returns GO.

### Stagecraft Phase 1 final Claude review — 2026-08-22

[[wa.review.2026-08-22_2032-stagecraft-phase1-final-claude]] returned **GO WITH CHANGES**. Every r0 B1/M1–M4 finding is closed, but G1 remains open for four new items:

- carried blank nodes fail the complete-output quad-key equality check because independent N3 parses allocate different blank-node labels
- absolute fallback/terminal failure safety branches lack direct tests
- the exact E2E oracle reuses a stateful parser whose base can leak between documents
- governing plan/review/founding notes referenced by the committed receipt are not yet committed

Repeated reviewer measurements reproduced the 1.9× observation and attributed about 92% of it to removable full-candidate reparse plus repeated directives. Suffix-only proof is ruled sound for the self-contained, no-blank-node append chunk and fixes the carried-blank-node failure. It must land before the Phase 3 founding receipt. Directive removal is not safe without proving trailing base/prefix state; keep directives unless that proof is implemented.

Do not infer a batch requirement from this receipt. All singular arms remain quadratic, the legacy arm also scales quadratically, and Stagecraft's budget remains the G3 input.

### Stagecraft Phase 1 suffix-proof follow-up — 2026-08-22

The bounded final-review follow-up is implemented:

- **N1 fixed.** `renderInventoryAppendPlan` proves only the self-contained rendered suffix against `plan.missing`; prepared current Turtle remains the exact output prefix. Carried blank nodes no longer participate in a second parser-local label allocation, and a dedicated regression passes.
- **N2 fixed.** Scheme-relative-looking and dot-segment edge IRIs prove the absolute fallback; a deliberately inconsistent append plan proves the terminal fail-closed branch.
- **N3 fixed.** The E2E exact RDF oracle now constructs a fresh N3 parser for actual, expected, and carried documents, preventing base leakage.
- **N4 ready for closure.** The plan, reviews, Founding task, plan template/schemas, and related guidance are staged for the planning-seat archive commit alongside this receipt.

Directive amortization is deliberately not included. Every compact suffix remains self-contained because naive directive removal fails the no-prefix/different-base adversarial cases. A later optimization must prove effective trailing base/prefix state before omitting directives. OS-level append and batch creation remain outside G1.

Focused receipt: 43 passed, 0 failed across append planner, ReferenceCatalog renderer, create index/core/runtime/probe, and create E2E tests.

Full gates: `deno task fmt`, `deno task lint`, `deno task check`, and `deno task ci` green; CI reports 861 passed, 0 failed and generated LCOV. Both repositories pass `git diff --check`.

Suffix-proof N=552 observation: 552/552 creates; 1,104 created files; 552 updated-file writes; 220,817 final MeshInventory bytes; 1,982.976 ms probe total; 1,919.791 ms create loop; 2.06 s wall; 222,432 KiB peak RSS. This recovers most of the full-candidate reparse cost while retaining self-contained directives and the physical full-file rewrite residual. The matching CSV row is `stagecraft-phase1-knop-create-scale-suffix-proof-n552`.

**Final review request:** verify N1–N3, committed governance N4, full gates, suffix-proof soundness, and the retained directive/physical-write residual. Gate G1 remains open until that review returns GO.

### Stagecraft Phase 1 Gate G1 closure — 2026-08-22

[[wa.review.2026-08-22_2303-stagecraft-phase1-g1-claude]] returned **GO** against Weave `8dfc7f3` and archive `c474401`. N1–N4 are closed, all earlier B1/M1–M4 fixes remain intact, and Gate G1 is complete.

Owned residuals:

- retain self-contained append directives until effective trailing base/prefix state can be proved safely
- retain physical full-file read/parse/replacement for singular create; batch remains evidence-triggered at G3
- board latent prepared/plan mispair hardening for the shared renderer
- fix stateful multi-file N3 parser reuse in import/integrate/version/payload-update validators separately
- regenerate stale create fixture ladders in the next deliberate broad fixture pass

Phase 1 does not close the broader append-onlyish task; remaining writers and progression separation stay owned here. The Weave FoundingReferentData runtime may now proceed under the parent plan.

### Fresh-Conversation Implementation Brief — Phase 1 Review Follow-Up

```text
Address the Claude Phase 1 review findings that are ruled in scope in [[wa.plan.2026.2026-05-17-append-onlyish-inventory]]. Work only on the Stagecraft Phase 1 follow-up; do not implement FoundingReferentData.

Work in /home/djradon/hub/semantic-flow/weave. Read AGENTS.md, product vision, wd.general-guidance, [[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]], this task's Phase 1 brief/receipts/follow-up scope, and [[wa.review.2026-08-22_1711-stagecraft-phase1-claude]] completely. Inspect the current uncommitted diff before editing. Preserve all unrelated planning/documentation changes; do not reset, discard, commit, push, or rename them. Use apply_patch for edits.

REQUIRED CORRECTNESS FIXES

1. Eliminate the plan/output mismatch in the create append renderer. Do not reconstruct requested facts independently from plan.missing. Build compact output only from plan.appendTurtle through one fact-preserving shared renderer, or use plan.outputTurtle's absolute-IRI suffix. Any compact chunk must be self-contained with respect to base/prefix directives.
2. After rendering, parse the complete output and prove its semantic quad set is exactly currentInventoryQuads union plan.missing. A mismatch must fail closed or use the known-correct absolute planner output; never silently write a different graph.
3. Add fail-on-current tests for both Claude reproductions: a carried inventory using full SFLO IRIs with no sflo prefix, and a carried inventory whose @base differs from meshBase. Add a coverage-drift case proving every planner-approved requested fact survives, including a literal-valued fact through the shared rendering helper.
4. Replace the second bespoke compactor with one shared append-rendering helper. Migrate the current-only ReferenceCatalog append consumer onto it when necessary to prove sharing, while preserving that consumer's semantics and focused tests. Do not broaden into a general Turtle rewrite.

FIRST-KNOP DATA-LOSS FIX

5. Migrate the first/legacy Knop creation path onto append/no-op/conflict semantics as well. Preserve all carried mesh config, unknown facts, histories, pages, comments, and prefixes as an exact input prefix. Delete the fixed first-Knop inventory template only if it becomes dead; retain any genuinely required bootstrap behavior through requested facts, not whole-document replacement.
6. Add a fail-on-current test based on the real Alice a.03 shape proving the first Knop preserves the complete _mesh/_config support graph. Scope Gate G1 truthfully after this change.
7. Restore an exact RDF acceptance oracle in the E2E test. Compare actual with expected union explicitly carried facts; do not use indefinite containment. Record stale fixture-branch regeneration as a named residual if updating the whole ladder is disproportionate, but do not weaken semantic equality.

PREPARED INVENTORY INVARIANT

8. Replace the independently supplied currentInventoryTurtle/currentInventoryQuads pair with a consistency-by-construction prepared-current-inventory value produced by the append-planner module from base IRI plus Turtle. A branded/opaque prepared type is acceptable. The same prepared quads must feed KnopCreateInventoryIndex and planInventoryAppend.
9. Preserve the existing string-only convenience path for other callers if useful, but document both arms and add direct planner tests for prepared-input append/no-op/conflict behavior. Make inconsistent string/quads input unrepresentable through the public TypeScript contract rather than trusting callers.
10. Reject non-default-graph current inventory input at preparation, matching the append contract. Make exact duplicate statements semantically consistent: named-node and literal object listing should both dedupe identical RDF terms before singleton cardinality checks. Add tests for duplicates, datatypes/languages, and named-graph refusal.

RUNTIME/PROBE/DOC TESTS

11. Add a runtime integration test proving a single-valued conflict creates zero files/directories and leaves MeshInventory byte-identical.
12. Fix scale-probe test cleanup with try/finally and the shared temp harness where applicable. Exclude final Deno.stat from createElapsedMs. Add --help and focused tests.
13. Reword timing receipts as single-run observations: no demonstrated speedup, with observed values reported but no precise regression claim. Do not erase the original numbers.
14. Update wd.codebase-overview and wd.todo to reflect the migrated create paths, prepared append input/shared renderer, and any named fixture/newline residual.
15. Keep these deferred: speculative extra pass optimization, OS-level append, helper-only no-op write avoidance, broad fixture-ladder regeneration, and source-text import guards.

VALIDATION AND FINAL RECEIPT

16. Run focused fail-on-current tests first, then all existing append-planner/create/index/probe unit, integration, and E2E tests. Run deno task fmt, lint, check, and ci; run git diff --check in Weave and weave-dev-archive.
17. Rerun the identical N=552 command/wrapper/environment once after the review fixes. Append a post-review-final CSV row and extend the task table without rewriting the prior baseline or first-after observations. State any comparability issue and retain the honest physical full-file parse/write residual.
18. Update this task with a point-by-point disposition of B1 and M1-M4 plus advisories taken/deferred. Do not mark Gate G1 complete; request a final read-only review first.

HANDOFF

Report files changed, exact test/gate receipts, final N=552 observation, fixture/newline/physical-write residuals, and detailed suggested semantic commit messages per repo. Do not commit or push. End with READ-IN/QUEUE DELTA: none | <what belongs where>.
```

- Focused command: `deno test --allow-read --allow-write --allow-env --allow-run=git src/core/weave/inventory_append_planner_test.ts src/core/knop/create_inventory_index_test.ts src/core/knop/create_test.ts tests/integration/knop_create_test.ts tests/integration/knop_create_scale_probe_test.ts` — 22 passed, 0 failed.
- CLI/Accord command: `deno test --allow-read --allow-write --allow-env --allow-run=git,deno tests/e2e/knop_create_cli_test.ts` — 4 passed, 0 failed. The sidecar MeshInventory assertion now requires the exact carried input prefix plus every prior target-fixture RDF fact because the old exact target omitted the carried config fact that block replacement deleted; no fixture topology changed.
- Full commands: `deno task fmt` — 257 files checked; `deno task lint` — 256 files checked; `deno task check` — all script/source/test modules checked; `deno task ci` — 847 passed, 0 failed, LCOV generated. Coverage reported only the existing deleted-temporary-source skips.
- Whitespace commands: `git diff --check` in Weave and `git -C dependencies/github.com/semantic-flow/weave-dev-archive diff --check` — both exited 0.

The residual physical cost is unchanged in kind: each singular create parses the complete growing MeshInventory once and `writeUpdatedFiles` still calls `Deno.writeTextFile` with the complete planner output. Existing bytes are a byte-stable logical prefix, but this phase does not perform an OS-level append and still incurs N complete-file replacement writes for N singular creates.

### Fresh-Conversation Implementation Brief — Stagecraft Phase 1

```text
Implement Phase 1 of [[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]] only: measure the current singular knop.create path at N=552, migrate the non-legacy MeshInventory create path to planInventoryAppend plus indexed membership validation, and rerun the identical measurement. Do not implement FoundingReferentData in this conversation.

Work in /home/djradon/hub/semantic-flow/weave. Read AGENTS.md, documentation/notes/product-vision.md, documentation/notes/wd.general-guidance.md, this append-onlyish task note, the parent plan, and src/core/knop/create.ts / src/runtime/knop/create.ts / src/core/weave/inventory_append_planner.ts plus their tests before editing. Preserve every existing uncommitted planning/documentation change; do not discard, rewrite, commit, push, or rename unrelated work.

BASELINE FIRST

1. Before changing production create code, add a reusable opt-in probe (prefer scripts/probe-knop-create-scale.ts) that creates a fresh minimal mesh in a temp directory and invokes the real executeKnopCreate path N times. It must accept a count, default to a small safe value, emit machine-readable totals, and clean up its own temp workspace unless an explicit preserve flag is supplied.
2. Add a small ordinary-CI test proving the probe/helper path works without running N=552 in CI.
3. Record git rev-parse HEAD, prove git diff -- src scripts tests contains no pre-existing production change other than the new probe/test, record Deno version and a broad non-secret host label, warm up at a small N, then run N=552 under /usr/bin/time -v (or an equivalently explicit Linux wrapper). Record wall-clock, max RSS, successful create count, and final MeshInventory bytes. Use the same DENO_DIR, command, flags, filesystem class, and workload for the after run.
4. Append the baseline observation to dependencies/github.com/semantic-flow/weave-dev-archive/timings/weave-performance.csv and add a dated Phase 1 receipt subsection to this task. Do not edit production create code until the baseline is durably recorded.

IMPLEMENTATION

5. Keep the legacy/first-Knop renderer only where its distinct bootstrap shape is genuinely required. Migrate the later/current knop.create MeshInventory growth path away from splitTurtleBlocks / replaceSubjectBlock / upsertSubjectBlockAfter and onto planInventoryAppend.
6. Request only the new settled facts: _mesh hasKnop D/_knop; D/_knop rdf:type Knop; D/_knop hasWorkingKnopInventoryFile its inventory file; and the inventory file rdf:type LocatedFile plus RdfDocument. hasKnop and rdf:type are multi-valued; hasWorkingKnopInventoryFile is single-valued.
7. Preserve the entire current MeshInventory byte-for-byte as the output prefix, append only missing facts, return exact input bytes for planner-level semantic no-op, and fail before writes with requested/existing facts named on a single-valued conflict. Existing executeKnopCreate behavior still refuses an already registered Knop; do not silently turn the public operation into an idempotent create.
8. Replace the carried-shape loop that calls a full quads.some scan for every existing Knop with indexes built from the one parsed graph. Preserve all current fail-closed shape checks and diagnostics unless a condition-specific improvement is required by the new tests.
9. Reuse the shared append planner. Do not add another inventory mutation abstraction. If compact suffix rendering needs extraction because this is now another consumer, keep it narrowly shared and preserve the planner's semantic behavior; never fall back to subject-block replacement.
10. Do not change page generation, FoundingReferentData, other MeshInventory writers, add-reference, extract/integrate, payload/support histories, progression placement, remote access, or fixture topology in this slice.

FAIL-ON-OLD AND REGRESSION TESTS

11. Add focused tests proving: an unknown predicate/comment remains an exact byte prefix; only missing facts append; a single-valued working-inventory locator conflict refuses with both facts named; requested duplicate facts no-op at the helper/planner boundary; no non-legacy create path calls block replacement; and membership validation performs one indexed pass rather than one full quad scan per existing Knop.
12. Keep existing knop.create core, integration, e2e/CLI, root, semantically equivalent Turtle, and already-registered refusal behavior green.

AFTER MEASUREMENT

13. Run the identical N=552 command with the same wrapper and environment. Record the same fields in weave-performance.csv and this task, with a before/after table. If anything is not comparable, say so and do not report a misleading ratio.
14. Run focused tests, deno task fmt, deno task lint, deno task check, and deno task ci. Run git diff --check in Weave and weave-dev-archive.

HANDOFF

Report the implementation outcome, exact test/gate receipts, before/after measurements, files changed, any residual physical full-file-write cost, and a detailed suggested semantic commit message per affected repo. Do not commit or push. End with READ-IN/QUEUE DELTA: none | <what belongs where>.
```

## Discussion

### Settled Inventory Facts

Inventory may append facts such as:

- a mesh has a Knop
- a Knop has support artifacts
- a Knop has a payload artifact
- an artifact has an ArtifactHistory
- an ArtifactHistory has a HistoricalState
- a HistoricalState has an ArtifactManifestation
- a state or manifestation has a LocatedFile
- a support artifact has a working LocatedFile
- a Knop inventory links a source registry, reference catalog, extraction-source pointer, or generated ResourcePage once that support surface exists

These are additive facts. Once the mesh has published them, rerunning the same operation should not restate the whole subject in a canonical order, remove unfamiliar predicates, or "clean up" older blocks.

### Current And Progression Facts

Facts such as `sflo:currentArtifactHistory`, `sflo:latestHistoricalState`, `sflo:nextHistoryOrdinal`, consumed next-state hints, and any current-selection convenience facts are mutable progression facts. They should be written to metadata/progression documents, not to inventory working files.

This is already the documented direction for MeshInventory: `_mesh/_inventory` keeps stable artifact-history and historical-state membership while `_mesh/_meta` advances `sflo:latestHistoricalState`, `sflo:nextStateOrdinal`, and related progression facts. Finish that split and apply the same shape to Knop-owned inventory and payload/support artifact histories.

The RDF subject can still be the artifact or history IRI. The important distinction is the graph/document home: settled membership facts live in inventory; mutable progression facts live in meta.

### ResourcePages

Generated ResourcePage files are derived outputs, but the fact that a ResourcePage exists for a resource can become a settled inventory fact once the page is materialized or promised by policy. ResourcePage policy should control whether new page facts are appended and whether page bytes are generated. It should not remove already-settled page facts from inventory during ordinary runs.

If renderer behavior, page config, or publication policy changes and historical pages need to be rebuilt, that is a regeneration mode. Regeneration may rewrite generated HTML bytes and may repair stale page facts if explicitly requested, but it should not be the default inventory mutation path for `weave`, `generate`, or composed branch-published release operations.

### Retractions And Repairs

Append-onlyish does not mean "never correct anything". It means correction is explicit.

Legitimate non-append modes include:

- privacy or security retraction
- repair of invalid previous output
- deliberate regeneration after a config or ontology-contract change
- retargeting an ArtifactHistory or source binding with user intent

Those modes should be named, audited, and tested separately. Ordinary mesh growth should not smuggle repair behavior through a canonical renderer.

### Release Automation Consequence

For CI/CD, rerunning publication should be safe because the command either sees no new facts, appends genuinely new release/source facts, or fails before writes. A repeated release action against the same source bytes and the same `releases/vX.Y.Z` target should not touch inventory. A new version should append the new HistoricalState membership and move current/latest pointers in meta, not rewrite the old inventory blocks.

## Open Issues

- **OPEN 2026-08-31 — repository floating-locator identity blocks versioned first-payload append.** `renderCurrentWorkingFileLocator` currently emits `sflo:hasRepositorySourceFloatingLocator [ ... ]` as an intentionally generated blank-node subgraph. The append planner correctly rejects blank nodes in newly requested facts, and Dave's product preference is to avoid blank nodes wherever practical. The choices are: (A) mint a deterministic named locator resource and update SFLO/Weave/docs/fixtures; (B) expand the append planner to admit and prove bounded blank-node request subgraphs; or (C) keep a split legacy rewrite only for floating repository payloads. Lean: A. Dave must rule the portable locator identity/path contract before the versioned first-payload renderer migrates; do not silently choose an identifier shape or weaken the append planner.
- **OPEN 2026-08-31 — first-Knop exposes legacy progression migration versus exact append.** The accepted pre-weave MeshInventory still carries mutable `currentArtifactHistory`, `nextHistoryOrdinal`, `latestHistoricalState`, and `nextStateOrdinal`; the old renderer deletes them while writing their authoritative successors to MeshMetadata. Exact-prefix append would preserve them and violate the storage split. Choices: (A) name ordinary first-Knop weave as an explicit one-time migration/removal; (B) fix producers/fixtures and fail old shapes pending explicit repair/regeneration; or (C) grandfather the duplicate pointers until a repair surface exists. Lean: B, consistent with pre-v1 fail-closed/no-shim policy, but only after current producers stop creating the stale shape.

- Which exact metadata document owns current/progression facts for Knop-owned payload and support histories? The likely target is `D/_knop/_meta/meta.ttl` for Knop-local artifact progression and `_mesh/_meta/meta.ttl` for MeshInventory progression.
- Should the ontology or config vocabulary name an explicit inventory write policy such as append-only/current-projection/repair, or is this initially a Weave runtime invariant?
- How should repair/retraction be exposed: CLI flags on existing commands, a separate `weave repair` surface, or an internal mode first?
- Should existing pre-v1 fixtures and branch-published meshes be regenerated into the new shape, or should the first implementation carry a one-time migration reader? Preference: regenerate fixtures and fail closed on stale published shapes unless a repair mode is invoked.
- Are generated ResourcePage facts always settled once present, or do some page promises belong in metadata until the page is actually materialized?

## Decisions

- Normal inventory operations append facts, no-op duplicate facts, and fail closed on conflicting facts.
- "Graph-preserving update" is not the desired endpoint; it is still too permissive because it allows silently replacing known subject blocks.
- Current/progression predicates belong in metadata/progression documents, even when their RDF subjects are inventory artifacts, payload artifacts, or ArtifactHistory resources.
- Inventory history/state membership facts are settled facts and may remain in inventory.
- Generated ResourcePage byte regeneration is a separate concern from inventory append semantics.
- `releases` is the preferred named ArtifactHistory segment for release histories. Singular `release` examples and tests should be corrected rather than propagated.

## Contract Changes

- `weave version` must not rewrite an inventory file when it has no new settled facts to append. This does not block an explicit versioning request from appending a new named HistoricalState even when source bytes are unchanged, such as when `weave set next-state` or `--payload-state-segment` supplies the next state name; in that case the new state membership is itself the new settled fact.
- `weave` must not rewrite an inventory file merely because the renderer can produce a prettier or more complete canonical block.
- `weave generate` must not remove existing inventory ResourcePage facts as a side effect of page generation policy; it should append new page facts only when the policy says the page surface is materialized or promised.
- Composed branch-published release operations must be idempotent for unchanged source bytes, source metadata, target designator, target `releases/<version>` state, and config.
- A requested append that contradicts a single-valued settled fact must fail before writes with an error that names the existing fact and the requested fact.
- Historical state files and historical inventory snapshots remain immutable once written.
- Metadata files may update mutable progression facts during ordinary runs, but those updates should be narrowly scoped to the predicates the operation owns.

## Testing And Receipts

- Add unit coverage for an inventory append planner: duplicate triples no-op, new triples append, conflicting settled triples fail, unknown existing triples are preserved byte-for-byte.
- Add tests that normal `weave` over an already-settled workspace produces no inventory writes when source and config are unchanged.
- Add tests that a composed branch-published release rerun over the same source commit leaves publication inventory files unchanged.
- Add tests that a new release state appends `sflo:hasHistoricalState <D/releases/vX.Y.Z>` to inventory while writing `sflo:latestHistoricalState` and next-state progression to metadata.
- Add guardrail tests that newly generated inventory files do not contain mutable progression predicates such as `sflo:currentArtifactHistory`, `sflo:latestHistoricalState`, `sflo:nextHistoryOrdinal`, or `sflo:nextStateOrdinal`.
- Add regression coverage for the source-registry/reference-catalog case: adding `_sources` must not drop `_references`, and neither path should rewrite unrelated inventory facts.
- Update fixture expectations after the inventory/meta split; do not add long-lived compatibility shims for stale pre-v1 fixture shapes.

## Non-Goals

- Do not design a full merge/conflict-resolution language for arbitrary RDF graphs in this task.
- Do not make Turtle formatting canonicalization part of normal inventory writes.
- Do not promise immutable generated HTML bytes; ResourcePage regeneration has its own policy surface.
- Do not add backward-compatibility shims for old pre-v1 inventory shapes unless an implementation slice proves a very narrow temporary reader is unavoidable.
- Do not solve privacy/security retraction UX beyond reserving explicit repair/retraction modes.

## Required Code Changes

- Introduce a shared inventory append planner/writer, likely under `src/core/weave` or a small RDF utility module, that parses current inventory with the RDF parser, compares requested triples semantically, and produces append/no-op/conflict results.
- Replace inventory mutation paths that call subject-block replacement or canonical re-rendering with the append planner. Primary targets include MeshInventory update helpers, KnopInventory weave renderers, source-registry insertion in `src/runtime/deploy/gh_pages.ts`, and support-artifact preservation helpers around `renderKnopInventoryWithPreservedSupportArtifacts`.
- Move remaining current/progression writes out of inventory renderers and into metadata writers. MeshInventory is already partially split; KnopInventory, payload histories, ReferenceCatalog histories, ResourcePageDefinition histories, and source-registry-related progression need the same treatment.
- Update runtime readers so current/latest resolution reads metadata/progression graphs rather than assuming the current pointers live in inventory. The reader should reject conflicting metadata/inventory current-pointer facts rather than picking one silently.
- Change ResourcePage policy filtering so it prevents appending disallowed new page facts and controls generated bytes, but does not remove already-settled inventory facts during ordinary generation.
- Update branch-published source-registry handling so linking `_knop/_sources` from Knop inventory is append-only and idempotent, while source registry detail replacement/retargeting remains outside inventory.
- Correct CLI/test examples that use `--payload-history-segment release` for ArtifactHistory IRIs to `--payload-history-segment releases`.

## Required Documentation Changes

- Replace the stale TODO entry about graph-preserving subject rewrites with this append/no-op/fail-closed task.
- Update [[wd.codebase-overview]] after implementation to describe inventory as settled additive membership and metadata as the current/progression graph.
- Add a [[wd.decision-log]] entry before closing the task that records the append-onlyish inventory contract and the explicit exception modes.
- Update [[wu.cli-reference]] to use `releases` consistently for release ArtifactHistory examples.
- Update or add user-facing release automation guidance that says automated reruns should use composed mesh/source/publication operations, stable source refs/commits, and explicit `releases/<version>` targets, and that unchanged reruns must not rewrite inventory.
- If the storage split becomes part of the portable Semantic Flow contract rather than only Weave runtime behavior, update the relevant SFLO ontology summary/spec notes to clarify settled inventory facts versus metadata-hosted progression facts.

## Exit Criteria

- Every normal-operation inventory writer either uses `planInventoryAppend`/`renderInventoryAppendPlan` or proves an equivalent append/no-op/conflict contract through a single shared abstraction.
- Existing bytes and unknown settled facts are retained; new settled facts append; conflicting single-valued facts refuse before writes.
- Mutable progression ownership is ruled and implemented without leaving conflicting pointers in inventory and metadata.
- ResourcePage generation policy never silently deletes settled facts; explicit repair/retraction remains a separate named mode.
- Whole-command reruns and release-shaped fixtures prove byte-stable no-op behavior where no new settled fact exists.
- Required fixture, developer, user, release-note, and decision-log updates land with full Weave gates green.
- Every child task is completed/cancelled or an explicit ruled-off branch, and no actionable migration remains only in this plan's prose.

## Plan Checklist

- [x] Add and harden the shared RDF-aware inventory append planner and renderer.
- [x] Migrate current-only ReferenceCatalog weave, first/later `knop.create`, FoundingReferentData settlement, and import source-registry insertion.
- [x] Stop routine ResourcePage generation from deleting settled inventory facts.
- [x] Deliver [[wa.completed.2026.2026-08-31_0106-add-reference-inventory-append]].
- [x] Deliver [[wa.completed.2026.2026-08-31_0845-batched-extracted-mesh-inventory-append]].
- [x] Deliver [[wa.completed.2026.2026-08-31_1026-versioned-sequential-extracted-mesh-inventory-append]].
- [x] Deliver [[wa.completed.2026.2026-08-31_1050-current-only-payload-like-mesh-inventory-append]].
- [ ] Deliver [[wa.task.2026.2026-08-31_1111-versioned-first-knop-mesh-inventory-append]].
- [x] Deliver [[wa.completed.2026.2026-08-31_1127-current-only-page-definition-inventory-append]].
- [x] Deliver [[wa.completed.2026.2026-08-31_1147-current-shape-extract-mesh-inventory-append]].
- [ ] Cut and deliver the MeshInventory weave child tasks.
- [ ] Cut and deliver remaining KnopInventory/PageDefinition and operation-specific writer tasks.
- [ ] Resolve and implement the progression-storage contract.
- [ ] Regenerate affected fixtures and update durable documentation.
- [ ] Run the final writer audit, full gates, and close this plan.

## Evidence audit + Kim brief — bite 1 (cut 2026-08-01, post-loop harvest)

Codex read-only audit against current code found this note partly stale: `planInventoryAppend` already exists with semantic `unchanged`/`append`/`conflict` plans, byte-stable no-op, and contract tests (`src/core/weave/inventory_append_planner.ts`, `_test.ts`); import's existing-payload `_sources` insertion already uses it; carried `_sources`/`_references` preservation consults it but applies facts to a freshly rendered inventory. The real remaining gap: normal inventory writers are still whole-document producers — MeshInventory growth splits/replaces subject blocks (`mesh_inventory_renderers.ts` ~28/179/428), KnopInventory and payload renderers emit mutable progression facts directly, later `knop create`/`add-reference`/extract/integrate render whole documents, and ResourcePage policy actively deletes disallowed page facts (`resource_page_policy.ts` ~141 — needs its own later slice).

### Bite 1: migrate current-only ReferenceCatalog weave to the append planner

File-disjoint from PRs #31/#32/#33 (touches only `knop_inventory_renderers.ts` + a NEW `knop_inventory_renderers_test.ts`); if work serializes, merge those PRs first and rebase. Change only `renderCurrentOnlyReferenceCatalogWovenKnopInventoryTurtle` (~line 304, currently `replaceSubjectBlock`/`upsertSubjectBlockAfter`) so a current-only ReferenceCatalog weave: preserves the entire existing KnopInventory byte-for-byte as a prefix; appends only missing settled facts (catalog types, its `sflo:hasWorkingLocatedFile`, `sflo:hasResourcePage <…/index.html>`, page types `sflo:ResourcePage` + `sflo:LocatedFile`); returns exact input bytes when all requested facts exist; fails with `WeaveInputError` naming requested and existing facts when the settled working-locator conflicts; keeps compact `sflo:` Turtle for the appended suffix; preserves semantic preconditions and full-plan behavior. Treat only `hasWorkingLocatedFile` as single-valued — `hasResourcePage` is not functional in the ontology.

Fail-on-old sequence: (1) append test whose input carries a comment and an unrelated `ex:` predicate — result must start with the exact input bytes (old code edits the block in place and rejoins); (2) exact no-op test requiring byte equality; (3) conflict test with expected + different carried locator requiring the named-facts diagnostic (old string-inclusion guard accepts it); (4) record failures against unmodified code; (5) implement via `planInventoryAppend` (planner is evidence — do not modify it); (6) rerun focused tests + the existing current-only ReferenceCatalog plan test WITHOUT editing `weave_test.ts`.

Validation: `deno task test --filter='planInventoryAppend|current-only ReferenceCatalog inventory'`; `deno fmt` on the two files; `deno task lint`; `deno task check`; `deno task build:npm-lib`; `deno task ci` (reviewer-side at landing).

Non-goals: no versioned-catalog history changes, no PageDefinition twin yet, no MeshInventory/payload/batch/metadata/progression refactor, no ResourcePage policy fix, no planner API redesign, no fixture regeneration or shims. Must not touch: `shape_assertions.ts`, `weave.ts`, `weave_test.ts`, `version_execution.ts`, `tests/integration/weave_test.ts`, the pending-heavy generator, MeshInventory/payload renderers, ResourcePage policy, import's migrated path, archive notes.

Report rather than implement: any evidence `hasResourcePage` needs single-valued treatment; any need to relax full-plan shape assertions; any exact-output fixture forcing active-lane edits; any global planner-serialization need; mutable progression facts in this current-only path; the generate-policy deletion defect (cite, separate bite).

Branch when fired: `lane/reference-catalog-append` off main (pre-created by the planning seat). Suggested commit: `fix(weave): append-only current-only ReferenceCatalog inventory weave`. End the return with `READ-IN/QUEUE DELTA: none | <what belongs where>`.

STATUS: brief READY; fire held until PRs #31–#33 merge or the loop re-arms (serialization per the carve's own lane-overlap analysis).

### Bite 1 receipt + residuals (2026-08-01)

DELIVERED on `lane/reference-catalog-append` (commit pending full gates): the current-only ReferenceCatalog weave now routes through `planInventoryAppend`. Fail-on-old recorded for all three tests, and the conflict probe found a real defect in the replaced code — the old substring guard accepted the requested locator appearing in an unrelated `ex:` predicate while a *different* working locator was carried, so a genuine conflict passed silently. New `knop_inventory_renderers_test.ts` deliberately kept out of `weave_test.ts`.

Residuals carried forward as later-bite evidence (per the return's delta line):

- **Compact planner serialization pressure.** `planInventoryAppend`'s append text expands RDF IRIs, so this slice locally compacts its own bounded suffix. If more append writers hit this, a shared compact serializer is warranted rather than per-consumer compaction — decide when the second or third consumer migrates, not now.
- **ResourcePage policy deletion defect (confirmed, unowned).** `resource_page_policy.ts` collects disallowed settled page paths (~line 141) and removes them (~line 197) — a direct violation of the append-onlyish contract, and a more serious one than block-rewriting because it *deletes settled facts*. Needs its own bite; it was also flagged independently in the evidence audit above.
- **`hasResourcePage` is not single-valued.** Verified against the core ontology (`owl:ObjectProperty`, no functional/max-cardinality constraint). Future append consumers must not assume otherwise.

Next consumers in the natural order: the PageDefinition twin renderer, then the whole-document writers (`knop create` later-path, `add-reference`, extract, integrate), then MeshInventory growth, with the ResourcePage policy deletion defect as its own correctness bite.

### Nit boarded 2026-08-01 (from the sflo publication)

The append path leaves a trailing blank line at EOF: in the regenerated sflo mesh exactly one file (`_knop/_inventory/inventory.ttl`, the root Knop whose inventory received a carried-source-registry append) ends with `\n\n` where the pre-append renderer emitted a single `\n`. Cosmetic and valid Turtle — it shipped in the published mesh — but `git diff --check` flags it, and Kim's bite-1 fail-on-old evidence already noted the old renderer "removed the trailing blank line". Whoever next touches the append planner should normalize the trailing newline.

### Bite 2 receipt — ResourcePage fact deletion (2026-08-07)

DELIVERED on `lane/resource-page-fact-deletion`. The confirmed correctness defect is fixed:
`filterResourcePageFactsFromInventoryTurtle` no longer deletes anything. `removeResourcePagePaths`
and its string-splitting machinery are gone; inventory Turtle and planned files are preserved
**byte-for-byte**, and HTML selection stays RDF-aware through `listGeneratedResourcePagePaths`.

**Option (a) chosen — retain settled facts, stop generating the disallowed page** — with evidence
rather than assumption. Kim checked that later planning treats retained page facts as settled state
without testing HTML existence, that validation has no ResourcePage file-existence invariant, and
that publication validation checks host readiness and path leakage rather than every
`hasResourcePage` target.

**Named cost of (a), accepted:** a retained fact whose page is no longer generated can leave a
dangling published link. It breaks no Weave validation, planning, rendering, or publication check —
but it is a real artifact of choosing append-onlyish over deletion, and the honest resolution is an
explicit repair/retraction mode, not silent removal. This is the same shape as the 2026-08-07 ruling
that states must be retractable *explicitly*.

Fail-on-old: `0 passed | 1 failed | 824 filtered out`. The old code removed both page facts **and**
both page subject blocks, **and** rewrote the surviving `ex:carried` predicate terminator — collateral
damage to an unrelated carried fact, which is the sharpest argument that string-based inventory
editing had to go.

### Remaining deletion-capable inventory paths (found 2026-08-07, unowned)

The audit asked for every other place that can destroy settled inventory facts. Three remain, in
rough order of risk:

1. **`knop add-reference`** (`src/core/knop/add_reference.ts:281`) — canonically rerenders the
   KnopInventory and only specially preserves *known* support artifacts, so **unrelated carried facts
   remain at risk**. This is live data-loss exposure, not merely a contract violation, and is the
   sharpest of the three.
2. **MeshInventory renderers** (`src/core/weave/mesh_inventory_renderers.ts:440`) — overwrite whole
   subject blocks; the batched extracted-Knop path explicitly filters target blocks before
   reconstruction.
3. **Later `knop create`** (`src/core/knop/create.ts:780`) — replaces the `_mesh` block wholesale.

Each wants the `planInventoryAppend` treatment. Sequence them by exposure: `add-reference` first.
