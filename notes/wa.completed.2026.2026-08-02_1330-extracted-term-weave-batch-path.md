---
id: mrh7559hrqyutlujcprm4d4
title: Extracted-term weave batch path
desc: 'Carved 2026-08-02 from the closed extractor defect pair: get extracted candidates onto the batch path so extract→weave→generate has headroom at thousand-term nested-source scale'
updated: 1785702640000
created: 1785702640000
---

## Goals

- Give extracted-term weave the same one-transaction batch treatment that `firstPayloadWeave` candidates got in `v0.7.0`, so peak memory and wall time stop growing with the per-candidate recursive loop.
- Restore real headroom under V8's ~4.09 GiB default old-space ceiling at N=1,700, rather than the ~300 MiB margin measured on 2026-08-01.
- Encode the result as a durable regression workload so the curve cannot silently regress.

## Summary

Carved from [[wa.completed.2026.2026-07-21_1603-extractor-defect-pair]] on 2026-08-02. That note's own scope — the two alleged extractor defects — is delivered and closed: F1 (workspace files excluded from all-terms discovery) landed, and F2 (the page-claim defect) was withdrawn by amendment r1 with no code deliverable. What remained open there was never the defect pair; it was the scale epic that got boarded onto the note, and this note now owns it.

Three of that epic's bites shipped: PR #31 removed the fictional ancestor-Knop requirement, PR #32 added faithful nested-source pending-heavy generation, and PR #33 added untargeted multi-pending first-payload batching (released in `v0.7.0`). The viability probe then measured the composed sequence end to end for the first time.

**The probe is the reason this note exists.** It found no break through N=1,700 on the faithful nested-source shape — but weave peaked at **3.79 GiB against a ~4.09 GiB ceiling**. It passes; it barely passes. The decisive detail: `payloadBatchCandidates=0`. Extracted candidates classify as `firstExtractedKnopWeave`, and the batch gate in `version_execution.ts` requires all-`firstPayloadWeave`, so they never enter the path PR #33 built and instead run the per-candidate recursive loop — candidate-cache hits reached **1,445,850** at N=1,700, the exact triangular per-candidate pattern.

So the batch path already exists and is already proven; extracted candidates simply are not eligible for it.

Completed 2026-08-21. PR #41 admitted homogeneous untargeted extracted candidates to a sibling coherent batch planner; PRs #43/#44 repaired the batch and sequential MeshInventory history-index paths. The dedicated generate carve [[wa.completed.2026.2026-08-21_1111-generate-streaming-memory]] then replaced corpus-wide rendered-byte retention with command-shared source contents and four-page render→write→discard batches. The final faithful current-only N=1,700 probe at Weave `5fd9dd1` measured 802 MiB for composed weave and 674 MiB for same-timestamp standalone generate. Both are below 1.5 GiB, and both N=500→1,700 RSS curves are sublinear.

## Discussion

The measured curve, from the probe receipt (current-only history, faithful `catalog/source` nested shape, no ancestor Knop):

| N | weave wall / max RSS | generate wall / max RSS |
|---:|---|---|
| 200 | 7.00s / 467 MiB | 4.00s / 482 MiB |
| 500 | 32.23s / 1.45 GiB | 15.61s / 1.47 GiB |
| 1,000 | 1m55.60s / 2.79 GiB | 50.44s / 2.76 GiB |
| 1,700 | 5m09.98s / **3.79 GiB** | 2m29.63s / **3.74 GiB** |

Final curve on 2026-08-21 from a clean detached worktree at `5fd9dd1`, using fresh meshes with current-only history, 1,024-byte terms, nested source `catalog/source`, fixed timestamp `2026-08-21T00:00:00.000Z`, `WEAVE_MEMORY_STATS=1`, `WEAVE_TIMING=1`, and external `/usr/bin/time -v`:

| N | weave wall / max RSS | generate wall / max RSS |
|---:|---|---|
| 500 | 10.81s / 408 MiB | 10.61s / 357 MiB |
| 1,000 | 39.90s / 542 MiB | 32.86s / 524 MiB |
| 1,700 | 1m18.54s / **802 MiB** | 1m34.12s / **674 MiB** |

For 3.4× cardinality, weave RSS grew 1.97× and generate RSS grew 1.89×. `payloadBatchCandidates` equaled N at every rung. N=1,700 rendered 6,814 pages in 1,704 batches, with at most four rendered files and 4,476,771 rendered bytes retained in a batch.

In the original curve, memory and wall time were both super-linear. Recursive planning (135.3s) and page generation (129.6s) dominated at N=1,700, while RDF validation (128.5ms) and writes (118.1ms) were noise — so this was a planning-shape problem, not an I/O or parsing problem.

Note that `generate` tracks weave closely (3.74 GiB). Batching weave alone therefore does not make the *sequence* safe; page generation needs its own look before any headroom claim covers extract→weave→generate end to end.

## Acceptance Criteria

