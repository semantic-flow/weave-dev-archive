---
id: watask20260729validmembound
title: 'Whole-mesh validate under a bounded memory budget'
desc: 'Make untargeted weave validate mesh complete on ~10^4-file / ~2x10^3-Knop meshes without approaching the ~4.09 GiB default V8 heap ceiling observed in the srd-rules environment: instrument and pin the actual retention profile, eliminate the identified accumulation pathologies, and land a scale acceptance workload. Cut 2026-07-29 12:20 by Jimbo from wd.consumer-feedback.0.5.1.2026-07-29_1213 §3. Codex r1 corrections folded 2026-07-29.'
---

## Goals

- Untargeted `weave validate mesh` completes on meshes in the ~10⁴-file / ~2×10³-pending-Knop range with peak memory bounded by the largest single validation unit plus compact whole-mesh indexes — not by pending-Knop count × mesh size.
- Raising the V8 heap is explicitly not the fix (anti-goal); the observed ~4 GiB default ceiling is treated as an external constraint, exactly as `wd.todo` already treats it for the extract/weave scale work.
- Validation findings and CLI output semantics are unchanged; this is an engine slice, not a behavior slice.
- **Instrumentation before design commitment:** the recorded exhaustion's dominant retention term is not yet attributed (see Mechanism); the first implementation gate measures retained bytes by role on a policy-faithful workload and *selects* the primary fix from evidence.

## Summary

Stagecraft's 2026-07-29 follow-up ([[wd.consumer-feedback.0.5.1.2026-07-29_1213]] §3) reports that on `weave 0.3.0` an untargeted `validate` over a ~6,900-file mesh exhausted a "fixed 4 GiB heap", flags honestly that they can no longer reproduce it (that mesh was deleted; their largest remaining mesh is 442 files and validates fine on both `0.3.0` and `0.5.1` at ~185–193 MB), and asks whether whole-mesh validation is expected to scale. **We do not need the ghost chased — this repo already holds same-corpus evidence:** the phase-A receipt of the srd-rules run recorded the untargeted exhaustion on the same ~6,900-file corpus, and the extractor-lane R2 diagnosis ([[wa.task.2026.2026-07-21_1603-extractor-defect-pair]], archive `origin/main`) measured the ceiling and the targeted-path costs. A 2026-07-29 architecture sweep of `main` at `c962ab2` identified several real accumulation pathologies in the untargeted path (below), though the dominant term for the specific recorded exhaustion remains to be pinned by instrumentation. Their inability to reproduce is explained, not mysterious: settled meshes short-circuit the expensive path entirely, and their surviving 442-file mesh is settled, while the 6,900-file mesh was fresh extraction output with ~1,693 pending extracted-term Knops.

This note was adversarially reviewed by Codex on 2026-07-29 (r1 below, verdict CHANGES-REQUIRED); the required corrections are folded into this revision — most importantly, the retained-snapshot mechanism is now presented as policy-conditional rather than as the established dominant term.

## Answers for the reply (their three §3 questions, evidence-backed)

1. **Known ceiling?** There is no fixed heap allocation anywhere in Weave — no `--v8-flags`, `--max-old-space-size`, or `NODE_OPTIONS` in the compile args (`scripts/build-binaries.ts:212-229`), the npm wrapper shim (a plain `spawnSync` exec of the platform binary, `scripts/release/npm.ts:151-199`), or `deno.json`. The ~4 GiB they hit is V8's *default* old-space limit as observed in that environment: the R2 diagnosis measured `heap_size_limit` = 4,395,630,592 bytes (~4.09 GiB), identical with and without `DENO_V8_FLAGS=--max-old-space-size=4096` — establishing the observed default, not an unchangeable constant. Their "fixed 4 GiB heap" observation is correct; the attribution to a Weave allocation is not. **On 0.4.0/0.5.1:** the recursive-loop sources are unchanged between the tags (`planning_context.ts`, `candidate_loader.ts`, `mesh_state.ts`, and the runtime `weave.ts` are snapshot-identical; the `prepareVersionExecution` body is unchanged), and no known memory-affecting change to the validate path exists — but `version_execution.ts` also gained the lazy repository-source seam and transitive config loading changed, and no cross-version pending-heavy measurement exists, so profile identity is *expected, not proven*. Their 442-file table proves settled-path equivalence only.
2. **Expected to scale?** Intended, yes — whole-mesh validation green at 10⁴-file scale is the product intent, and this note owns delivering it. Currently it does not scale on pending-heavy meshes, for the architectural reasons below. Targeted validation is the present mitigation, not the endorsed long-term pattern.
3. **Their reconstruction offer:** accept, gratefully, as independent confirmation of our acceptance workload — but not as a blocker. We can synthesize a pending-heavy mesh ourselves, sharing the ~1,700-term regression-workload substrate already boarded for the extract/weave scale task (`wd.todo` TODO 23), and a policy-faithful reconstruction from them is exactly the right instrument for pinning the dominant term (question 1 of the Implementation Plan).

