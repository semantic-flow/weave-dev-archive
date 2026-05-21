---
id: x3ec2py2xjy1nl2zp0qczxo
title: 2026 05 19_0022 Lateststate Improvement
desc: ''
updated: 1779175354969
created: 1779175354969
---

## Goals

- Define the remaining `artifactResolutionMode_latestState` runtime/API work as a separate follow-up from [[wa.completed.2026.2026-05-18_0627-remove-prepare]].
- Make latest-state resolution useful for source bindings and page/source resolution when callers want settled bytes but do not know the exact `HistoricalState` IRI.
- Keep mutable working-source resolution and settled-state resolution sharply separated.
- Confirm that this work is not required before fixture-ladder regeneration unless a regenerated ladder is meant to demonstrate `latestState` specifically.
- Preserve the rule that exact target coordinates, such as requested state, manifestation, commit, or digest, fix identity without requiring an additional pinned mode.

## Summary

`artifactResolutionMode_latestState` is the settled-byte counterpart to `artifactResolutionMode_working`.

`working` means "follow mutable current bytes under operational policy." It is right for source checkouts, drafts, and branch-published ontology source files that are already present locally and are made reproducible with repository commit/digest evidence.

`latestState` means "resolve to the latest settled `HistoricalState` for the target artifact." It is right when the consumer wants stable mesh state, not whatever happens to be in the working file today.

Examples that show why this matters:

- Public page regeneration with local drafts: Alice's `alice-bio.ttl` working file may contain draft edits that are not ready to publish. A page definition or source binding using `working` would read the draft. A binding using `latestState` should read the newest settled state for `alice/bio`, so ResourcePages can be regenerated safely from the last woven artifact even while local work continues.
- Cross-artifact term pages: an extracted term page can say "my source is the latest settled `ontology` payload" without hard-coding `ontology/releases/v0.1.0` or `_history001/_s0007`. When `ontology` is later versioned, regenerating pages can intentionally follow the newest settled state rather than the editable ontology file.
- Reusable dependency meshes: a documentation mesh may depend on the latest settled state of a separate vocabulary mesh. It should not need a host-local checkout grant, and it should not read a maintainer's uncommitted vocabulary edits.
- CI release checks after source validation: a release workflow may validate source working files first, run `weave version`, and then run generation or extraction steps over the latest settled state. That lets "validate draft/source" and "publish settled state" remain separate phases.

For the current branch-published SFLO work, the implemented repository-backed working-source path is enough: the source checkout is explicit, and commit/digest evidence makes the observed bytes reproducible. `latestState` should therefore be a separate improvement task, not a blocker for the planned fixture-ladder regeneration.

## Discussion

### Current State

The vocabulary already has the important terms:

- `sflo:artifactResolutionMode_working` for mutable working/source bytes.
- `sflo:artifactResolutionMode_latestState` for settled state resolution.
- exact coordinates such as `hasRequestedTargetState`, target manifestation, commit, or digest imply exact identity without `artifactResolutionMode_pinned`.
- a requested `ArtifactHistory` is intended to bound resolution to the latest state in that history.

Weave also already has pieces of latest-state behavior in related areas. It understands `latestHistoricalState` for payload/version progression, ResourcePage history display, exact extraction-source evidence, and generated page rendering. What is missing is the general `ArtifactResolutionTarget` resolver path that can take a source binding or page-source target and turn "latest state" into a concrete observed state, manifestation, located file, digest, and rendered/readable bytes.

There is already at least one fail-closed behavior around this boundary: artifact-backed page source resolution rejects `latestState` today. That is good as a guardrail; this task is about replacing that rejection with precise resolution behavior where it is safe.

### Semantics To Preserve

`latestState` is not "current working file." It must not read `workingLocalRelativePath` merely because a working file exists. It should resolve through artifact history facts.

The basic policy should be:

