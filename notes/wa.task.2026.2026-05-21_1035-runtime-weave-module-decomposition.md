---
id: akfrzoedu3djcikk32wha0p
title: 2026 05 21_1035 Runtime Weave Module Decomposition
desc: ''
updated: 1779384959022
created: 1779384959022
---

## Goals

- Reduce the size and review risk of `src/runtime/weave/weave.ts` by extracting runtime orchestration, prepared execution, candidate loading, and version-write helpers into cohesive modules.
- Preserve CLI/runtime behavior, generated RDF, generated ResourcePage output, timing counters, logging behavior, and local-path policy semantics.
- Keep `executeValidate`, `executeVersion`, `executeGenerate`, and `executeWeave` available from the current runtime weave module while moving implementation details behind smaller seams.
- Leave runtime ResourcePage generation decomposition to [[wa.task.2026.2026-05-21_1036-runtime-resource-page-generation-decomposition]] so this task does not become another broad rewrite.

## Summary

`src/runtime/weave/weave.ts` is still about 4,900 lines after [[wa.task.2026.2026-04-08_1615-weave-orchestration-refactor]]. The orchestration refactor created a useful `PreparedWeaveExecution` seam and moved portable target semantics into `core/targeting.ts`, but the runtime file still combines public command entry points, request normalization, mesh-state loading, local-path policy handling, staged candidate loading, overlay/cache behavior, version plan preparation, writes, RDF validation, page generation, logging, and utility helpers.

This task should split the non-page-generation runtime execution seams into smaller modules. It should be a behavior-preserving module decomposition, not a semantic refactor.

## Discussion

Candidate extraction boundaries:

- `prepared_execution.ts`: `NormalizedWeaveRequest`, `PreparedWeaveExecution`, `normalizeWeaveRequest`, and `prepareWeaveExecution` if the dependencies stay manageable.
- `version_execution.ts`: `PreparedVersionExecution`, `prepareVersionExecution`, `writePreparedVersion`, version-plan RDF validation, and planned-file write checks.
- `candidate_loader.ts`: `loadWeaveableKnopCandidates`, `loadWeaveableKnopCandidate`, and weaveable-candidate coverage checks, consuming the command-scoped planning context without owning staged filesystem/cache state.
- `mesh_state.ts`: runtime `MeshState`, `loadMeshState`, `ensureWorkspaceRootExists`, and workspace path conversion if those helpers do not create cycles.
- `planning_context.ts`: command-scoped read/staged-file overlay, dependency-aware candidate cache, planned-file staging, and cache/timing counters if this boundary keeps candidate loading and version execution from owning cross-cutting staged filesystem state.
- `artifact_loaders.ts`: shared runtime artifact-loading helpers used by both candidate loading and page-generation context loading, especially payload, reference catalog, source registry, extraction-source target, and ResourcePageDefinition working artifact loaders.
- `request_normalization.ts`: validate/version/generate request normalization and supported-key checks if they are useful outside the façade.

The current `src/runtime/weave/weave.ts` should remain the public façade for the four runtime operations and result descriptions. It can import extracted helpers from neighboring modules under `src/runtime/weave/`.

The page-generation block is large enough to deserve its own task. In this task, `generatePreparedPages` can remain where it is or move only if doing so is a small prerequisite for the execution split. Do not attempt to split designator contexts, raw source panels, history groups, and reference rendering here.

## Open Issues

- [x] Which exact boundary should own `TextFileOverlay`: candidate loading, prepared version execution, or a small runtime planning context module? Decision: put it in a small runtime planning context module because it owns command-scoped staged filesystem state, read caching, dependency-aware candidate caching, planned-file staging, cache invalidation, and cache counters across mesh-state loading, candidate loading, and version preparation.
- [x] Should `executeVersion` and `executeWeave` share `writePreparedVersion` from a `version_execution.ts` helper, or should `executeVersion` itself move to a smaller façade module later? Decision: keep sharing `writePreparedVersion` from `version_execution.ts`; keep `executeVersion` itself in `weave.ts` during this task so `weave.ts` remains the public façade.
- [x] Can `MeshState` be split cleanly before ResourcePage generation moves, or will page generation keep enough dependencies that `loadMeshState` should remain in the façade until the page task? Decision: split `MeshState` and `loadMeshState` into `mesh_state.ts` now. Page generation can depend on this shared runtime value without forcing the façade to own it. If needed, move `WeaveRuntimeError` to a small `errors.ts` and re-export it from `weave.ts` to avoid import cycles.
- [x] Should timing phase names remain byte-for-byte stable, or is a controlled phase-prefix cleanup acceptable as long as `WEAVE_TIMING=1` remains diagnostic? Decision: keep phase names byte-for-byte stable during this behavior-preserving task. Any timing name cleanup should be a separate tiny task with recorded before/after implications.
- [x] Should artifact-loading helpers live under `candidate_loader.ts` if they are first noticed while extracting candidates? Decision: no. Add an `artifact_loaders.ts` boundary for helpers shared by candidate loading and page-generation context loading so page generation does not import from a semantically candidate-specific module.

