---
id: c09d608076d1eec106a246a1
title: 2026 05 27 2215 ResourcePageSource Resolution Semantics
desc: >-
  Implement the narrow ResourcePageSource exact/latest/working/fallback runtime
  semantics after per-target effective config settles.
updated: 1779945357157
created: 1779945357157
---

## Goals

- Turn the remaining page-source resolution work from [[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]] into a small implementation task.
- Align Weave's `ResourcePageSource` runtime behavior with [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]] and the shared `ArtifactResolutionSpec` vocabulary.
- Replace the remaining custom page-source resolution branches with the shared artifact-resolution service where that reduces duplicated working/latest-state logic.
- Support exact in-mesh state resolution for `ResourcePageSource` instead of rejecting `targetHistoricalState`.
- Support one explicit `hasFallbackArtifactResolutionSpec` for page-source resolution, with bounded fail-closed behavior.
- Preserve the first-pass safety boundaries: direct `targetAccessUrl`, remote `workingAccessUrl`, repository/floating locators, and outside-the-tree live sources remain out of page-generation scope unless explicit operational policy and tests are added.

## Summary

The first ResourcePage implementation landed the useful surface: `_knop/_page/page.ttl`, local `targetLocalRelativePath` sources, artifact-backed working/current sources, and latest-state artifact sources. The remaining source-resolution gap is narrower than the original umbrella task:

- exact `targetHistoricalState` sources still fail as unsupported in `src/runtime/weave/page_definition.ts`
- `hasFallbackArtifactResolutionSpec` is modeled in ontology/specs but still rejected by Weave page generation
- page-source artifact resolution duplicates logic that now exists in `src/runtime/artifact_resolution`
- page-source resolution should keep fail-closed behavior and should not become an ambient fetcher

This task should implement the missing exact/fallback behavior without reopening page templating, non-RDF reference semantics, import provenance, or remote-resolution policy.

## Discussion

### Why This Is Next

After [[wa.completed.2026.2026-05-27-2031-per-target-effective-config-resolution]], page generation will already be moving from one global config toward per-target effective config. That is the right moment to clean up the next page-generation resolver edge, because page-source resolution is both user-visible and currently split from the shared artifact resolver.

This is also the likely best first consumer of [[wa.completed.2026.2026-05-24_1748-shared-artifact-resolution-runtime-service]] after config-source resolution. It proves the service against visible generated-page behavior rather than another hidden bootstrap path.

### Current Runtime Shape

`src/runtime/weave/page_definition.ts` already handles:

- direct `targetLocalRelativePath` sources as current local files, with path-policy checks
- `targetArtifact` with `artifactResolutionMode_working` or omitted mode as current/working artifact resolution
- `targetArtifact` with `artifactResolutionMode_latestState` or `targetArtifactHistory` as latest-state resolution within the requested/current history
- direct `targetAccessUrl`, target distributions, and target located files as unsupported/fail-closed

It still rejects:

- `targetHistoricalState`
- `hasFallbackArtifactResolutionSpec`
- direct local path sources that also carry artifact-resolution policy fields

Those rejections were good first-slice safety. This task should remove only the exact-state and bounded fallback rejections.

### Exact State Resolution

When a `ResourcePageSource` names `targetArtifact` plus `targetHistoricalState`, the source is exact. The runtime should:

- verify the historical state is governed by the target artifact and belongs to a declared history of that artifact
- resolve the state's located file or manifestation bytes under the existing local path policy
- fail closed if the state, manifestation, located file, or bytes cannot be resolved
- not fall back to working or latest bytes unless an explicit fallback spec is present

If `targetArtifactHistory` is also present with `targetHistoricalState`, the history must be consistent with the state. A mismatch should fail closed.

### Fallback Resolution

`hasFallbackArtifactResolutionSpec` is an explicit alternate spec, not a search strategy. The first Weave slice should support at most one fallback spec and evaluate it only after the primary spec fails for a resolution reason that is allowed to fall back.

Conservative first behavior:

- fallback from exact/latest artifact resolution to another explicit artifact-resolution spec is allowed
- fallback must be local/in-mesh under existing operational policy
- fallback must not cross into direct remote `targetAccessUrl`
- fallback must not silently jump to an unrelated source unless the authored fallback spec names that source directly
- fallback should preserve useful diagnostics: if both primary and fallback fail, report both enough for the user to repair the page definition

Direct `targetLocalRelativePath` sources should remain exact local files with no mode/fallback broadening in this task. If a local helper file is missing, page generation should fail closed.

### Shared Resolver Use

Prefer reusing `src/runtime/artifact_resolution` over adding another custom page-source resolver. That service already understands `ArtifactResolutionSpec` parsing, local path policy, working/latest-state mode, digest checks, unsupported fallback rejection, and failure diagnostics. If the shared service still lacks exact/fallback behavior, add it there in a way that config sources and future source families can also use.

The page-definition layer should stay responsible for ResourcePage-specific concerns:

- finding `ResourcePageRegion` and its source node
- accepting exactly one first-pass source per region
- converting resolved text bytes into Markdown/page-region content
- applying page-definition fail-closed behavior

It should not re-implement artifact-state traversal if the shared resolver can do that.

### Remote And Import Boundaries

Do not turn page generation into a live fetcher in this task. `targetAccessUrl`, remote `workingAccessUrl`, repository source locators, and floating repository locators may remain modeled but unsupported by ResourcePage generation until [[wa.task.2026.2026-04-11_1723-operational-config-for-runtime-resolution]] and [[wa.task.2026.2026-05-20_2152-workingAccessUrl]] are deliberately broadened.

