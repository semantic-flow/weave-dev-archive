---
id: puqc8gfadcjlhr5ljo2yjxs
title: 2026 05 22_2139 Core Weave Mesh Inventory Renderer Extraction
desc: ''
updated: 1779512774164
created: 1779511157732
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_2117-core-weave-knop-support-render-preservation-extraction]].
- Move mesh-inventory Turtle renderers and their tight local block helpers out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve generated Turtle text, current support-history policy behavior, planner dispatch, legacy fixture fallback behavior, and public imports through `src/core/weave/weave.ts`.
- Allow small helper extraction inside the new mesh-inventory renderer module when it reduces real duplication without changing output.
- Keep KnopInventory, ReferenceCatalog, page-definition, ResourcePage model, and runtime renderers out of this slice.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

`src/core/weave/weave.ts` still owns the mesh-inventory render paths for first Knop weave, first payload weave, current-only first payload weave, first extracted Knop weave, generic extracted Knop weave, and MeshMetadata progression updates. These renderers are cohesive but bulky: they edit MeshInventory Turtle blocks, preserve carried mesh Knop entries, append MeshInventory historical states, and add ResourcePage/LocatedFile declarations.

This is a behavior-preserving module extraction. It should reduce `weave.ts` while leaving planner branch selection, generated RDF shape, and fixture-sensitive legacy render output unchanged.

## Discussion

Candidate module:

- `src/core/weave/mesh_inventory_renderers.ts`

Expected exported helper set:

- `renderFirstKnopWovenMeshInventoryTurtle`
- `renderFirstPayloadWovenMeshInventoryTurtle`
- `renderFirstPayloadWovenCurrentOnlyMeshInventoryTurtle`
- `renderFirstExtractedKnopWovenMeshInventoryTurtle`
- `renderGenericFirstExtractedKnopWovenMeshInventoryTurtle`
- `renderMeshMetadataWithMeshInventoryProgression`

Expected private helper set:

- `renderLegacyFirstKnopWovenMeshInventoryTurtle`
- `renderLegacyFirstPayloadWovenMeshInventoryTurtle`
- `renderMeshIdentifierBlock`
- `renderMeshRootBlock`
- `renderMeshKnopBlockWithResourcePage`
- `renderMeshPayloadArtifactBlockWithResourcePage`
- `renderMeshInventoryArtifactBlock`
- `renderMeshInventoryHistoryBlock`
- `renderMeshInventoryStateBlock`
- `renderMeshInventoryStateManifestationBlock`
- `renderMeshInventoryHistoryWithFourthStateBlock`
- `renderMeshInventoryStateFourBlock`
- `renderMeshInventoryStateFourManifestationBlock`
- `renderMeshInventoryMetaProgressionBlock`
- `renderMeshInventoryHistoryMetaProgressionBlock`
- `renderLocatedFileBlock`
- `renderResourcePageLocatedFileBlock`
- `resolveMeshRootKnopPaths`
- `tryToMeshPath`

Commonality that may be factored during the move:

- The repeated "advance MeshInventory history" edit sequence may become a private helper if it is clearer than duplicating the same `replaceSubjectBlock` / `upsertSubjectBlockAfter` calls.
- `renderMeshInventoryStateFourBlock` and `renderMeshInventoryStateFourManifestationBlock` may be replaced by the generic state/state-manifestation helpers only if the generated output remains byte-identical.
- Do not create a grand unified mesh renderer. Prefer one or two small private helpers over a parameter-heavy abstraction that hides fixture-sensitive order and anchor choices.

Boundary notes:

- `weave.ts` remains the planner dispatcher and public compatibility façade.
- The new module may depend on `designator_segments.ts`, `rdf_helpers.ts`, `turtle_blocks.ts`, `source_locator_renderers.ts`, `progression_models.ts`, and `source_models.ts`.
- The new module must not depend on `src/core/weave/weave.ts` or `src/runtime/**`.
- `renderLocatedFileBlock` and `renderResourcePageLocatedFileBlock` are duplicated in a few modules today. Keep them private in this slice unless a tiny shared helper module is needed to avoid worse coupling; broader shared LocatedFile/ResourcePage block cleanup can be a later slice.
- Legacy Alice Bio / fixture-ladder renderers should be moved as-is. Replacing them with generalized renderers is a separate behavior-sensitive cleanup after the move-only phase.

## Open Issues

- [x] Update stale durable docs/task links from the pre-move 2117 task note to [[wa.completed.2026.2026-05-22_2117-core-weave-knop-support-render-preservation-extraction]].
- [x] Confirm the pre-slice import graph is clean before moving the renderers.
- [x] Confirm whether `renderLocatedFileBlock` and `renderResourcePageLocatedFileBlock` should stay private to the new module for this slice or move into a small shared block-renderer module.
- [x] Keep generated Turtle output byte-identical for fixture-sensitive paths.
- [x] Keep planner branch selection and support-history policy behavior unchanged.
- [x] Confirm the new module does not import from `src/core/weave/weave.ts` or `src/runtime/**`.
- [x] After extraction, reassess whether legacy Alice Bio / fixture-ladder renderers are ready for a separate generalized-renderer cleanup task.

