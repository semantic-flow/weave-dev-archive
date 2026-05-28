---
id: wgv1vdga8zodyooa52gfeqt
title: 2026 05 27_1246 Config Source Discovery and Resolution
desc: Teach Weave to discover and resolve sfcfg:ConfigSource attachments into effective config.
updated: 1779934447201
created: 1779911218761
---

## Goals

- Teach Weave's config resolver to discover `sfcfg:ConfigSource` attachments and resolve them through the `sflo:ArtifactResolutionSpec` model.
- Preserve attachment-point authority: referenced config participates at the scope and local/inheritable role determined by the attachment subject and generic property that selected it.
- Keep the first implementation fail-closed and deliberately narrow: no unpinned remote config retrieval, no host-trust expansion from portable config, no silent fallback to broad local path access.
- Add enough provenance and in-memory resolution trace detail that config-source participation can be explained without persisting `sfcfg:ResolvedConfig` or `sfcfg:ConfigResolutionRecord` by default.
- Leave ordinary `_mesh/_config/config.ttl` behavior intact while allowing it, mesh metadata, or later Knop support graphs to attach additional config sources.
- Make this the runtime follow-up to [[ont.completed.2026.2026-05-26_2205-artifact-resolution-spec-and-observation-cleanup]] and the source-discovery follow-up to [[wa.completed.2026.2026-05-25_1609-config-policy-ontology-and-runtime]].

## Summary

The config-policy runtime now compiles application defaults, conventional mesh-local `_mesh/_config/config.ttl`, and command overrides. The ontology also now has `sfcfg:ConfigSource` as a config-specific `sflo:ArtifactResolutionSpec`, plus generic attachment properties: `sfcfg:hasConfigSource` for local config on recognized config-bearing subjects and `sfcfg:hasInheritableConfigSource` for Knop-authored descendant defaults.

The missing slice is discovery and resolution of those source attachments. A mesh or Knop should be able to point at reusable config bytes or a `sfcfg:ConfigArtifact`; Weave should resolve that source under active runtime trust policy, parse it as config, and compile it into the same effective config runtime as inline/conventional config. The layer comes from the attachment property, not from the physical location or reusable identity of the referenced config.

The artifact-resolution cleanup substrate is now in place. This ticket should build on that vocabulary directly and begin with a quick verification that `sfcfg:ConfigSource` and `sflo:ArtifactResolutionSpec` remain stable in the active ontology, SHACL, runtime, and fixture surfaces.

## Discussion

### Dependency on artifact-resolution cleanup

`sfcfg:ConfigSource` reuses `sflo:ArtifactResolutionSpec` so it can target config bytes through the same coordinate vocabulary as other source bindings: `sflo:targetArtifact`, `sflo:targetHistoricalState`, `sflo:targetLocatedFile`, `sflo:targetManifestation`, `sflo:targetRepositorySource`, `sflo:targetLocalRelativePath`, `sflo:targetAccessUrl`, `sflo:hasFallbackArtifactResolutionSpec`, and `sflo:expectsContentDigest`.

This ticket should not reopen that vocabulary. If a needed coordinate cannot be expressed cleanly, record the gap against [[ont.completed.2026.2026-05-26_2205-artifact-resolution-spec-and-observation-cleanup]] or a new ontology task rather than creating config-only aliases.

### Seed config versus discovered config

Weave already has seed config inputs:

- application defaults in `defaults/application.ttl`
- conventional mesh-local config at `_mesh/_config/config.ttl`
- command/request overrides

Those seed inputs are trusted according to their existing layer and runtime path. Config-source discovery is recursive from those seeds and, later, from mesh/Knop support graphs. A source discovered through a mesh-local attachment participates as mesh-local config. A source discovered through a Knop-local attachment participates as Knop-local config. A source discovered through a Knop-inheritable attachment is an outbound offer that is projected into descendants before it is consumed.

Do not infer authority from where the `ConfigSource` resource is stored. A Knop-owned `ConfigArtifact` referenced by a mesh-level attachment participates at mesh scope because the mesh attachment used it. The same artifact referenced by a Knop-local attachment participates at Knop-local scope.

This means the first mesh-local slice can resolve config bytes or a `ConfigArtifact` physically stored under a Knop if the mesh subject attaches that source. That is different from scanning Knop subjects as independent config attachment points. Discovering `sflo:Knop sfcfg:hasConfigSource ?source` or `sflo:Knop sfcfg:hasInheritableConfigSource ?source` belongs to the later Knop discovery/inheritance slice.

### Attachment discovery

The first implementation should support mesh-local discovery before full Knop inheritance. Reasonable first slice:

