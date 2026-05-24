---
id: 3vv8pywpvezdr6l40sniuin
title: 2026 05 22_2239 Source Registry Extraction Source Helper Extraction
desc: ''
updated: 1779635111111
created: 1779514781521
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.completed.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_2222-core-weave-html-page-renderer-extraction]].
- Move source-registry / ExtractionSource block helpers out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve generated RDF text, exact ExtractionSource evidence facts, planner dispatch, source-registry replacement behavior, and public imports through `src/core/weave/weave.ts`.
- Keep progression resolver extraction, source-registry redesign, generalized provenance work, runtime source loading, and ResourcePage templating out of this slice.
- Treat "Open Issues" as a watchlist unless an item is explicitly framed as an unresolved decision.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

After the legacy HTML page renderer extraction, `src/core/weave/weave.ts` still owns a small source-registry helper island: replacing an existing ExtractionSource block, rendering the exact ExtractionSource block for extracted Knop weaves, converting optional source evidence into Turtle facts, and escaping Turtle string literals.

This is a behavior-preserving module extraction. It should remove the last small RDF-rendering helper cluster from `weave.ts` while leaving planner branch selection, generated RDF shape, exact extraction-source evidence, and fixture-sensitive output unchanged.

## Discussion

Candidate module:

- `src/core/weave/extraction_source_blocks.ts`

Expected exported helper set:

- `replaceExtractionSourceBlock`
- `renderExactExtractionSourceBlock`

Expected private helper set:

- `toExtractionSourceEvidenceFacts`
- `escapeTurtleString`

Expected dependencies:

- `candidates.ts` for `ExtractionSourceEvidenceModel`.
- `errors.ts` for fail-closed block replacement errors.
- `turtle_blocks.ts` for subject-block lookup and splitting.

Boundary notes:

- `weave.ts` remains the planner dispatcher and public compatibility façade.
- The new module must not depend on `src/core/weave/weave.ts` or `src/runtime/**`.
- Do not move progression resolver helpers in this slice.
- Do not alter the current source-registry or extraction provenance model; this is a move-only extraction.
- Do not start generalized replay/provenance work from Accord or framework-level behavior changes here.

## Open Issues

- [x] Confirm baseline `src/core/weave/weave.ts` line count and import graph before implementation.
- [x] Keep generated ExtractionSource Turtle byte-identical for fixture-sensitive paths; core and integration fixture tests passed without fixture regeneration.
- [x] Confirm the new module does not import from `src/core/weave/weave.ts` or `src/runtime/**`.
- [x] Reassess whether progression resolver extraction is the next clean move-only slice after this extraction.
- [x] Keep generalized provenance/replay ideas visible as later behavior work, not part of this helper move.

## Decisions

- Use `extraction_source_blocks.ts` because the moved helpers operate on Turtle subject blocks and ExtractionSource evidence facts, not on full source-registry planning.
- Keep extraction move-first. Any cleanup inside the new module must be small, local, and output-preserving.
- Defer source-registry model redesign and generalized provenance work.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.

## Testing

- Baseline before implementation: `src/core/weave/weave.ts` is 1,715 lines.
- Baseline import graph/cycle audit rooted at `src/core/weave/weave.ts`: 34 modules, 128 edges, 38 direct root imports, 0 core-to-runtime imports, 0 cycles.
- Before editing implementation, run `deno task check`.
- After extraction, run:
  - `deno task fmt`
  - `deno task lint`
  - `deno task check`
  - import graph/cycle audit rooted at `src/core/weave/weave.ts`
  - confirm there are no new `src/core/` -> `src/runtime/` import edges
  - confirm there are no local cycles among `src/` modules
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`

## Non-Goals

- Do not change generated RDF, generated ResourcePage output, source locator behavior, payload versioning behavior, overwrite-state behavior, or planner dispatch.
- Do not move progression resolver helpers.
- Do not redesign source-registry or extraction provenance semantics.
- Do not introduce generalized replay/provenance behavior or Accord acceptance changes.
- Do not introduce fixture regeneration or semantic cleanup.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No new correctness bug found during this slice.
- Progression resolver extraction now looks like the next clean move-only slice; it is larger and more planner-adjacent than this helper move, so it deserves a task shell with explicit resolver boundaries.
- Generalized replay/provenance should remain a later behavior task. This slice only moved the current exact ExtractionSource block rendering and replacement helpers.

## Implementation Results

- Added `src/core/weave/extraction_source_blocks.ts`.
- Moved `replaceExtractionSourceBlock`, `renderExactExtractionSourceBlock`, `toExtractionSourceEvidenceFacts`, and `escapeTurtleString` out of `src/core/weave/weave.ts`.
- Kept planner branch selection and source-registry replacement call sites in `weave.ts`.
- Resulting `src/core/weave/weave.ts`: 1,608 lines.
- New `src/core/weave/extraction_source_blocks.ts`: 112 lines.
- Post-slice rooted import graph for `src/core/weave/weave.ts`: 35 modules, 131 edges, 38 direct root imports, 0 core-to-runtime imports, 0 cycles.
- Verification passed: `deno task fmt`, `deno task lint`, `deno task check`, 56 core weave tests, 72 integration weave/version/generate tests, rooted import graph audit, whole-`src` cycle audit, core-to-runtime import search, and `git diff --check` in both weave and weave-dev-archive.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.completed.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest pre-slice count is 1,715 lines.
- [x] Run pre-slice `deno task check`.
- [x] Move source-registry / ExtractionSource block helpers into `src/core/weave/extraction_source_blocks.ts`.
- [x] Update `src/core/weave/weave.ts` imports and remove dead local helper definitions/imports.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, core/integration weave tests, and import-cycle/runtime-edge audits.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.completed.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving source-registry ExtractionSource helper extraction.
