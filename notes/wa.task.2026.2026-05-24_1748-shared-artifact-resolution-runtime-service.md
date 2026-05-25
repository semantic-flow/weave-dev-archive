---
id: trbrhxgf7qgd17x0ec2u3g3
title: 2026 05 24_1748 Shared Artifact Resolution Runtime Service
desc: ''
updated: 1779670258967
created: 1779670128331
---

## Goals

- Define a shared runtime service for resolving `sflo:ArtifactResolutionTarget`-shaped requests into concrete observed artifact evidence.
- Consolidate duplicated working/latest/exact resolution logic currently spread across page-source loading, extraction-source loading, source-registry parsing, raw source panels, and payload history readers.
- Keep the resolver explicitly separate from active acquisition: arbitrary URL fetch remains an explicit `weave import` operation, not a general resolution behavior.
- Return both requested coordinates and observed evidence so operations can render, record, validate, or audit what actually resolved.
- Make `working`, `latestState`, exact state, exact manifestation, exact located-file, repository-backed local source, and digest evidence behave consistently across consumers.
- Keep this as a follow-up after the first import/source-binding fixture proof, not a blocker for immediate fixture regeneration unless that regeneration intentionally exercises the shared resolver.

## Summary

Weave now has several runtime paths that all perform pieces of artifact resolution:

- payload loading resolves current working files and latest settled snapshots
- page-definition region loading resolves `ResourcePageSource` targets in working or latest-state mode
- extraction-source loading resolves source artifacts and records observed state/manifestation/located-file/digest evidence
- source-registry readers parse `ExtractionSource`, `IntegrationSource`, and `ImportSource` bindings with overlapping requested/observed fields
- raw source panels derive similar state/file information for display

Those paths are converging on the same concept: take an `ArtifactResolutionTarget`-style description and answer, fail-closed, "what concrete artifact bytes or settled state did this request mean right now?"

The shared service should not be a URL loader. It should be a local runtime resolver for governed artifact coordinates. If the request carries `sflo:targetAccessUrl` on an `sflo:ImportSource`, that URL is acquisition provenance unless the caller is the explicit import command. Normal resolution should follow the governed local working file or settled historical state, not refetch the original import URL.

The service should return a structured result containing:

- requested target artifact and optional requested history/state/manifestation/located-file/digest coordinates
- requested resolution mode, such as `working` or `latestState`
- observed state, manifestation, located file, local relative path, digest, and observation timestamp when those are available or required
- enough source-binding identity/context for the caller to write or expose evidence without reparsing raw RDF

The immediate value is not new behavior. It is consistency: page sources, extraction sources, reference/source panels, source registries, and future manifest-driven integrate should agree about the same vocabulary and the same fail-closed boundaries.

## Discussion

### Why This Still Matters Without Ambient URL Fetch

Most artifact resolution is local and governed. `targetAccessUrl` is only one possible coordinate, and after import it is usually provenance rather than the current byte locator.

Useful resolver cases include:

- `working`: resolve the active current working file through local path policy, including mesh-local files, approved extra-mesh local paths, and repository floating locators.
- `latestState`: resolve the latest settled `HistoricalState`, manifestation, and located file through artifact history facts.
- exact requested state: validate the state belongs to the requested artifact/history and resolve its file/manifestation evidence.
- exact requested manifestation or located file: validate that the coordinate is in-mesh and belongs to the requested artifact state when that relationship is asserted.
- repository-backed local source bindings: resolve to the allowed local checkout path under repository/local-path policy, not by inspecting live remote refs.
- digest evidence: compute or compare observed bytes when the caller requires content evidence.

That means the service is useful even if `weave`, `generate`, extraction, and page rendering never fetch arbitrary URLs. Explicit import remains the one command that may acquire outside HTTP(S) bytes.

### Current Implementation Shape

The completed latest-state task implemented the first real consumer for payload-backed page sources. See [[wa.completed.2026.2026-05-19_0022-lateststate-improvement]] and the later conformance task [[wa.task.2026.2026-05-19_1536-latest-state-conformance]].

Current duplication worth consolidating:

- `src/runtime/weave/page_definition.ts` has private page-source artifact resolution for current working and latest-state region sources.
- `src/runtime/weave/artifact_loaders.ts` resolves extraction source evidence, reads selected snapshots, and computes observed digests.
- `src/runtime/mesh/inventory.ts` parses payload current/latest fields plus `ExtractionSource`, `IntegrationSource`, and `ImportSource` requested/evidence fields.
- `src/runtime/weave/raw_source_panels.ts` has display-oriented source resolution logic that shadows parts of payload/source evidence resolution.
- `src/runtime/import/import.ts` and `src/runtime/integrate/integrate.ts` compute digests and emit evidence, but import is active acquisition while integrate records existing local/source bindings.

