---
id: jqe3oraeuyqf2f8mt81ru9u
title: 2026 05 22_2117 Core Weave Knop Support Render Preservation Extraction
desc: ''
updated: 1779635111117
created: 1779509844508
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.completed.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_2252-payload-render-helpers]].
- Move Knop support artifact preservation helpers out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve generated Turtle text, carried KnopSourceRegistry/ReferenceCatalog behavior, planner dispatch, and public imports through `src/core/weave/weave.ts`.
- Keep mesh-inventory renderers, payload renderers, ReferenceCatalog renderers, and page-definition renderers out of this slice.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

`src/core/weave/weave.ts` still owns a compact support-artifact preservation path used after rendering KnopInventory Turtle for several weave slices. The helper detects already-carried Knop support artifacts in the current KnopInventory and re-inserts their facts/blocks into the newly rendered KnopInventory when the main render path would otherwise omit them.

This logic is not specific to payload rendering, ReferenceCatalog rendering, page-definition rendering, or extracted-Knop rendering. Pulling it into a focused module should reduce `weave.ts` while keeping the fixture-sensitive renderers themselves in place.

## Discussion

Candidate module:

- `src/core/weave/knop_support_renderers.ts`

Expected moved helper set:

- `renderKnopInventoryWithPreservedSupportArtifacts`
- private `resolveCurrentKnopSourceRegistry`
- private `resolveCurrentKnopReferenceCatalog`
- private `renderKnopBlockWithCarriedSupportFacts`
- private `CurrentKnopSourceRegistry`
- private `CurrentKnopReferenceCatalog`

Expected dependencies:

- RDF helpers from `src/core/weave/rdf_helpers.ts`
- Turtle block helpers from `src/core/weave/turtle_blocks.ts`
- `WeaveInputError` from `src/core/weave/errors.ts`
- `SFLO_NAMESPACE` from `src/core/rdf/namespaces.ts`

Boundary notes:

- This is preservation glue, not a general KnopInventory renderer.
- Keep `renderLocatedFileBlock` private in the new module for this slice if needed; broader located-file/resource-page block extraction belongs with a later mesh-inventory block extraction.
- Do not move the caller renderers. The only intended call-site change is importing the preservation helper from the new module.

## Open Issues

- [x] Confirm the pre-slice import graph is clean before moving the preservation helpers.
- [x] Keep carried KnopSourceRegistry and ReferenceCatalog preservation behavior unchanged.
- [x] Do not move mesh-inventory, payload, ReferenceCatalog, or page-definition renderers.
- [x] Confirm the new module does not import from `src/core/weave/weave.ts` or `src/runtime/**`.
- [x] Record whether source-registry preservation should later gain narrower tests or remain covered by existing core/integration weave tests.

## Decisions

- Use `knop_support_renderers.ts` rather than adding this preservation logic to payload, ReferenceCatalog, or page-definition renderer modules.
- Keep the preservation helper exported and keep all source-registry/reference-catalog discovery helpers private.
- Keep `weave.ts` as the caller and planner dispatcher.
- Preserve current error messages and Turtle block insertion behavior.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.

## Testing

- Before editing, record `src/core/weave/weave.ts` line count and import graph/cycle audit rooted at `src/core/weave/weave.ts`.
- Before editing, run `deno task check`.
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

- Do not change support artifact preservation semantics.
- Do not change generated Turtle, generated ResourcePage output, source registry behavior, reference catalog behavior, or planner dispatch.
- Do not move mesh-inventory renderers, payload renderers, ReferenceCatalog renderers, page-definition renderers, or source locator renderers.
- Do not introduce fixture regeneration or semantic cleanup.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- Existing performance opportunity: `renderKnopInventoryWithPreservedSupportArtifacts` still parses `currentKnopInventoryTurtle` separately in `resolveCurrentKnopSourceRegistry` and `resolveCurrentKnopReferenceCatalog`. A later cleanup could parse once and pass the quads through both resolvers, but this slice kept the moved behavior unchanged.

## Implementation Results

- Added `src/core/weave/knop_support_renderers.ts` with `renderKnopInventoryWithPreservedSupportArtifacts` plus private current KnopSourceRegistry/ReferenceCatalog resolution and carried-fact rendering helpers.
- Updated `src/core/weave/weave.ts` to import the preservation helper and removed the duplicated helper definitions and now-dead constants/imports.
- `src/core/weave/weave.ts` moved from 4,066 lines before the slice to 3,814 lines after the slice; the new focused module is 275 lines.
- Post-slice import graph audit rooted at `src/core/weave/weave.ts`: 31 modules, 113 edges, 39 direct root imports, 0 core-to-runtime imports, 0 cycles.
- Whole-`src` local import-cycle audit: 131 modules, 463 edges, 0 cycles.
- Source-registry preservation remains covered by the existing fixture-sensitive core and integration weave tests. Add a narrower unit test only if future preservation behavior changes; this slice did not create a new test-only seam that needs extra coverage.

Verification passed:

- `deno task fmt`
- `deno task lint`
- `deno task check`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`
- `rg -n "from [\"'][^\"']*runtime|from [\"']\\.\\.?/[^\"']*runtime" src/core`
- `git diff --check`

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.completed.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest pre-slice count is 4,066 lines.
- [x] Run pre-slice `deno task check`.
- [x] Move Knop support preservation helpers into `src/core/weave/knop_support_renderers.ts`.
- [x] Update `src/core/weave/weave.ts` imports and remove dead local helper definitions/constants.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, and focused core/integration tests.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.completed.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving Knop support preservation extraction.
