---
id: 32a3xrm6ciha0whhsu1z8ib
title: 2026 05 21_1036 Runtime Resource Page Generation Decomposition
desc: ''
updated: 1779384990627
created: 1779384990627
---

## Goals

- Reduce the size and risk of the runtime ResourcePage generation area currently embedded in `src/runtime/weave/weave.ts`.
- Preserve generated HTML byte shape, generated page paths, timestamp-skip behavior, raw source panels, reference link rendering, history group rendering, and ResourcePage policy behavior.
- Create clearer module boundaries for future page-model work without doing the page-model redesign in this task.
- Keep `executeGenerate` and composed `executeWeave` behavior stable while moving implementation details into smaller runtime modules.

## Summary

The runtime weave module contains a large ResourcePage generation subsystem: generated page collection, runtime ResourcePage policy checks, designator context loading, child identifier discovery, RDF type/title extraction, raw source panels, reference link models, history group collection, custom page definitions, and final page writes. This is separable from the runtime orchestration/candidate/version execution split tracked in [[wa.task.2026.2026-05-21_1035-runtime-weave-module-decomposition]].

This task should decompose the runtime ResourcePage generation code into cohesive modules while preserving output. It should make later page-model redesign work easier, but it should not do that redesign. Name churn is acceptable for internal module names because there are no external users yet; use the clearest boundaries rather than preserving awkward transitional names.

## Discussion

Current extraction families inside `src/runtime/weave/weave.ts`:

- `resource_page_models.ts`: shared ResourcePage model types currently exported from `src/core/weave/weave.ts`, including `ResourcePageModel`, raw-source panel models, reference-link models, child identifier models, history-group models, and Knop artifact link models.
- `history_groups.ts`: pure history group collection, recency sorting, historical state model resolution, history component classification, and history-aware descriptions. The pure RDF/model parts are good shared-core candidates; workspace inventory loading wrappers remain runtime.
- `reference_links.ts`: parsed ReferenceCatalog link extraction, reference role labels, and target model conversion. Keep canonical reference target raw-source panel loading out of this module because that code reads local workspaces and belongs with runtime raw-source panels.
- `raw_source_panels.ts`: raw source panel collection/read helpers for payloads, support artifacts, reference targets, extraction sources, historical states, and current artifacts. Most of this is runtime-only today because it depends on `Deno.stat`, `Deno.readTextFile`, `OperationalLocalPathPolicy`, local-path allowlists, floating repository source resolution, and best-effort missing-file behavior.
- `page_contexts.ts`: `GenerateDesignatorContext`, `loadGenerateDesignatorContexts`, best-effort child context loading, child identifiers, owner titles, and Knop artifact link collection. This is runtime-only for now because it reads current workspace inventories, loads runtime artifacts, resolves effective ResourcePageDefinition state, and tolerates runtime failures when loading child type hints.
- `page_generation.ts`: `generatePreparedPages`, `collectGeneratedPageFiles`, generated path collection, page model assembly, favicon resolution, and generated HTML page write/upsert behavior. Move this after the lower-level modules exist; it is currently the top of the dependency cone.
- `resource_page_rdf.ts` or `rdf_helpers.ts`: only if the extraction leaves reusable RDF query helpers that do not clearly belong to history groups or reference links. Avoid a miscellaneous dumping ground.

The extraction should proceed from low-risk helper clusters outward. The safest first moves are type-only model moves and pure collectors that do not change rendering order. Avoid reshaping data models unless the existing tests prove byte-identical output.

Do not extract `generatePreparedPages` or `collectGeneratedPageFiles` first. Those functions sit at the top of the page-generation dependency cone and will either drag most helper families with them or create awkward temporary cycles. Create the destination file name early only if it helps land imports, then move leaf clusters first.

This task should coordinate with [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice]] because core and runtime both use ResourcePage model names. Avoid creating duplicate incompatible type families.

## Open Issue Resolutions

- ResourcePage model types should move out of `src/core/weave/weave.ts` into a shared core model module before the runtime page-generation split goes far. Use the current shared layer, likely `src/core/weave/resource_page_models.ts`, rather than adding a new top-level `src/shared/` convention in this task unless [[wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice]] makes that architecture decision first.
- The helpers that are truly runtime-only today are the workspace readers, raw-source panel file readers, `OperationalLocalPathPolicy` calls, floating repository source resolution, effective-config loading, ResourcePageDefinition artifact loading, best-effort child context loading, and generated HTML writing.
- The helpers most likely to become shared/core are ResourcePage model types, pure history group parsing/merging/modeling, pure ReferenceCatalog parsing/model conversion, path-to-resource helpers, small RDF query helpers, and maybe byte-limit raw-source panel model construction once an API/service wants to present source excerpts.
- `writeGeneratedPagesUpsert` should move with `page_generation.ts`, not with general planned-file writing. Its timestamp-only skip behavior is generated-HTML-specific and depends on the current ResourcePage footer shape.
- Timing phase-name churn is acceptable, but keep it out of the first move-only slices when practical. If timing names are changed, do it as a deliberate follow-up slice with `WEAVE_TIMING=1` e2e assertions updated in the same commit.