## Mechanism (measured 2026-07-29, `main` at `c962ab2`; corrected per Codex r1)

`weave validate mesh` is not a validator with its own traversal — it is a dry run of the full recursive version planner: `executeValidate` (`src/runtime/weave/weave.ts:117-215`) → `prepareVersionExecution` (`src/runtime/weave/version_execution.ts:255-625`) plans one pending candidate per loop iteration, then `validateVersionPlanRdf` parses the final plan's `.ttl` files, then publication-preset checks run (already streaming). **Why a settled mesh is cheap (and why their repro vanished):** when nothing is pending, `prepareVersionExecution` short-circuits before the loop (`version_execution.ts:440-459`); untargeted validate on a settled mesh never enters the accumulation path. The consumer's srd mesh was fresh extraction output — ~1,693 never-woven extracted-term Knops — the pending-heavy shape that maximizes it.

Verified accumulation pathologies, with their conditions:

- **Retained plan text, append-only.** Every planned created file's full contents accumulate in the `createdFiles` array for the whole command (`version_execution.ts:468-473`, `:550-567`); repeated updates are deduplicated to latest-per-path in `updatedFileByPath` (`:569-579`). Planned files are staged into the overlay as well (`planning_context.ts:73-86`).
- **Quadratic snapshot retention — conditional on history-materializing policy (Codex F6).** When the effective mesh-inventory history policy materializes history, each first-Knop/first-payload plan emits a fresh full MeshInventory copy as a new history-state file at an advancing path (`src/core/weave/weave.ts:1110-1145`, `:1240-1259`; extracted-term arm policy gate at `:1334-1346`, `:1423-1430`), and each distinct snapshot is retained in `createdFiles` ⇒ ~O(N²) retained text. **Under `currentOnly`, this arm does not fire** — the planner only updates the shared current inventory path (`:1384-1394`, `:1477-1481`), which dedups to one retained copy — and the archive receipt records that the srd mesh's effective current-only policy avoided the history arm. **The dominant retained term for the recorded current-only exhaustion is therefore NOT yet attributed**; candidates, read cache, staged plan text, and parse churn are the suspects the baseline instrumentation must separate.
- **Unbounded command-scoped caches, by design.** `TextFileOverlay` (`src/runtime/weave/planning_context.ts:14-102`) retains every file ever read (`#readCache`) and every loaded candidate object (`#candidateCache`) for the whole command; [[wd.codebase-overview.caching]] documents no-eviction-within-a-command as intentional. (Correction per Codex F7: the MeshInventory is parsed outside candidate dependency capture, so it is *not* automatically a captured dependency of every candidate, and the earlier claim that candidate-cache invalidation fires essentially every iteration is withdrawn — actual hit/invalidation counts are an instrumentation deliverable.)
- **Candidate retention shape (corrected per Codex F7).** Candidates retain each text-like payload as text (bytes discarded), binary payloads as bytes plus decoded text, and latest snapshots as text or bytes by file type (`src/runtime/weave/artifact_loaders.ts:87-94`, `:109-123`, `:139-157`). The earlier "text and bytes simultaneously, generally" claim is withdrawn; actual retained candidate bytes on the baseline workload are an instrumentation deliverable.
- **O(N²) parse *work* (churn, transient).** Every loop iteration re-runs `loadMeshState` and re-parses the full MeshInventory (`version_execution.ts:476-496`, `candidate_loader.ts:87-96`); parsed quads are throwaway per call (`src/core/weave/rdf_helpers.ts:14-24`). High allocation churn even where retention is linear.

