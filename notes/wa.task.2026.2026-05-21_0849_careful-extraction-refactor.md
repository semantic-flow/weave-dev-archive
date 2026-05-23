---
id: 4cql6p0o4p6kvdz50s5k7hh
title: 2026 05 21_0849_careful Extraction Refactor
desc: ''
updated: 1779511141737
created: 1779378607538
---

## Goals

- Reduce the size and risk of `src/core/weave/weave.ts` by extracting cohesive helper modules after the next release.
- Preserve externally visible behavior, public CLI behavior, generated RDF shape, and generated ResourcePage output while moving code.
- Make future release-state, payload-versioning, ResourcePage, and extraction-source changes easier to reason about without editing an 8k+ line file.
- Keep the refactor boring: small slices, focused tests after each slice, and no opportunistic semantic redesign.

## Summary

`src/core/weave/weave.ts` has grown into the central planning file for local weave behavior. It currently contains request/result types, candidate selection, pending slice classification, payload version layout, overwrite policy, mesh inventory progression, shape assertions, Turtle block rendering, RDF query helpers, ResourcePage model construction, and several legacy fixture-specific render paths.

That concentration made [[wa.task.2026.2026-05-21_0820-ci-idempotency-tests]] harder than it needed to be: a narrow generate/version idempotency change crossed planner code, payload state policy, low-level RDF helpers, and CLI-facing result behavior. This task is a careful post-release extraction refactor. It is about moving stable responsibilities into smaller modules while keeping `planWeave` and `planVersion` behavior unchanged.

This is a code-extraction refactor of the weave planner. It is not a refactor of the `weave extract` command.

## Discussion

Current rough shape:

- `src/core/weave/weave.ts` is about 2,016 lines after [[wa.completed.2026.2026-05-21_1037-core-weave-first-extraction-slice]] moved shared request, source, candidate, planning, and slice-name model types into focused modules, [[wa.completed.2026.2026-05-22_1358-core-weave-rdf-and-turtle-helper-extraction]] moved low-level RDF and Turtle helpers, [[wa.completed.2026.2026-05-22_1424-core-weave-slice-classification-extraction]] moved pending slice classification, [[wa.completed.2026.2026-05-22_1441-payload-version-layout-and-overwrite-state-planning]] moved payload layout and overwrite planning, [[wa.completed.2026.2026-05-22_1644-shape-assertions]] moved settled current-shape and source-locator assertions, [[wa.completed.2026.2026-05-22_2225-referencecatalog-current-link-extraction]] moved ReferenceCatalog current-link parsing, [[wa.completed.2026.2026-05-22_2235-resourcepage-model-builder-extraction]] moved ResourcePage model builders, [[wa.completed.2026.2026-05-22_2244-extract-source-locator-render-helpers]] moved source-locator render helpers, [[wa.completed.2026.2026-05-22_2252-payload-render-helpers]] moved payload KnopInventory render helpers, [[wa.completed.2026.2026-05-22_2117-core-weave-knop-support-render-preservation-extraction]] moved Knop support artifact preservation helpers, [[wa.task.2026.2026-05-22_2139-core-weave-mesh-inventory-renderer-extraction]] moved mesh-inventory renderer helpers, and [[wa.task.2026.2026-05-22_2206-core-weave-knop-inventory-renderer-extraction]] moved KnopInventory renderer helpers.
- `src/runtime/weave/weave.ts` is about 4,900 lines and has its own cleanup track in [[wa.completed.2026.2026-05-21_1035-runtime-weave-module-decomposition]] and [[wa.completed.2026.2026-05-21_1036-runtime-resource-page-generation-decomposition]]. This core planner task should not absorb those runtime refactors.
- The core file mixes high-level orchestration with low-level RDF/Turtle utilities. That makes small behavior changes feel like broad edits and makes review harder.
- `core/targeting.ts` now owns portable target request semantics after [[wa.task.2026.2026-04-08_1615-weave-orchestration-refactor]]. Do not pull target normalization, shared target derivation, or requested-target coverage back into `src/core/weave/weave.ts` during this extraction.
- Several helper families already have natural names and call boundaries: slice detection, payload state layout, rendered artifact history collection, ResourcePage model builders, Turtle block editing, RDF fact assertions/queries, and current-working-file source locator rendering.

