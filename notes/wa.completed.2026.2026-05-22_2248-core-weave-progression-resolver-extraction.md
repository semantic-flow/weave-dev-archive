---
id: cq4vidjhu02zzdf51h0fnbp
title: 2026 05 22_2248 Core Weave Progression Resolver Extraction
desc: ''
updated: 1779515319967
created: 1779515319967
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_2239-source-registry-extraction-source-helper-extraction]].
- Move planner-adjacent progression resolver helpers out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve fail-closed settled-state checks, generated RDF/Page behavior, planner dispatch, naming semantics, and public imports through `src/core/weave/weave.ts`.
- Keep planner branch orchestration, generalized renderer replacement, ResourcePage templating, and progression semantic redesign out of this slice.
- Treat "Open Issues" as a watchlist unless an item is explicitly framed as an unresolved decision.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

After the source-registry ExtractionSource helper extraction, `src/core/weave/weave.ts` still owns progression resolver helpers that validate and compute the current/next history state paths for page-definition weaves, KnopInventory updates, and MeshInventory updates. These helpers are larger and more planner-adjacent than the prior renderer moves, but their boundaries are now clear enough for a move-only slice.

This is a behavior-preserving module extraction. It should reduce `weave.ts` while leaving planner branch selection, generated RDF, generated pages, and fail-closed progression validation unchanged.

## Discussion

Candidate module:

- `src/core/weave/progression_resolvers.ts`

Expected exported helper set:

- `resolvePageDefinitionWeaveProgression`
- `resolveCurrentKnopInventoryProgressionForPageDefinitionWeave`
- `resolveCurrentMeshInventoryProgressionForFirstKnopWeave`
- `resolveCurrentMeshInventoryProgressionForFirstPayloadWeave`

Expected private helper set:

- `parseStateOrdinalFromPath`
- `toStateSegment`
- `toLastPathSegment`
- local `toHistoryPathFromStatePath` copy if needed

Boundary notes:

- `weave.ts` remains the planner dispatcher and public compatibility façade.
- The new module must not depend on `src/core/weave/weave.ts` or `src/runtime/**`.
- `toHistoryPathFromStatePath` is still used by planner-created history pages in `weave.ts`; keep that tiny helper there and duplicate it privately in the resolver module rather than exporting it just for page paths.
- Do not move `planFirst*`, `planSecond*`, or target-selection orchestration in this slice.
- Do not alter state-segment naming, ordinal validation, page-definition history policy, or mesh/Knop inventory progression semantics.

## Open Issues

- [x] Confirm baseline `src/core/weave/weave.ts` line count and import graph before implementation.
- [x] Preserve fail-closed progression validation exactly.
- [x] Confirm the new module does not import from `src/core/weave/weave.ts` or `src/runtime/**`.
- [x] Decide whether to keep `toHistoryPathFromStatePath` local in `weave.ts` for page rendering call sites.
- [x] Reassess whether the move-only extraction phase is now essentially complete, or whether remaining small planner helpers still deserve another slice.

## Decisions

- Use `progression_resolvers.ts` rather than mixing these helpers into `shape_assertions.ts`; the functions compute next planning progression, while shape assertions should stay focused on validating settled current shapes.
- Keep extraction move-first. Any cleanup inside the new module must be small, local, and output-preserving.
- Keep planner branch functions in `weave.ts`.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.

## Testing

- Baseline before implementation: `src/core/weave/weave.ts` is 1,608 lines.
- Baseline import graph/cycle audit rooted at `src/core/weave/weave.ts`: 35 modules, 131 edges, 38 direct root imports, 0 core-to-runtime imports, 0 cycles.
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
- Do not move planner branch functions.
- Do not redesign progression, state naming, history naming, or fail-closed validation semantics.
- Do not introduce fixture regeneration or semantic cleanup.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No new performance issues or bugs surfaced in this slice.
- After this move, the remaining `weave.ts` helpers are mostly branch orchestration plus tiny guards. The next highest-value work is likely the already-tracked [[wa.task.2026.2026-05-22_2253-resourcepage-config-and-templating]] work and a follow-up for legacy fixture-renderer generalization, not another micro extraction.

## Implementation Results

- Added `src/core/weave/progression_resolvers.ts` with page-definition, KnopInventory, and MeshInventory progression resolvers.
- Kept `toHistoryPathFromStatePath` local in `weave.ts` for planner-created ResourcePage history models and duplicated it privately in `progression_resolvers.ts` to avoid a needless shared-export dependency.
- Removed resolver-only RDF helpers, shape assertion imports, constants, and ordinal path helpers from `src/core/weave/weave.ts`.
- Final line counts: `src/core/weave/weave.ts` is 1,224 lines; `src/core/weave/progression_resolvers.ts` is 403 lines.
- Post-slice import graph rooted at `src/core/weave/weave.ts`: 36 modules, 136 edges, 37 direct root imports, 0 core-to-runtime imports, 0 cycles.
- Whole-`src` import-cycle audit: 136 modules, 493 edges, 0 cycles.
- Verification passed: `deno task fmt`, `deno task lint`, `deno task check`, core weave tests, and the focused validate/version/generate plus weave integration tests.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest pre-slice count is 1,608 lines.
- [x] Run pre-slice `deno task check`.
- [x] Move progression resolver helpers into `src/core/weave/progression_resolvers.ts`.
- [x] Update `src/core/weave/weave.ts` imports and remove dead local helper definitions/imports/constants.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, core/integration weave tests, and import-cycle/runtime-edge audits.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving progression resolver extraction.