**Cross-file read set — hypothesis, not established (Codex F9).** Beyond the MeshInventory, extracted candidates read their source Knop inventory and selected source historical state (`artifact_loaders.ts:222-274`, `:300-335`), and effective target config reads ancestor Knop metadata (`execution_config.ts:205-229`). Any cache-eviction or skip-staging design must define safety in terms of *observed* future reads (instrumented), not an assumed MeshInventory-only coupling. The overlay latest-per-path law stands: later Knops must observe earlier staged advancement ([[wd.performance]], [[wd.codebase-overview.caching]]).

## Design direction (sketch — the spec review rules the exact plan; the baseline instrumentation selects the primary fix)

- **Instrumentation first.** Extend `WEAVE_TIMING`-style output with retained-bytes accounting by role (created plan text, staged overlay, read cache, candidates) plus peak-RSS/heap statistics; run it on a policy-faithful pending-heavy workload (current-only *and* history-materializing variants) to pin the dominant term before committing to a primary fix.
- **Plan-and-drop validation sink.** For validation, a planned file's contents need to exist only long enough to parse-check. Give `prepareVersionExecution` a validate-mode sink that parses planned `.ttl` content at plan time and retains only findings plus compact metadata. Per Codex F8, the sink must distinguish **unique creates** (parse once, drop) from **latest-per-path updates** (today only the *final* deduplicated content is validated, fail-fast on first error — parsing transient intermediate update states would change semantics); the sink preserves final-state-only validation and first-error behavior unless the spec review explicitly amends them.
- **Validation-only execution mode, fenced (Codex F10).** `prepareVersionExecution` also serves `version` and full `weave`; the sink and any cache-release behavior are parameterized to validate mode, with unchanged-behavior regressions for `executeVersion` and `executeWeave`. Reusable infrastructure (e.g. a reused MeshInventory read model) that would also serve TODO 23's write-path work is coordinated with that item explicitly — shared substrate, single owner per seam.
- **Stop staging what nothing reads / bound the caches — evidence-gated.** If instrumentation shows staged snapshot files or completed-candidate cache entries are never re-read, skip/release them in validate mode; eviction safety is defined by the observed dependency read set (including source-Knop and ancestor-config reads), keeping the documented invalidation laws intact.
- **Parse the MeshInventory once per staged generation, not per iteration** — coordinate with the RDF parse/render boundary cleanup task; the parked Oxigraph decision stays parked.
- **Progress output** at scale ties into the existing validation-progress backlog item (quiet/default/verbose policy).

## Acceptance Criteria (proposed budgets; ratify at spec review)

- Untargeted `weave validate mesh` over a synthetic pending-heavy mesh at ~2×10³ pending Knops / ~10⁴ files completes rc 0 with peak RSS under a ruled budget (proposed: < 1 GiB), receipt via `/usr/bin/time -v`, under BOTH current-only and history-materializing mesh-inventory policies.
- A CI-safe proportionality guard discriminates linear from super-linear retention at small scale (N ∈ {~50, ~200} pending Knops), asserting on the internal retained-bytes accounting counters rather than RSS (noisy in CI).
- Findings parity: identical findings before/after this slice on settled, pending-heavy, and defect-seeded fixtures — including a fixture where one updated path is planned in multiple iterations, proving final-state-only validation is preserved (Codex F8).
- `executeVersion` and `executeWeave` behavior unchanged (regression-pinned), targeted validation stays in its current class (the R2-measured targeted run was ~244 MB RSS / ~0.44 s at 1,698-Knop scale).

## Testing

- Synthetic mesh generator as test tooling (parameterized Knop count and history policy, `createTestTmpDir()`-based, no fixture-repo refs) — shared substrate with the boarded ~1,700-term extract/weave regression workload where practical.
- Fail-on-old at reduced scale via the retained-bytes counters (the honest discriminator), not a deliberate 4 GiB OOM in CI — the extractor-lane precedent stands: another deliberate exhaustion adds risk without discriminating anything. One manual full-scale before/after receipt (old trajectory vs new peak) recorded in this note at lane end.
- Multi-iteration updated-path parity fixture; unchanged-behavior regressions for the write-path callers.
- Full `deno task ci` green; instrumentation covered by focused tests.

