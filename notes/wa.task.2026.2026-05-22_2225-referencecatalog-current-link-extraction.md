---
id: jvk7l35jg67zbrl9m1dlrxk
title: 2026 05 22_2225 Referencecatalog Current Link Extraction
desc: ''
updated: 1779506751753
created: 1779506751753
---

## Goals

- Execute the next conservative core weave extraction slice from [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_1644-shape-assertions]].
- Move ReferenceCatalog current-link parsing out of `src/core/weave/weave.ts` into a focused core weave module.
- Preserve planner dispatch, generated RDF, generated ResourcePage output, ReferenceCatalog validation behavior, and public imports through `src/core/weave/weave.ts`.
- Keep this as a move-only slice unless an existing test exposes a blocker.
- Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this task.

## Summary

`extractCurrentReferenceCatalogLinks` is a compact validation/parsing helper that reads a current ReferenceCatalog Turtle file and returns `ReferenceCatalogCurrentLinkModel` records used by the first reference-catalog weave slice.

The helper is adjacent to shape assertions, but it is more specific than generic settled-shape validation: it parses managed `ReferenceLink` fragment subjects, validates owner/link-target consistency, resolves role labels, and captures optional version-pinned target state paths. Pulling it into a focused module should reduce `weave.ts` without moving renderers, ResourcePage builders, or planner dispatch.

## Discussion

Candidate module:

- `src/core/weave/reference_catalog_links.ts`

Expected moved helper set:

- `extractCurrentReferenceCatalogLinks`
- a private `toReferenceRoleLabel` helper if needed
- a private `toLastPathSegment` helper if needed for role-label parsing

Expected dependencies:

- `SFLO_NAMESPACE` from `src/core/rdf/namespaces.ts`
- `WeaveInputError` from `src/core/weave/errors.ts`
- `ReferenceCatalogCurrentLinkModel` from `src/core/weave/resource_page_models.ts`
- RDF helpers from `src/core/weave/rdf_helpers.ts`

The extraction should not move ResourcePage builders even though they consume `ReferenceCatalogCurrentLinkModel`. Those builders have a broader page-model dependency shape and should stay in a later ResourcePage slice.

## Open Issues

- [x] Keep ReferenceCatalog parsing and validation behavior unchanged.
- [x] Preserve exact error messages.
- [x] Do not move ReferenceCatalog renderers, ResourcePage builders, or planner dispatch.
- [x] Do not broaden this into general ReferenceCatalog or ResourcePage cleanup.
- [x] Confirm the new module does not import from `src/core/weave/weave.ts` or `src/runtime/**`.

## Decisions

- Use a focused `reference_catalog_links.ts` module rather than adding this helper to `shape_assertions.ts`.
- Keep `ReferenceCatalogCurrentLinkModel` in `resource_page_models.ts`; this slice only moves parser logic.
- Keep `weave.ts` as the caller and public façade.
- Preserve role-label derivation exactly, including the `referenceRole_` prefix behavior.

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

- Do not change ReferenceCatalog or ReferenceLink ontology semantics.
- Do not change generated Turtle, generated ResourcePage output, target normalization, source locator behavior, or planner dispatch.
- Do not move ResourcePage builders, reference-catalog renderers, shape assertions, or payload renderers.
- Do not introduce new ReferenceCatalog behavior or fixture regeneration.
- Do not introduce a broad constants module.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No bugs or performance opportunities were found while implementing this slice.

## Implementation Results

Implemented behavior-preserving extraction:

- Added `src/core/weave/reference_catalog_links.ts` for `extractCurrentReferenceCatalogLinks` and its private role-label helpers.
- Updated `src/core/weave/weave.ts` to import the parser and removed now-local ReferenceLink constants and the old private helper body.
- Kept ReferenceCatalog renderers, ResourcePage builders, planner dispatch, generated RDF, and generated ResourcePage output unchanged.
- Reduced `src/core/weave/weave.ts` from 5,857 lines to 5,721 lines.
- Updated [[wd.codebase-overview]] and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] with the new module layout.

Verification:

- Pre-slice baseline: `src/core/weave/weave.ts` was 5,857 lines; import audit reported 25 modules, 87 edges, 35 direct root imports, 0 `src/core/weave` -> `src/runtime` imports, and 0 cycles.
- Pre-slice `deno task check` passed.
- Post-slice line counts: `weave.ts` 5,721; `reference_catalog_links.ts` 154.
- Post-slice import audit: 26 modules, 91 edges, 36 direct root imports, 0 `src/core/weave` -> `src/runtime` imports, 0 cycles.
- `deno task fmt`
- `deno task lint`
- `deno task check`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts` passed 56 tests.
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts` passed 72 tests.
- `git diff --check`

Suggested commit message:

```text
refactor(core-weave): extract reference catalog link parsing

- move current ReferenceCatalog link parsing into reference_catalog_links.ts
- keep role-label derivation private to the parser module
- preserve planner dispatch, generated RDF/Page output, and public weave façade exports

Verification:
- deno task fmt
- deno task lint
- deno task check
- WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts
- WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts
- import graph audit: 26 modules, 91 edges, 0 core->runtime imports, 0 cycles
- git diff --check
```

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and import graph/cycle audit for `src/core/weave/weave.ts`; latest handoff count is 5,857 lines.
- [x] Move `extractCurrentReferenceCatalogLinks` into `src/core/weave/reference_catalog_links.ts`.
- [x] Keep any helper role-label parsing private to the new module unless another current caller needs it.
- [x] Update `src/core/weave/weave.ts` imports and remove dead constants/helpers.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, and focused core/integration tests.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities".
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving ReferenceCatalog current-link extraction.