- read the active mesh IRI/base from mesh metadata
- inspect the conventional mesh config graph for statements about that mesh using `sfcfg:hasConfigSource`
- optionally inspect mesh metadata for the same attachment property if that is where the mesh support surface naturally records the link
- resolve each attached `sfcfg:ConfigSource`
- inspect accepted resolved config graphs for additional mesh-local `sfcfg:hasConfigSource` attachments so source declarations can chain
- compile resolved config into the mesh-local layer after the conventional mesh config that referenced it, preserving deterministic order

The second slice should add:

- `sfcfg:hasConfigSource` on `sflo:Knop` subjects as Knop-local config sources
- `sfcfg:hasInheritableConfigSource` on `sflo:Knop` subjects as Knop-authored defaults projected into descendant Knop scopes
- ancestor traversal and cycle detection across inherited config-source graphs

Generic attachment authority comes from the attachment subject plus property. `sflo:SemanticMesh sfcfg:hasConfigSource ?source` is mesh-local. `sflo:Knop sfcfg:hasConfigSource ?source` is Knop-local. `sflo:Knop sfcfg:hasInheritableConfigSource ?source` is a Knop-authored descendant offer. The runtime should fail closed for these generic attachment properties on unrecognized subjects rather than guessing.

### Resolution policy

Config-source resolution must be capped by the active runtime policy. Portable config can request stricter behavior but must not grant itself broader host or network access.

Initial supported source forms should be conservative but useful enough for mesh-local reuse:

- in-mesh or workspace-bounded `sflo:targetLocalRelativePath` only when local path policy permits it
- governed `sfcfg:ConfigArtifact` / `sflo:targetArtifact` only when the artifact can be resolved through current mesh state
- exact `sflo:targetHistoricalState`, `sflo:targetLocatedFile`, or `sflo:targetManifestation` when the target is governed and available
- `sflo:expectsContentDigest` verification when present

Defer remote `sflo:targetAccessUrl` fetches, unpinned repository fetches, and arbitrary host-local file resolution. Repository-backed current-source resolution may be added later only when it uses the same host-local access profile and repository-source policy as ordinary source bindings.

### Merge and trace behavior

Each resolved config source should become a config input with:

- attachment property
- attachment subject
- source IRI or blank-node identity
- resolved coordinate summary
- source document path/IRI where known
- content digest or fingerprint when available
- layer role and scope used for compilation
- inclusion, rejection, or cycle/conflict reason

The effective config compiler should continue to fail closed on malformed Turtle, unsupported config terms, duplicate singletons, unsupported policy values, unsafe source coordinates, cycles, digest mismatch, and policy conflicts.

## Open Issues

- None currently. The first implementation slice should stay mesh-local, recursive, deterministic, and fail closed.

## Decisions

- Build on [[ont.completed.2026.2026-05-26_2205-artifact-resolution-spec-and-observation-cleanup]]; `sfcfg:ConfigSource` is the config-specific `sflo:ArtifactResolutionSpec` subclass for this runtime work.
- Resolve config source content at the scope and local/inheritable role determined by the attachment subject and generic attachment property that selected it.
- Do not let portable config source declarations expand network, repository, or host-local trust.
- Do not append `sflo:ArtifactResolutionObservation` records for ordinary config reads by default. Record provenance in the in-memory resolution trace; persist observations or `sfcfg:ConfigResolutionRecord` only when a deliberate diagnostic/export operation asks for it.
- Treat generic `sfcfg:hasConfigSource` as authoritative only on recognized config-bearing subjects, currently `sflo:SemanticMesh` and `sflo:Knop`.
- Allow conventional `_mesh/_config/config.ttl` and accepted resolved config-source graphs to attach additional mesh-local config sources with `<meshIri> sfcfg:hasConfigSource ?source`. Chaining is necessary for reusable config composition.
- Treat `sfcfg:hasConfigSource` or `sfcfg:hasInheritableConfigSource` on unrecognized subjects in authored config inputs as hard errors.
- Support workspace-bounded `sflo:targetLocalRelativePath` in the first slice only when the existing local path policy permits it; a config source must not grant itself the access needed to read itself.
- Use deterministic source processing for reproducibility and as a same-layer tie-breaker: earlier sources in the effective source chain win over later sources. The config graph that attaches a source is earlier than the source it attaches, so reusable config can supply defaults behind the referring config without overriding it. Existing policy resolution semantics still apply first: layer, selector specificity, and `sfcfg:policyPriority` resolve before source order.
- Keep Knop attachment-subject discovery out of the first mesh-local slice. A mesh-local attachment may resolve a Knop-owned `ConfigArtifact`, but Weave should not yet scan Knop config graphs or consume `sflo:Knop sfcfg:hasConfigSource` / `sflo:Knop sfcfg:hasInheritableConfigSource` as ordinary source-discovery entry points. If those Knop attachment statements appear in an active first-slice config input, reject them as unsupported until Knop source discovery is implemented.
- Runtime/cache code may compute internal source byte digests, parsed graph fingerprints, and dependency-set fingerprints for invalidation and diagnostics. Reserve the RDF predicate `sfcfg:hasConfigSourceFingerprint` for persisted or exported `sfcfg:ConfigResolutionRecord` output until its exact persisted semantics are settled.