## Decisions

- Preserve generated output. This task is move/refactor only.
- Do this after runtime orchestration seams are stable enough that `generatePreparedPages` has a clear call boundary.
- Keep page visual redesign, page content-model redesign, and generated HTML changes out of scope.
- Keep `executeGenerate` and `executeWeave` import/call surfaces stable.
- Put API/service-shareable models and pure helpers in the shared core layer. Prefer modules under `src/runtime/weave/` only when a helper depends on local filesystem runtime behavior, operational policy, logging/timing, or best-effort local recovery.
- Move `ResourcePageModel` and related model types first, or coordinate so [[wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice]] does it before this task starts significant runtime extraction.
- Treat `page_generation.ts` as the final assembly/orchestration module, not the first extraction target.

## Contract Changes

- No generated RDF, generated HTML, CLI, timing-env, or ontology behavior change is intended.
- Internal module layout may change under `src/core/weave/` and `src/runtime/weave/`.
- Page-generation timing may become more granular, but normal stdout/stderr semantics should remain compatible. Existing `WEAVE_TIMING=1` assertions should be updated only when phase names intentionally change.

## Testing

- Before editing, run a focused baseline:
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/weave_test.ts tests/integration/validate_version_generate_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/e2e/weave_cli_test.ts`
- Baseline captured during task review:
  - integration focused baseline: 72 passed, 0 failed, 17.17s real
  - e2e focused baseline: 35 passed, 0 failed, 13.15s real
  - representative `WEAVE_TIMING=1 weave generate --target designatorPath=alice/bio`: `generate.total 139.3ms`, `generate.collectGeneratedPageFiles 125.1ms`, `generate.collectGeneratedPageFiles.renderResourcePages 107.9ms`, `generate.writePages 3.6ms`
- After each extraction, run the smallest tests that cover the moved helper family, usually selected `executeGenerate` and `executeWeave` tests from `tests/integration/weave_test.ts`.
- After moving shared model types, run at least `deno task check` plus core weave tests that assert `createdPages`.
- After moving history/reference/raw-source/page-context helpers, run the focused `executeGenerate` tests covering sidecar histories, managed references, latest-state raw panels, floating repository source locators, Semantic Flow metadata, and support artifact source panels.
- Before finishing, run `deno task fmt`, `deno task lint`, `deno task check`, and the focused integration/e2e page-generation tests.
- Use fixture/output diffs and timestamp-stable test runs to confirm generated page bytes did not drift.

## Non-Goals

- Do not redesign generated page visuals, CSS, HTML structure, page content models, or raw source panel semantics.
- Do not split candidate loading, version execution, or prepared weave orchestration; use [[wa.task.2026.2026-05-21_1035-runtime-weave-module-decomposition]].
- Do not split the core weave planner; use [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]].
- Do not introduce a new rendering engine or template system.
- Do not regenerate fixture ladders unless an existing test reveals drift that must be resolved.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], and the latest `src/runtime/weave/weave.ts` page-generation section before editing.
- [x] Record the starting line count and list the page-generation helper families currently in `src/runtime/weave/weave.ts`; review baseline was 3,430 lines.
- [x] Coordinate with [[wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice]] and move ResourcePage model types from `src/core/weave/weave.ts` into a shared core model module before broad runtime extraction.
- [x] Extract pure history group model helpers into a shared core module if dependency direction stays clean; keep runtime inventory-loading wrappers under `src/runtime/weave/`.
- [ ] Extract pure ReferenceCatalog parsing and reference target model helpers; leave canonical reference source raw-panel loading with raw-source/runtime helpers.
- [ ] Extract raw source panel helpers under `src/runtime/weave/raw_source_panels.ts`, with any obviously pure byte-limit/model construction helper separated only if it keeps the module simpler.
- [ ] Extract `GenerateDesignatorContext`, `loadGenerateDesignatorContexts`, best-effort child context loading, child identifier collection, owner title resolution, and Knop artifact link collection into `src/runtime/weave/page_contexts.ts`.
- [ ] Extract the page model assembly helpers that are still local to generation, such as generated resource path selection, displayed child path collection, favicon resolution, and Semantic Flow resource descriptions.
- [ ] Move `collectGeneratedPageFiles` and `generatePreparedPages` into `src/runtime/weave/page_generation.ts` after lower-level helpers have clean module homes.
- [ ] Move `writeGeneratedPagesUpsert` and timestamp-footer normalization into `page_generation.ts` with generated page collection/write behavior.
- [ ] Keep timing phase names stable during move-only slices; if clearer page-generation timing names are desired, land them as a distinct slice with e2e timing assertions updated.
- [ ] Run focused tests after each slice and keep diffs reviewable.
- [ ] Update [[wd.codebase-overview]] if the runtime page-generation module layout changes materially.
- [ ] Provide commit message(s) that identify move-only page-generation slices and any timing diagnostic changes separately.