## Decisions

- Use `mesh_inventory_renderers.ts` as the first extraction boundary rather than folding these helpers into payload, Knop, ReferenceCatalog, or page-definition renderer modules.
- Move `renderMeshMetadataWithMeshInventoryProgression` with this slice because it renders MeshInventory progression facts into MeshMetadata and is part of the same support-artifact progression surface.
- Allow small private helper extraction inside the new module when it keeps the moved code easier to review and output-stable.
- Do not replace legacy fixture-specific render strings in this task.
- Keep `weave.ts` as the caller and planner dispatcher.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.

## Testing

- Baseline before this task shell: `src/core/weave/weave.ts` is 3,814 lines.
- Baseline import graph/cycle audit rooted at `src/core/weave/weave.ts`: 31 modules, 113 edges, 39 direct root imports, 0 core-to-runtime imports, 0 cycles.
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
- If implementation touches shared LocatedFile/ResourcePage block helpers outside the mesh-inventory module, broaden beyond the focused weave tests according to the affected caller; there is no dedicated mesh-support-pages unit test today, so the integration weave tests remain the minimum coverage for that path.

## Non-Goals

- Do not change support artifact progression semantics.
- Do not change generated Turtle, generated ResourcePage output, source locator behavior, payload versioning behavior, overwrite-state behavior, or planner dispatch.
- Do not move KnopInventory renderers, ReferenceCatalog renderers, page-definition renderers, ResourcePage model builders, runtime renderers, or page HTML renderers.
- Do not replace legacy Alice Bio-specific or fixture-ladder renderers with generalized renderers in this slice.
- Do not introduce fixture regeneration or semantic cleanup.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No bugs or performance opportunities recorded during this slice.
- Small follow-up cleanup opportunity: `renderLocatedFileBlock` and `renderResourcePageLocatedFileBlock` remain duplicated across `weave.ts`, `mesh_inventory_renderers.ts`, `mesh_support_pages.ts`, and `knop_support_renderers.ts`. This is intentionally left for a later shared block-renderer cleanup because moving it now would widen the slice beyond mesh-inventory renderer extraction.

## Implementation Results

- Added `src/core/weave/mesh_inventory_renderers.ts` with the mesh-inventory Turtle renderers, MeshMetadata progression renderer, legacy mesh-inventory renderers, and tight private mesh block helpers.
- Updated `src/core/weave/weave.ts` to import the exported mesh-inventory renderers and removed the moved helper definitions and now-dead imports/constants.
- Kept local copies of `renderLocatedFileBlock`, `renderResourcePageLocatedFileBlock`, `renderMeshInventoryStateBlock`, and `renderMeshInventoryStateManifestationBlock` in `weave.ts` because remaining page-definition/ReferenceCatalog render paths still use them; shared block-renderer extraction is deferred.
- `src/core/weave/weave.ts` moved from 3,814 lines before the slice to 2,806 lines after the slice; the new focused module is 1,072 lines.
- Post-slice import graph audit rooted at `src/core/weave/weave.ts`: 32 modules, 119 edges, 39 direct root imports, 0 core-to-runtime imports, 0 cycles.
- Whole-`src` local import-cycle audit: 132 modules, 471 edges, 0 cycles.
- Legacy Alice Bio / fixture-ladder renderers are not ready for replacement in this move-only slice; the parent task keeps a follow-up reminder to create a behavior-sensitive generalized-renderer cleanup task after the remaining renderer boundaries settle.

Verification passed:

- `deno task fmt`
- `deno task lint`
- `deno task check`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`
- `rg -n "from [\"'][^\"']*runtime|from [\"']\\.\\.?/[^\"']*runtime" src/core`
- `git diff --check`

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before filling the task.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest pre-slice count is 3,814 lines.
- [x] Update durable stale links for the completed Knop support preservation slice.
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] with this eleventh extraction slice and the deferred legacy-renderer cleanup reminder.
- [x] Run pre-slice `deno task check`.
- [x] Move mesh-inventory renderers and tight local helpers into `src/core/weave/mesh_inventory_renderers.ts`.
- [x] Avoid broad commonality factoring beyond the move-only extraction; keep duplicated shared block helpers as a recorded follow-up.
- [x] Update `src/core/weave/weave.ts` imports and remove dead local helper definitions/constants.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, core/integration weave tests, and import-cycle/runtime-edge audits.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving mesh-inventory renderer extraction.