## Decisions

- This task is runtime-only and behavior-preserving.
- `core/targeting.ts` remains the home of portable target semantics; do not duplicate or move those rules back into runtime.
- Keep public imports from `src/runtime/weave/weave.ts` valid during this task.
- Preserve timing counters and cache counters unless an intentional diagnostic naming adjustment is recorded.
- Do not decompose runtime ResourcePage generation in this task except for small call-site preparation needed by the orchestration split.
- `TextFileOverlay` belongs to a command-scoped planning context boundary rather than candidate loading or version writing.
- `writePreparedVersion` should be extracted and shared by `executeVersion` and `executeWeave`; the public command façades stay in `weave.ts` in this slice.
- `MeshState` and `loadMeshState` should move to a shared runtime mesh-state module before the ResourcePage generation task.
- Shared artifact loaders used by both version candidate loading and page-generation context loading should move to `artifact_loaders.ts`, not `candidate_loader.ts`.
- Timing phase names should remain stable in this decomposition.

## Contract Changes

- No CLI, daemon-facing runtime, generated RDF, generated HTML, logging, or ontology contract change is intended.
- Internal module layout may change under `src/runtime/weave/`.
- `WEAVE_TIMING=1` should continue to expose enough phase detail to compare before/after runs.

## Testing

- Before editing, run focused baseline checks for runtime weave behavior:
  - `deno test src/core/targeting_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/weave_test.ts tests/integration/validate_version_generate_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/e2e/weave_cli_test.ts --filter "WEAVE_TIMING emits aggregate timings to stderr"`
- After each extraction slice, run the narrowest matching tests plus `deno task check`.
- Before finishing, run `deno task fmt`, `deno task lint`, `deno task check`, and focused integration/e2e tests affected by the moved seam.
- For any change that could affect performance or timing shape, run a small before/after `WEAVE_TIMING=1` comparison and record material observations in `timings/weave-performance.csv` if useful.

## Non-Goals

- Do not redesign ResourcePage generation, page models, raw source panels, reference rendering, or history group collection; use [[wa.task.2026.2026-05-21_1036-runtime-resource-page-generation-decomposition]].
- Do not split `src/core/weave/weave.ts`; use [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice]].
- Do not change target semantics, exact-vs-recursive coverage behavior, or version target parsing.
- Do not add persistent caches, file watchers, or daemon-level state.
- Do not combine this with page visual changes, fixture regeneration, or CLI documentation rewrites.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[wd.performance]], and [[wa.task.2026.2026-04-08_1615-weave-orchestration-refactor]] before editing.
- [x] Record current line counts and public exports from `src/runtime/weave/weave.ts`. Baseline before this slice was 4,864 lines; after timing instrumentation plus `errors.ts`, `mesh_state.ts`, `planning_context.ts`, `artifact_loaders.ts`, `candidate_loader.ts`, `request_normalization.ts`, and `timing_helpers.ts`, `weave.ts` is 3,974 lines and the public façade still exports the runtime result/option types, `executeValidate`, `executeVersion`, `executeGenerate`, `executeWeave`, result describers, and `WeaveRuntimeError`.
- [x] Identify import-cycle risks around `MeshState`, `TextFileOverlay`, candidate loaders, and page generation.
- [x] If needed to avoid cycles, extract `WeaveRuntimeError` to `src/runtime/weave/errors.ts` and re-export it from the public façade.
- [x] Extract `MeshState` and `loadMeshState` into `mesh_state.ts` while keeping existing error behavior.
- [x] Extract command-scoped overlay/cache behavior into `planning_context.ts`, preserving staged read semantics and cache counter field names.
- [x] Extract shared runtime artifact-loading helpers into `artifact_loaders.ts` before extracting candidate loading, so page generation does not depend on candidate-specific modules.
- [x] Extract the smallest helper module first, preferably version-write or request-normalization helpers.
- [ ] Extract `PreparedWeaveExecution` preparation if dependencies stay local and readable.
- [x] Extract candidate loading after `planning_context.ts` and `artifact_loaders.ts` are stable.
- [ ] Extract version execution/write helpers once candidate loading boundaries are stable.
- [x] Keep timing phase names stable during extraction unless a separate diagnostic cleanup task is opened.
- [x] Keep `src/runtime/weave/weave.ts` as the public façade for runtime commands.
- [x] Run focused tests after each slice and keep diffs reviewable.
- [x] Update [[wd.codebase-overview]] if the runtime module layout changes materially.
- [ ] Provide commit message(s) that separate move-only module extraction from any incidental cleanup.