## Contract Changes

### Weave Runtime

- Add config-source discovery to the effective-config load path after seed config parsing.
- Represent resolved config-source inputs as ordinary layer inputs carrying source/provenance metadata.
- Resolve supported `sfcfg:ConfigSource` shapes using the shared artifact-resolution semantics rather than custom config-only path parsing.
- Detect and reject cycles across config-source references.
- Verify `sflo:expectsContentDigest` when resolved bytes are available.
- Include accepted and rejected config sources in the in-memory resolution trace.

### SFLO / Framework

- No new ontology terms are expected for the first implementation because [[ont.completed.2026.2026-05-26_2205-artifact-resolution-spec-and-observation-cleanup]] supplied the needed substrate.
- Update [[sf.spec.2026-05-25-config-behavior]] only if implementation decisions clarify discovery locations, source ordering, or the first supported source-coordinate subset.
- Add durable docs once user-visible config-source authoring is supported.

## Testing

- Mesh-local config source attached with `sfcfg:hasConfigSource` on a `sflo:SemanticMesh` resolves and affects history/resource-page policy exactly as inline mesh-local config would.
- The same reusable `sfcfg:ConfigArtifact` participates at different layers when attached through `hasConfigSource` on a mesh, `hasConfigSource` on a Knop, or `hasInheritableConfigSource` on a Knop.
- Mesh-local config-source chaining works when an accepted config source declares another `<meshIri> sfcfg:hasConfigSource ?source`.
- A `sfcfg:hasConfigSource` attachment on an unrecognized subject is rejected as a hard error.
- A `sflo:Knop sfcfg:hasConfigSource ?source` statement in a resolved mesh-local graph is rejected as unsupported until Knop source discovery is implemented.
- Cyclic config-source references fail closed and report the cycle.
- Digest mismatch on `sflo:expectsContentDigest` fails closed.
- Unsupported remote `sflo:targetAccessUrl` fails closed under the default resolver policy.
- Workspace/local path config sources require the same local path policy as other source resolution and cannot grant themselves access.
- Multiple same-layer sources with conflicting singleton settings or policy bindings are resolved by effective source order after layer, selector specificity, and `sfcfg:policyPriority`; unresolved ties within the same effective source order fail closed with diagnostics that include participating source identities.
- Command overrides still beat mesh-local config-source values.
- Conventional `_mesh/_config/config.ttl` behavior remains unchanged when no config-source attachments are present.
- Ordinary config-source reads do not write `sfcfg:ResolvedConfig`, `sfcfg:ConfigResolutionRecord`, or `sflo:ArtifactResolutionObservation` records to the mesh.

## Non-Goals

- Do not implement general remote config fetching in the first slice.
- Do not use config sources to add host-local grants or broader path access.
- Do not persist `sfcfg:ResolvedConfig`, `sfcfg:ConfigResolutionRecord`, or `sflo:ArtifactResolutionObservation` by default.
- Do not add config-editing CLI commands here.
- Do not redesign artifact-resolution vocabulary in this ticket.
- Do not implement full Knop inheritance if mesh-local source discovery is enough to unblock the next runnable slice.

## Implementation Plan

- [ ] Verify [[ont.completed.2026.2026-05-26_2205-artifact-resolution-spec-and-observation-cleanup]] remains reflected in active ontology, SHACL, runtime, docs, and fixtures before implementing source discovery.
- [x] Inventory the current effective-config loader and identify the smallest layer-input abstraction needed for resolved config sources.
- [x] Confirm the first supported config-source coordinate subset and update [[sf.spec.2026-05-25-config-behavior]] if the spec needs that first-slice boundary.
- [x] Add config-source attachment discovery for mesh-local attachments from the conventional mesh config graph and/or mesh metadata.
- [x] Add recursive mesh-local attachment discovery from accepted resolved config-source graphs.
- [x] Implement safe config-source byte resolution for the first supported coordinate subset.
- [x] Parse resolved bytes as Turtle with an appropriate base IRI and compile them into the attachment property's layer.
- [x] Add cycle detection and deterministic source processing.
- [x] Add digest verification for `sflo:expectsContentDigest`.
- [x] Add trace metadata for accepted/resolved config sources.
- [x] Add focused unit tests for source discovery, source resolution, layer participation, cycle rejection, digest mismatch, and unsupported remote/path cases.
- [x] Add an integration test showing existing-mesh commands honoring a policy supplied through `sfcfg:hasConfigSource` on the mesh subject.
- [ ] Update CLI/user docs only after the authoring pattern is stable enough to recommend.
- [x] Run focused Weave tests plus `deno task check` and `deno task lint` if runtime structure changes broadly.