- If `hasRequestedTargetState` is present, resolution is exact state resolution and `latestState` is unnecessary or a warning.
- If `hasRequestedTargetHistory` is present, resolve the latest state in that history.
- If no history is requested, resolve the latest settled state according to a deterministic policy.
- The first deterministic policy should prefer the artifact's `currentArtifactHistory` when present and typed, because that is the artifact's selected current/default history.
- If there is no usable current/default history, later policy may choose latest across all known histories, but only if recency can be determined unambiguously.

"Latest across histories" is useful, but it is also the easiest place to smuggle in surprising behavior. The implementation should start narrow and fail closed when there is no unambiguous latest settled state.

### What Gets Recorded

Resolving `latestState` should leave evidence, not just read bytes.

When resolution succeeds, the consumer should be able to record or expose:

- the requested target artifact.
- the requested history if one bounded the search.
- the observed source state.
- the observed manifestation or located file when bytes are read.
- the observed digest when bytes are available locally.
- `observedAt` as the time resolution observed those bytes.

This mirrors the existing distinction: `observedAt` is evidence for resolution, not an import/copy timestamp.

### Relationship To Exact Targets

Exact targets do not need `latestState`.

If a binding names a `HistoricalState`, manifestation, commit, or digest, it already says "use exactly this thing." A resolver may still normalize and record observed located-file evidence, but it should not reinterpret the target as "latest."

### Relationship To Ladder Regeneration

This task should not block the near-term branch/sidecar fixture-ladder regeneration.

The regenerated branch/SFLO ladders can use repository-backed working-source integration with release refs, commits, and digests. That path proves the important remove-prepare symmetry: `integrate` leaves source bytes where they are and branch publication is not a special operation.

Add a `latestState` ladder only when we want to demonstrate this specific behavior, such as a page source or extraction source that intentionally follows the latest settled payload state.

## Open Issues

- Should no-history `latestState` resolve only through `currentArtifactHistory` at first, or should it attempt latest-across-all-histories when ordinals or dates make that unambiguous?
- Should `latestState` on a repository-backed target ever inspect repository refs directly, or should repository-backed latest behavior always require exact commit/digest or a mesh `ArtifactHistory` target?
- Which operation should first exercise `latestState`: page-source generation, extraction-source resolution, source registry integration, or a narrower resolver test harness?
- Should `latestState` resolution write observed evidence back into the source registry automatically, or should it only expose evidence to the operation plan/result and let each operation decide what to persist?
- What is the correct SHACL severity for `artifactResolutionMode_latestState` combined with exact coordinates: warning because it is redundant, or violation because it is misleading?

## Decisions

- Treat this as a separate follow-up task, not as remaining work inside [[wa.completed.2026.2026-05-18_0627-remove-prepare]].
- Do not block fixture-ladder regeneration on `latestState` unless the ladder's purpose is to demonstrate latest-state resolution.
- Keep `artifactResolutionMode_latestState` in the ontology; the improvement is runtime/API behavior, not new vocabulary naming.
- Implement the first resolver fail-closed rather than guessing when no unambiguous latest settled state is available.
- Prefer `currentArtifactHistory` as the initial no-history default, because it is explicit current/progression metadata for the artifact.
- Do not reintroduce `pinned`; exact target coordinates remain exact by default.

## Contract Changes

- Clarify [[sf.spec.2026-04-04-integrate-behavior]] and [[sf.spec.2026-05-18-publication-source-binding]] when a source binding may request `latestState` versus `working`.
- Clarify [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]] for page-source resolution from latest settled state.
- Update [[sf.glossary]] with the fail-closed no-history policy once chosen.
- Update [[sf.api]] only as an overview/link change if the behavior spec moves; avoid restating the resolver contract there.
- Update SHACL if necessary so redundant or contradictory `latestState` combinations have the intended severity.

## Testing

- Add resolver tests for `ArtifactResolutionTarget`:
  - requested history resolves to that history's `latestHistoricalState`.
  - no requested history resolves through `currentArtifactHistory` when present.
  - exact requested state ignores or warns on redundant `latestState`.
  - no unambiguous latest state fails closed.
  - missing located-file/manifestation evidence fails with a useful error when bytes are required.