Outside-the-tree or extra-mesh content should still enter page generation through import: the imported in-tree governed artifact becomes the page source, and its working or settled state is resolved locally.

## Open Issues

- Should fallback be allowed after any primary resolution failure, or only after "not found/unavailable" failures? Recommendation: start conservative and allow fallback only for resolution failures, not malformed source specs, contradictory coordinates, unsafe paths, digest mismatches, unsupported locator kinds, or failed validation.
- Should exact-state page-source resolution accept both `targetHistoricalState` and `targetLocatedFile` in the same source when they identify the same bytes? Recommendation: keep one target locator per source for now; consistency checks across multiple locators can wait until a real use case appears.
- Should page-source resolved bytes verify `expectsContentDigest` in this task? Recommendation: yes if the shared resolver already has it for the locator kind being used; do not invent page-specific digest behavior.
- Should media-type/content-kind hints be required before reading non-Markdown artifact sources? Recommendation: no for this task; keep current Markdown/text behavior and fail closed on unsupported binary/text decoding cases.

## Decisions

- Implement exact `targetHistoricalState` support for artifact-backed `ResourcePageSource`.
- Implement one explicit `hasFallbackArtifactResolutionSpec` for artifact-backed page sources.
- Treat fallback as an explicit alternate `ArtifactResolutionSpec`, not a fallback-policy enum; the live ontology has working/latest-state resolution modes but no current fallback enum vocabulary.
- Attempt fallback only when an exact/latest payload-artifact primary source fails because requested bytes are unavailable. Do not fall back after malformed specs, contradictory coordinates, unsafe paths, unsupported locators, digest mismatches, failed validation, or text decoding failures.
- Keep direct `targetLocalRelativePath` sources exact and fail-closed; no mode/fallback broadening for local helper files.
- Allow an explicit fallback spec to name a direct `targetLocalRelativePath` source, but only when the primary source is an eligible exact/latest artifact-backed source.
- Keep `targetAccessUrl`, remote `workingAccessUrl`, repository locators, and floating locators unsupported for page generation in this slice.
- Prefer shared artifact-resolution service changes over duplicating state/history traversal in page-definition code.
- Preserve fail-closed behavior for malformed, contradictory, unsafe, unsupported, or unresolved page sources.
- Do not change ontology vocabulary for this task.

## Contract Changes

- Weave page generation should support `ResourcePageSource` exact state resolution through `targetArtifact` + `targetHistoricalState`.
- Weave page generation should support one bounded explicit fallback spec for artifact-backed `ResourcePageSource`.
- Page-source resolution failures should include enough context to identify the owning designator path, region key, primary source spec, and fallback source spec when present.
- ResourcePageDefinition parsing should continue to reject multiple unordered sources per region and multiple independent target locators in one source.
- No Semantic Flow Framework spec change is expected unless implementation discovers a mismatch with [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]].

## Testing

- Unit-test `ResourcePageSource` exact state resolution for a governed payload artifact, including the rendered region text.
- Unit-test exact state mismatch cases: state outside target artifact history, requested history inconsistent with requested state, missing state file, and missing located file.
- Unit-test bounded fallback: primary exact/latest source missing or unavailable, fallback explicit in-mesh source succeeds, rendered page uses fallback content.
- Unit-test fallback failure diagnostics when both primary and fallback fail.
- Unit-test that malformed/contradictory/unsafe/unsupported primary specs do not fall back silently.
- Regression-test that direct `targetLocalRelativePath` remains fail-closed and does not honor fallback/mode fields.
- Regression-test that `targetAccessUrl` and remote current-byte locators remain unsupported in page generation.
- Add or update an integration fixture only if the focused runtime tests do not exercise enough of the real `weave generate` path.
- Run focused `page_definition` and artifact-resolution tests first, then `deno task check` and `deno task lint` for touched runtime modules.

## Non-Goals

- Do not implement multiple ordered sources per region.
- Do not implement arbitrary fallback chains.
- Do not add remote fetching or repository checkout resolution for page generation.
- Do not add media-type/content-kind negotiation.
- Do not redesign ResourcePage presentation, generated panels, or template selection.
- Do not change import provenance or source-registry vocabulary.
- Do not close or rename [[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]]; leave it as the historical umbrella until the remaining slices have explicit homes.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]], [[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]], and [[wa.completed.2026.2026-05-24_1748-shared-artifact-resolution-runtime-service]].
- [x] Inventory current custom page-source resolution in `src/runtime/weave/page_definition.ts` and the shared artifact resolver behavior in `src/runtime/artifact_resolution`.
- [x] Decide the smallest shared resolver API shape for resolving a `ResourcePageSource` node into text bytes and resolved coordinates.
- [x] Add exact `targetHistoricalState` support in the shared resolver or page-source adapter.
- [x] Add one-level `hasFallbackArtifactResolutionSpec` support for artifact-backed sources.
- [x] Preserve direct `targetLocalRelativePath` handling as exact local file resolution with no fallback/mode broadening.
- [x] Update `loadActiveCustomIdentifierPage` to use the shared resolver/adapter for artifact-backed sources.
- [x] Improve error messages to include designator path, region key, and primary/fallback source context.
- [x] Add focused unit tests for exact state, fallback success, fallback failure, no fallback on malformed/unsafe specs, and remote-locator rejection.
- [x] Run focused tests, then `deno task check` and `deno task lint` for touched Weave modules.
- [x] Update [[wd.todo]] and provide a semantic-commit-style commit message for the Weave repo.
