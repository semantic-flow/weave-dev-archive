---
id: 46677d2797bf294d8f43f9c7
title: 2026 05 21_0820 CI Idempotency Tests
desc: ''
updated: 1779361200000
created: 1779361200000
---

## Goals

- Add focused tests for the CI/release rerun contract that was split out of [[wa.completed.2026.2026-05-18_0627-remove-prepare]].
- Prove that rerunning generation and validation over unchanged source/config/designators is clean and explainable.
- Prove that rerunning release/version operations with the same explicit state target fails closed or reports an intentional no-op instead of minting duplicate states.
- Keep this out of generic dirty-source heuristics for default `weave`.

## Summary

`prepare gh-pages` has been removed and its useful behaviors have been decomposed into ordinary `mesh create`, `integrate`, `weave`, `version`, `generate`, and `validate` operations. Before those operations become the basis for automated SFLO release workflows, Weave needs direct tests for rerun safety.

This task is about executable idempotency and fail-closed behavior, not about changing the semantic model. The expected release automation contract is: if source bytes, config, target designators, and explicit version targets are unchanged, reruns should either produce byte-identical generated output, update nothing, or fail closed with a deliberate duplicate-state/no-op signal.

## Discussion

Existing tests already cover pieces of the story:

- `prepare gh-pages` is no longer accepted.
- `executeGenerate` does not mutate RDF artifacts.
- scoped `validate mesh` and `validate publication` have integration coverage.
- `extract --all-terms` reruns can be no-op for already extracted terms.
- several creation paths fail closed when their target files already exist.

The missing coverage is a release-shaped rerun suite that connects those pieces:

- `generate` rerun over the same mesh should update nothing when generated timestamps are pinned.
- `validate mesh` rerun should return the same clean findings and not write files.
- `version` or top-level `weave` rerun with the same explicit release state should not mint a duplicate state.
- CLI-shaped coverage should use `WEAVE_GENERATED_AT` where byte-identical HTML matters.

## Open Issues

- Should duplicate explicit release-state reruns become a deliberate no-op result, or continue to fail closed with an existing-state error?

## Decisions

- Make `generate` avoid rewriting generated pages when the only content difference is the generated timestamp footer, and report that as an info notification so CI reruns are explainable without requiring pinned `WEAVE_GENERATED_AT`.
- Treat duplicate explicit state creation as fail-closed by default. Add an explicit overwrite flag for deliberate replacement of the current payload state named by an explicit history and state target, behind tight validation: broad recursive, inferred-state, or older-state retargeting overwrites must stay rejected.
- Keep dirty worktree detection out of scope. CI should ask for explicit operations and compare their results, not infer semantic intent from repository dirtiness.

## Contract Changes

- `generate` should leave existing generated pages untouched when the planned rewrite differs only in the generated timestamp footer. The generated result should expose skipped timestamp-only paths, and the CLI should print an info line when that skip occurs.
- `version` and top-level `weave` should accept an explicit overwrite-state option for release automation repair cases, while continuing to fail closed for duplicate explicit states unless that option is present. The overwrite option is current-state-only in this pass.
- `WEAVE_GENERATED_AT` remains useful for deterministic fixture construction, but idempotent generate reruns should no longer depend on it.

## Testing

- Add an integration test that runs `executeGenerate` twice with different `now` values; the second run should report no `createdPaths` and no `updatedPaths`, report timestamp-only skipped paths, and leave representative generated HTML byte-identical.
- Add an integration test that runs `executeValidate({ scope: "mesh" })` twice over the same settled fixture; both runs should produce no findings and leave representative files unchanged.
- Add an integration test that versions a payload to an explicit release state, then repeats the same `executeVersion` request; the second request should fail closed before writing, and the history/inventory files should remain unchanged.
- Add an integration test that repeats an explicit release-state request with the overwrite option enabled; it should rewrite only the targeted current-state snapshot, keep inventory pointers stable, and reject missing explicit state/history inputs.
- Add a black-box CLI test for the generate rerun path that verifies the timestamp-only skip notification, because CI release workflows exercise CLI behavior.

## Non-Goals

- Do not add a generic dirty-source heuristic to default `weave`.
- Do not introduce a persistent cache, file watcher, or repository status dependency.
- Do not implement source fetch/import/update behavior.
- Do not redesign generated-page timestamp policy unless idempotency evidence shows it is necessary.

## Implementation Plan

- [x] Add focused runtime integration coverage for generate reruns with timestamp-only skips.
- [x] Add focused runtime integration coverage for validation reruns.
- [x] Add focused runtime integration coverage for explicit release-state duplicate reruns.
- [x] Add guarded overwrite support for explicit release-state reruns.
- [x] Add a CLI-shaped generate rerun test for the timestamp-only skip notification.
- [c] Update `remove-prepare` only if this task changes the remaining completion criteria.
