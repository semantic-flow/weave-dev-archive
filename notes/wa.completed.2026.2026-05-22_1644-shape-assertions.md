---
id: t7zz2hl53q9c1wiv9h51gek
title: 2026 05 22_1644 Shape Assertions
desc: ''
updated: 1779493459483
created: 1779493459483
---

## Goals

- Execute the next small core weave extraction slice from [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_1441-payload-version-layout-and-overwrite-state-planning]].
- Move shape assertion helpers out of `src/core/weave/weave.ts` into focused core weave modules without changing validation policy, generated RDF, generated ResourcePage output, runtime behavior, or CLI behavior.
- Preserve `planWeave`, `planVersion`, extracted payload/layout helpers, renderers, and ResourcePage builders in their current modules.
- Keep current error messages and fail-closed checks stable.
- Record any confusing shape policy, source-locator coupling, or possible bug under "Orthogonal Opportunities" instead of broadening this slice.

## Summary

After the payload layout and overwrite extraction, `src/core/weave/weave.ts` is about 7,100 lines. A large remaining non-rendering cluster is shape validation: helpers that parse current Turtle and assert the current Mesh/Knop/Payload/ReferenceCatalog/PageDefinition shape before a weave slice proceeds.

This slice should separate validation policy from planner orchestration and rendering. The move should make it easier to review later changes to current-state shape support without scanning page rendering, payload layout, or planner dispatch logic.

The work is still behavior-sensitive. These assertions encode the currently supported local weave shapes, including sparse working files, repository source locators, current-only support history policy, and fixture-era expectations. The implementation should be move-only unless an existing test reveals a blocker.

## Discussion

Candidate modules:

- `src/core/weave/shape_assertions.ts`: shared assertion primitives such as `assertHasNamedNodeFacts`, `assertHasLiteralFacts`, and top-level settled-shape validators if the dependency set stays reasonable.
- `src/core/weave/source_locator_assertions.ts`: optional focused module for `assertHasCurrentPayloadSourceLocator`, `assertHasCurrentSourceLocator`, `assertHasRepositorySourceFloatingLocator`, and `assertHasCurrentWorkingFileLocator` if source-locator validation would otherwise make `shape_assertions.ts` too broad.
- `src/core/weave/current_shape_assertions.ts`: optional name if the implementation benefits from distinguishing current supported weave shapes from generic RDF fact assertions.

Likely helper set to consider:

- Fact assertion primitives:
  - `assertHasNamedNodeFacts`
  - `assertHasLiteralFacts`
- Source locator assertion helpers:
  - `assertHasCurrentPayloadSourceLocator`
  - `assertHasCurrentSourceLocator`
  - `assertHasRepositorySourceFloatingLocator`
  - `assertHasCurrentWorkingFileLocator`
- Current shape assertion families:
  - `assertCurrentKnopMetadataShape`
  - `assertCurrentKnopInventoryBaseShape`
  - `assertCurrentKnopInventoryWithoutHistory`
  - `assertCurrentPayloadArtifactShape`
  - `assertCurrentKnopInventoryShapeForSecondPayloadWeave`
  - `assertCurrentMeshInventoryShapeForFirstExtractedKnopWeave`
  - `assertCurrentKnopInventoryShapeForFirstExtractedKnopWeave`
  - `assertCurrentSourceRegistryShapeForFirstExtractedKnopWeave`
  - `assertReferenceTargetSourcePayloadShapeForFirstExtractedKnopWeave`
  - `assertCurrentMeshInventoryShapeForFirstReferenceCatalogWeave`
  - `assertCurrentKnopInventoryShapeForFirstReferenceCatalogWeave`
  - `assertCurrentKnopInventoryShapeForFirstPageDefinitionWeave`
  - `assertCurrentKnopInventoryShapeForSubsequentPageDefinitionWeave`

Helpers to be careful with:

