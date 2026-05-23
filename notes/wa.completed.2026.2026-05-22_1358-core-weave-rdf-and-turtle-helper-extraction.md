---
id: m1yk51lgpabbyjj0aaq7b8s
title: 2026 05 22_1358 Core Weave Rdf and Turtle Helper Extraction
desc: ''
updated: 1779517781926
created: 1779483532145
---

## Goals

- Execute the next small slice of [[wa.completed.2026.2026-05-22_1422-core-weave-rdf-and-turtle-helper-extraction]] after [[wa.completed.2026.2026-05-21_1037-core-weave-first-extraction-slice]].
- Move pure RDF parsing/query/resolution helpers and pure Turtle block-editing helpers out of `src/core/weave/weave.ts`.
- Preserve planner behavior, public TypeScript compatibility from `src/core/weave/weave.ts`, generated RDF bytes, generated ResourcePage output, and CLI/runtime behavior.
- Establish clean leaf helper modules that make later slice-classification and payload-version extraction easier.

## Summary

At the start of this task, `src/core/weave/weave.ts` was still 8,515 lines after the first type/model extraction. The next safest reduction was to move low-level helpers that already behaved like leaf utilities: RDF parsing and fact resolution, plus Turtle block splitting/replacement/upsert helpers. These helpers are called by many planner sections but should not own planner state or operation semantics.

This task is a move-only refactor. It should not move `detectPendingWeaveSlice`, shape assertion families, payload-version layout helpers, source-locator rendering helpers, ResourcePage model builders, or large Turtle renderers. Those can move later once the shared query and block-editing utilities are no longer buried in the façade.

## Discussion

Candidate modules:

- `src/core/weave/rdf_helpers.ts`: parser and query helpers such as `parseWeaveShapeQuads`, `parseTurtleQuads`, `hasNamedNodeFact`, `hasLiteralFact`, `hasSubject`, `hasPredicateFact`, `hasSubjectPredicateFact`, `hasTermKeyNamedNodeFact`, `resolveUniqueLiteralValuesForTermKey`, `matchesRdfTermKey`, `toRdfTermKey`, `requireSingleNamedNodeObject`, `requireOptionalNamedNodeObject`, `requireSingleNonNegativeIntegerLiteral`, `resolveOptionalNonNegativeIntegerLiteral`, `resolveOptionalLiteralObject`, `resolveNamedNodeObjectPaths`, `resolveOptionalNamedNodePath`, `resolveOptionalSegmentHint`, `requireLiteralValue`, `requireNamedNodePath`, `toMeshRelativePath`, and `toAbsoluteIri`.
- `src/core/weave/turtle_blocks.ts`: block-string helpers such as `splitTurtleBlocks`, `normalizeMeshInventoryHeader`, `replaceSubjectBlock`, `upsertSubjectBlockAfter`, `findSubjectBlockIndex`, `getSubjectPathFromBlock`, `collectSubjectSubtreeBlocks`, `appendPredicateToSubjectBlock`, and `renderSubjectPredicateBlock`.

Prefer the two focused modules above over one broad helper module. The split should leave RDF graph inspection and string-block editing as separate responsibilities.

Expected dependency shape:

- `rdf_helpers.ts` may import `Parser` / `Quad` from `n3`, `SAFE_DESIGNATOR_SEGMENT_PATTERN` from `../designator_segments.ts` if `resolveOptionalSegmentHint` moves, and `WeaveInputError` from `./errors.ts`.
- `rdf_helpers.ts` may define its own RDF/XSD constants needed by generic literal helpers, but should not become a large Semantic Flow IRI constant module.
- `turtle_blocks.ts` should have no RDF parser dependency. It may import `WeaveInputError` only if a moved helper already throws planner-compatible input errors.
- `weave.ts` should import moved helpers directly from these modules.
- No helper module should import from `src/runtime/**`.
- `../designator_segments.ts` is already in `src/core/weave/weave.ts`'s direct import graph today, so moving `resolveOptionalSegmentHint` into `rdf_helpers.ts` should not add a new dependency to the `weave.ts` root graph. It would, however, make every future direct importer of `rdf_helpers.ts` load `designator_segments.ts` at module level. That is acceptable for this internal core-weave helper slice because the helper is designator-aware, but if a later caller needs generic RDF helpers without designator semantics, split `resolveOptionalSegmentHint` out or leave it near slice/payload naming code.

