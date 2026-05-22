---
id: evh0tcow0l532ujdetucpe6
title: 2026 05 21_1037 Core Weave Request Candidate Model Extraction
desc: ''
updated: 1779483483682
created: 1779385025853
---

## Goals

- Execute the next small slice of [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] by moving remaining shared request, candidate, and planning model definitions out of `src/core/weave/weave.ts`.
- Preserve public imports from `src/core/weave/weave.ts` through façade re-exports.
- Avoid behavior changes, generated RDF changes, generated ResourcePage changes, and semantic helper rewrites.
- Establish a clean dependency direction for later core weave planner extraction slices.

## Summary

The safest next step in decomposing `src/core/weave/weave.ts` is to move stable exported type/model definitions into focused modules. The file still starts with request types, runtime candidate models, planning inputs/results, and slice/progression-local interfaces. ResourcePage model types have already moved to `src/core/weave/resource_page_models.ts`; this task should not repeat that work.

This task is intentionally narrow. It should not move planner functions, Turtle renderers, RDF helpers, or payload layout logic except where a tiny type-only move is needed to avoid cycles.

## Current State

- `src/core/weave/weave.ts` is still the main pressure point at 8,597 lines.
- `ResourcePageModel` and related generated-page model types already moved to `src/core/weave/resource_page_models.ts` during the runtime page-generation decomposition. Do not repeat that move in this task.
- Runtime page generation is no longer the blocker for this core slice: `src/runtime/weave/weave.ts` is now a small façade and ResourcePage generation lives under `src/runtime/weave/page_generation.ts`, `page_model_assembly.ts`, `page_contexts.ts`, and `raw_source_panels.ts`.
- This task should continue as the first remaining type/model extraction slice rather than being closed. It is now scoped to request, candidate, and planning models.

## Discussion

Candidate modules:

- `src/core/weave/requests.ts`: `WeaveRequest`, `ValidateRequest`, `GenerateRequest`, and `VersionRequest`.
- `src/core/weave/source_models.ts`: `RepositorySourceFloatingLocator`. This is a tiny neutral move, but it keeps source-location models from living under a ResourcePage-specific name while candidate models start importing them directly.
- `src/core/weave/candidates.ts`: `PayloadWorkingArtifact`, `ReferenceCatalogWorkingArtifact`, `ReferenceTargetSourcePayloadArtifact`, `ExtractionSourceEvidenceModel`, `ResourcePageDefinitionWorkingArtifact`, and `WeaveableKnopCandidate`.
- Already done: `src/core/weave/resource_page_models.ts` now contains the ResourcePage model types needed by runtime page generation.
- `src/core/weave/planning_models.ts`: `PlanWeaveInput`, `WeavePlan`, and any planning-only public types that are not better kept near implementation.
- `src/core/weave/slices.ts`: `WeaveSlice`. Move this with the model slice before later slice-classification extraction, so `detectPendingWeaveSlice` can import the slice-name contract without making `planning_models.ts` the owner of slice detection semantics.

Prefer these focused modules over one broad `models.ts`. The split mirrors current use: requests are public operation contracts, source models describe reusable source-location structures, candidates are runtime-loaded planner input models, planning models describe planner input/output, and slices define the public slice-name union. The main rule is to avoid a huge mechanical move that creates awkward import cycles.

Expected type-only dependency shape:

- `requests.ts` imports only `TargetSpec` and `VersionTargetSpec` from `../targeting.ts`.
- `source_models.ts` has no local dependencies unless a later source model needs one.
- `resource_page_models.ts` imports `RepositorySourceFloatingLocator` from `source_models.ts` and may re-export it for compatibility with existing direct imports.
- `candidates.ts` imports `RepositorySourceFloatingLocator` from `source_models.ts`.
- `planning_models.ts` imports `PlannedFile`, request types, candidate types, policy types, and `ResourcePageModel`.
- `slices.ts` has no local dependencies.
- `weave.ts` imports the extracted names as types and re-exports them as the public façade.

The existing `src/core/weave/weave.ts` should continue to export the same public type names. Existing tests should not need broad changes in this first slice. Runtime modules that currently import moved type names through the large façade should be updated to import from the new focused type modules directly when touched, while continuing to import `WeaveInputError`, `planWeave`, `planVersion`, and `detectPendingWeaveSlice` from the façade until those behavior slices move.

## Open Issue Resolutions

- Keep request types under `src/core/weave/requests.ts` for now. They are public operation contracts that use shared target semantics from `src/core/targeting.ts`; they should not move into targeting itself.
- Extract focused modules now: `requests.ts`, `source_models.ts`, `candidates.ts`, `planning_models.ts`, and `slices.ts`. Avoid a broad `models.ts` because it would become a second junk drawer beside `weave.ts`.
- Move `RepositorySourceFloatingLocator` out of `resource_page_models.ts` in this slice, then re-export it from `resource_page_models.ts` and `weave.ts` so existing imports remain valid.
- Move `WeaveSlice` into a tiny `slices.ts` module rather than `planning_models.ts`; this keeps the slice-name union available to both the current detection function and the later slice-classification extraction without giving planning result models ownership of slice semantics.
- Keep `WeaveRequest` and `VersionRequest` as separate operation contracts even though their current shapes match.
- Keep `PlanWeaveInput.request: VersionRequest` unchanged. That is the current planner API shape, not a contract cleanup target for this slice.
- Move only exported/shared model types in this slice. Keep non-exported implementation-local interfaces in `weave.ts` until their behavior moves:
  - `SelectedWeaveableKnopCandidate` should move later with target selection/candidate filtering.
  - `PayloadVersionLayout` should move later with payload version layout planning.
  - `MeshInventoryProgression` and `PageDefinitionWeaveProgression` should move later with inventory/page-definition progression helpers.
  - `RenderedArtifactHistoryModel`, `RenderedHistoricalStateModel`, and `HistoricalStateLocatedFileFallback` should move later with Turtle rendering helpers.
  - `CurrentKnopSourceRegistry` and `CurrentKnopReferenceCatalog` should move later with RDF query/source registry helpers.


