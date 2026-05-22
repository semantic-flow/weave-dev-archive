---
id: tn9dqed12wnkywhong8nls9
title: 2026 05 22_1424 Core Weave Slice Classification Extraction
desc: ''
updated: 1779485117696
created: 1779485071599
---

## Goals

- Execute the next small slice of [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_1358-core-weave-rdf-and-turtle-helper-extraction]].
- Move pending weave slice detection and candidate slice classification out of `src/core/weave/weave.ts` into focused core weave modules.
- Preserve `src/core/weave/weave.ts` public compatibility, especially the existing `detectPendingWeaveSlice` export.
- Preserve planner behavior, generated RDF bytes, generated ResourcePage output, runtime candidate loading behavior, CLI behavior, and current error messages.
- Keep this slice narrow enough that payload-version layout and overwrite-state planning can remain separate follow-up work.

## Summary

After the RDF/Turtle helper extraction, `src/core/weave/weave.ts` is 8,026 lines. The next cohesive island is pending weave slice classification: the private `classifyWeaveSlice` helper and the exported `detectPendingWeaveSlice` function near the top of the file. These functions decide whether a candidate is ready for `firstKnopWeave`, `firstPayloadWeave`, `firstExtractedKnopWeave`, `firstReferenceCatalogWeave`, `pageDefinitionWeave`, or `secondPayloadWeave`.

This is more semantic than the RDF/Turtle helper slice. The code reads SFLO facts and applies local pending-state policy. It should move as an explicit classification module, not into generic RDF helpers or broad constants drawers.

## Discussion

Candidate modules:

- `src/core/weave/slice_classification.ts`: owns `detectPendingWeaveSlice`, `classifyWeaveSlice`, and small classification-only helpers such as `hasPayloadVersionNamingTarget`.
- Optional `src/core/weave/artifact_history_queries.ts`: if implementation needs to share history query helpers between classification and existing payload/overwrite/render code, move the minimal shared query helpers here instead of making payload code import from `slice_classification.ts`.

Likely classification dependencies:

- `NormalizedVersionTargetSpec` from `src/core/targeting.ts`.
- `toKnopPath` from `src/core/designator_segments.ts`.
- `WeaveableKnopCandidate` from `src/core/weave/candidates.ts`.
- `WeaveSlice` from `src/core/weave/slices.ts`.
- RDF helpers from `src/core/weave/rdf_helpers.ts`.
- `WeaveInputError` from `src/core/weave/errors.ts`.
- A narrow set of SFLO/SFCFG/RDF/XSD IRI constants used by classification predicates.

Prefer moving only the constants needed by classification into the new module as private constants. Do not introduce a broad shared constants module in this slice unless the import graph proves there is no cleaner option.

The classification code currently depends on helper behavior that is also useful elsewhere:

- `isDeclaredArtifactHistory` is used by classification, payload shape assertions, and overwrite planning.
- `hasNextStateSegmentHint` is classification-specific today.
- `requirePayloadCurrentStatePathFromInventory` is used by classification and payload overwrite/render helpers.
- `countArtifactHistoryPaths` is not classification-specific and should stay out of this slice unless a tiny shared history-query module makes that move obviously cleaner.

The clean dependency shape is `weave.ts` importing classification helpers, and classification helpers importing only leaf/core modules. Avoid any import from `slice_classification.ts` back to `weave.ts`, and avoid runtime imports from any core helper module.

## Open Issues

- [x] `detectPendingWeaveSlice` must remain available from `src/core/weave/weave.ts` because runtime and tests import it from that path today.
- [x] `classifyWeaveSlice` may become a named export from `slice_classification.ts` for internal use, but it should not be re-exported publicly from `weave.ts` unless an existing caller requires it.
- [x] Shared history query helpers should not force payload/overwrite code to import from `slice_classification.ts`. If they must move, use a narrowly named shared helper module.
- [x] Keep SFLO/SFCFG IRI constants narrow and local to the extracted module where possible. A broad constants drawer is still deferred.
- [x] Treat this as behavior-preserving. If classification exposes a semantic ambiguity, record it under "Orthogonal Opportunities" rather than changing policy in this slice.

## Decisions

- Create a focused classification module before extracting payload layout.
- Keep `WeaveSlice` in `src/core/weave/slices.ts`.
- Keep `src/core/weave/weave.ts` as the public façade for planner functions and current public type/function imports.
- Prefer direct internal imports from `slice_classification.ts` rather than barrel exports.
- Preserve exact slice names and decision order.
- Preserve current `WeaveInputError` messages unless a message must move unchanged with a helper.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts`, including `detectPendingWeaveSlice`, should remain compatible.

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

- Do not change slice classification behavior or decision priority.
- Do not move `planWeave`, `planVersion`, payload-version layout, overwrite-state planning, mesh/page-definition progression resolution, ResourcePage model builders, large Turtle renderers, or shape assertion families.
- Do not move generic RDF helpers again; they belong in `rdf_helpers.ts`.
- Do not introduce a broad SFLO constants module.
- Do not change generated RDF formatting, block ordering, ResourcePage output, target semantics, or error messages.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- None recorded yet.

## Deferred Follow-Up Ideas

- Payload version layout slice: move `PayloadVersionLayout`, first/second payload layout resolution, current-state overwrite helpers, and related naming-policy checks after classification is isolated.
- Artifact history query helper slice: if this task only moves the minimal helpers needed for classification, revisit whether remaining history query helpers should live together after payload layout extraction.
- Shape assertion slice: keep assertion wrappers out of classification, but consider extracting validation policy once the slice and payload helpers are out.
- Render helper slice: move larger Turtle renderers after classification and payload layout no longer depend on them.

## Implementation Plan

- [ ] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [ ] Record current line count and exported function/type names from `src/core/weave/weave.ts`; latest handoff count is 8,026 lines.
- [ ] Run a pre-slice import graph/circular-dependency audit rooted at `src/core/weave/weave.ts`.
- [ ] Identify the exact helper set to move first, especially whether a tiny shared history-query module is warranted.
- [ ] Move `detectPendingWeaveSlice` into `src/core/weave/slice_classification.ts`.
- [ ] Move `classifyWeaveSlice` into `src/core/weave/slice_classification.ts` for internal planner use.
- [ ] Keep `detectPendingWeaveSlice` available from `src/core/weave/weave.ts`.
- [ ] Preserve `weave.ts` planner functions and generated output logic.
- [ ] Run `deno task check` after the first classification module move.
- [ ] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, and focused core/integration tests.
- [ ] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this implementation slice.
- [ ] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [ ] Provide a commit message that clearly says this is a behavior-preserving slice-classification extraction.