Suggested extraction order:

1. Completed first executable slice: [[wa.completed.2026.2026-05-21_1037-core-weave-first-extraction-slice]] moved shared type/model definitions while preserving façade re-exports from `src/core/weave/weave.ts`.
2. Completed second executable slice: [[wa.completed.2026.2026-05-22_1358-core-weave-rdf-and-turtle-helper-extraction]] moved RDF query helpers and Turtle block helpers that do not depend on planner state.
3. Implemented third executable slice: [[wa.completed.2026.2026-05-22_1424-core-weave-slice-classification-extraction]] moved `detectPendingWeaveSlice`, `classifyWeaveSlice`, and the minimal supporting history/query helpers needed to avoid cycles.
4. Implemented fourth executable slice: [[wa.completed.2026.2026-05-22_1441-payload-version-layout-and-overwrite-state-planning]] moved `PayloadVersionLayout`, first/second payload layout resolution, and guarded overwrite-state planning helpers into focused modules.
5. Implemented fifth executable slice: [[wa.completed.2026.2026-05-22_1644-shape-assertions]] moved settled current-shape validators, source-locator assertions, shared working-file path normalization, and progression structural types into focused modules.
6. Implemented sixth executable slice: [[wa.completed.2026.2026-05-22_2225-referencecatalog-current-link-extraction]] moved ReferenceCatalog current-link parsing into a focused module.
7. Implemented seventh executable slice: [[wa.completed.2026.2026-05-22_2235-resourcepage-model-builder-extraction]] moved planned ResourcePage model builders into a focused module.
8. Implemented eighth executable slice: [[wa.completed.2026.2026-05-22_2244-extract-source-locator-render-helpers]] moved current working-file and repository floating-locator render helpers into a focused module.
9. Implemented ninth executable slice: [[wa.completed.2026.2026-05-22_2252-payload-render-helpers]] moved first and second payload KnopInventory render helpers into a focused module and moved shared support-history omission postprocessors.
10. Implemented tenth executable slice: [[wa.completed.2026.2026-05-22_2117-core-weave-knop-support-render-preservation-extraction]] moved carried KnopSourceRegistry/ReferenceCatalog preservation helpers into a focused module.
11. Implemented eleventh executable slice: [[wa.task.2026.2026-05-22_2139-core-weave-mesh-inventory-renderer-extraction]] moved mesh-inventory Turtle renderers and small mesh-inventory block helpers into a focused module.
12. Implemented twelfth executable slice: [[wa.task.2026.2026-05-22_2206-core-weave-knop-inventory-renderer-extraction]] moved first Knop, first ReferenceCatalog, current-only ReferenceCatalog, subsequent page-definition, and first extracted-Knop KnopInventory renderers into a focused module.
13. Later likely slices: rendered artifact history collection/render helpers, remaining page/HTML helper renderers, and legacy fixture-sensitive renderer replacement after the move-only phase. The remaining render helpers are still verbose and fixture-sensitive.

The healthiest end state is probably a small `src/core/weave/weave.ts` that coordinates target selection and dispatch, with neighbor modules doing specific work. It can continue to re-export stable public types from the same path until a later API cleanup is warranted.

## Open Issues

