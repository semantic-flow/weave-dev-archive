---
id: y6jyev4hxpw8ru1npk598ib
title: 2026 05 23_0040 Further Refactoring
desc: ''
updated: 1779522014232
created: 1779522009683
---

## Goals

- Keep a durable backlog of refactors that should follow the first core/runtime weave extraction wave.
- Separate test-structure cleanup from production behavior changes so future diffs remain reviewable.
- Continue reducing very large files such as `src/runtime/weave/pages.ts`, `scripts/fixture-ladder.ts`, `src/core/weave/weave_test.ts`, `tests/integration/weave_test.ts`, and `tests/scripts/fixture_ladder_test.ts`.
- Make fixture, renderer, and generated-output tests easier to extend for `weave import`, ResourcePage presentation config, and fixture-helper generalization work.
- Preserve settled generated RDF and HTML output unless a slice explicitly records and tests an intended semantic or presentation change.


## Summary

The recent weave extraction work made the core planner and renderer seams much better, but several files are still large enough that they slow future changes and make review harder. This task is a placeholder for the refactoring cleanup we have discussed but intentionally did not bundle into the move-only slices or the CodeRabbit review fixes.

The main shape: split large tests into topic-oriented files, extract repeated fixture/assertion helpers, and continue separating runtime page model collection from HTML rendering. This should happen in small output-preserving slices, with focused tests proving no fixture drift unless the task is explicitly about changing behavior.

## Discussion

We chose not to refactor tests aggressively during the production extraction because the existing fixture and integration tests were serving as the behavioral oracle. That was the right tradeoff during move-only work: changing the tests and production code at the same time would have made fixture drift harder to interpret.

Now that the first extraction wave is released, test cleanup becomes more attractive. The current tests are valuable but heavy: `src/core/weave/weave_test.ts`, `tests/integration/weave_test.ts`, and `tests/scripts/fixture_ladder_test.ts` each cover multiple behavioral layers. Future work such as [[wa.task.2026.2026-05-22_2253-resourcepage-config-and-templating]], [[wa.task.2026.2026-05-21_0907-import]], and [[wa.task.2026.2026-05-22_2308-fixture-helper-generalization]] will be easier if the tests expose smaller named helpers and more local assertions.

The production side has similar pressure. `src/runtime/weave/pages.ts` still mixes page model collection, panel construction, HTML escaping/rendering, built-in CSS, and some template-like behavior. `scripts/fixture-ladder.ts` mixes scenario definitions, CLI parsing, plan rendering, materialization/execution, git branch updates, and guardrail evaluation. These are not emergency problems, but they are the places future functionality will repeatedly touch.

The broad preference is conservative: extract cohesive modules when they let a future task touch fewer lines, not because a line count is aesthetically annoying. The outer fixture ladder and integration tests should remain as acceptance coverage even after helper extraction.

## Open Issues

- How far should fixture-ladder scenario declarations move toward data files versus TypeScript builders?
- Should `scripts/fixture-ladder.ts` remain a single public script module with internal submodules, or should scenario definitions, execution, and rendering become separate importable modules?
- Which generated-output assertions should keep exact string comparisons, and which should move to normalized RDF/HTML model assertions?
- Should helper extraction for fixture tests happen before or after [[wa.task.2026.2026-05-22_2308-fixture-helper-generalization]]? Recommendation: extract neutral test helpers first, then change fixture-specific production helpers.
- How much `src/runtime/weave/pages.ts` extraction should wait for [[wa.task.2026.2026-05-22_2253-resourcepage-config-and-templating]] so the eventual panel/template model does not churn twice?
- Should codebase-overview updates be batched after a cluster of refactors, or updated after each extraction slice?

## Decisions

- Test refactors should be separate from production semantic changes unless the test refactor is required to make a production change safe.
- Keep fixture ladder acceptance coverage even if lower-level tests become stronger.
- Prefer output-preserving extraction slices before behavior-changing generalization.
- Keep generated RDF/HTML fixture drift explicit: if output changes, the task must say why and tests should name the intended difference.
- Do not use this task as a dumping ground for unrelated cleanup; create smaller child tasks when a refactor has a clear target.

## Contract Changes

- No user-facing contract change is intended by this placeholder task.
- Refactoring may add test helpers, support modules, and internal renderer/model modules.
- Public CLI behavior, generated RDF, generated HTML, and fixture branch expectations should remain stable unless a child task explicitly changes them.

## Testing

- Run targeted tests for the module being extracted plus the relevant fixture/integration tests before and after each slice.
- For fixture-ladder refactors, keep `tests/scripts/fixture_ladder_test.ts` green and add smaller tests for any new helper modules.
- For core weave test splits, preserve all existing behavioral assertions while grouping them by concern such as payload history, source registry, resource page definitions, references, progression, and preservation.
- For runtime page refactors, run `src/runtime/weave/pages_test.ts`, runtime page-generation integration tests, and fixture comparisons that cover customized pages.
- Use exact output tests when preserving settled fixture behavior; add normalized Turtle/HTML assertions only where exact string comparisons are causing noise rather than catching meaningful regressions.
- Finish significant slices with `deno task fmt`, `deno task lint`, `deno task check`, and a focused `deno task test` or equivalent targeted test run.

## Non-Goals

- Do not redesign ResourcePage presentation config here; that belongs to [[wa.task.2026.2026-05-22_2253-resourcepage-config-and-templating]].
- Do not implement `weave import` here; that belongs to [[wa.task.2026.2026-05-21_0907-import]].
- Do not replace Alice Bio-specific or fixture-ladder production helpers here except as part of [[wa.task.2026.2026-05-22_2308-fixture-helper-generalization]].
- Do not remove broad integration coverage just because lower-level tests exist.
- Do not rename completed/task notes as part of this placeholder unless explicitly requested.

## Implementation Plan

- [ ] Inventory the remaining largest production and test files, including `src/runtime/weave/pages.ts`, `scripts/fixture-ladder.ts`, `src/core/weave/weave.ts`, `src/core/weave/weave_test.ts`, `tests/integration/weave_test.ts`, and `tests/scripts/fixture_ladder_test.ts`.
- [ ] Split `tests/scripts/fixture_ladder_test.ts` into helper-backed sections without changing assertions: argument parsing, planning, asset inventory, materialization, execution, branch updates, guardrails, and render output.
- [ ] Extract reusable fixture-ladder test helpers for temp repo setup, fake CLI scripts, git assertions, transition lookup, and scenario-invariant checks.
- [ ] Split `scripts/fixture-ladder.ts` into scenario definitions, CLI parsing, planning, materialization/execution, renderers, branch update helpers, and guardrail evaluation while preserving the public script behavior.
- [ ] Split `src/core/weave/weave_test.ts` into topic-oriented tests that mirror the extracted core weave modules.
- [ ] Split `tests/integration/weave_test.ts` into narrower integration files where fixture setup and assertions can be shared without hiding the acceptance scenarios.
- [ ] Continue extracting `src/runtime/weave/pages.ts` toward page model collection, panel models, HTML shell rendering, built-in stylesheet rendering, and custom page-source rendering seams.
- [ ] Revisit remaining `src/core/weave/weave.ts` orchestration after the current child extraction tasks land, and decide whether any planner orchestration can move without obscuring the main plan flow.
- [ ] Add or update `documentation/notes/wd.codebase-overview.md` after a meaningful refactor cluster so the module map stays truthful.
- [ ] Create smaller child tasks before implementing any slice that changes generated output, fixture semantics, CLI behavior, or ResourcePage presentation behavior.
