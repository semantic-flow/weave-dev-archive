---
id: nhbtxz75ssfh3h43sl0fe0i
title: 2026 05 22_2222 Core Weave HTML Page Renderer Extraction
desc: ''
updated: 1779513775127
created: 1779513775127
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_2206-core-weave-knop-inventory-renderer-extraction]].
- Move the remaining legacy/core HTML page render helpers out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve generated HTML text, planner dispatch, fixture-sensitive page output, and public imports through `src/core/weave/weave.ts`.
- Keep runtime ResourcePage rendering, ResourcePage presentation/config/templating design, RDF/Turtle renderers, and generalized fixture-renderer replacement out of this slice.
- Treat "Open Issues" as a watchlist unless an item is explicitly framed as an unresolved decision.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

After the KnopInventory renderer extraction, `src/core/weave/weave.ts` is mostly planner orchestration plus progression resolution and a small cluster of legacy HTML page renderers. These HTML helpers generate fixture-sensitive pages for artifact histories and extracted identifiers during core planning.

This is a behavior-preserving module extraction. It should reduce `weave.ts` while leaving generated HTML, generated RDF, planner branch selection, and runtime page-generation behavior unchanged.

## Discussion

Candidate module:

- `src/core/weave/legacy_page_renderers.ts`

Expected exported helper set:

- `renderArtifactHistoryIndexPage`
- `renderAliceIdentifierPageAfterFirstExtractedWeave`
- `renderExtractedPersonIdentifierPage`
- `renderGenericExtractedIdentifierPage`

Expected dependencies:

- `designator_segments.ts` for display/resource page paths.
- `html.ts` for existing HTML escaping, mesh labels, relative links, and resource-path normalization.
- `rdf_helpers.ts` for reading fixture-sensitive payload literals/nodes.
- `errors.ts` for local error construction where needed.

Boundary notes:

- `weave.ts` remains the planner dispatcher and public compatibility façade.
- The new module must not depend on `src/core/weave/weave.ts` or `src/runtime/**`.
- Keep `escapeTurtleString`, `renderExactExtractionSourceBlock`, and source-registry patching in `weave.ts`; they are source-registry/RDF helpers, not HTML page renderers.
- Do not begin the broader [[wa.task.2026.2026-05-22_2253-resourcepage-config-and-templating]] design/behavior task in this slice.
- Do not replace Alice Bio-specific or fixture-ladder HTML with generalized renderers in this slice.

## Open Issues

- [x] Confirm baseline `src/core/weave/weave.ts` line count and import graph before implementation.
- [x] Keep generated HTML output byte-identical for fixture-sensitive paths; core and integration fixture tests passed without fixture regeneration.
- [x] Confirm the new module does not import from `src/core/weave/weave.ts` or `src/runtime/**`.
- [x] Reassess whether progression resolvers or source-registry helpers are the next clean move-only slice after this extraction.
- [x] Keep the legacy-renderer replacement idea visible as a later behavior-sensitive task.

## Decisions

- Use `legacy_page_renderers.ts` to make the temporary nature of these helper renderers visible without mixing them into the runtime ResourcePage presentation pipeline.
- Keep extraction move-first. Any cleanup inside the new module must be small, local, and output-preserving.
- Defer generalized ResourcePage templating and fixture-renderer replacement to later tasks.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.

## Testing

- Baseline before implementation: `src/core/weave/weave.ts` is 2,016 lines.
- Baseline import graph/cycle audit rooted at `src/core/weave/weave.ts`: 33 modules, 125 edges, 38 direct root imports, 0 core-to-runtime imports, 0 cycles.
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

- Do not change generated HTML, generated RDF, generated ResourcePage output, planner dispatch, payload versioning behavior, source locator behavior, or overwrite-state behavior.
- Do not move runtime page renderers or start ResourcePage presentation/config templating work.
- Do not replace legacy Alice Bio-specific or fixture-ladder renderers with generalized renderers in this slice.
- Do not introduce fixture regeneration or semantic cleanup.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No new correctness bug found during this slice.
- The extracted module makes the legacy Alice/person fixture renderers easier to target later. The follow-up should replace these page strings with generalized ResourcePage/panel rendering only after the move-only phase is complete.
- The next clean move-only slice looks like progression resolver extraction or source-registry/extraction-source block helper extraction. Progression resolvers are larger and planner-adjacent; source-registry helpers are smaller but more semantically specific.

## Implementation Results

- Added `src/core/weave/legacy_page_renderers.ts`.
- Moved these helpers out of `src/core/weave/weave.ts`: `renderArtifactHistoryIndexPage`, `renderAliceIdentifierPageAfterFirstExtractedWeave`, `renderExtractedPersonIdentifierPage`, and `renderGenericExtractedIdentifierPage`.
- Kept `escapeTurtleString`, `renderExactExtractionSourceBlock`, and source-registry patching in `weave.ts` because they are RDF/source-registry helpers, not HTML page renderers.
- Resulting `src/core/weave/weave.ts`: 1,715 lines.
- New `src/core/weave/legacy_page_renderers.ts`: 311 lines.
- Post-slice rooted import graph for `src/core/weave/weave.ts`: 34 modules, 128 edges, 38 direct root imports, 0 core-to-runtime imports, 0 cycles.
- Verification passed: `deno task fmt`, `deno task lint`, `deno task check`, 56 core weave tests, 72 integration weave/version/generate tests, rooted import graph audit, whole-`src` cycle audit, core-to-runtime import search, and `git diff --check` in weave.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest pre-slice count is 2,016 lines.
- [x] Run pre-slice `deno task check`.
- [x] Move legacy/core HTML page renderers into `src/core/weave/legacy_page_renderers.ts`.
- [x] Update `src/core/weave/weave.ts` imports and remove dead local helper definitions/imports.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, core/integration weave tests, and import-cycle/runtime-edge audits.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving legacy HTML page renderer extraction.
