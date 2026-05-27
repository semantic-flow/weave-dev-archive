---
id: trbrhxgf7qgd17x0ec2u3g3
title: 2026 05 24_1748 Shared Artifact Resolution Runtime Service
desc: Define a shared runtime resolver for ArtifactResolutionSpec bytes, evidence, and config-source discovery.
updated: 1779670258967
created: 1779670128331
---

## Goals

- Define a shared runtime service for resolving `sflo:ArtifactResolutionSpec`-shaped requests, including subclasses such as `sfcfg:ConfigSource`, `sflo:ResourcePageSource`, `sflo:ExtractionSource`, `sflo:ReferenceSource`, `sflo:ImportSource`, and `sflo:IntegrationSource`.
- Make the first implementation a config-unblocking resolver core for [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]], not a broad migration of every existing source consumer.
- Resolve governed local bytes/text under active operational policy while preserving the semantic split between requested coordinates and observed evidence.
- Consolidate duplicated working/latest/exact resolution logic currently spread across page-source loading, extraction-source loading, source-registry parsing, raw source panels, and payload history readers.
- Keep the resolver explicitly separate from active acquisition: arbitrary URL fetch remains an explicit `weave import` operation, not a general resolution behavior.
- Return enough requested coordinates, observed coordinates, byte/text content, digest, and trace context for callers to parse config, render pages, validate sources, or intentionally persist observations.
- Keep page-source, extraction-source, raw-source-panel, and source-registry migrations as follow-on cleanup after the first config-source-ready resolver slice lands.

## Summary

The ontology has moved from the old `ArtifactResolutionTarget` framing to `sflo:ArtifactResolutionSpec`. The current config ontology also defines `sfcfg:ConfigSource` as a config-specific `ArtifactResolutionSpec` subclass. That makes config-source discovery depend on a runtime capability that does not yet exist as a shared service: take an artifact-resolution spec, enforce operational policy, and return the concrete bytes/text plus observed resolution evidence.

Today, Weave has several runtime paths that perform pieces of this behavior independently:

- payload loading resolves current working files and latest settled snapshots
- page-definition region loading resolves `sflo:ResourcePageSource` targets in working or latest-state mode
- extraction-source loading resolves source artifacts and records observed state/manifestation/located-file/digest evidence
- source-registry readers parse `sflo:ExtractionSource`, `sflo:IntegrationSource`, and `sflo:ImportSource` bindings with overlapping requested/observed fields
- raw source panels derive similar state/file information for display

The immediate value is to stop config-source discovery from adding a new config-only resolver. A first slice should expose a small service that resolves the coordinate subset needed by `sfcfg:ConfigSource`, then lets the config resolver parse the returned bytes as Turtle and compile the result into the appropriate config layer. Once that exists, page sources and extraction sources can migrate incrementally.

The shared service should not be a URL loader. It should be a local runtime resolver for governed artifact coordinates. If a request carries `sflo:targetAccessUrl` on an `sflo:ImportSource`, that URL is acquisition provenance unless the caller is the explicit import command. Normal resolution should follow governed local working files, local relative paths allowed by operational policy, or settled historical state coordinates.

## Discussion

### Current Motivation: ConfigSource