The shared service should grow out of those existing paths rather than introducing a parallel vocabulary model.

### Proposed Shape

Start with a small runtime module, probably under `src/runtime/artifact_resolution/` or `src/runtime/weave/artifact_resolution.ts`, with core-ish types that can later move if they become planner-facing.

The service can expose two layers:

1. parse or normalize target coordinates from already-loaded source binding/page source state
2. resolve those coordinates against loaded mesh/Knop inventory plus local path policy

Illustrative shape:

```ts
interface ArtifactResolutionRequest {
  targetArtifactPath: string;
  mode?: "working" | "latestState";
  requestedHistoryPath?: string;
  requestedStatePath?: string;
  requestedManifestationPath?: string;
  requestedLocatedFilePath?: string;
  expectedContentDigest?: string;
}

interface ArtifactResolutionResult {
  requested: ArtifactResolutionRequest;
  observed: {
    statePath?: string;
    manifestationPath?: string;
    locatedFilePath?: string;
    localRelativePath?: string;
    contentDigest?: string;
    observedAt?: string;
  };
}
```

The final type should probably distinguish "resolved locator only" from "read bytes and computed digest." Some consumers only need paths; extraction/import evidence often needs digest.

### Requested Versus Observed

The resolver should not collapse requested and observed coordinates.

Examples:

- A request for `latestState` may observe `alice/page-main/_history001/_s0003`.
- A request for a history may observe that history's `latestHistoricalState`.
- A request for exact state should echo the requested state and may observe a located file or digest.
- A request for `working` may observe only a local working file and digest, with no historical state.

This distinction should align with the ontology split from [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]]: coordinates live on the target/source binding; evidence lives in an `ArtifactResolutionObservation` when an operation records it.

### Relationship To Import

`weave import` is active acquisition. It may fetch an explicit HTTP(S) URL, copy bytes into a governed local working file, compute an observed digest, and record the original URL on `sflo:ImportSource` as provenance.

The shared resolver should not make ordinary runtime operations follow `ImportSource.targetAccessUrl`. After import succeeds, the artifact's active bytes are the governed working file or settled historical states created later. If a future caller wants to "refresh from import URL," that should be an explicit repeated import operation, not a generic artifact resolution side effect.

### Relationship To Integrate And Extraction

`weave integrate` records existing bytes where they already live. Repository-backed or floating source coordinates are good resolver inputs, but only through local checkout policy. The resolver may compute observed digest evidence from local bytes when requested; it should not inspect live repository remotes.

Extraction currently carries source evidence through `ExtractionSource` and generated Knop source registries. That is probably the first source-registry consumer that would benefit from the shared resolver after page sources, because it already needs observed state/manifestation/located-file/digest evidence.

### Relationship To Append-Onlyish Inventory

This task is adjacent to [[wa.task.2026.2026-05-17-append-onlyish-inventory]], but it should not be bundled with that migration.

The resolver can initially read today's inventory/meta shape. The append-onlyish storage migration can later change where current/progression facts live, and the resolver should become the single runtime place that absorbs that storage split. That is another reason to create the service before or during the inventory migration, but not as a hidden part of fixture regeneration.

### Suggested Sequencing

Do not block the immediate import/source-binding fixture regeneration on this service.

Useful order:

- land and commit import cleanly
- regenerate fixtures to prove `ImportSource`, `IntegrationSource`, and observations
- add latest-state conformance if it fits that regeneration pass
- implement the shared resolver as the cleanup step before broadening latest/exact source binding behavior or before the append-onlyish inventory migration

## Open Issues

- Should the first implementation live under runtime-only code, or should request/result types live in `src/core` because planners will eventually consume them?
- Should the resolver parse RDF source bindings directly, or should `inventory.ts` continue parsing RDF into typed state objects that the resolver accepts?
- Should reading bytes and computing digest be an optional resolver capability, or a separate helper layered on top of path/state resolution?
- How should exact target plus redundant `latestState` be reported: tolerated with a warning-like result, or rejected by runtime before SHACL has a say?
- Should no-history `latestState` continue to resolve only through `currentArtifactHistory`, or should the shared resolver also support unambiguous latest-across-all-histories later?
- Should observed evidence ever be written back by the resolver itself, or should it only return evidence and leave persistence to the calling operation?
- What is the minimum consumer migration that proves the abstraction without boiling the lake?