- [x] Public imports from `src/core/weave/weave.ts` should remain valid during this refactor. New internal modules may be imported directly by internal callers when useful, but `weave.ts` remains the compatibility façade for the current public core weave surface.
- [x] The first extraction slice should be model/type extraction, tracked separately in [[wa.completed.2026.2026-05-21_1037-core-weave-first-extraction-slice]].
- [x] The second extraction slice should be pure RDF query and Turtle block helper extraction, tracked separately in [[wa.completed.2026.2026-05-22_1358-core-weave-rdf-and-turtle-helper-extraction]].
- [x] The third extraction slice should be pending slice classification extraction, tracked separately in [[wa.completed.2026.2026-05-22_1424-core-weave-slice-classification-extraction]].
- [x] The fourth extraction slice should be payload version layout and overwrite-state planning extraction, tracked separately in [[wa.completed.2026.2026-05-22_1441-payload-version-layout-and-overwrite-state-planning]].
- [x] The fifth extraction slice should be current shape and source-locator assertion extraction, tracked separately in [[wa.completed.2026.2026-05-22_1644-shape-assertions]].
- [x] The sixth extraction slice should be ReferenceCatalog current-link extraction, tracked separately in [[wa.completed.2026.2026-05-22_2225-referencecatalog-current-link-extraction]].
- [x] The seventh extraction slice should be ResourcePage model-builder extraction, tracked separately in [[wa.completed.2026.2026-05-22_2235-resourcepage-model-builder-extraction]].
- [x] The eighth extraction slice should be source-locator render helper extraction, tracked separately in [[wa.completed.2026.2026-05-22_2244-extract-source-locator-render-helpers]].
- [x] The ninth extraction slice should be payload render helper extraction, tracked separately in [[wa.completed.2026.2026-05-22_2252-payload-render-helpers]].
- [x] The tenth extraction slice should be Knop support preservation helper extraction, tracked separately in [[wa.completed.2026.2026-05-22_2117-core-weave-knop-support-render-preservation-extraction]].
- [x] The eleventh extraction slice should be mesh-inventory renderer extraction, tracked separately in [[wa.task.2026.2026-05-22_2139-core-weave-mesh-inventory-renderer-extraction]].
- [x] The twelfth extraction slice should be KnopInventory renderer extraction, tracked separately in [[wa.task.2026.2026-05-22_2206-core-weave-knop-inventory-renderer-extraction]].
- [x] Runtime planner cleanup should not be done in this task. Use [[wa.completed.2026.2026-05-21_1035-runtime-weave-module-decomposition]] for runtime orchestration/candidate/version seams and [[wa.completed.2026.2026-05-21_1036-runtime-resource-page-generation-decomposition]] for runtime page-generation seams.

## Decisions

- Do this after the next release, not during the v0.1.2 release stabilization window.
- Treat the first pass as behavior-preserving. Move code, rename only when it clarifies an extracted module boundary, and avoid changing generated output.
- Keep `planWeave`, `planVersion`, `WeaveInputError`, `detectPendingWeaveSlice`, and current public type exports available through `src/core/weave/weave.ts` during the extraction.
- Prefer Deno-native module boundaries and existing local helper APIs. Do not add build tooling or code generation.
- Run focused tests after each meaningful slice and full `deno task test` before considering the refactor complete.
- Keep this task focused on core planner decomposition. Runtime module decomposition and runtime ResourcePage generation decomposition are related but separate tasks.
- Treat `core/targeting.ts` as an already-extracted shared dependency for target semantics, not as part of the remaining core weave planner split.

## Contract Changes

- No ontology, CLI, generated page, RDF, or public behavior change is intended.
- Module organization may change internally.
- Public TypeScript import compatibility from `src/core/weave/weave.ts` should be preserved for this task unless a specific import proves impossible to keep without a cycle.

## Testing

- Before moving code, capture a baseline with `deno task check`, `deno task lint`, and `deno task test` on the release branch.
- After each extraction slice, run the narrowest matching tests first:
  - `deno test src/core/weave/weave_test.ts`
  - `deno test tests/integration/validate_version_generate_test.ts`
  - `deno test tests/integration/weave_test.ts`
  - selected e2e tests when CLI-visible planner behavior could be affected.
