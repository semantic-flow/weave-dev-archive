---
id: 22buqe8c44lfbo7r8ptyw58
title: 2026 05 22_2252 Payload Render Helpers
desc: ''
updated: 1779508363273
created: 1779508363273
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.task.2026.2026-05-22_2244-extract-source-locator-render-helpers]].
- Move payload-specific KnopInventory render helpers out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve generated Turtle text, planner dispatch, generated ResourcePage output, and public imports through `src/core/weave/weave.ts`.
- Keep mesh-inventory renderers in `weave.ts` for this slice because they share generic mesh/root/page-definition helper boundaries.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

`src/core/weave/weave.ts` still owns the verbose payload KnopInventory render path for first payload weave, second payload weave, and the multi-history second payload branch. These helpers are payload-specific and already consume extracted payload layout/source-locator modules, so they are a good next step after the source-locator renderer extraction.

This slice should move those payload KnopInventory renderers and their private rendered-history collection helpers into `payload_renderers.ts`. It should also move the shared support-history string omission helpers into a small renderer-support module so the new payload module and remaining non-payload renderers do not import from `weave.ts`.

## Discussion

Candidate modules:

- `src/core/weave/payload_renderers.ts`
- `src/core/weave/support_history_renderers.ts`

Expected moved helper set:

- `renderFirstPayloadWovenKnopInventoryTurtle`
- `renderSecondPayloadWovenKnopInventoryTurtle`
- private `renderMultiHistoryPayloadWovenKnopInventoryTurtle`
- private rendered-history collection/render helpers currently used by the multi-history payload branch
- shared `omitInitialKnopMetadataHistory`
- shared `omitKnopInventoryHistory`

Expected dependencies:

- `PayloadVersionLayout` from `src/core/weave/payload_version_layout.ts`
- `RepositorySourceFloatingLocator` from `src/core/weave/source_models.ts`
- `renderCurrentWorkingFileLocator` and `renderCurrentWorkingFileDeclaration` from `src/core/weave/source_locator_renderers.ts`
- `shouldMaterializeSupportHistory` and `SupportArtifactHistoryPolicy` from `src/core/weave/support_history_policy.ts`
- RDF helpers from `src/core/weave/rdf_helpers.ts`
- `renderSubjectPredicateBlock` from `src/core/weave/turtle_blocks.ts`

Boundary notes:

- Do not move `renderFirstPayloadWovenMeshInventoryTurtle`, `renderFirstPayloadWovenCurrentOnlyMeshInventoryTurtle`, or `renderLegacyFirstPayloadWovenMeshInventoryTurtle` in this slice. Those functions depend on generic mesh-inventory/root/block render helpers that are also used by non-payload renderers.
- The rendered-history helper family moves as private support for the multi-history payload renderer, not as a new public model contract.
- The support-history omission helpers are moved separately because they are shared by payload, Knop, ReferenceCatalog, and page-definition renderers.

## Open Issues

- [x] Confirm the pre-slice import graph is clean before moving the payload renderers.
- [x] Keep generated Turtle snippets byte-for-byte equivalent for first payload and second payload KnopInventory outputs.
- [x] Do not move mesh-inventory payload renderers in this slice.
- [x] Confirm the new modules do not import from `src/core/weave/weave.ts` or `src/runtime/**`.
- [x] Record whether rendered-history collection should remain private to `payload_renderers.ts` or become a later standalone module.

## Decisions

- Use `payload_renderers.ts` for payload-specific KnopInventory renderers.
- Use `support_history_renderers.ts` for shared string postprocessors that omit initial KnopMetadata and KnopInventory histories when support history policy is current-only.
- Keep mesh-inventory payload renderers in `weave.ts` until a broader generic mesh-inventory renderer boundary is clear.
- Keep rendered-history collection private to `payload_renderers.ts` for now because only the multi-history payload renderer currently consumes it.
- Keep `weave.ts` as the planner dispatcher and public façade.
- Preserve the current string-template rendering strategy and current post-render omission behavior.

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

- Do not change payload version layout, overwrite planning, source-locator semantics, support-history policy semantics, or generated output.
- Do not move mesh-inventory payload renderers, page-definition renderers, ReferenceCatalog renderers, source-registry preservation, or runtime page generation in this slice.
- Do not introduce fixture regeneration or semantic cleanup.
- Do not make the rendered-history helper family public unless another current module needs it.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No bugs or performance opportunities were found while implementing this slice.

## Implementation Results

Implemented behavior-preserving extraction:

- Added `src/core/weave/payload_renderers.ts` for first payload and second payload KnopInventory Turtle renderers.
- Moved the multi-history payload KnopInventory renderer and its private rendered-history collection/render helpers into `payload_renderers.ts`.
- Added `src/core/weave/support_history_renderers.ts` for shared KnopMetadata and KnopInventory support-history omission postprocessors.
- Updated `src/core/weave/weave.ts` to import the extracted renderers and support-history postprocessors.
- Kept mesh-inventory payload renderers, page-definition renderers, ReferenceCatalog renderers, source-registry preservation, planner dispatch, generated RDF, and generated ResourcePage output unchanged.
- Reduced `src/core/weave/weave.ts` from 5,195 lines to 4,066 lines.
- Updated [[wd.codebase-overview]] and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] with the new module layout.

Verification:

- Pre-slice baseline: `src/core/weave/weave.ts` was 5,195 lines; import audit reported 28 modules, 99 edges, 36 direct root imports, 0 `src/core/weave` -> `src/runtime` imports, and 0 cycles.
- Pre-slice `deno task check` passed.
- Post-slice line counts: `weave.ts` 4,066; `payload_renderers.ts` 1,022; `support_history_renderers.ts` 172.
- Post-slice import audit: 30 modules, 109 edges, 38 direct root imports, 0 `src/core/weave` -> `src/runtime` imports, 0 cycles.
- `deno task fmt`
- `deno task lint`
- `deno task check`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts` passed 56 tests.
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts` passed 72 tests.
- `git diff --check`

Suggested commit message:

```text
refactor(core-weave): extract payload render helpers

- move first and second payload KnopInventory Turtle renderers into payload_renderers.ts
- keep multi-history payload rendering and rendered-history collection private to the payload renderer module
- move shared support-history omission postprocessors into support_history_renderers.ts
- keep mesh-inventory renderers, planner dispatch, and generated RDF/Page output unchanged

Verification:
- deno task fmt
- deno task lint
- deno task check
- WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts
- WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts
- import graph audit: 30 modules, 109 edges, 0 core->runtime imports, 0 cycles
- git diff --check
```

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest pre-slice count is 5,195 lines.
- [x] Run pre-slice `deno task check`.
- [x] Move shared support-history omission helpers into `src/core/weave/support_history_renderers.ts`.
- [x] Move payload KnopInventory render helpers and private rendered-history helpers into `src/core/weave/payload_renderers.ts`.
- [x] Update `src/core/weave/weave.ts` imports and remove dead local helper definitions.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, and focused core/integration tests.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving payload renderer extraction.