## Decisions

- The shared service is not a remote fetch mechanism. Ambient URL following remains out of scope; explicit HTTP(S) acquisition belongs to `weave import`.
- Preserve the semantic split between requested coordinates and observed evidence.
- Keep exact coordinates exact. Do not revive `artifactResolutionMode_pinned`.
- Start fail-closed: if requested coordinates cannot be proven to belong to the target artifact, or if a required located file/digest cannot be observed, resolution fails before writes.
- Treat `working` and `latestState` as distinct modes. `latestState` must not fall back to mutable working bytes.
- Let callers decide whether and where to persist returned observations.
- Do not bundle this with fixture regeneration or append-onlyish inventory migration unless one of those tasks explicitly chooses it as scope.

## Contract Changes

- Add a runtime API for resolving `ArtifactResolutionTarget`-style requests into requested/observed result objects.
- Replace page-source latest-state private resolution with the shared service once the service supports the existing behavior.
- Replace extraction-source observed evidence resolution with the shared service once it can return state/manifestation/located-file/digest evidence.
- Align source-registry parser outputs for `ExtractionSource`, `IntegrationSource`, and `ImportSource` with the request/result model.
- Clarify developer documentation that `targetAccessUrl` on `ImportSource` is acquisition provenance, not a current-byte remote locator for ordinary runtime resolution.
- If the behavior becomes portable Semantic Flow API surface, update the relevant framework spec notes rather than only Weave developer docs.

## Testing

- Unit-test request normalization for working, latest-state, exact state, requested history, exact manifestation, exact located file, and expected digest.
- Unit-test fail-closed ownership checks: requested history belongs to target artifact, state belongs to requested history, manifestation belongs to state, located file belongs to state/manifestation where asserted.
- Unit-test that `targetAccessUrl` is not fetched by the resolver.
- Unit-test latest-state resolution through `currentArtifactHistory` and requested-history `latestHistoricalState`.
- Unit-test working resolution through mesh-local `hasWorkingLocatedFile`, `workingLocalRelativePath`, and repository floating locator under local path policy.
- Integration-test page-source generation after migrating it to the shared resolver, preserving the existing working versus latest-state behavior.
- Integration-test extraction evidence after migrating it to the shared resolver, preserving observed state/manifestation/located-file/digest output.
- Add regression coverage that import provenance remains provenance: repeated `weave` or `generate` does not fetch an `ImportSource.targetAccessUrl`.

## Non-Goals

- Do not implement arbitrary URL import/fetch in the resolver.
- Do not refresh imports from their original URL. Use repeated `weave import --replace-working` for that.
- Do not inspect live git remotes or floating repository refs.
- Do not redesign the ontology vocabulary in this task.
- Do not move current/progression facts out of inventory; that belongs to [[wa.task.2026.2026-05-17-append-onlyish-inventory]].
- Do not implement manifest-driven integrate; that belongs to [[wa.task.2026.2026-05-18_1846-integrate-manifest]].
- Do not add a conformance fixture solely for the service abstraction. Fixtures should prove externally visible behavior, not internal factoring.

## Implementation Plan

- [ ] Re-inventory current resolver-like paths in `page_definition.ts`, `artifact_loaders.ts`, `inventory.ts`, `raw_source_panels.ts`, `import.ts`, and `integrate.ts`.
- [ ] Decide module placement and public type names for artifact resolution requests/results.
- [ ] Add focused unit tests for request normalization and fail-closed exact/latest/working resolution.
- [ ] Implement the first resolver slice for payload artifacts: working file resolution, requested-history latest-state, no-history latest-state through `currentArtifactHistory`, exact requested state, and located-file evidence.
- [ ] Add optional byte-reading/digest helper or resolver option, without remote URL fetching.
- [ ] Migrate payload-backed `ResourcePageSource` working/latest-state resolution to the shared service.
- [ ] Migrate extraction-source selected-source evidence to the shared service.
- [ ] Align source-registry parser output types with the shared request/result model where that reduces duplication.
- [ ] Add regression coverage that `ImportSource.targetAccessUrl` is not used by ordinary resolution.
- [ ] Update developer docs or task notes with the final resolver contract and consumer migration status.
- [ ] Run focused tests, then `deno task fmt`, `deno task lint`, `deno task check`, and relevant integration tests.