- [x] An untargeted candidate set that classifies wholly as `firstExtractedKnopWeave` plans and executes as one policy-validated `VersionPlan`, advancing the MeshInventory once, with canonical ordering independent of discovery order.
- [x] Mixed sets (extracted + first-payload + later-payload), recursive sets, and `overwrite` requests keep their current behavior; single-target and explicit multi-target behavior is unchanged.
- [x] At N=1,700 on the faithful nested-source shape, `payloadBatchCandidates` equals the candidate count (not 0), **weave peak RSS is under 1.5 GiB**, and growth from N=500 to N=1,700 is no worse than linear (RULED 2026-08-06).
- [x] Byte-stable same-timestamp regeneration is preserved: post-weave pending is zero, every term and Knop carries its canonical page claim, source facts and non-HTML artifact hashes are unchanged, and standalone `generate` creates 0 files and updates 0 pages.
- [x] A durable regression encodes the measured curve at a cardinality CI can actually afford.

## Open Issues

- **RULED 2026-08-06 (Dave): the headroom target is peak RSS at N=1,700 under 1.5 GiB, with growth from N=500 to N=1,700 no worse than linear.** That is roughly 2.5x margin under V8's ~4.09 GiB ceiling, and the growth bound is the part that matters — a one-off number can be met by luck, a shape cannot. This is the bite's pass condition.
- **RULED 2026-08-06 (Dave): `generate` is in scope too**, as its own carve with the dependency stated. Batching weave alone would leave the sequence bounded by generation (3.74 GiB at N=1,700), so **no viability claim for extract→weave→generate may rest on the weave carve alone.**

  Dave's hypothesis was that generate "holds a bunch of stuff to calculate links." **Checked, and it is simpler than that — no cross-page link index exists.** `page_model_assembly.ts` builds no backlink or link-graph structure. The accumulation is three full-corpus materializations in sequence:

  1. every `ResourcePageModel` built up front;
  2. `renderResourcePages` (`src/runtime/weave/pages.ts:223`) rendering all of them through **one unbounded `Promise.all`**, so every render is in flight simultaneously;
  3. the result — `readonly PlannedFile[]` containing **every page's full rendered bytes** — returned to the caller before anything is written (`page_generation.ts:148`).

  Good news: no link graph means no ordering constraint forcing whole-corpus residency. The fix shape is streaming — render, write, discard — with bounded concurrency, rather than an algorithmic redesign. The unbounded `Promise.all` is the cheap half; the materialized `PlannedFile[]` return type is the structural half and will drive the interface change.
- **RESOLVED 2026-08-21:** per-PR CI uses N=40 and N=120. It asserts a four-file maximum render batch, rendered-batch byte growth no faster than cardinality, and RSS growth no faster than cardinality. Existing PR #41 tests assert non-zero all-extracted batching and mixed/recursive sequential routing.
- Only current-only history was measured, matching the ruled acceptance substrate. No versioned comparison was required for closure; the generator still exposes no `--generated-at`, so the probe applied the fixed timestamp to measured weave/generate operations.

## Decisions

- **2026-08-02 (Dave):** cut this work into its own note rather than leaving it boarded on a delivered defect-pair lane, and pursue getting extracted candidates onto the batch path.

## Contract Changes

None intended. This is an internal planning-path change: the same weave outcomes, reached in one transaction instead of N. Any user-visible change — diagnostics, ordering, or MeshInventory state cardinality for extracted sets — must be named here and in the release notes before it ships, per the runbook's behavioral-changelog rule.

## Testing

- Fail-on-old core test: a reverse-order untargeted all-extracted set producing canonical ordering, one MeshInventory state, and one working update.
- Regression that mixed and recursive sets still take the sequential path (guards against widening the gate too far).
- Instrumented assertion that `payloadBatchCandidates` is non-zero for an all-extracted set — the probe's smoking gun becomes the test.
- A cardinality run under `WEAVE_MEMORY_STATS=1` recording peak RSS and candidate-cache hits, at whatever N is affordable.

Final N=1,700 byte-stability receipt: zero pending candidates; zero missing term claims; zero missing Knop claims; first/middle/last term pages contained their source names; `catalog/_knop` remained absent; the working and exact historical source hashes matched; all 5,108 non-HTML file hashes were identical before and after standalone generate; standalone generate created 0 files and updated 0 pages.

## Non-Goals

- Retiring Stagecraft's claim-synthesis workaround. That still gates on a real-corpus replay or their own confirmation, exactly as [[wd.consumer-feedback.0.5.1.reply]] §2 promises. A synthetic pass is not a retirement authorization.
- Changing extraction's non-publication-bearing lifecycle, or any `hasResourcePage` emission in `src/core/extract`.
- MeshInventory history-index rendering from actual progression (separate residual).
- Page-generation memory work inside the weave batch-path implementation bite. It was delivered separately on [[wa.completed.2026.2026-08-21_1111-generate-streaming-memory]], and the end-to-end claim relies on both completed lanes.

## Implementation Plan

- [x] Headroom target and generate scope RULED 2026-08-06 — see Open Issues.
- [x] Carve the generate-side memory work as its own note, with the no-premature-victory dependency stated.
- [x] Locate the batch gate in `src/runtime/weave/version_execution.ts` (~913 at the time of the probe) and the classifier boundary that separates `firstExtractedKnopWeave` from `firstPayloadWeave`.
- [x] Determine whether extracted candidates can join the existing coherent payload-batch planner as-is, or need a sibling planner — PR #41 shipped the sibling planner.
- [x] Implement, with the fail-on-old tests above recorded before the fix.
- [x] Re-run the probe's evidence sequence at N=1,700 and record the new curve against the table in Discussion.
- [x] Board the durable regression at the affordable cardinality.
- [x] Update `wd.todo` and board the Known Limitations change for the next release notes.
