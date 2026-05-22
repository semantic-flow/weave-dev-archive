---
id: yg6v6jgfp191rku67wue5zl
title: 2026 05 22_1441 Payload Version Layout and Overwrite State Planning
desc: ''
updated: 1779486147457
created: 1779486147457
---

## Goals

- Execute the next small slice of [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] after [[wa.completed.2026.2026-05-22_1424-core-weave-slice-classification-extraction]].
- Move payload version layout resolution and guarded overwrite-state planning out of `src/core/weave/weave.ts` into focused core weave modules.
- Preserve planner behavior, generated RDF bytes, generated ResourcePage output, runtime behavior, CLI behavior, and current error messages.
- Keep `planWeave` and `planVersion` in `weave.ts` while letting them call extracted payload-version helpers.
- Avoid moving large Turtle renderers or ResourcePage builders in this slice.

## Summary

After the slice-classification extraction, `src/core/weave/weave.ts` is 7,703 lines. The next cohesive area is payload version intent and layout: resolving which payload history/state/manifestation paths should be used for first and later payload weaves, plus guarded overwrite-state planning when `overwriteExistingState` is requested.

This slice is more behavior-sensitive than the prior classification move because it sits directly on version target fields: `historySegment`, `stateSegment`, `manifestationSegment`, naming policies, next-state hints, and overwrite validation. The implementation should remain move-only and should preserve every error message, generated path, and fail-closed guard.

## Discussion

Candidate modules:

- `src/core/weave/payload_version_layout.ts`: owns `PayloadVersionLayout`, `resolveFirstPayloadVersionLayout`, `resolveSecondPayloadVersionLayout`, `requirePayloadHistoryPath`, `resolveCurrentPayloadManifestationPathFromInventory`, `resolveNextOrdinalStatePathFromHistory`, and the payload-specific path/naming guards needed by layout resolution.
- `src/core/weave/payload_overwrite.ts`: owns `assertOverwriteExistingStateTargets` and `planOverwriteExistingPayloadState` if keeping overwrite planning separate from layout keeps dependencies clearer.
- Optional `src/core/weave/artifact_manifestation_paths.ts`: only if moving layout exposes a clean shared home for `toArtifactManifestationPath`, `defaultManifestationSegment`, and `toManifestationSegment`, which are currently also used by reference-catalog planning/rendering. Do not introduce this module unless it reduces coupling; duplication or a narrow private helper may be safer for this slice.

Likely helper set to consider:

- `PayloadVersionLayout`
- `resolveFirstPayloadVersionLayout`
- `resolveSecondPayloadVersionLayout`
- `toPayloadManifestationPath`
- `resolveCurrentPayloadManifestationPathFromInventory`
- `resolveNextOrdinalStatePathFromHistory`
- `requirePayloadHistoryPath`
- `requirePayloadCurrentStatePath`, if the same history/state invariant naturally belongs with payload layout and does not drag unrelated rendering code
- `assertRequestedStateSegmentSatisfiesPolicy`
- `assertAutoStateSegmentSupported`
- `defaultHistorySegment`
- `defaultStateSegment`
- `assertOverwriteExistingStateTargets`
- `planOverwriteExistingPayloadState`

Helpers to be careful with:

- `toArtifactManifestationPath`, `defaultManifestationSegment`, and `toManifestationSegment` are shared by payload and reference-catalog code today. Moving them may be useful, but it is not required unless payload layout extraction would otherwise duplicate too much logic.
- `toFileName` and `toParentPath` are broad path string helpers used throughout `weave.ts`. Do not move them just for this slice unless the extracted module needs one and the import direction stays clean.
- `assertPayloadNamingSupportedForSlice` is a target guard close to payload versioning, but it also sits in the `planWeave` dispatch path. Move it only if the extracted module can own the guard without needing planner orchestration state.

Expected dependency shape:

