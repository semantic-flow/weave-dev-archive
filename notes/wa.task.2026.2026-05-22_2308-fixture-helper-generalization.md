---
id: w1atk5wuk9doldnsaiv7hlm
title: 2026 05 22_2308 Fixture Helper Generalization
desc: ''
updated: 1779516518268
created: 1779516506731
---

## Goals

Replace legacy Alice Bio-specific and fixture-ladder render helpers with generalized renderer/model paths, preserving generated RDF and ResourcePage output while removing fixture-name assumptions from core weave rendering.

- Make `scripts/fixture-ladder.ts` easier to extend for new whole-mesh, sidecar, branch-published, import, and custom ResourcePage fixture scenarios.
- Reduce copy/paste in fixture transition definitions by introducing small reusable data builders for file operations, command actions, branch-publication actions, replay inputs, and fixture asset references.
- Keep this slice behavior-preserving: no intended generated RDF, generated HTML, manifest, or fixture branch output changes.
- Prepare for [[wa.task.2026.2026-05-21_0907-import]] and [[wa.task.2026.2026-05-23_2230-custom-resourcepage-shared-shell-fixture]] without adding Carol/import/custom ResourcePage fixture rungs yet.
- Generate Accord `ScenarioIndex` documents for the existing fixture scenarios as part of the helper generalization, without replacing transition manifests or changing `accord check` behavior.

## Summary

The fixture ladder script has become important infrastructure. It now describes multiple fixture scenarios and transition families, but parts of the code still read like the Alice Bio ladder was the only fixture. That makes the next wave awkward: import needs GitHub-backed HTTP fixture content, Carol custom ResourcePages need stackable rungs, and sidecar/branch-published cases need topology-specific source/output boundaries.

This task should generalize fixture helper code before we add more fixture rungs. It should not regenerate existing ladders. If this work changes fixture output, that is a bug or an accidentally mixed semantic change unless explicitly approved.

## Discussion

Known pressure points:

- Alice Bio transition definitions carry several hand-authored file-operation blocks that could become reusable helpers.
- Fixture asset copy entries repeat source/target/description shapes.
- Command, file-operation, and branch-publication transition construction each have enough common shape to merit small builders.
- Future import fixtures need explicit support for GitHub-backed source acquisition without requiring a local HTTP server.
- Future custom ResourcePage fixtures should be easy to append as new rungs without rewriting old scenario definitions.
- Sidecar and branch-published fixture definitions should remain first-class, not special cases buried under Alice-specific helper names.
- Fixture support modules still carry temporary branch-prefix and replay-policy constants that should eventually move into Accord scenario indexes.

The desired change is a code-organization refactor of fixture planning and execution helpers. It should make the script easier to read and extend, not introduce a new fixture DSL unless the existing shape is genuinely blocking. Conservative helper extraction is better than a grand framework.

Good extraction targets:

- `fixtureAssetSource(...)` or similar for deterministic fixture asset copy declarations.
- `fileOperationTransition(...)` for transitions that materialize checked-in assets and optional inventory patches.
- `commandTransition(...)` for ordinary CLI replay transitions.
- `branchPublicationTransition(...)` for source-to-publication branch replay transitions.
- shared functions for replay input paths, fixture asset descriptions, transition id/path normalization, and guardrail lists.
- scenario-local constants for repeated descriptions or asset path roots.
- pure rendering helpers for an Accord `ScenarioIndex` document from the generalized fixture scenario definitions.
- script support to write or verify generated scenario index JSON-LD files for Alice Bio, Sidecar Fantasy Rules, and Branch Fantasy Rules.

Current code already has early `commandTransition`, `fileTransition`, and `branchPublicationTransition` helpers. Treat those as the starting point: improve their names, inputs, defaults, and exported test surface where useful instead of replacing them with a larger abstraction.

This should probably happen before the Carol/import/custom ResourcePage fixture if the helper extraction stays small and mechanical. If it starts turning into a larger fixture framework redesign, pause and do the narrow Carol/import path first instead.

### Accord Scenario Indexes

Accord already defines `ScenarioIndex` as the portable JSON-LD shape for fixture-owned topology documents. The Accord user guide is explicit that a scenario index is not a replacement for a transition manifest and is not consumed by `accord check` today. It describes how multiple manifests fit into an ordered scenario.

That makes scenario-index generation a good fit for this task. The fixture ladder already owns enough scenario topology to render indexes: default fixture repo, branch prefix, asset root, ordered steps, manifest paths, and lane state bindings. Generating these indexes would move the "temporary until replay prefix moves into an Accord/scenario master manifest" comments toward the intended shape.

Scope boundary: generate and validate scenario index files, but keep the TypeScript fixture definitions as the implementation source of truth for this refactor. Do not make the fixture-ladder script read scenario indexes as its runtime source yet, and do not change how `accord check` evaluates individual transition manifests.

## Open Issues

No blocking open issues remain for this behavior-preserving helper-generalization slice.

Watch during implementation:

- If scenario-index lane rendering for the first branch-published publication step cannot represent the empty publication root cleanly with Accord's current optional `fromLaneState`, pause that part rather than inventing new Accord vocabulary in this Weave task.
- If helper extraction starts changing generated fixture output, fixture branch refs, or dry-run text beyond explicitly reviewed wording, split the semantic/output change into a separate task.

## Decisions