- Before finishing, run `deno task fmt`, `deno task lint`, `deno task check`, and `deno task test`.
- Use git diffs and tests to confirm no fixture/generated-output drift is introduced by move-only changes.

## Non-Goals

- Do not change Semantic Flow ontology terms or shape semantics.
- Do not redesign payload versioning, ResourcePage rendering, generated timestamp policy, or release-state overwrite behavior in this task.
- Do not combine this with feature work, CLI documentation edits, or fixture regeneration unless a moved helper exposes an existing failing test.
- Do not hard-split everything in one patch. A partial extraction that leaves the codebase easier to review is better than a huge heroic diff.
- Do not rename task notes to completed notes as part of this task unless explicitly requested.

## Implementation Plan

- [x] Record the starting line count and current public exports from `src/core/weave/weave.ts`.
- [x] Use [[wa.completed.2026.2026-05-21_1037-core-weave-first-extraction-slice]] for the first model/type extraction slice.
- [x] Use [[wa.completed.2026.2026-05-22_1358-core-weave-rdf-and-turtle-helper-extraction]] for the second RDF/Turtle helper extraction slice.
- [x] Extract RDF query/resolution helpers that are pure and planner-independent.
- [x] Extract Turtle block editing/render support helpers that are pure and planner-independent.
- [x] Use [[wa.completed.2026.2026-05-22_1424-core-weave-slice-classification-extraction]] for the third slice-classification extraction slice.
- [x] Extract weave slice detection/classification helpers.
- [x] Use [[wa.completed.2026.2026-05-22_1441-payload-version-layout-and-overwrite-state-planning]] for the fourth payload layout and overwrite extraction slice.
- [x] Extract payload version layout and current-state overwrite helpers.
- [x] Use [[wa.completed.2026.2026-05-22_1644-shape-assertions]] for the fifth shape assertion extraction slice.
- [x] Extract settled shape assertions, source-locator assertions, working-file path normalization, and progression structural types.
- [x] Use [[wa.completed.2026.2026-05-22_2225-referencecatalog-current-link-extraction]] for the sixth ReferenceCatalog current-link extraction slice.
- [x] Extract ReferenceCatalog current-link parsing.
- [x] Use [[wa.completed.2026.2026-05-22_2235-resourcepage-model-builder-extraction]] for the seventh ResourcePage model-builder extraction slice.
- [x] Extract ResourcePage model-builder helpers.
- [x] Use [[wa.completed.2026.2026-05-22_2244-extract-source-locator-render-helpers]] for the eighth source-locator render helper extraction slice.
- [x] Extract current working-file and repository floating-locator render helpers.
- [x] Use [[wa.completed.2026.2026-05-22_2252-payload-render-helpers]] for the ninth payload render helper extraction slice.
- [x] Extract first and second payload KnopInventory render helpers plus shared support-history omission postprocessors.
- [x] Use [[wa.completed.2026.2026-05-22_2117-core-weave-knop-support-render-preservation-extraction]] for the tenth Knop support preservation extraction slice.
- [x] Extract carried KnopSourceRegistry and ReferenceCatalog preservation helpers.
- [x] Use [[wa.task.2026.2026-05-22_2139-core-weave-mesh-inventory-renderer-extraction]] for the eleventh mesh-inventory renderer extraction slice.
- [x] Extract mesh-inventory Turtle renderers and small local block helpers.
- [x] Use [[wa.task.2026.2026-05-22_2206-core-weave-knop-inventory-renderer-extraction]] for the twelfth KnopInventory renderer extraction slice.
- [x] Extract KnopInventory Turtle renderers and small local block helpers.
- [ ] After the move-only phase exposes stable renderer boundaries, create a follow-up task for replacing legacy Alice Bio-specific and other fixture-ladder render helpers with generalized renderers.
- [x] Update [[wd.codebase-overview]].