- `normalizeWorkingLocalRelativePathLiteral` is used by current working-file and repository-source locator assertions, but it also belongs conceptually to a future source/artifact locator helper slice. Move it only if needed to avoid awkward imports.
- `usesMeshLocalWorkingLocatedFile`, `renderCurrentWorkingFileLocator`, `renderCurrentWorkingFileDeclaration`, and `renderRepositorySourceFloatingLocatorBlankNode` are renderer/source-locator helpers, not assertion helpers. Do not move them here unless the import graph forces a tiny shared source-locator module and tests stay behavior-preserving.
- `extractCurrentReferenceCatalogLinks`, ResourcePage model builders, payload renderers, and mesh inventory progression helpers are adjacent consumers of asserted shapes but should stay out of this slice.
- `assertPayloadNamingSupportedForSlice` is a request/target guard, not a shape assertion. Leave it near planner dispatch unless implementation reveals a cleaner home.

Expected dependency shape:

- Assertion modules may import `Quad` from `n3`, `toKnopPath` from `src/core/designator_segments.ts`, `SFLO_NAMESPACE` / `SFCFG_NAMESPACE` from `src/core/rdf/namespaces.ts`, model types from `src/core/weave/candidates.ts`, policy types from `src/core/weave/support_history_policy.ts`, and `WeaveInputError` from `src/core/weave/errors.ts`.
- Assertion modules may import RDF helpers from `src/core/weave/rdf_helpers.ts`, artifact history helpers from `src/core/weave/artifact_history_queries.ts`, and payload layout guards from `src/core/weave/payload_version_layout.ts`.
- Assertion modules must not import from `src/runtime/**`.
- Assertion modules must not import from `src/core/weave/weave.ts`.
- `weave.ts` should import extracted assertion helpers directly and keep planner function behavior unchanged.

The safest implementation may be two modules: one source-locator assertion module plus one current-shape assertion module. A single module is acceptable if it stays cohesive and does not become a new bucket for every validator in the file.

## Open Issues

- [x] Keep shape assertions behavior-preserving; no new supported shapes and no relaxed guards in this slice.
- [x] Preserve exact error messages unless a message already has to move unchanged with the helper.
- [x] Do not move rendering helpers or ResourcePage builders with assertion helpers.
- [x] Do not move source-locator renderers just because source-locator assertions move.
- [x] If source-locator assertions require too many unrelated helpers, split a narrowly named `source_locator_assertions.ts` module instead of broadening `shape_assertions.ts`.
- [x] Record any source-locator or current-shape policy confusion under "Orthogonal Opportunities".

## Decisions

- Treat this as a move-only validation extraction.
- Keep `planWeave` and `planVersion` in `src/core/weave/weave.ts`.
- Keep planner dispatch, payload layout, overwrite planning, generated Turtle renderers, and ResourcePage builders unchanged.
- Keep source-locator assertion helpers separate from source-locator renderers unless a tiny shared helper is unavoidable.
- Prefer narrow named exports from any assertion modules; do not create a barrel.
- Preserve the current distinction between shape assertions that parse Turtle and low-level assertion primitives that operate on parsed quads.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.
- The set of accepted current weave shapes should remain unchanged.

## Testing

- Before editing, record current line count and exported surface from `src/core/weave/weave.ts`.
- Before editing, run an import graph/cycle audit rooted at `src/core/weave/weave.ts`.
- Before editing, run:
  - `deno task check`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