## Non-Goals

- Extracted-term weave/extract batch viability — `wd.todo` TODO 23 owns the write path; shared infrastructure is coordinated explicitly, not silently co-owned.
- The `validateMesh` API contract (sibling note [[wa.completed.2026.2026-07-29_1219-programmatic-validate-mesh-api]]); the acceptance bench here runs through both surfaces once both land.
- Oxigraph adoption; raising the heap via `--v8-flags` as a fix (documenting the observed default ceiling in `wu.*` docs may ride the lane); write-path streaming; publication-preset changes (already streaming); CLI output/format changes.

## Implementation Plan

- [ ] Spec review r1 (instrumentation-first gate, sink rules for creates vs latest-per-path updates, fail-fast preservation, validate-mode fencing + TODO 23 coordination, eviction-safety law, budget numbers, CI metric); amendment; PM GO.
- [ ] Retained-bytes/RSS instrumentation + synthetic mesh generator (policy-parameterized) + baseline receipts on current `main`; **pin the dominant retention term for the current-only pending-heavy shape and select the primary fix from that evidence.**
- [ ] Plan-and-drop validation sink in the `prepareVersionExecution` seam (validate mode only), preserving final-state-only validation, first-error behavior, and the overlay latest-per-path law.
- [ ] Evidence-gated cache release/staging reduction with invalidation laws intact.
- [ ] Reused MeshInventory read model across loop iterations (coordinated with TODO 23 / RDF-boundary cleanup ownership).
- [ ] Acceptance bench receipts at 10⁴ scale (both history policies) + CI proportionality guard + write-path regressions.
- [ ] Docs: [[wu.cli-reference.validate]] scaling note, [[wd.performance]] and [[wd.codebase-overview.caching]] updates.
- [ ] Reply input: the three §3 answers above + acceptance of their reconstructed-mesh offer as the policy-faithful instrumentation corpus.

## Open Issues

- The ruled peak-RSS budget number and the exact CI-safe proportionality metric.
- What the baseline instrumentation names as the dominant retained term for the current-only pending-heavy shape (candidates vs read cache vs staged/created plan text vs churn-driven heap growth) — this selects the primary fix.
- Whether staged historical snapshots and completed-candidate entries are ever re-read during later planning (observed, not assumed; determines the cheapest big win).
- Whether the reused MeshInventory read model becomes the shared parse-once substrate TODO 23 also needs, and which task owns it.

## Spec review r1 — Codex (2026-07-29, adversarial, read-only sandbox)