The first pass should move the helpers with the least planner-specific coupling. If a helper drags in many SFLO-specific constants or operation-specific error semantics, leave it in `weave.ts` and record the reason rather than widening the slice.

## Open Issue Resolutions

- Move `renderSubjectPredicateBlock` with `turtle_blocks.ts`. It is a leaf string renderer, has no model dependency, and moving it lets later render helpers compose through the same small Turtle support module.
- Move `resolveOptionalSegmentHint` with `rdf_helpers.ts` only after confirming the import graph stays clean. Its dependency on `SAFE_DESIGNATOR_SEGMENT_PATTERN` is still a portable designator rule and `../designator_segments.ts` is already part of `weave.ts`'s direct imports, so this does not pull planner state into the helper module. Record it and reconsider if `rdf_helpers.ts` starts being used as a broadly generic RDF utility.
- Do not move `normalizeWorkingLocalRelativePathLiteral`, `usesMeshLocalWorkingLocatedFile`, `renderCurrentWorkingFileLocator`, `renderCurrentWorkingFileDeclaration`, or `renderRepositorySourceFloatingLocatorBlankNode` in this slice. They belong to a later source/artifact locator helper slice because they encode current-source locator behavior rather than generic RDF or block editing.
- Do not move SFLO-specific semantic convenience helpers such as `isDeclaredArtifactHistory`, `hasNextStateSegmentHint`, or `countArtifactHistoryPaths` in this slice. They should stay near slice/planner semantics until a later semantic-helper or slice-classification extraction gives them a clearer home. This avoids creating a broad SFLO constants/helper drawer.
- Do not move assertion wrappers such as `assertHasNamedNodeFacts`, `assertHasLiteralFacts`, `assertHasCurrentPayloadSourceLocator`, or `assertHasCurrentWorkingFileLocator` in this slice. They are shape/assertion policy helpers, not generic query primitives.

## Decisions

- Keep `src/core/weave/weave.ts` as the public façade for planner functions and existing public type imports.
- Do not re-export these internal helper modules from `src/core/weave/weave.ts` unless an existing public import requires it.
- Prefer direct internal imports from `rdf_helpers.ts` and `turtle_blocks.ts`.
- Keep planner/stateful functions in `weave.ts` during this slice, including `detectPendingWeaveSlice`, `classifyWeaveSlice`, shape assertions, progression resolution, payload-version layout, and renderers.
- Do not introduce a generic constants module unless the move is otherwise impossible; a constants drawer would just recreate the current problem in smaller form.
- Preserve current error messages. When helpers accept `errorMessage` or `label`, keep those call contracts stable.
- Leave source-locator and SFLO-semantic convenience helpers in `weave.ts` unless the implementation exposes a concrete cycle or duplication risk; record that risk under "Orthogonal Opportunities" rather than broadening the slice.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.

## Testing

- Before editing, record current line count and exported surface from `src/core/weave/weave.ts`.
- Before editing, run an import graph/cycle audit rooted at `src/core/weave/weave.ts`.
- Before editing, run:
  - `deno task check`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
