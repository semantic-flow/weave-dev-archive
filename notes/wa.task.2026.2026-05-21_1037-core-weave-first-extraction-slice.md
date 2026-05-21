---
id: evh0tcow0l532ujdetucpe6
title: 2026 05 21_1037 Core Weave First Extraction Slice
desc: ''
updated: 1779385025853
created: 1779385025853
---

## Goals

- Execute the first small slice of [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] by moving shared type/model definitions out of `src/core/weave/weave.ts`.
- Preserve public imports from `src/core/weave/weave.ts` through façade re-exports.
- Avoid behavior changes, generated RDF changes, generated ResourcePage changes, and semantic helper rewrites.
- Establish a clean dependency direction for later core weave planner extraction slices.

## Summary

The safest first step in decomposing `src/core/weave/weave.ts` is to move stable type/model definitions into one or more small modules. The current file starts with request types, runtime candidate models, planning inputs/results, ResourcePage model interfaces, and slice/progression-local interfaces. Moving the public and broadly shared model types first should reduce import pressure for later helper extraction without changing planner behavior.

This task is intentionally narrow. It should not move planner functions, Turtle renderers, RDF helpers, or payload layout logic except where a tiny type-only move is needed to avoid cycles.

## Discussion

Candidate modules:

- `src/core/weave/requests.ts`: `WeaveRequest`, `ValidateRequest`, `GenerateRequest`, `VersionRequest` if keeping them separate from result/planning models helps.
- `src/core/weave/candidates.ts`: `PayloadWorkingArtifact`, `RepositorySourceFloatingLocator`, `ReferenceCatalogWorkingArtifact`, `ReferenceTargetSourcePayloadArtifact`, `ExtractionSourceEvidenceModel`, `ResourcePageDefinitionWorkingArtifact`, and `WeaveableKnopCandidate`.
- `src/core/weave/resource_page_models.ts`: `IdentifierResourcePageModel`, `SimpleResourcePageModel`, `ReferenceCatalogResourcePageModel`, `KnopResourcePageModel`, `CustomIdentifierResourcePageModel`, raw-source panel, child identifier, reference link, and history group models.
- `src/core/weave/planning_models.ts`: `PlanWeaveInput`, `WeavePlan`, `WeaveSlice`, and any planning-only public types that are not better kept near implementation.

The exact module count should be conservative. One `models.ts` may be acceptable if it prevents churn, but separate modules may make later runtime page-generation decomposition easier. The main rule is to avoid a huge mechanical move that creates awkward import cycles.

The existing `src/core/weave/weave.ts` should continue to export the same public type names. Existing tests and runtime imports should not need broad changes in this first slice.

## Open Issues

- Should request types live with targeting/request normalization long term, or stay under core weave as the operation request surface?
- Is it better to extract one broad `models.ts` first, then split later, or create `requests.ts`, `candidates.ts`, and `resource_page_models.ts` now?
- Which currently non-exported interfaces should move now, and which should stay local until their implementation functions move?
- Are ResourcePage model types needed by runtime page-generation decomposition soon enough to justify moving them first?

## Decisions

- Preserve `src/core/weave/weave.ts` as the public façade for existing public type imports.
- Prefer type-only exports and imports where possible.
- Do not move behavior in this slice unless TypeScript cycles force a tiny supporting move.
- Do not change names unless a local name is actively misleading after extraction.
- Keep `core/targeting.ts` as the already-extracted home for target semantics; this slice should not move targeting types back under `core/weave`.

## Contract Changes

- No runtime, CLI, RDF, generated page, or ontology contract change is intended.
- Public TypeScript import compatibility from `src/core/weave/weave.ts` should be preserved.
- Internal module layout under `src/core/weave/` may change.

## Testing

- Before editing, record current exports and run:
  - `deno task check`
  - `deno test src/core/weave/weave_test.ts`
- After the type extraction, run:
  - `deno task check`
  - `deno test src/core/weave/weave_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`
- Before finishing, run `deno task fmt`, `deno task lint`, and `deno task check`.

## Non-Goals

- Do not move `planWeave`, `planVersion`, `detectPendingWeaveSlice`, or slice planner functions in this task.
- Do not move Turtle renderers, RDF query helpers, or payload version layout helpers unless a tiny type-only move requires it.
- Do not change public operation request shapes, target semantics, or generated outputs.
- Do not split runtime weave files here; use [[wa.task.2026.2026-05-21_1035-runtime-weave-module-decomposition]] and [[wa.task.2026.2026-05-21_1036-runtime-resource-page-generation-decomposition]].
- Do not rename this task to completed as part of the implementation unless explicitly requested.

## Implementation Plan

- [ ] Re-read [[wd.general-guidance]], [[wd.testing]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [ ] Record current line count and exported type/interface names from `src/core/weave/weave.ts`.
- [ ] Choose the first model module shape with the fewest import-cycle risks.
- [ ] Move request and/or candidate model types into the new module.
- [ ] Move ResourcePage model types if they do not create cycles and if doing so helps runtime page-generation decomposition.
- [ ] Re-export moved public types from `src/core/weave/weave.ts`.
- [ ] Prefer `import type` for moved types in implementation files.
- [ ] Run `deno task check` after the first move.
- [ ] Run focused core and integration tests.
- [ ] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] if this slice reveals a better extraction order.
- [ ] Provide a commit message that clearly says this is a behavior-preserving type/model extraction.