Findings as reported (severities are Codex's proposals): **F6 BLOCKER** — the claimed dominant O(N²) retained-snapshot mechanism does not describe the cited srd workload: the snapshot arm is policy-conditional and the srd mesh's effective current-only policy avoided it; the cited `progression_resolvers.ts` ranges were unrelated (page-definition / KnopInventory progression). **F7 HIGH** — two supporting claims false: the MeshInventory is not automatically a captured dependency of every candidate (so per-iteration invalidation was overclaimed), and candidates do not generally retain text+bytes simultaneously. **F8 HIGH** — parse-at-plan-time changes validation semantics unless the sink distinguishes unique creates from latest-per-path updates and preserves fail-fast, final-state-only behavior. **F9 HIGH** — the asserted load-bearing read set was unverified and materially incomplete (source-Knop inventory, source historical state, ancestor-config reads). **F10 HIGH** — cache/read-model work overlaps TODO 23 through the shared `prepareVersionExecution` seam; validate-mode fencing and write-path regressions required; the `--validate-before/--validate-after` deferral question was moot (same `executeValidate` function). **F11 MEDIUM** — release-diff and heap conclusions stated stronger than their evidence (lazy repo-source seam + transitive config changes exist between tags; the measurement establishes the observed default ceiling, not a fixed constant).

Disposition (this revision, 2026-07-29): all six folded — the mechanism section now separates verified pathologies from the unpinned dominant term and withdraws the two false claims with dated corrections; instrumentation-first is the explicit gate that selects the primary fix; the sink rules, validate-mode fencing, write-path regressions, and evidence-gated eviction are in the design and plan; the reply answers were softened to what the evidence proves; the moot open issue was removed. Codex's proposed verdict was **CHANGES-REQUIRED**; the required changes are applied in this revision, with the instrumentation gate standing as the first implementation item.

## Build receipts — instrumentation/baseline phase (2026-07-30)

Implementation seat: Codex (`codex exec`, workspace-write sandbox, `model_reasoning_effort=high`). PM direction 2026-07-30 authorized proceeding directly to the instrumentation gate (it needs no product rulings). Deliverables landed on branch `lane/validate-memory-baseline` (two commits: `3e0dd68` `feat(validate): instrument retained memory`, `b91aca5` `test(validate): add pending-heavy baseline mesh`). Sandbox note: the run's `.git` mount was read-only, so the commits were minted in a shadow repo and fetched into the real repository afterward; the real worktree carries the identical content (verified byte-identical file-by-file against the branch tip).

- **A. Instrumentation:** opt-in `WEAVE_MEMORY_STATS=1` JSON on stderr (`src/runtime/weave/memory_stats.ts`; overlay/cache accounting in `planning_context.ts`; wiring in `executeValidate` + `prepareVersionExecution`). Disabled path: no scanning, no RSS sampling, no `node:v8` import (dynamic, on finish only). `src/api/fs_purity_test.ts` green. No behavior change.
- **B. Generator:** `scripts/generate-pending-heavy-mesh.ts` — creates and settles a flat `source` Knop, authors N mesh-scoped terms, extracts all; parameterized `--mesh-inventory-history current-only|versioned`. Honesty proven: untargeted validate performs exactly N recursive iterations (`tests/scripts/pending_heavy_mesh_test.ts`).
- **C. Baselines** (raw receipts under `/tmp/weave-validate-memory-baseline-final.ozIbVa`):

| Policy | N | Wall | Peak RSS | Iter | MeshInv history snapshots | updated map | overlay staged | read cache | end V8 used |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| current-only | 50 | 0.80s | 212.8 MiB | 50 | 0 / 0 B | 176 KB | 176 KB | 154 KB | 74.2 MiB |
| current-only | 200 | 3.50s | 287.7 MiB | 200 | 0 / 0 B | 695 KB | 695 KB | 590 KB | 25.0 MiB |
| current-only | 500 | 16.71s | 445.0 MiB | 500 | 0 / 0 B | 1.73 MB | 1.73 MB | 1.46 MB | 244.2 MiB |
| versioned | 50 | 1.20s | 227.9 MiB | 50 | 50 / 2.55 MB | 230 KB | 2.78 MB | 156 KB | 99.1 MiB |
| versioned | 200 | 9.21s | 575.3 MiB | 200 | 200 / 36.8 MB | 899 KB | 37.7 MB | 593 KB | 319.3 MiB |

- **Growth:** current-only retained text is linear (≈N-proportional) while wall goes super-linear (2.5× N → 4.77× wall at the 200→500 interval, consistent with O(N²) parse work). Versioned MeshInventory-snapshot retention grew 14.4× at 4× N — the conditional O(N²) arm, confirmed.
- **Evidence verdict (supersedes the design lean):** for the **current-only** pending-heavy shape — the consumer's shape — every counted retained role totals under ~5 MB against 445 MiB peak RSS at N=500; retention of plan text is NOT the dominant term. The dominant term is loop allocation/heap occupancy consistent with repeated full-MeshInventory parsing. **Plan-and-drop is demoted for the current-only shape; parse-reuse/churn-reduction is the evidence-selected primary direction.** For the **versioned** shape, snapshot retention dominates and plan-and-drop/skip-staging remains the right lever.
- **D. Gates:** `deno task fmt` 243 files; `deno task lint` 242 files; focused instrumentation/generator/integration/fs-purity 39/39; `deno task ci` 722/723 under this workspace's injected `NO_COLOR=1` (the pre-existing ANSI byte-stability test), 723/723 with `env -u NO_COLOR`; LCOV generated.

### Adversarial review of the build (Jimbo, 2026-07-30) — ACCEPTED, three findings

Independently verified: fs-purity + memory-stats tests green; fresh N=20 current-only mesh generated and untargeted validate reported exactly `planningLoopIterations: 20` with sane counters; disabled run emits nothing; worktree byte-identical to the branch tip; no `documentation/notes/` or `dependencies/` paths touched by the commits.

- **R1 (attribution caveat):** end-of-run V8 used-heap is noisy (74 → 25 → 244 MiB across current-only N=50/200/500) and 244 MiB at N=500 vastly exceeds the ~5 MB counted retention — GC-lag vs uncounted-retention is not yet discriminated. The churn verdict is supported by wall-time super-linearity but needs a forced-GC measurement (`--v8-flags=--expose-gc`, collect before reporting) or a heap snapshot before it is final.
- **R2 (magnitude gap):** the synthetic per-term content is far smaller than srd's (~1 MB MeshInventory, real payloads); current-only peak RSS at N=500 is 445 MiB with sub-linear RSS growth, which does not obviously extrapolate to the recorded ~4 GiB exhaustion at N≈1,693. Before final attribution, the generator needs a payload-content-size knob and one N≈1,700 srd-scale run (their offered reconstructed mesh is the ideal corpus).
- **R3 (mechanics):** shadow-repo commit flow worked but leaves the real checkout on `main` with the branch content uncommitted in the worktree; resolve by checking out the branch (or stashing) before the next build slice.

Next slice (evidence-directed): forced-GC/heap-snapshot attribution run + content-size-parameterized N≈1,700 baseline, then ratify the primary fix (parse-reuse for current-only; plan-and-drop/skip-staging for versioned) at spec review r1.

### Attribution follow-up receipts (2026-07-30, second Codex slice — closes R1 and R2)

Deliverables (commit `5a994c4` `feat(validate): discriminate retained memory after gc` on `lane/validate-memory-baseline`, installed from the sandbox shadow repo): post-GC discrimination in the memory-stats report (`postGcUsedHeapSize`, `null` when `--expose-gc` absent; JSON shape stable) and a deterministic `--term-content-bytes` knob on the generator (honesty property re-proven: exactly N iterations). Gates: fmt 243 / lint 242 / focused 4-4; full ci intentionally not run for this measurement slice. Raw receipts: `/tmp/weave-validate-memory-attribution.RQhLYt`.

Measurements (current-only, `--expose-gc`, `/usr/bin/time -v`):

| N | Content/term | Wall | Peak RSS | Counted roles | Pre-GC heap | Post-GC heap |
|---:|---:|---:|---:|---:|---:|---:|
| 500 | 0 B | 16.5s | 441.9 MiB | ~4.9 MB | 248.3 MB | **14.4 MB** |
| 1700 | 0 B | 3m16s | 1.96 GiB | ~10.8 MB | 1.537 GB | **15.0 MB** |
| 500 | 1024 B | 18.1s | **2.42 GiB** | ~4.9 MB (unchanged) | 1.105 GB | 15.3 MB |
| 1700 | 1024 B | 14.2s → **V8 OOM** | **4.14 GiB** | not emitted (died in-loop) | — | — |

**Attribution verdict (R1 + R2 closed):**

1. **Content-0 churn CONFIRMED (R1).** Post-GC heap collapses to the counted-retention order (~14–15 MB) at both N=500 and N=1700 — the RSS is allocation/parse churn plus GC lag, not hidden retention. Churn alone reaches 1.96 GiB RSS at N=1700 (~48% of the ceiling).
2. **The srd OOM is REPRODUCED (R2).** At the recorded cardinality (N=1700) with srd-like ~1 KB/term content, validate hits V8's actual limit ("Reached heap limit", last-resort full GC at 4095.4 MB, 4.14 GiB peak RSS) and dies in 14 seconds. The 2026-07-21 exhaustion is no longer an anecdote — it is a reproducible benchmark.
3. **Content-sensitive mechanism identified: per-candidate full-source duplication.** Every pending extracted candidate carries the full source payload text in BOTH `currentPayloadTurtle` and `latestHistoricalSnapshotTurtle`, and the loop's candidate list retains them all: with the 1.9 MB generated source, ~N × 2 × sourceSize ≈ 6.5 GB implied at N=1700. Supported by the N=500 probe: +2 GB peak vs content-0 while every counted role stayed identical — the retention lives in the candidate live-set, which the role counters do not cover.

**Adversarial review of the follow-up (Jimbo, 2026-07-30) — ACCEPTED.** Independently verified: 4/4 focused tests; `postGcUsedHeapSize` present and sane on an untouched pre-knob N=20 mesh (31.1 → 14.5 MB); commit path-scoped, doc notes untouched. New open items for the fix design:

- **R4 (copies vs references):** the +2 GB delta at N=500 implies the per-candidate source strings are distinct copies, not shared references to one read-cache string. The fix slice must locate where the copy is minted (decode path vs slicing); if candidates can share one immutable source string, the dominant content term collapses outright — dedup/lazy-load is the obvious lever either way.
- **R5 (blind spot):** the candidate live-set is invisible to the role counters (`candidateCache` reported 0 while gigabytes sat in candidates). The fix slice adds a live-set counter.
- **R6:** synthetic MeshInventory reached 422 KB vs srd's 1.06 MB — same order, close enough for attribution; noted, not blocking.
- Honest deviations recorded: the first N=1700/content-0 attempt was discarded (runner vanished; rerun completed); the OOM run has no finish JSON by nature.

**Evidence-ratified fix direction for spec review r1:** primary — eliminate per-candidate full-source duplication in validate-mode candidate loading (share/dedupe or lazy-load-and-drop); secondary — parse-reuse to cut the churn term (1.96 GiB at N=1700 from churn alone is still uncomfortable headroom); versioned-policy snapshot arm keeps plan-and-drop/skip-staging as previously recorded.

**Landing:** this lane lands SECOND, rebased onto main after `lane/validate-mesh-api` merges — the two lanes collide on `weave.ts`/`version_execution.ts` and this one's overlap is the small additive side. The full landing plan (D1–D4) is framed in [[wa.completed.2026.2026-07-29_1219-programmatic-validate-mesh-api]]; the fix slice cuts only after both lanes are on main.

**LANDED (2026-07-31):** rebased onto post-#25 main (conflicts exactly the two predicted files; `memoryStats` wiring re-threaded through the classified `executeValidate` signature), one dnt-shim fix rode the lane (`929d5e8`: `TextEncoder` used as a type annotation — dnt's Node type-checking has only the shimmed value), full ci 737/737 + npm-lib build/smoke green, merged as PR #26 (`c0b25b3`).

## Fix-slice design (amendment r2 — ruled 2026-07-31 under the PM's blanket GO; build is evidence-gated on R4)

**Primary lever — share, don't copy (the content term).** R4 hypothesis to be PROVEN before any implementation: `readPayloadFileWithOverlay` (`src/runtime/weave/artifact_loaders.ts:166-183`) reads and decodes a FRESH text string per call — it consults only the overlay's staged map (`overlay?.get`), not the command-scoped text read cache — so every pending extracted candidate mints its own copy of the same source payload text (`currentPayloadTurtle`) and snapshot text (`latestHistoricalSnapshotTurtle`): N × 2 × sourceSize live across the loop. Ruled fix, contingent on that proof: route payload text reads through the command-scoped read cache so all candidates hold references to ONE immutable string per distinct path (≈ 2 shared strings at the srd shape instead of ~3,400 copies), registering the path as a candidate-cache dependency (which also closes the earlier dependency-capture gap for payload paths). Binary-payload bytes handling unchanged. Sharing immutable strings is behavior-neutral for validate AND the write paths — byte-out tests protect.

**Instrumentation rider (R5):** a candidate live-set retained-bytes counter, so the fixed shape is CI-guardable: shared-source retention must be O(sourceSize), not O(N × sourceSize).

**Deliberately deferred, boarded:** the parse-churn term (a true fix is an incrementally-updated parsed MeshInventory model — owned jointly with the RDF parse/render boundary cleanup and TODO 23, not this slice; churn alone completes at srd scale, 1.96 GiB peak, and collapses post-GC) and the versioned-policy snapshot arm (plan-and-drop/skip-staging; policy-conditional, not the consumer shape).

**Acceptance (revised from the earlier <1 GiB aspiration, which belongs to the deferred churn work):**
- N=1700 @ 1024 B/term, current-only: completes rc 0 (no OOM — today it dies at 14 s) with peak RSS ≤ ~1.25× the same-N content-0 run (content-size independence), post-GC used heap on the counted-retention order.
- N=500 @ 1024 B/term: peak drops from the 2.42 GiB class to the content-0 class (~450 MiB).
- Findings parity (settled / pending-heavy / defect-seeded), full `deno task ci`, and `build:npm-lib` + off-tree smoke run LOCALLY before push (the dnt-shim lesson, twice learned).
- New live-set counter proportionality: retained candidate source bytes stay flat as N grows.

### FIX LANDED (2026-07-31, PR #27 merged `0852de0`)

R4 gate PASSED before implementation: copies were minted by fresh `Deno.readFile` + per-call `TextDecoder` in `loadPayloadWorkingArtifact` (`artifact_loaders.ts:193-209` pre-fix) and the resolver's selected-history read (`resolver.ts:686-713`), both bypassing the command cache; N=200 proof showed payload sizes growing while read-cache bytes stayed byte-identical. Implementation (commits `df4587e` + `61b9e53`): text-like payload and historical-snapshot reads intern through the staged-first command cache (`readTextFileWithOverlay` gains `contentsIfUncached`; digest verification on bytes precedes interning; binary bytes path unchanged); payload paths now register as candidate dependencies; identity-deduplicated `candidateLiveSet` role added to `WEAVE_MEMORY_STATS`; shared-identity + proportionality tests added.

**Acceptance, measured (implementer) and independently reproduced (reviewer):** N=1700 @ 1 KB/term: V8 OOM at 14 s / 4.14 GiB → **rc 0, 3m27s, 514 MiB peak** (1.035× the content-0 run; ruled ceiling 1.25×). N=500 @ 1 KB: 2.42 GiB → 369–378 MiB. N=1700 @ 0 B: 1.96 GiB → 496 MiB. At N=1700 the 3,400 source-text references collapse to 2 cache identities / 3.8 MB. Gates: full ci 739/739 (re-earned by reviewer), `build:npm-lib` + off-tree Node smoke green locally before push (both seats). **The whole-mesh validation green Stagecraft never had now exists at their scale.** Remaining on this note: the deferred parse-churn term and versioned-arm plan-and-drop (boarded above); the ~10⁴ acceptance bench through both CLI and API surfaces rides ordinary regression use of the generator.

### FIX LANDED (2026-07-31, PR #27 → main `0852de0`)

R4 gate PASSED before implementation (N=200: +210 KB source/snapshot text left the read cache byte-identical while peak RSS rose 292 → 665 MiB — the cache-bypass signature; both fresh-decode sites traced). Implementation per the ruled arm: payload/snapshot text reads route through the command-scoped overlay read cache (seeded variant for the resolver's already-decoded text); payload paths register as candidate dependencies; identity-deduplicated candidate live-set counter added (commits `df4587e` + `61b9e53`; seat: Codex-as-Kim, interrupted mid-acceptance by a session restart — the reviewing seat verified the installed commits byte-matched the gated worktree, re-earned every gate, and ran the acceptance matrix itself).

**Acceptance results (all criteria EXCEEDED):**

| Run (current-only) | Before | After |
|---|---|---|
| N=500 @ 1 KB/term | 2.42 GiB peak RSS | **373 MiB** |
| N=1700 @ 1 KB/term (srd scale) | **V8 OOM at 14 s** (4.14 GiB) | **rc 0, 3m23s, 554 MiB** |
| N=1700 @ 0 B | 3m16s, 1.96 GiB | 3m26s, **485 MiB** |

Live-set at N=1700: 3,400 source-text references → **2 distinct identities** (~3.8 MB). Bonus beyond the ruled scope: sharing also eliminated the per-iteration re-read/decode churn, so the content-0 peak fell ~4× with no wall-time regression. Gates: full ci 739/739, `build:npm-lib` + off-tree smoke green locally before push. **The consumer's §3 report is now: reproduced, attributed, fixed, and regression-guarded.** Remaining on this note: the deferred parse-churn term (incremental MeshInventory model — RDF-boundary/TODO-23 coordination) and the versioned-policy snapshot arm, both boarded, neither urgent at current evidence.
