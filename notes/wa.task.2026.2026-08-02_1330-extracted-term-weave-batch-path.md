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

## Discussion

The measured curve, from the probe receipt (current-only history, faithful `catalog/source` nested shape, no ancestor Knop):

| N | weave wall / max RSS | generate wall / max RSS |
|---:|---|---|
| 200 | 7.00s / 467 MiB | 4.00s / 482 MiB |
| 500 | 32.23s / 1.45 GiB | 15.61s / 1.47 GiB |
| 1,000 | 1m55.60s / 2.79 GiB | 50.44s / 2.76 GiB |
| 1,700 | 5m09.98s / **3.79 GiB** | 2m29.63s / **3.74 GiB** |

Memory and wall time are both super-linear. Recursive planning (135.3s) and page generation (129.6s) dominate at N=1,700, while RDF validation (128.5ms) and writes (118.1ms) are noise — so this is a planning-shape problem, not an I/O or parsing problem.

Note that `generate` tracks weave closely (3.74 GiB). Batching weave alone therefore does not make the *sequence* safe; page generation needs its own look before any headroom claim covers extract→weave→generate end to end.

## Acceptance Criteria

- An untargeted candidate set that classifies wholly as `firstExtractedKnopWeave` plans and executes as one policy-validated `VersionPlan`, advancing the MeshInventory once, with canonical ordering independent of discovery order.
- Mixed sets (extracted + first-payload + later-payload), recursive sets, and `overwrite` requests keep their current behavior; single-target and explicit multi-target behavior is unchanged.
- At N=1,700 on the faithful nested-source shape, `payloadBatchCandidates` equals the candidate count (not 0) and weave peak RSS leaves meaningful headroom under the ~4.09 GiB ceiling. **The target margin is a decision, not an assumption — see Open Issues.**
- Byte-stable same-timestamp regeneration is preserved: post-weave pending is zero, every term and Knop carries its canonical page claim, source facts and non-HTML artifact hashes are unchanged, and standalone `generate` creates 0 files and updates 0 pages.
- A durable regression encodes the measured curve at a cardinality CI can actually afford.

## Open Issues

- **What headroom is enough?** "Under the ceiling" is what we have now and it is not comfortable. A stated target (a peak-RSS ceiling at N=1,700, or a bound on the growth exponent) should be ruled before implementation, so the bite has a pass condition rather than a vibe.
- **Does generate need its own bite?** At 3.74 GiB it is nearly as exposed as weave. Batching weave may leave the sequence still bounded by generation. Probably a separate carve, but the dependency should be stated before we claim the sequence is viable.
- **What cardinality can CI afford?** The N=1,700 run is ~5 minutes of weave plus ~2.5 minutes of generate. The durable regression likely runs at a much smaller N with the curve asserted rather than the endpoint, or runs nightly rather than per-PR.
- Only current-only history was measured. No versioned comparison exists, and the generator exposes no `--generated-at` (the probe applied fixed timestamps to the measured operations only).

## Decisions

- **2026-08-02 (Dave):** cut this work into its own note rather than leaving it boarded on a delivered defect-pair lane, and pursue getting extracted candidates onto the batch path.

## Contract Changes

None intended. This is an internal planning-path change: the same weave outcomes, reached in one transaction instead of N. Any user-visible change — diagnostics, ordering, or MeshInventory state cardinality for extracted sets — must be named here and in the release notes before it ships, per the runbook's behavioral-changelog rule.

## Testing

- Fail-on-old core test: a reverse-order untargeted all-extracted set producing canonical ordering, one MeshInventory state, and one working update.
- Regression that mixed and recursive sets still take the sequential path (guards against widening the gate too far).
- Instrumented assertion that `payloadBatchCandidates` is non-zero for an all-extracted set — the probe's smoking gun becomes the test.
- A cardinality run under `WEAVE_MEMORY_STATS=1` recording peak RSS and candidate-cache hits, at whatever N is affordable.

## Non-Goals

- Retiring Stagecraft's claim-synthesis workaround. That still gates on a real-corpus replay or their own confirmation, exactly as [[wd.consumer-feedback.0.5.1.reply]] §2 promises. A synthetic pass is not a retirement authorization.
- Changing extraction's non-publication-bearing lifecycle, or any `hasResourcePage` emission in `src/core/extract`.
- MeshInventory history-index rendering from actual progression (separate residual).
- Page-generation memory work, unless the ruling above folds it in deliberately.

## Implementation Plan

- [ ] Rule the headroom target and whether generate is in scope (Open Issues 1–2).
- [ ] Locate the batch gate in `src/runtime/weave/version_execution.ts` (~913 at the time of the probe) and the classifier boundary that separates `firstExtractedKnopWeave` from `firstPayloadWeave`.
- [ ] Determine whether extracted candidates can join the existing coherent payload-batch planner as-is, or need a sibling planner — the probe says the per-candidate recursion is the cost, not the classification itself.
- [ ] Implement, with the fail-on-old tests above recorded before the fix.
- [ ] Re-run the probe's evidence sequence at N=1,700 and record the new curve against the table in Discussion.
- [ ] Board the durable regression at the affordable cardinality.
- [ ] Update `wd.todo` and the release notes' Known Limitations when the headroom claim actually changes.
