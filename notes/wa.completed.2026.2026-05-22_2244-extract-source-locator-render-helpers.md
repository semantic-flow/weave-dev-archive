---
id: ublteg1v9q9o07fwysiqhhn
title: 2026 05 22_2244 Extract Source Locator Render Helpers
desc: ''
updated: 1779507905211
created: 1779507905211
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_2235-resourcepage-model-builder-extraction]].
- Move current working-file/source-locator render helpers out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve generated Turtle text, planner dispatch, generated ResourcePage output, and public imports through `src/core/weave/weave.ts`.
- Keep source-locator assertions and working-file path normalization in their existing focused modules.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

`src/core/weave/weave.ts` still contains a compact cluster of helper functions that render the current working-file locator predicate used across several generated Turtle blocks. These helpers decide between:

- `sflo:hasRepositorySourceFloatingLocator` with a blank node
- `sflo:hasWorkingLocatedFile` for mesh-local current bytes
- `sflo:workingLocalRelativePath` for local paths outside the mesh surface

The source-locator assertion and path-normalization pieces have already been extracted. This slice should move only the rendering side into a focused module so the remaining planner file loses one more small, repeated low-level responsibility without changing the generated RDF shape.

## Discussion

Candidate module:

- `src/core/weave/source_locator_renderers.ts`

Expected moved helper set:

- `renderCurrentWorkingFileLocator`
- `renderCurrentWorkingFileDeclaration`
- `renderRepositorySourceFloatingLocatorBlankNode`

Expected dependencies:

- `RepositorySourceFloatingLocator` from `src/core/weave/source_models.ts`
- `usesMeshLocalWorkingLocatedFile` from `src/core/weave/working_file_paths.ts`

Boundary notes:

- The helper dependency on `working_file_paths.ts` is acceptable because that module owns the shared current working-file path predicate and does not pull planner state.
- The pre-slice import graph already reaches `working_file_paths.ts` directly from `weave.ts`; this extraction moves that edge behind `source_locator_renderers.ts` rather than introducing a new class of dependency for the planner.
- Do not merge this with `source_locator_assertions.ts`. Assertions read parsed RDF and validate shape; renderers emit Turtle snippets.
- Do not add a general Turtle renderer abstraction in this slice. These helpers are still string renderers with deliberately unchanged formatting.

## Open Issues

- [x] Confirm `working_file_paths.ts` is already in the `weave.ts` import graph before the move, so the new renderer module is not adding a surprising transitive dependency class.
- [x] Keep generated Turtle snippets byte-for-byte equivalent for the covered helper outputs.
- [x] Do not move source-locator assertions or shape validation.
- [x] Confirm the new module does not import from `src/core/weave/weave.ts` or `src/runtime/**`.
- [x] Record any formatter-induced whitespace changes that might affect generated output review.

## Decisions

- Use a focused `source_locator_renderers.ts` module rather than broadening `source_locator_assertions.ts` or `working_file_paths.ts`.
- Keep `renderRepositorySourceFloatingLocatorBlankNode` exported from the renderer module even if the initial external caller only needs `renderCurrentWorkingFileLocator`; it is part of the render helper family and makes the boundary explicit.
- Keep `weave.ts` as the planner dispatcher and public façade.
- Preserve the current JSON string literal rendering strategy for locator fields and local paths.

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

- Do not change source-locator ontology semantics.
- Do not change generated Turtle, generated ResourcePage output, target normalization, source locator assertion behavior, or planner dispatch.
- Do not move payload render helpers, rendered-history collection, ResourcePage model builders, or ReferenceCatalog parsing in this slice.
- Do not introduce new validation behavior, fixture regeneration, or semantic cleanup.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No bugs or performance opportunities were found while implementing this slice.

## Implementation Results

Implemented behavior-preserving extraction:

- Added `src/core/weave/source_locator_renderers.ts` for current working-file locator rendering, current working located-file declaration rendering, and repository source floating-locator blank-node rendering.
- Updated `src/core/weave/weave.ts` to import the renderer helpers and removed the old local helper bodies.
- Kept `source_locator_assertions.ts`, `working_file_paths.ts`, planner dispatch, generated RDF, and generated ResourcePage output unchanged.
- Reduced `src/core/weave/weave.ts` from 5,234 lines to 5,195 lines.
- Updated [[wd.codebase-overview]] and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] with the new module layout.

Verification:

- Pre-slice baseline: `src/core/weave/weave.ts` was 5,234 lines; import audit reported 27 modules, 97 edges, 36 direct root imports, 0 `src/core/weave` -> `src/runtime` imports, and 0 cycles.
- Pre-slice `deno task check` passed.
- Post-slice line counts: `weave.ts` 5,195; `source_locator_renderers.ts` 44.
- Post-slice import audit: 28 modules, 99 edges, 36 direct root imports, 0 `src/core/weave` -> `src/runtime` imports, 0 cycles.
- `deno task fmt`
- `deno task lint`
- `deno task check`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts` passed 56 tests.
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts` passed 72 tests.
- `git diff --check`

Suggested commit message:

```text
refactor(core-weave): extract source locator render helpers

- move current working-file and repository floating-locator render helpers into source_locator_renderers.ts
- keep source locator assertions, path normalization, planner dispatch, and generated RDF/Page output unchanged
- preserve the existing core weave public façade while reducing weave.ts renderer clutter

Verification:
- deno task fmt
- deno task lint
- deno task check
- WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts
- WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts
- import graph audit: 28 modules, 99 edges, 0 core->runtime imports, 0 cycles
- git diff --check
```

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest pre-slice count is 5,234 lines.
- [x] Confirm `working_file_paths.ts` is already in the `weave.ts` import graph before this extraction.
- [x] Move source-locator render helpers into `src/core/weave/source_locator_renderers.ts`.
- [x] Update `src/core/weave/weave.ts` imports and remove dead local helper definitions.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, and focused core/integration tests.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving source-locator renderer extraction.
