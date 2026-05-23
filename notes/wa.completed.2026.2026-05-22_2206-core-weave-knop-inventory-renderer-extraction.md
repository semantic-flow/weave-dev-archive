---
id: jxyftamaoe6tst1lzb2p4vp
title: 2026 05 22_2206 Core Weave Knop Inventory Renderer Extraction
desc: ''
updated: 1779512786487
created: 1779512786488
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.task.2026.2026-05-22_2139-core-weave-mesh-inventory-renderer-extraction]].
- Move remaining KnopInventory Turtle renderers out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve generated Turtle text, support-history policy behavior, planner dispatch, legacy fixture output, and public imports through `src/core/weave/weave.ts`.
- Keep mesh-inventory renderers, payload renderers, ResourcePage model builders, HTML page renderers, and runtime renderers out of this slice.
- Treat "Open Issues" as a watchlist unless an item is explicitly framed as an unresolved decision.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

`src/core/weave/weave.ts` still owns several bulky KnopInventory RDF render paths after the mesh-inventory renderer extraction. These paths render first Knop weave inventories, first ReferenceCatalog weave inventories, current-only ReferenceCatalog inventory updates, subsequent page-definition inventory updates, and first extracted-Knop inventories.

This is a behavior-preserving module extraction. It should reduce `weave.ts` while leaving planner branch selection, generated RDF shape, and fixture-sensitive output unchanged.

## Discussion

Candidate module:

- `src/core/weave/knop_inventory_renderers.ts`

Expected exported helper set:

- `renderFirstKnopWovenKnopInventoryTurtle`
- `renderFirstReferenceCatalogWovenKnopInventoryTurtle`
- `renderCurrentOnlyReferenceCatalogWovenKnopInventoryTurtle`
- `renderSubsequentPageDefinitionWovenKnopInventoryTurtle`
- `renderFirstExtractedKnopWovenKnopInventoryTurtle`

Expected private helper set:

- `renderPageDefinitionStateBlock`
- `renderPageDefinitionStateManifestationBlock`
- local `renderLocatedFileBlock` / `renderResourcePageLocatedFileBlock` copies if needed
- local path/segment helpers such as `toStateSegment` if the extracted module needs them

Boundary notes:

- `weave.ts` remains the planner dispatcher and public compatibility façade.
- The new module may depend on `designator_segments.ts`, `artifact_manifestation_paths.ts`, `support_history_policy.ts`, `support_history_renderers.ts`, `source_locator_renderers.ts`, `progression_models.ts`, `turtle_blocks.ts`, and `rdf/namespaces.ts`.
- The new module must not depend on `src/core/weave/weave.ts` or `src/runtime/**`.
- Keep shared LocatedFile/ResourcePage block helper cleanup deferred unless avoiding duplication becomes necessary for this module boundary.
- Do not replace legacy fixture-sensitive strings or alter generated Turtle ordering in this slice.
- `renderExactExtractionSourceBlock` should stay in `weave.ts` for this slice because the planner still calls it directly while replacing the existing source-registry ExtractionSource block. Moving it would either force an awkward export from the KnopInventory renderer module or widen the slice into source-registry rendering.

## Open Issues

- [x] Confirm baseline `src/core/weave/weave.ts` line count and import graph before implementation.
- [x] Keep `renderLocatedFileBlock` and `renderResourcePageLocatedFileBlock` duplicated locally for this move-only slice; shared block-renderer extraction remains deferred.
- [x] Keep generated Turtle output byte-identical for fixture-sensitive paths; core and integration fixture tests passed without fixture regeneration.
- [x] Keep planner branch selection and support-history policy behavior unchanged.
- [x] Confirm the new module does not import from `src/core/weave/weave.ts` or `src/runtime/**`.
- [x] After extraction, reassess whether remaining HTML/page helper renderers are the next clean slice.

## Decisions

- Use `knop_inventory_renderers.ts` rather than mixing these helpers into payload, mesh-inventory, ReferenceCatalog link parsing, or page-definition model modules.
- Keep extraction move-first. Any helper factoring inside the new module must be small, local, and output-preserving.
- Defer legacy fixture-renderer replacement to a later behavior-sensitive task after move-only extraction has exposed stable boundaries.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.

## Testing

- Baseline before implementation: `src/core/weave/weave.ts` is 2,806 lines.
- Baseline import graph/cycle audit rooted at `src/core/weave/weave.ts`: 32 modules, 119 edges, 39 direct root imports, 0 core-to-runtime imports, 0 cycles.
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

- Do not change support artifact progression semantics.
- Do not change generated Turtle, generated ResourcePage output, source locator behavior, payload versioning behavior, overwrite-state behavior, or planner dispatch.
- Do not move mesh-inventory renderers, payload renderers, ResourcePage model builders, runtime renderers, or page HTML renderers.
- Do not replace legacy Alice Bio-specific or fixture-ladder renderers with generalized renderers in this slice.
- Do not introduce fixture regeneration or semantic cleanup.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No new correctness bug found during this slice.
- The duplication of `renderLocatedFileBlock` / `renderResourcePageLocatedFileBlock` now appears in several renderer-focused modules. A later generalized RDF block-renderer cleanup can reduce this once the move-only phase settles.
- `renderExactExtractionSourceBlock` remains planner-local because it patches an existing source-registry block. If source-registry rendering grows, extract it into a dedicated source-registry renderer module rather than hiding it inside KnopInventory renderers.

## Implementation Results

- Added `src/core/weave/knop_inventory_renderers.ts`.
- Moved these renderers out of `src/core/weave/weave.ts`: `renderFirstKnopWovenKnopInventoryTurtle`, `renderFirstReferenceCatalogWovenKnopInventoryTurtle`, `renderCurrentOnlyReferenceCatalogWovenKnopInventoryTurtle`, `renderSubsequentPageDefinitionWovenKnopInventoryTurtle`, and `renderFirstExtractedKnopWovenKnopInventoryTurtle`.
- Kept `renderExactExtractionSourceBlock` in `weave.ts` because it is still a planner-side source-registry replacement helper, not a pure KnopInventory renderer boundary.
- Kept small LocatedFile/ResourcePage block helpers private to the new module for move-only safety.
- Resulting `src/core/weave/weave.ts`: 2,016 lines.
- New `src/core/weave/knop_inventory_renderers.ts`: 824 lines.
- Post-slice rooted import graph for `src/core/weave/weave.ts`: 33 modules, 125 edges, 38 direct root imports, 0 core-to-runtime imports, 0 cycles.
- Verification passed: `deno task fmt`, `deno task lint`, `deno task check`, 56 core weave tests, 72 integration weave/version/generate tests, rooted import graph audit, whole-`src` cycle audit, core-to-runtime import search, and `git diff --check` in both weave and weave-dev-archive.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest pre-slice count is 2,806 lines.
- [x] Run pre-slice `deno task check`.
- [x] Move KnopInventory renderers and tight local helpers into `src/core/weave/knop_inventory_renderers.ts`.
- [x] Update `src/core/weave/weave.ts` imports and remove dead local helper definitions/constants.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, core/integration weave tests, and import-cycle/runtime-edge audits.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving KnopInventory renderer extraction.