- After extraction, run:
  - `deno task fmt`
  - `deno task lint`
  - `deno task check`
  - another import graph/cycle audit rooted at `src/core/weave/weave.ts`
  - confirm there are no new `src/core/` -> `src/runtime/` import edges
  - confirm there are no local cycles among `src/` modules
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/weave_test.ts`
  - `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`

## Non-Goals

- Do not move `planWeave`, `planVersion`, `detectPendingWeaveSlice`, `classifyWeaveSlice`, or slice planner functions.
- Do not move shape assertion families unless a tiny leaf assertion helper must move with the RDF helper it wraps.
- Do not move payload-version layout, overwrite planning, mesh/page-definition progression resolution, or support-history behavior.
- Do not move large Turtle renderers or ResourcePage model builders.
- Do not change generated RDF formatting, block ordering, ResourcePage output, target semantics, or error messages.
- Do not introduce behavior cleanup, fixture regeneration, or semantic redesign.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction; keep this slice narrow and create/update follow-up tasks when needed.

- No performance optimization opportunities, behavior bugs, or suspicious planner behavior were found during this move-only slice.

## Implementation Results

- Baseline `src/core/weave/weave.ts` size was 8,515 lines.
- Baseline import graph rooted at `src/core/weave/weave.ts`: 18 local `src/` modules, 35 local edges, 16 direct local imports from the root, 0 `src/core/` -> `src/runtime/` edges, and 0 local cycles.
- Added `src/core/weave/rdf_helpers.ts` for low-level RDF parsing, fact matching, literal/path resolution, RDF term-key conversion, mesh-relative path conversion, and optional segment-hint resolution.
- Added `src/core/weave/turtle_blocks.ts` for Turtle subject-block splitting, lookup, replacement, upsert, subtree collection, predicate append, and small predicate-block rendering.
- Left source-locator rendering helpers, SFLO semantic convenience helpers, shape assertion wrappers, slice classification, payload layout, and large renderers in `weave.ts` as planned.
- Post-slice `src/core/weave/weave.ts` size is 8,026 lines.
- Post-slice import graph rooted at `src/core/weave/weave.ts`: 20 local `src/` modules, 40 local edges, 18 direct local imports from the root, 0 `src/core/` -> `src/runtime/` edges, and 0 local cycles. The added direct root imports are the intentional `rdf_helpers.ts` and `turtle_blocks.ts` leaf modules.
- `rdf_helpers.ts` imports `../designator_segments.ts` because `resolveOptionalSegmentHint` moved with the RDF literal resolver family. This preserves the existing `weave.ts` root dependency shape but means future direct importers of `rdf_helpers.ts` inherit the designator-segment dependency unless that helper is split later.
- Verification passed with `deno task fmt`, `deno task lint`, `deno task check`, the core weave test, the focused validate/version/generate integration test, the focused weave integration test, and the post-slice import graph/cycle audit.

## Deferred Follow-Up Ideas

These are intentionally not part of this slice, but should not be lost:

- Source/artifact locator helper slice: move `normalizeWorkingLocalRelativePathLiteral`, `usesMeshLocalWorkingLocatedFile`, `renderCurrentWorkingFileLocator`, `renderCurrentWorkingFileDeclaration`, `renderRepositorySourceFloatingLocatorBlankNode`, and the repository-source locator assertion helpers into a focused module once the RDF helpers are no longer embedded in `weave.ts`.
- Semantic RDF convenience helper slice: after the generic RDF helpers move, consider a small module for SFLO-specific query shorthands such as `isDeclaredArtifactHistory`, `hasNextStateSegmentHint`, and history-count helpers if slice classification or payload layout extraction needs them.
- Shape assertion slice: keep assertion wrappers out of `rdf_helpers.ts`, but consider extracting shape assertion families after the pure query helpers move so validation policy is separate from low-level graph access.
- Slice-classification slice: once RDF helpers are in place, move `detectPendingWeaveSlice`, `classifyWeaveSlice`, payload naming policy checks, and related pending-slice helpers into a slice module.
- Render helper slice: move larger Turtle renderers last, after dependency direction is clear. `renderSubjectPredicateBlock` is the one intentionally small exception for this task.
- Potential future constants cleanup: avoid a constants drawer now, but if later modules duplicate the same SFLO IRI constants repeatedly, create a narrowly named constants module for a specific domain rather than a broad global bucket.
- Possible generic RDF split: if `rdf_helpers.ts` becomes useful outside core-weave-specific planning, separate truly generic parser/query helpers from designator-aware helpers such as `resolveOptionalSegmentHint` so generic callers do not inherit designator path dependencies unnecessarily.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.completed.2026.2026-05-22_1422-core-weave-rdf-and-turtle-helper-extraction]] before editing.
- [x] Record current line count and exported function/type names from `src/core/weave/weave.ts`; latest handoff count is 8,515 lines.
- [x] Run a pre-slice import graph/circular-dependency audit rooted at `src/core/weave/weave.ts`.
- [x] Identify the exact helper set to move first and leave any high-coupling helpers in `weave.ts` with a note.
- [x] Move pure RDF parser/query/resolution helpers into `src/core/weave/rdf_helpers.ts`.
- [x] Move pure Turtle block-editing helpers into `src/core/weave/turtle_blocks.ts`.
- [x] Update `weave.ts` imports without changing planner functions or generated output logic.
- [x] Prefer narrow named exports from helper modules; avoid broad barrel exports.
- [x] Run `deno task check` after the first helper module move.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, and focused core/integration tests.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this implementation slice.
- [x] Update [[wa.completed.2026.2026-05-22_1422-core-weave-rdf-and-turtle-helper-extraction]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving RDF/Turtle helper extraction.