- Payload layout helpers may import `NormalizedVersionTargetSpec` from `src/core/targeting.ts`, naming policy types from `src/core/weave/naming_policy.ts`, `PayloadWorkingArtifact` from `src/core/weave/candidates.ts`, RDF helpers from `src/core/weave/rdf_helpers.ts`, artifact history helpers from `src/core/weave/artifact_history_queries.ts`, and `WeaveInputError` from `src/core/weave/errors.ts`.
- Overwrite planning may import `WeavePlan` from `src/core/weave/planning_models.ts`, `WeaveableKnopCandidate` from `src/core/weave/candidates.ts`, and payload layout/path helpers.
- New modules must not import from `src/runtime/**`.
- New modules must not import from `src/core/weave/weave.ts`.
- `weave.ts` should import extracted helpers directly and keep the public façade stable.

The safest split is probably `payload_version_layout.ts` plus `payload_overwrite.ts`. If the implementation shows that overwrite planning is tiny and only depends on the layout module, keeping both in one `payload_versioning.ts`-style module is acceptable, but avoid a broad bucket that also absorbs renderers and ResourcePage builders.

## Open Issues

- [x] Preserve all existing target-field and overwrite error messages exactly unless a test reveals they already differ from current behavior.
- [x] Do not move first/second payload Turtle renderers in this slice; they consume `PayloadVersionLayout` but are not layout resolution.
- [x] Do not move payload ResourcePage model builders in this slice; they consume `PayloadVersionLayout` but belong to a later ResourcePage/render helper pass.
- [x] Keep shared manifestation path helpers local to `weave.ts` unless a clean narrow shared module is clearly better than duplicated private helper code.
- [x] `assertPayloadNamingSupportedForSlice` is optional for this slice; move it only if it stays leaf-like and does not pull planner dispatch state into payload modules.
- [x] Record any confusing payload-version policy or suspicious overwrite behavior under "Orthogonal Opportunities" instead of changing it here.

## Decisions

- Treat this as a behavior-preserving extraction, not a payload versioning redesign.
- Keep `planWeave` and `planVersion` in `src/core/weave/weave.ts`.
- Keep `WeavePlan` result shape unchanged.
- Keep `PayloadVersionLayout` as an internal core weave type unless an existing public import requires a façade re-export.
- Prefer narrow named exports from any new payload modules; do not create a barrel.
- Keep Semantic Flow path conventions unchanged: payload histories under the designator path, historical states below their history, and manifestations below the state.
- Keep naming policy enforcement fail-closed for named, semver, and date policies.

## Contract Changes

- No ontology, CLI, runtime, generated RDF, generated ResourcePage, or public behavior change is intended.
- Internal module layout under `src/core/weave/` may change.
- Existing public imports from `src/core/weave/weave.ts` should remain compatible.
- The `overwriteExistingState` request behavior must remain unchanged: exact targets only, explicit history/state required, and only the current historical state may be overwritten.

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

- Do not change payload versioning semantics, naming policy behavior, overwrite safety rules, generated Turtle, generated ResourcePage output, or target normalization.
- Do not move `planWeave`, `planVersion`, slice classification, candidate selection, large Turtle renderers, ResourcePage model builders, mesh inventory progression, or shape assertion families.
- Do not change how `historySegment`, `stateSegment`, `manifestationSegment`, or next-state segment hints are interpreted.
- Do not introduce fixture regeneration or semantic cleanup.
- Do not introduce a broad constants module.
- Do not rename this task to completed as part of implementation unless explicitly requested.

## Orthogonal Opportunities

Record performance optimization opportunities, bugs, and suspicious behavior found while implementing this task here. Do not fix them in this slice unless they block the behavior-preserving extraction.

- No performance optimization opportunities, behavior bugs, or suspicious planner behavior were found during this move-only slice.

## Implementation Results