- Add operation-level tests for the first consumer:
  - page-source generation can render from latest settled payload state while the working file contains different draft bytes.
  - source/extraction evidence records the observed historical state and optional digest.
  - working mode still reads the working file under operational policy and does not silently fall back to latest state.
- Add SHACL tests for severity distinctions around `latestState`.
- Add one fixture-ladder transition only if this task intentionally becomes part of a conformance story; otherwise keep coverage at unit/integration level.

## Non-Goals

- Do not block remove-prepare completion or branch-published ladder regeneration.
- Do not fetch remote repositories or inspect live refs as part of `latestState`.
- Do not implement broad source update/refresh.
- Do not copy bytes into the mesh; that remains `import`.
- Do not revive `artifactResolutionMode_current` or `artifactResolutionMode_pinned`.
- Do not make latest-across-all-histories guess from filesystem mtimes or lexical path ordering.

## Conformance Fixture Decision

Dedicated `latestState` conformance is worth adding, but it is not worth reopening the just-completed fixture-ladder regeneration solely for this behavior.

The current implementation has focused integration coverage for payload-backed `ResourcePageSource` latest-state resolution. That is enough for the immediate release/regen path. The reason to add a conformance fixture later is that latest-state page-source resolution is externally visible Semantic Flow behavior: a portable implementation should be able to demonstrate that a page source can follow the latest settled payload state without reading mutable working bytes.

When the next conformance-ladder pass is already happening, add one small Alice Bio transition rather than a new ladder:

- start after `alice/page-main` has a settled payload state.
- mutate the `alice/page-main` working file so it visibly differs from the settled state.
- configure an Alice page region source with `sflo:hasTargetArtifact <.../alice/page-main>` and `sflo:hasArtifactResolutionMode sflo:artifactResolutionMode_latestState`.
- weave the page.
- assert that the rendered ResourcePage uses the settled `alice/page-main` state, not the draft working file.
- assert the source triples show `ResourcePageSource`, `hasTargetArtifact`, and `artifactResolutionMode_latestState`.

That fixture would prove the key semantic distinction: `working` follows editable current bytes, while `latestState` follows settled mesh history. More exotic cases, such as requested-history latest-state, ambiguous no-history failure, or exact-state redundancy, can stay in unit/integration tests until they become conformance-level behavior.

## Implementation Plan

- [x] Inventory existing resolver-like code paths for page sources, extraction sources, source registries, and payload history progression.
- [ ] Define a shared `ArtifactResolutionTarget` runtime resolver that returns the requested target plus observed state/manifestation/located-file/digest evidence.
- [x] Implement requested-history latest-state resolution for payload-backed `ResourcePageSource` bindings.
- [x] Implement no-history latest-state resolution through `currentArtifactHistory` for payload-backed `ResourcePageSource` bindings.
- [x] Fail closed for ambiguous latest-across-histories until an explicit deterministic policy is chosen.
- [x] Wire the first operation-level consumer: page-source generation.
- [x] Update behavior specs and glossary after the first consumer is implemented.
- [x] Add focused integration tests for working versus latest-state behavior in page-source generation.
- [d] Add a dedicated latest-state conformance fixture during the next fixture-ladder regeneration pass that is already happening; do not reopen the just-completed regeneration solely for this.

## Implementation Progress

2026-05-19:

- Weave now resolves payload-backed `ResourcePageSource` bindings with `artifactResolutionMode_latestState` by following the target payload artifact's `currentArtifactHistory` to `latestHistoricalState`.
- Weave now treats a `ResourcePageSource` with `hasRequestedTargetHistory` and no explicit mode as a history-bounded latest-state request.
- Page generation fails closed when no current/default history is available, when the requested history is not a history of the target artifact, when the history is not typed as `ArtifactHistory`, or when the latest state cannot be materialized to a settled file.
- The first implementation reads `locatedFileForState` when present and only derives the conventional payload snapshot path as a fallback.
- The broader shared resolver, exact-state page-source resolution, extraction/source-registry consumers, SHACL tests, and latest-across-all-histories policy remain future work.