- This is a behavior-preserving refactor. No full fixture ladder regeneration is expected.
- Keep scenario names such as Alice Bio, Sidecar Fantasy Rules, and Branch Fantasy Rules where they are genuine fixture labels.
- Remove Alice-specific assumptions from shared helper names and helper implementation paths.
- Prefer incremental helper extraction over a new declarative fixture framework.
- Use a modest typed transition-builder API for the repeated `command`, `fileOperation`, and `branchPublication` shapes, but avoid a new fixture DSL. The builder should reduce boilerplate and make scenario-index rendering easier; it should not hide the actual transition data.
- Generalize deterministic fixture asset declarations now, because asset declarations are already repeated in Alice Bio and Fantasy Rules. Keep the helper small, such as `fixtureAssetSource(...)`, so Carol can reuse it later without another refactor.
- Add targeted tests for the new builders and scenario-index rendering in addition to existing fixture-ladder dry-run/execute tests.
- Keep Alice-specific names only for scenario labels, scenario constants, and Alice fixture content. Shared helpers should be scenario-neutral.
- Do this before import/custom ResourcePage fixture expansion if the diff remains small and reviewable.
- Generate Accord scenario indexes from fixture-ladder scenario definitions, but do not make them the runtime source of truth in this slice.
- Name generated fixture scenario index files `scenario-index.jsonld` and keep them next to the transition manifests in each scenario's `conformance/` directory.
- Render single-lane scenario indexes for Alice Bio and Sidecar Fantasy Rules with one fixture-state lane, and render Branch Fantasy Rules with separate `source` and `publication` lanes. For branch-publication steps that only read the source lane, it is acceptable for the source lane binding to point from and to the same source ref.
- Validate generated scenario indexes by reading the checked-in `scenario-index.jsonld` files with Accord's `readScenarioIndexSource`, then calling `validateScenarioIndexDocument` with the scenario `conformance/` directory as `rootPath`.

## Contract Changes

- No CLI, runtime, ontology, generated RDF, generated HTML, or fixture manifest contract changes are intended.
- The internal fixture-ladder script structure may change, but `deno task fixture:ladder` behavior should remain stable.
- Existing dry-run output should remain stable unless a helper extraction exposes a clearly accidental formatting issue that is separately approved.
- Add generated Accord scenario-index files for the existing fixture scenarios, validating them against Accord's current `ScenarioIndex` model.
- Generate scenario-index files as `scenario-index.jsonld` next to the scenario transition manifests.
- Do not remove or collapse individual transition manifests; the index references them.

## Testing

- Run `deno test tests/scripts/fixture_ladder_test.ts` after each meaningful extraction.
- Run `deno task fmt:check`, `deno task lint`, and `deno task check` after the refactor.
- Run full `deno task test` if helper changes touch transition execution, branch update logic, or shared test support.
- Compare `deno run --allow-read --allow-write --allow-run=git,deno --allow-env scripts/fixture-ladder.ts --scenario alice-bio --dry-run` output before/after if dry-run rendering helpers are touched.
- Test rendered/written scenario indexes with Accord's `readScenarioIndexSource` and `validateScenarioIndexDocument`, passing the scenario `conformance/` directory as validation `rootPath`, without making the index the runtime source of truth yet.
- Compare rendered scenario-index documents against checked-in `scenario-index.jsonld` files so fixture topology drift is explicit.
- Do not regenerate fixture branches unless an explicit semantic fixture change is added later.

## Non-Goals

- Do not add Carol/import/custom ResourcePage fixture rungs in this task.
- Do not change generated fixture RDF or HTML.
- Do not redesign Accord manifests or fixture branch naming.
- Do not require live network access.
- Do not change `weave` command behavior.
- Do not make generated Accord scenario indexes the runtime source of truth in this pass.
- Do not change `accord check` to consume scenario indexes.
- Do not rename this task to completed unless explicitly requested.

## Implementation Plan

- [ ] Re-read [[wd.general-guidance]], [[wd.testing.fixture-ladder-regeneration]], this note, [[wa.task.2026.2026-05-21_0907-import]], and [[wa.task.2026.2026-05-23_2230-custom-resourcepage-shared-shell-fixture]] before editing.
- [ ] Inspect `scripts/fixture-ladder.ts` for repeated Alice Bio-specific helper shapes that are actually generic transition or asset patterns.
- [ ] Capture current focused fixture-ladder test behavior with `deno test tests/scripts/fixture_ladder_test.ts`.
- [ ] Extract the smallest useful helper set for fixture asset copy declarations and file-operation transitions.
- [ ] Extract or rename shared command/branch-publication helpers only where doing so reduces real duplication or clarifies topology boundaries.
- [ ] Add pure Accord scenario-index rendering helpers from generalized fixture scenario definitions.
- [ ] Render scenario-index `manifestPath` values as manifest filenames relative to the containing `conformance/` directory.
- [ ] Generate scenario index JSON-LD files for the existing Alice Bio, Sidecar Fantasy Rules, and Branch Fantasy Rules scenarios if the output validates cleanly.
- [ ] Add tests that read and validate the generated scenario indexes through Accord's scenario-index helpers.
- [ ] Keep scenario-specific constants and labels scenario-local.
- [ ] Run focused fixture-ladder tests after each extraction batch.
- [ ] Run formatting, lint, check, and broad tests according to touched code.
- [ ] Provide a commit message emphasizing behavior preservation and no fixture regeneration.