## Decisions

- Preserve `src/core/weave/weave.ts` as the public façade for existing public type imports.
- Prefer type-only exports and imports where possible.
- Allow internal runtime modules to import extracted type modules directly when it reduces dependency pressure on the large façade, but preserve façade re-exports for public compatibility.
- When moving a type used by runtime modules, prefer updating those runtime imports to the focused module in the same small patch instead of leaving new internal imports pointed at `src/core/weave/weave.ts`.
- Do not move behavior in this slice unless TypeScript cycles force a tiny supporting move.
- Do not change names unless a local name is actively misleading after extraction.
- Keep `core/targeting.ts` as the already-extracted home for target semantics; this slice should not move targeting types back under `core/weave`.

## Contract Changes

- No runtime, CLI, RDF, generated page, or ontology contract change is intended.
- Public TypeScript import compatibility from `src/core/weave/weave.ts` should be preserved.
- Internal module layout under `src/core/weave/` may change.

## Testing

- Before editing, record current exports and run a local import-graph audit rooted at `src/core/weave/weave.ts`.
- Before editing, run:
  - `deno task check`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
- After the type extraction, run:
  - `deno task check`
  - another import-graph/cycle check rooted at `src/core/weave/weave.ts`
  - confirm there are no new `src/core/` -> `src/runtime/` import edges
  - confirm there are no local cycles among `src/` modules
  - confirm runtime modules touched by the move import extracted type names from focused modules rather than from `src/core/weave/weave.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`
- Before finishing, run `deno task fmt`, `deno task lint`, and `deno task check`.

## Non-Goals

- Do not move `planWeave`, `planVersion`, `detectPendingWeaveSlice`, or slice planner functions in this task.
- Do not move Turtle renderers, RDF query helpers, or payload version layout helpers unless a tiny type-only move requires it.
- Do not move implementation-local interfaces merely to reduce line count; move them with their owning behavior later.
- Do not change public operation request shapes, target semantics, or generated outputs.
- Do not change the current ResourcePageDefinition support-history behavior in this type/model extraction. The mismatch between current-only support-artifact defaults and the legacy versioned `_knop/_page` progression is tracked as follow-up audit/documentation work in [[wd.todo]].
- Do not split runtime weave files here; use [[wa.task.2026.2026-05-21_1035-runtime-weave-module-decomposition]] and [[wa.completed.2026.2026-05-21_1036-runtime-resource-page-generation-decomposition]].
- Do not rename this task to completed as part of the implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction; keep this slice narrow and create/update follow-up tasks when needed.

- None found during implementation.

## Implementation Results

- Starting `src/core/weave/weave.ts` line count was 8,597; post-slice line count is 8,515.
- Pre-slice import graph rooted at `src/core/weave/weave.ts`: 13 local `src` modules, 21 local import edges, 12 direct local imports from `weave.ts`, 0 `src/core` -> `src/runtime` edges, and 0 local cycles.
- Post-slice import graph rooted at `src/core/weave/weave.ts`: 18 local `src` modules, 35 local import edges, 16 direct local imports from `weave.ts`, 0 `src/core` -> `src/runtime` edges, and 0 local cycles.
- `src/core/weave/weave.ts` now re-exports the moved public type names while planner behavior remains in place.
- Runtime modules touched by the move now import moved type names from `requests.ts`, `candidates.ts`, `source_models.ts`, or `slices.ts` instead of from the large façade.
- Verification passed:
  - `deno task fmt`
  - `deno task lint`
  - `deno task check`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Groom this note against the current post-1036 code before implementation.
- [x] Record current line count and exported type/interface names from `src/core/weave/weave.ts`; latest handoff count is 8,597 lines.
- [x] Run a pre-slice import graph/circular-dependency audit rooted at `src/core/weave/weave.ts`.
- [x] Move operation request types into `src/core/weave/requests.ts`.
- [x] Move `RepositorySourceFloatingLocator` into `src/core/weave/source_models.ts`, and re-export it from `resource_page_models.ts` and `weave.ts`.
- [x] Move exported working-artifact and candidate model types into `src/core/weave/candidates.ts`.
- [x] Move `PlanWeaveInput` and `WeavePlan` into `src/core/weave/planning_models.ts` unless the import graph audit exposes a real cycle.
- [x] Move `WeaveSlice` into `src/core/weave/slices.ts`.
- [x] ResourcePage model types have already moved via [[wa.completed.2026.2026-05-21_1036-runtime-resource-page-generation-decomposition]] into `src/core/weave/resource_page_models.ts`.
- [x] Re-export moved public types from `src/core/weave/weave.ts`.
- [x] Prefer `import type` for moved types in implementation files, and update touched runtime imports to the focused type modules.
- [x] Run `deno task check` after the first move.
- [x] Run focused core and integration tests.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this implementation slice.
- [d] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] if this slice reveals a better extraction order.
- [x] Provide a commit message that clearly says this is a behavior-preserving type/model extraction.

## Suggested Commit Message

```text
refactor(core-weave): extract request and candidate model types

- move core weave request contracts into requests.ts while preserving weave.ts type re-exports
- move reusable source locator, candidate, planning, and slice-name types into focused core weave modules
- update runtime weave type imports to use the focused model modules directly
- keep planner functions and generated RDF/Page behavior unchanged
- verify with fmt, lint, check, core weave tests, integration weave tests, and import-cycle audit
```