- Baseline `src/core/weave/weave.ts` size was 7,703 lines.
- Baseline public surface from `src/core/weave/weave.ts` included `planWeave`, `planVersion`, `detectPendingWeaveSlice`, type re-exports, `WeaveInputError`, and `planMeshSupportResourcePages`.
- Baseline import graph rooted at `src/core/weave/weave.ts`: 22 local `src/` modules, 67 local edges, 20 direct local imports from the root, 0 `src/core/` -> `src/runtime/` edges, and 0 local cycles.
- Added `src/core/weave/payload_version_layout.ts` for `PayloadVersionLayout`, first/second payload version layout resolution, payload current/history guards, payload manifestation lookup, and payload state naming-policy checks.
- Added `src/core/weave/payload_overwrite.ts` for `overwriteExistingState` target validation and guarded current payload-state overwrite planning.
- Added `src/core/weave/artifact_manifestation_paths.ts` for the shared state-to-manifestation path helper used by payload and reference-catalog planning. This kept the shared helper narrow without moving large renderers or broad path utilities.
- Kept `planWeave`, `planVersion`, first/second payload Turtle renderers, ResourcePage builders, shape assertions, and mesh inventory progression in `weave.ts` as planned.
- Kept general helpers such as `toFileName`, `toStateSegment`, and `toLastPathSegment` in `weave.ts`; the extracted modules use private copies only where needed to avoid widening this slice into a general path-helper refactor.
- Post-slice `src/core/weave/weave.ts` size is 7,135 lines.
- Post-slice import graph rooted at `src/core/weave/weave.ts`: 25 local `src/` modules, 92 local edges, 23 direct local imports from the root, 0 `src/core/` -> `src/runtime/` edges, and 0 local cycles. The added direct root imports are the intentional `artifact_manifestation_paths.ts`, `payload_version_layout.ts`, and `payload_overwrite.ts` helper modules.
- Verification passed with `deno task fmt`, `deno task lint`, `deno task check`, the core weave test, the focused validate/version/generate integration test, the focused weave integration test, and the post-slice import graph/cycle audit.

## Deferred Follow-Up Ideas

- Payload render helper slice: after layout is separate, move first/second payload Turtle renderers only if their dependency direction is clear.
- Payload ResourcePage builder slice: move payload page model builders after layout and render helpers are disentangled.
- Manifestation path helper cleanup: if reference-catalog and payload code keep sharing the same path helpers, consider a narrowly named shared module after this slice.
- Shape assertion slice: payload shape validation may become easier to extract after payload layout and overwrite planning are isolated.
- Legacy fixture-ladder renderer cleanup remains deferred until move-only core planner decomposition is complete.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[ont.summary.core]], and [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] before editing.
- [x] Record current line count and exported function/type names from `src/core/weave/weave.ts`; latest handoff count is 7,703 lines.
- [x] Run a pre-slice import graph/circular-dependency audit rooted at `src/core/weave/weave.ts`.
- [x] Identify the exact helper set to move first, especially whether overwrite planning should live with or beside payload layout.
- [x] Move `PayloadVersionLayout` and first/second payload layout resolution into a focused module.
- [x] Move overwrite-state target validation and planning into a focused module if dependency direction stays clean.
- [x] Keep large payload renderers and ResourcePage builders in `weave.ts`.
- [x] Preserve `weave.ts` planner functions and generated output logic.
- [x] Run `deno task check` after the first payload module move.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, post-slice graph audit, and focused core/integration tests.
- [x] Record any discovered bugs or performance opportunities under "Orthogonal Opportunities" instead of widening this implementation slice.
- [x] Update [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and [[wd.codebase-overview]] with the resulting module layout.
- [x] Provide a commit message that clearly says this is a behavior-preserving payload version layout and overwrite planning extraction.

## Suggested Commit Message

```text
refactor(core-weave): extract payload version layout planning

- move payload history/state/manifestation layout resolution into payload_version_layout.ts
- move guarded overwriteExistingState validation and planning into payload_overwrite.ts
- move shared artifact manifestation path construction into artifact_manifestation_paths.ts
- keep payload renderers, ResourcePage builders, shape assertions, and planner façade behavior in weave.ts
- verify with fmt, lint, check, core weave tests, integration weave tests, and import-cycle audit
```
