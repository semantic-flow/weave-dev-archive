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

This task should decompose the runtime ResourcePage generation code into cohesive modules while preserving output. It should make later page-model redesign work easier, but it should not do that redesign.

## Discussion

Current likely extraction families inside `src/runtime/weave/weave.ts`:

- `generated_pages.ts`: `generatePreparedPages`, `collectGeneratedPageFiles`, generated path collection, favicon resolution, and page upsert/write behavior if dependencies are clean.
- `designator_contexts.ts`: `GenerateDesignatorContext`, designator context loading, best-effort context loading, child identifiers, owner titles, and Knop artifact link collection.
- `raw_source_panels.ts`: raw source panel collection for payloads, support artifacts, reference targets, extraction sources, historical states, and current artifacts.
- `reference_links.ts`: parsed reference link extraction, reference role labels, target model conversion, and reference target/source raw panels if that boundary stays coherent.
- `history_groups.ts`: history group collection, recency sorting, historical state model resolution, and owner raw-source linking.
- `resource_page_rdf.ts`: small RDF query helpers local to page generation if they are not shared with core planner extraction.

The extraction should proceed from low-risk helper clusters outward. The safest first moves are likely pure collectors or type-only model moves that do not change rendering order. Avoid reshaping data models unless the existing tests prove byte-identical output.

This task should coordinate with [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] because core and runtime both have ResourcePage model names. Avoid creating duplicate incompatible type families.

## Open Issues

- Should runtime ResourcePage model types stay imported from `src/core/weave/weave.ts`, or should they move to a shared core model module first under [[wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice]]?
- Which page-generation helpers are truly runtime-only, and which should eventually become core/page model helpers?
- Should `writeGeneratedPagesUpsert` move with generated page collection or remain near general planned-file write helpers?
- How much timing phase-name churn is acceptable for `WEAVE_TIMING=1` when page generation phases become more explicit?

## Decisions

- Preserve generated output. This task is move/refactor only.
- Do this after runtime orchestration seams are stable enough that `generatePreparedPages` has a clear call boundary.
- Keep page visual redesign, page content-model redesign, and generated HTML changes out of scope.
- Keep `executeGenerate` and `executeWeave` import/call surfaces stable.
- Prefer modules under `src/runtime/weave/` until a helper clearly belongs in `src/core/weave/` or another shared core location.

## Contract Changes

- No generated RDF, generated HTML, CLI, timing-env, or ontology behavior change is intended.
- Internal module layout may change under `src/runtime/weave/`.
- Page-generation timing may become more granular, but normal stdout/stderr semantics should remain compatible.

## Testing

- Before editing, run a focused baseline:
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/weave_test.ts tests/integration/validate_version_generate_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/e2e/weave_cli_test.ts`
- After each extraction, run the smallest tests that cover the moved helper family, usually selected `executeGenerate` and `executeWeave` tests from `tests/integration/weave_test.ts`.
- Before finishing, run `deno task fmt`, `deno task lint`, `deno task check`, and the focused integration/e2e page-generation tests.
- Use fixture/output diffs and timestamp-stable test runs to confirm generated page bytes did not drift.

## Non-Goals

- Do not redesign generated page visuals, CSS, HTML structure, page content models, or raw source panel semantics.
- Do not split candidate loading, version execution, or prepared weave orchestration; use [[wa.task.2026.2026-05-21_1035-runtime-weave-module-decomposition]].
- Do not split the core weave planner; use [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]].
- Do not introduce a new rendering engine or template system.
- Do not regenerate fixture ladders unless an existing test reveals drift that must be resolved.

## Implementation Plan

- [ ] Re-read [[wd.general-guidance]], [[wd.testing]], and the latest `src/runtime/weave/weave.ts` page-generation section before editing.
- [ ] Record the starting line count and list the page-generation helper families currently in `src/runtime/weave/weave.ts`.
- [ ] Confirm whether ResourcePage model types should be extracted by [[wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice]] first.
- [ ] Extract a low-risk pure helper cluster, such as generated path collection or child identifier collection.
- [ ] Extract raw source panel helpers once their dependencies are clear.
- [ ] Extract reference link helpers if they do not create cycles with raw source panels.
- [ ] Extract history group helpers after raw source panel and RDF helper boundaries are stable.
- [ ] Move `collectGeneratedPageFiles` / `generatePreparedPages` only after lower-level helpers have a clean module home.
- [ ] Run focused tests after each slice and keep diffs reviewable.
- [ ] Update [[wd.codebase-overview]] if the runtime page-generation module layout changes materially.
- [ ] Provide commit message(s) that identify move-only page-generation slices and any timing diagnostic changes separately.