[[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] needs to discover `sfcfg:hasConfigSource` and `sfcfg:hasInheritableConfigSource` attachments, resolve each `sfcfg:ConfigSource`, parse the resolved bytes as config, and add the result as an ordinary config input at the scope and role determined by the attachment point.

That task should not learn its own path grammar, artifact-history traversal, digest semantics, or no-ambient-fetch boundary. This resolver task should provide the runtime primitive it needs:

- parse or accept a normalized `ArtifactResolutionSpec` request
- resolve a supported coordinate subset under `OperationalLocalPathPolicy`
- optionally read bytes/text
- optionally compute or verify digest evidence
- return requested and observed coordinates without writing observation records by default
- report unsupported coordinate forms as fail-closed errors with source identity and context

### Why This Still Matters Without Ambient URL Fetch

Most artifact resolution is local and governed. `sflo:targetAccessUrl` is only one possible coordinate, and after import it is usually provenance rather than the current byte locator.

Useful resolver cases include:

- `working`: resolve active current working files through local path policy, including mesh-local files, approved extra-mesh local paths, and repository floating locators once repository policy is supported.
- `latestState`: resolve the latest settled `sflo:HistoricalState`, manifestation, and located file through artifact history facts.
- exact requested state: validate the state belongs to the requested artifact/history and resolve its file/manifestation evidence.
- exact requested manifestation or located file: validate that the coordinate is in-mesh and belongs to the requested artifact state when that relationship is asserted.
- direct `sflo:targetLocalRelativePath`: resolve workspace/mesh-local source bytes only when local path policy permits the `targetLocalRelativePath` locator kind.
- digest evidence: compute or compare observed bytes when the caller requires content evidence.

That means the service is useful even if `weave`, `generate`, extraction, and page rendering never fetch arbitrary URLs. Explicit import remains the one command that may acquire outside HTTP(S) bytes.

### First Slice

The first implementation should be deliberately smaller than the full migration imagined by the original note. It should be enough to unblock config-source discovery and to prove that the service shape can later absorb page/extraction duplication.

Initial supported request shapes:

- `sflo:targetLocalRelativePath` resolved under `OperationalLocalPathPolicy` with locator kind `targetLocalRelativePath`
- governed `sflo:targetArtifact` with `sflo:hasArtifactResolutionMode sflo:artifactResolutionMode_working` when the artifact has current working bytes resolvable by current inventory helpers
- governed `sflo:targetArtifact` with `sflo:hasArtifactResolutionMode sflo:artifactResolutionMode_latestState` when current history/latest state can be proven and read
- exact `sflo:targetHistoricalState` for a governed payload artifact when the state belongs to the requested artifact/history
- exact `sflo:targetLocatedFile` when it is an in-mesh located file path and no broader provenance proof is required by the first caller
- `sflo:expectsContentDigest` verification when bytes are read

Initial unsupported request shapes should fail closed:

- `sflo:targetAccessUrl`
- unpinned or remote repository fetches
- fallback specs, unless the first implementation includes explicit cycle detection and deterministic fallback order
- multiple independent target locators that cannot be proven to name the same bytes
- exact manifestation resolution if the existing inventory helpers do not yet make it cheap to prove ownership

### Requested Versus Observed

The resolver must not collapse requested and observed coordinates.

Examples:

- A request for `latestState` may observe `alice/page-main/_history001/_s0003`.
- A request for a history may observe that history's `latestHistoricalState`.
- A request for exact state should echo the requested state and may observe a located file or digest.
- A request for `working` may observe only a local working file and digest, with no historical state.
- A direct `targetLocalRelativePath` may observe the same local relative path plus a digest, without pretending there is a governed historical state.

This distinction should align with [[ont.completed.2026.2026-05-26_2205-artifact-resolution-spec-and-observation-cleanup]]: coordinates live on the target/source binding; evidence lives in an `sflo:ArtifactResolutionObservation` only when an operation intentionally records it.

### Proposed Runtime Shape

Start with a small runtime module under `src/runtime/artifact_resolution/`. Keep it runtime-owned for now; move request/result types toward `src/core` only if planner-facing or portable API needs emerge.

The service can expose two layers:

1. parse or normalize target coordinates from RDF quads for a known `ArtifactResolutionSpec` subject
2. resolve normalized coordinates against mesh inventory plus local path policy

Illustrative shape:

```ts
interface ArtifactResolutionRequest {
  sourceIri?: string;
  targetArtifactIri?: string;
  mode?: "working" | "latestState";
  targetArtifactHistoryIri?: string;
  targetHistoricalStateIri?: string;
  targetManifestationIri?: string;
  targetLocatedFileIri?: string;
  targetLocalRelativePath?: string;
  expectedContentDigest?: string;
}

interface ArtifactResolutionResult {
  requested: ArtifactResolutionRequest;
  observed: {
    historicalStateIri?: string;
    manifestationIri?: string;
    locatedFileIri?: string;
    localRelativePath?: string;
    contentDigest?: string;
    observedAt?: string;
  };
  content?: {
    bytes: Uint8Array;
    text?: string;
    mediaType?: string;
  };
}
```

The exact type should distinguish locator-only resolution from byte-reading resolution. Config-source discovery needs bytes/text. Some later page or source-panel callers may need text. Other callers may only need coordinates and digest evidence.

### Relationship To Config Resolution

Config-source discovery should call this service instead of parsing artifact-resolution coordinates itself. The config resolver should remain responsible for:

- deciding which attachment subject/property gives the resolved config its scope and role
- parsing returned text as Turtle with the appropriate base IRI
- compiling parsed config into effective config inputs
- detecting config-source cycles and layer conflicts
- deciding whether to expose in-memory trace detail or persist a diagnostic record

The artifact resolver should remain responsible for:

- interpreting the `ArtifactResolutionSpec` coordinate vocabulary
- enforcing local path policy and no-ambient-fetch boundaries
- proving artifact/history/state/located-file relationships for supported forms
- reading bytes/text when requested
- computing or verifying digest evidence

### Relationship To Import

`weave import` is active acquisition. It may fetch an explicit HTTP(S) URL, copy bytes into a governed local working file, compute an observed digest, and record the original URL on `sflo:ImportSource` as provenance.

The shared resolver should not make ordinary runtime operations follow `ImportSource.targetAccessUrl`. After import succeeds, the artifact's active bytes are the governed working file or settled historical states created later. If a future caller wants to refresh from an import URL, that should be an explicit repeated import operation, not a generic artifact resolution side effect.

### Relationship To Page Sources And Extraction

`sflo:ResourcePageSource` and `sflo:ExtractionSource` remain important consumers, but they should be follow-on migrations after the first config-source-ready service exists.

Page-source migration should preserve current behavior around direct `targetLocalRelativePath`, working artifact-backed sources, latest-state sources, requested history, and fail-closed unsupported combinations.

Extraction migration should preserve current source evidence behavior and continue to leave observation persistence to the operation that intentionally records source evidence.

Raw source panels should eventually use the same resolver for display, but display failure handling may stay softer than write-time resolution.

### Relationship To Append-Onlyish Inventory

This task is adjacent to [[wa.task.2026.2026-05-17-append-onlyish-inventory]], but it should not be bundled with that migration.

The resolver can initially read today's inventory/meta shape. The append-onlyish storage migration can later change where current/progression facts live, and the resolver should become the single runtime place that absorbs that storage split. That is another reason to create the service before or during the inventory migration, but not as a hidden part of config-source discovery.

## Open Issues

- Should the first implementation parse RDF source bindings directly from quads, or should callers normalize RDF into `ArtifactResolutionRequest` values before invoking the resolver? Current leaning: provide both a small RDF parser helper and a normalized resolver function, so config-source discovery can stay simple without making the resolver RDF-only.
- Should reading bytes and computing digest be a resolver option or a separate helper layered on top of coordinate resolution? Current leaning: make byte reading an explicit resolver option because config sources need it and digest verification must cover the exact bytes read.
- Should exact `targetLocatedFile` require full provenance proof in the first slice, or is in-mesh located-file resolution acceptable when no target artifact is supplied? Current leaning: allow in-mesh exact located files only when the caller's policy permits a direct located-file coordinate and the path is under the mesh.
- What is the minimum page-source migration that proves the abstraction after config-source discovery lands?

## Decisions

- Use `sflo:ArtifactResolutionSpec` terminology. `ArtifactResolutionTarget` is historical wording and should not appear in new runtime API names.
- Make `sfcfg:ConfigSource` the immediate first consumer and [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] the first downstream task.
- The shared service is not a remote fetch mechanism. Ambient URL following remains out of scope; explicit HTTP(S) acquisition belongs to `weave import`.
- Preserve the semantic split between requested coordinates and observed evidence.
- Keep exact coordinates exact. Do not revive `artifactResolutionMode_pinned`.
- Start fail-closed: if requested coordinates cannot be proven to belong to the target artifact when such proof is required, or if a required located file/digest cannot be observed, resolution fails before writes.
- Treat `working` and `latestState` as distinct modes. `latestState` must not fall back to mutable working bytes.
- Let callers decide whether and where to persist returned observations. Ordinary config-source reads must not append `sflo:ArtifactResolutionObservation` records.
- Do not bundle every page/extraction/source-registry migration into the first implementation.

## Contract Changes

- Add a runtime API for resolving `sflo:ArtifactResolutionSpec`-style requests into requested/observed result objects and optional byte/text content.
- Add a small RDF normalization helper for supported artifact-resolution spec properties.
- Use the shared service from config-source discovery instead of custom config-only path parsing.
- Replace page-source latest-state private resolution with the shared service after the service supports the existing behavior.
- Replace extraction-source observed evidence resolution with the shared service once it can return state/manifestation/located-file/digest evidence.
- Align source-registry parser outputs for `sflo:ExtractionSource`, `sflo:IntegrationSource`, and `sflo:ImportSource` with the request/result model where that reduces duplication.
- Clarify developer documentation that `sflo:targetAccessUrl` on `sflo:ImportSource` is acquisition provenance, not a current-byte remote locator for ordinary runtime resolution.
- If the behavior becomes portable Semantic Flow API surface, update the relevant framework spec notes rather than only Weave developer docs.

## Testing

- Unit-test request normalization for direct `targetLocalRelativePath`, working `targetArtifact`, latest-state `targetArtifact`, requested history, exact state, exact located file, and expected digest.
- Unit-test fail-closed ownership checks: requested history belongs to target artifact, state belongs to requested history, and located file belongs to state/manifestation where asserted and required by the supported request shape.
- Unit-test that `targetAccessUrl` is not fetched by the resolver.
- Unit-test latest-state resolution through `currentArtifactHistory` and requested-history `latestHistoricalState`.
- Unit-test working resolution through mesh-local `workingLocalRelativePath` and repository floating locator only if repository local policy is included in the first slice.
- Unit-test direct `targetLocalRelativePath` through local path policy, including allowed in-mesh paths, denied path escapes, and missing files.
- Unit-test `sflo:expectsContentDigest` success and mismatch failure using the exact bytes returned.
- Integration-test config-source discovery after it calls the shared resolver, preserving the no-persist-by-default decision for observations.
- Integration-test page-source generation after migrating it to the shared resolver, preserving the existing working versus latest-state behavior.
- Integration-test extraction evidence after migrating it to the shared resolver, preserving observed state/manifestation/located-file/digest output.
- Add regression coverage that import provenance remains provenance: repeated `weave` or `generate` does not fetch an `ImportSource.targetAccessUrl`.

## Non-Goals

- Do not implement arbitrary URL import/fetch in the resolver.
- Do not refresh imports from their original URL. Use repeated `weave import --replace-working` for that.
- Do not inspect live git remotes or floating repository refs.
- Do not redesign the ontology vocabulary in this task.
- Do not move current/progression facts out of inventory; that belongs to [[wa.task.2026.2026-05-17-append-onlyish-inventory]].
- Do not implement full config-source discovery; that belongs to [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] and should consume this resolver.
- Do not implement manifest-driven integrate; that belongs to [[wa.task.2026.2026-05-18_1846-integrate-manifest]].
- Do not add a conformance fixture solely for the service abstraction. Fixtures should prove externally visible behavior, not internal factoring.

## Implementation Plan

- [x] Re-inventory current resolver-like paths in `page_definition.ts`, `artifact_loaders.ts`, `inventory.ts`, `raw_source_panels.ts`, `import.ts`, and `integrate.ts`, noting what the first config-source slice can reuse without broad migration.
- [x] Decide module placement and public type names under `src/runtime/artifact_resolution/`.
- [x] Add focused unit tests for RDF request normalization and fail-closed direct/local/exact/latest/working resolution.
- [x] Implement normalized request/result types using `sflo:ArtifactResolutionSpec` naming.
- [x] Implement direct `targetLocalRelativePath` resolution with local path policy and optional byte/text read.
- [x] Implement governed payload-artifact working resolution using existing inventory helpers and local path policy.
- [x] Implement governed payload-artifact latest-state and requested-history latest-state resolution using existing inventory helpers.
- [x] Implement exact state and simple exact located-file resolution if doing so is cheap enough for the config-source first slice; otherwise leave explicit fail-closed errors and tests.
- [x] Add optional byte-reading/digest verification, without remote URL fetching.
- [x] Add parser/normalizer support for `sfcfg:ConfigSource` subjects as `ArtifactResolutionSpec` instances.
- [ ] Wire the new resolver into [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] once the first resolver slice is green.
- [ ] After config-source discovery lands, migrate payload-backed `sflo:ResourcePageSource` working/latest-state resolution to the shared service.
- [ ] After page sources, migrate extraction-source selected-source evidence to the shared service.
- [ ] Align source-registry parser output types with the shared request/result model where that reduces duplication.
- [ ] Add regression coverage that `sflo:ImportSource` / `sflo:targetAccessUrl` is not used by ordinary resolution.
- [ ] Update developer docs or task notes with the final resolver contract and consumer migration status.
- [x] Run focused tests, then `deno task fmt`, `deno task lint`, `deno task check`, and relevant integration tests.