- After extraction, run:
  - `deno task fmt`
  - `deno task lint`
  - `deno task check`
  - another import graph/cycle audit rooted at `src/core/weave/weave.ts`
  - confirm there are no new `src/core/` -> `src/runtime/` import edges
  - confirm there are no local cycles among `src/` modules
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`

## Non-Goals

- Do not change Semantic Flow ontology terms or shape semantics.
- Do not change current weave slice classification, payload layout, overwrite planning, generated Turtle, generated ResourcePage output, target normalization, or source locator serialization.
- Do not move large Turtle renderers, ResourcePage builders, mesh inventory progression helpers, payload render helpers, or reference-catalog link extraction.
- Do not introduce new shape support, fixture regeneration, or semantic cleanup.
- Do not introduce a broad constants module.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Implementation Results

Implemented behavior-preserving extraction:

- Added `src/core/weave/shape_assertions.ts` for current settled-shape validators, low-level fact assertions, and the shared mesh-inventory progression resolver that one moved assertion already depended on.
- Added `src/core/weave/source_locator_assertions.ts` for current working-file and repository-source floating locator assertions.
- Added `src/core/weave/working_file_paths.ts` so source-locator assertions and source-locator renderers share the same `workingLocalRelativePath` normalization instead of duplicating it.
- Added `src/core/weave/progression_models.ts` for `MeshInventoryProgression` and `PageDefinitionWeaveProgression` so planner/render code and assertion helpers can share the same structural types without importing from `weave.ts`.
- Reduced `src/core/weave/weave.ts` from the recorded handoff count of 7,135 lines to 5,857 lines.
- Kept `planWeave`, `planVersion`, renderers, ResourcePage builders, payload layout, overwrite planning, and public façade re-exports behavior-compatible.
- Updated [[wd.codebase-overview]] with the new helper modules.

Verification:

- Pre-slice baseline: `src/core/weave/weave.ts` was 7,135 lines; public exports were unchanged façade exports plus `planWeave` and `planVersion`.
- Pre-slice checks run: `deno task check`; `deno test -A src/core/weave/weave_test.ts` passed 56 tests.
- Pre-slice import audit: 21 modules, 71 edges, 32 direct root imports, 0 `src/core/weave` -> `src/runtime` imports using the local audit script.
- Post-slice line counts: `weave.ts` 5,857; `shape_assertions.ts` 1,099; `source_locator_assertions.ts` 247; `working_file_paths.ts` 43; `progression_models.ts` 19.
- Post-slice import audit: 25 modules, 87 edges, 35 direct root imports, 0 `src/core/weave` -> `src/runtime` imports, 0 cycles.
- `deno task fmt`
- `deno task lint`
- `deno task check`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts` passed 56 tests.
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts` passed 72 tests.
- `git diff --check`

Suggested commit message:

```text
refactor(core-weave): extract current shape assertions

- move settled current-shape validators into shape_assertions.ts
- move current source-locator assertions into source_locator_assertions.ts
- share workingLocalRelativePath normalization through working_file_paths.ts
- move weave progression structural types into progression_models.ts
- preserve planner dispatch, rendering, generated RDF/Page output, and public weave façade exports

Verification:
- deno task fmt
- deno task lint
- deno task check
- WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts
- WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts
- import graph audit: 25 modules, 87 edges, 0 core->runtime imports, 0 cycles
- git diff --check
```

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No bugs or performance opportunities were found while implementing this slice.

## Deferred Follow-Up Ideas

- Source/artifact locator helper slice: after assertion helpers move, consider extracting shared locator normalization/rendering helpers if the dependency direction is clearer.
- Payload render helper slice: move first/second payload Turtle renderers after validation and layout are isolated.
- ResourcePage builder slice: move page model builders only after renderer and assertion dependencies are clearer.
- ReferenceCatalog current-link extraction slice: `extractCurrentReferenceCatalogLinks` is validation-like but should remain separate from generic shape assertions unless a later task targets reference-catalog helpers.
- Legacy fixture-ladder renderer cleanup remains deferred until move-only core planner decomposition is complete.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and exported function/type names from `src/core/weave/weave.ts`; latest handoff count is about 7,135 lines.
- [x] Run a pre-slice import graph/circular-dependency audit rooted at `src/core/weave/weave.ts`.
- [x] Identify whether source-locator assertions should move with current shape assertions or into a separate module.
- [x] Move low-level fact assertion primitives into a focused assertion module.
- [x] Move source-locator assertion helpers if dependency direction stays clean.
- [x] Move current shape assertion families into focused assertion module(s).
- [x] Keep renderers, ResourcePage builders, planner functions, and payload layout helpers unchanged.
- [x] Run `deno task check` after the first assertion module move.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, and focused core/integration tests.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this implementation slice.
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving shape assertion extraction.
