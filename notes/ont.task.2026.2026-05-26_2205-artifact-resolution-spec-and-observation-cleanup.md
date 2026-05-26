---
id: n99q1gmwn8q81efbdu85tuw
title: 2026 05 26_2205 Artifact Resolution Spec and Observation Cleanup
desc: ''
updated: 1779771923786
created: 1779771923786
---

## Goals

- Rename `sflo:ArtifactResolutionTarget` to `sflo:ArtifactResolutionSpec` so source subclasses no longer read as sources that are targets.
- Rename resolution-spec coordinate predicates to concise target-oriented names that stay distinct from config policy targeting.
- Replace enum-only fallback behavior with recursive fallback `ArtifactResolutionSpec` links.
- Simplify `sflo:ArtifactResolutionObservation` so observed coordinates are represented by an observed concrete `ArtifactResolutionSpec` rather than mirrored observation-only target predicates.
- Add `sfcfg:ConfigSource` as the config-side source/resolution-spec class for config bytes and `ConfigArtifact` resolution.
- Update SFLO SHACL, Weave runtime parsing/rendering, Semantic Flow Framework examples/conformance manifests, and docs that still use the old resolution-target or mirrored-observation shape.
- Keep this cleanup separate from the config policy-binding implementation while allowing the config task to depend on the new `ConfigSource` shape.

## Summary

[[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] introduced `ArtifactResolutionObservation`, source subclasses such as `ImportSource` and `IntegrationSource`, and nested observation records. That was useful, but the current vocabulary still has two conceptual snags:

- `ExtractionSource`, `ImportSource`, `IntegrationSource`, `ReferenceSource`, and `ResourcePageSource` are subclasses of `ArtifactResolutionTarget`, which reads backwards to humans and LLMs.
- Observation records mirror target coordinates with properties such as `hasObservedTargetState`, `hasObservedTargetManifestation`, and `hasObservedTargetLocatedFile`, instead of reusing the same expressive coordinate shape used by requested resolution specs.

The new direction is to rename the shared relator to `ArtifactResolutionSpec`. A spec describes target coordinates, source locators, expected digest checks, resolution mode, and fallback specs. Application-specific source classes are then kinds of resolution specs:

```turtle
<#import-source> a sflo:ImportSource ;
  sflo:targetAccessUrl "https://raw.githubusercontent.com/example/data.ttl"^^xsd:anyURI ;
  sflo:expectsContentDigest "sha256:expected..." ;
  sflo:hasResolutionObservation <#import-observation-001> .
```

An observation should record event metadata and byte-specific evidence, while pointing at the concrete spec that actually resolved:

```turtle
<#import-observation-001> a sflo:ArtifactResolutionObservation ;
  sflo:observedArtifactResolutionSpec [
    a sflo:ArtifactResolutionSpec ;
    sflo:targetLocatedFile <alice/data/_history001/_s0002/ttl/alice-data-v2.ttl> ;
    sflo:targetManifestation <alice/data/_history001/_s0002/ttl>
  ] ;
  sflo:observedContentDigest "sha256:observed..." ;
  sflo:observedAt "2026-05-26T22:05:00Z"^^xsd:dateTime .
```

Fallback should use the same spec model:

```turtle
<#requested-source> a sflo:ImportSource ;
  sflo:targetHistoricalState <alice/data/_history001/_s0003> ;
  sflo:hasFallbackArtifactResolutionSpec [
    a sflo:ArtifactResolutionSpec ;
    sflo:targetArtifactHistory <alice/data/_history001> ;
    sflo:hasArtifactResolutionMode sflo:artifactResolutionMode_latestState
  ] .
```

This keeps "resolution" as the name of the model, not the predicate family. Runtime code resolves specs, but the RDF says which target coordinates and fallback alternatives are allowed.

## Discussion

### Why ArtifactResolutionSpec

`ArtifactResolutionTarget` made sense when the node was read narrowly as the target coordinates for a resolver. It becomes misleading once concrete subclasses are named `ReferenceSource`, `ExtractionSource`, `ImportSource`, `IntegrationSource`, `ResourcePageSource`, and now `ConfigSource`.

`ArtifactResolutionSpec` is better because the node is a specification/relator, not the resolver itself and not merely the target resource. It can contain:

- target coordinates, such as artifact, history, state, manifestation, located file, URL, path, or repository locator
- requested mode, such as working bytes or latest state
- expected digest constraints
- fallback specs
- intentionally recorded observations

Avoid `ArtifactResolver`; that name should be reserved for code or an agent/process that performs resolution.

### Target predicate names

Keep "target" for spec coordinate predicates. The problem was not the word target; the problem was a source class inheriting from a class named target. Target-oriented predicates still read well on `ArtifactResolutionSpec`.

Preferred renames:

- `sflo:hasTargetArtifact` -> `sflo:targetArtifact`
- `sflo:hasTargetLocatedFile` -> `sflo:targetLocatedFile`
- `sflo:hasTargetDistribution` -> `sflo:targetManifestation`
- `sflo:hasRequestedTargetHistory` -> `sflo:targetArtifactHistory`
- `sflo:hasRequestedTargetState` -> `sflo:targetHistoricalState`
- `sflo:hasTargetRepositorySource` -> `sflo:targetRepositorySource`

Keep `sflo:targetAccessUrl` and `sflo:targetLocalRelativePath` unless a later naming pass chooses shorter path terminology. They already read correctly on `ArtifactResolutionSpec`.

Do not reuse these predicates for config policy targets. Policy targeting should use config-side predicates such as `sfcfg:targetsArtifact`, because policy targets answer "what governed thing does this policy apply to?" while resolution specs answer "what bytes/state/source should this operation resolve?"

### Fallback specs

The current `ArtifactResolutionFallbackPolicy` enum is too small for the model we need. It can say "accept latest in requested history," but it cannot express the full alternate coordinates, expected digest, repository locator, located file, or nested fallback behavior.

Prefer:

```turtle
sflo:hasFallbackArtifactResolutionSpec
  rdfs:domain sflo:ArtifactResolutionSpec ;
  rdfs:range sflo:ArtifactResolutionSpec .
```

No fallback spec means exact-only behavior unless a resolution mode has a well-defined default such as latest state within a targeted history. If multiple fallback specs are allowed, ordering must be explicit before runtime can use more than one. For the first pass, prefer zero or one fallback spec unless a clear ordered-list use case lands.

`ArtifactResolutionFallbackPolicy` can be removed or reduced to non-normative shorthand only if implementation needs a temporary bridge. Since this is pre-v1, prefer removing the enum-driven fallback model rather than keeping two ways to say the same thing.

### Observation shape

The completed observation task moved observed evidence off `ExtractionSource` and onto `ArtifactResolutionObservation`, but it still created mirrored observation predicates:

- `hasObservedTargetState`
- `hasObservedTargetManifestation`
- `hasObservedTargetLocatedFile`
- `observedTargetLocalRelativePath`

That mirrors part of `ArtifactResolutionTarget` and will keep growing as specs gain expressive power. Instead, an observation should point to an observed/concrete `ArtifactResolutionSpec`:

```turtle
sflo:observedArtifactResolutionSpec
  rdfs:domain sflo:ArtifactResolutionObservation ;
  rdfs:range sflo:ArtifactResolutionSpec .
```

The observation keeps event and byte-specific evidence:

- `observedContentDigest`
- `observedAt`
- `observedBy`
- possible future byte size or media/content type

The observed spec carries the concrete resolved coordinates. It may be anonymous when only serialized as nested evidence, or named when the exact observed spec is reusable or worth addressing directly.

### ConfigSource

The config ontology should add `sfcfg:ConfigSource` as a subclass of `sflo:ArtifactResolutionSpec`. It identifies config bytes or a `sfcfg:ConfigArtifact` to resolve and parse as authored config.

Attachment properties such as `sfcfg:hasMeshConfigSource`, `sfcfg:hasKnopLocalConfigSource`, and `sfcfg:hasKnopInheritableConfigSource` should range over `sfcfg:ConfigSource`, not generic `sflo:ArtifactResolutionSpec`.

Do not add `MeshConfigSource`, `KnopLocalConfigSource`, or similar subclasses unless different source classes need distinct validation. The attachment property already carries the role.

### Migration posture

This is pre-v1 vocabulary churn. Prefer direct replacement over long-lived compatibility aliases. Weave may use short-lived parser normalization for existing fixture branches if needed to keep tests meaningful during regeneration, but the ontology should not keep old names as normative authoring vocabulary.

## Open Issues

- Should `hasArtifactResolutionMode` be shortened to `artifactResolutionMode` or `resolutionMode` while this vocabulary is already changing? Recommendation: decide during ontology edit; `hasArtifactResolutionMode` is verbose but not conceptually wrong.
- Should `targetLocalRelativePath` remain as-is or become `targetLocalPath`? Recommendation: keep the current name unless the path model itself changes.
- Should `hasRepositorySourceFloatingLocator` be renamed in this task? It is related but not strictly a spec-coordinate predicate because its domain intentionally includes artifacts as well as specs. Recommendation: leave it unless implementation shows it causes the same confusion.
- Should multiple fallback specs be allowed? Recommendation: allow the ontology to express multiple only if ordering is also modeled; otherwise validate max one in SHACL for now.
- Should `observedArtifactResolutionSpec` be required for every observation? Recommendation: require it when the observation records concrete resolved coordinates; allow digest-only observations only if SHACL can distinguish that intentional shape.
- Should failed resolution attempts be modeled as observations? Recommendation: not in this pass; command/runtime logs can report failures, while observations remain successful evidence unless a later audit model needs failure records.

## Decisions

- Rename `sflo:ArtifactResolutionTarget` to `sflo:ArtifactResolutionSpec`.
- Treat `ArtifactResolutionSpec` as a relator/specification for target coordinates, resolution mode, expected digest constraints, fallback specs, and observations.
- Use concise target predicates on `ArtifactResolutionSpec`, not `resolves*` predicates.
- Use `targetManifestation` for ordinary Semantic Flow manifestation targeting instead of `targetDistribution`.
- Do not use spec target predicates for config policy targeting; config policy exact-target matching gets config-side properties such as `sfcfg:targetsArtifact`.
- Replace enum-only fallback behavior with recursive `sflo:hasFallbackArtifactResolutionSpec`.
- Simplify observations by linking to an observed concrete `ArtifactResolutionSpec` rather than mirroring target coordinate predicates on `ArtifactResolutionObservation`.
- Keep `observedContentDigest`, `observedAt`, and `observedBy` on `ArtifactResolutionObservation`.
- Add `sfcfg:ConfigSource` as a subclass of `sflo:ArtifactResolutionSpec`.
- Do not mutate [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] except to reference it from this newer cleanup. It remains useful as the completed intermediate design history.

## Contract Changes

### SFLO core ontology

- Rename class `sflo:ArtifactResolutionTarget` to `sflo:ArtifactResolutionSpec`.
- Update subclasses to inherit from `sflo:ArtifactResolutionSpec`:
  - `sflo:ReferenceSource`
  - `sflo:ExtractionSource`
  - `sflo:ImportSource`
  - `sflo:IntegrationSource`
  - `sflo:ResourcePageSource`
- Rename target coordinate properties:
  - `sflo:hasTargetArtifact` -> `sflo:targetArtifact`
  - `sflo:hasTargetLocatedFile` -> `sflo:targetLocatedFile`
  - `sflo:hasTargetDistribution` -> `sflo:targetManifestation`
  - `sflo:hasRequestedTargetHistory` -> `sflo:targetArtifactHistory`
  - `sflo:hasRequestedTargetState` -> `sflo:targetHistoricalState`
  - `sflo:hasTargetRepositorySource` -> `sflo:targetRepositorySource`
- Update property domains from `sflo:ArtifactResolutionTarget` to `sflo:ArtifactResolutionSpec`.
- Replace `sflo:hasArtifactResolutionFallbackPolicy` / `sflo:ArtifactResolutionFallbackPolicy` authoring semantics with `sflo:hasFallbackArtifactResolutionSpec`.
- Add `sflo:observedArtifactResolutionSpec` from `sflo:ArtifactResolutionObservation` to `sflo:ArtifactResolutionSpec`.
- Remove or retire mirrored observation-coordinate predicates:
  - `sflo:hasObservedTargetState`
  - `sflo:hasObservedTargetManifestation`
  - `sflo:hasObservedTargetLocatedFile`
  - `sflo:observedTargetLocalRelativePath`
- Update comments for repository locators, source classes, observations, and expected digest to use `ArtifactResolutionSpec`.

### SFLO config ontology

- Add `sfcfg:ConfigSource` as a subclass of `sflo:ArtifactResolutionSpec`.
- Change `sfcfg:hasConfigSource` and role-specific config source properties to range over `sfcfg:ConfigSource`.
- Update comments that mention `ArtifactResolutionTarget` or old target predicate names.
- Keep config policy targeting separate with config-side predicates, such as `sfcfg:targetsArtifact`, for exact policy targets.

### SFLO SHACL

- Rename shapes and path references from `ArtifactResolutionTarget` to `ArtifactResolutionSpec`.
- Validate the renamed target coordinate predicates.
- Validate `sflo:hasFallbackArtifactResolutionSpec` and guard against unbounded unordered fallback ambiguity in the first pass.
- Validate `sflo:observedArtifactResolutionSpec` on observations and remove old mirrored-observation-coordinate validation.
- Update source subclass constraints for `ReferenceSource`, `ExtractionSource`, `ImportSource`, `IntegrationSource`, `ResourcePageSource`, and `ConfigSource`.

### Weave

- Update RDF constants, parser paths, serializers, and test fixtures from `ArtifactResolutionTarget` / old target predicates to `ArtifactResolutionSpec` / new predicates.
- Update source registry rendering and parsing so observations expose an observed spec instead of mirrored observed target fields.
- Keep internal convenience accessors if useful, but make them derive values from the observed spec.
- Update fixture normalization only as a short-lived bridge for pre-regeneration fixture branches.
- Update code that reads or writes config source properties once `sfcfg:ConfigSource` lands.

### Semantic Flow Framework

- Update specs, examples, conformance manifests, and Alice fixture expectations that assert old source or observation vocabulary.
- Regenerate fixture branches when the corresponding Weave source rendering is updated.
- Update any explanatory docs that describe `ArtifactResolutionTarget`, mirrored observation fields, or enum-only fallback policy.

## Testing

- Ontology parse tests for the renamed `ArtifactResolutionSpec` class and renamed target predicates.
- SHACL tests for valid source subclasses using the new target coordinate properties.
- SHACL tests rejecting old mirrored observation-coordinate predicates if old vocabulary is removed from the ontology.
- SHACL tests for valid observations with `observedArtifactResolutionSpec`, digest, timestamp, and observer.
- SHACL tests for recursive fallback specs, including max-one or ordered fallback constraints if selected.
- Weave parser tests for source registry RDF using the new spec and observation shape.
- Weave serializer tests for extraction/import/integration source records that intentionally include observations.
- Weave fixture normalization tests if temporary old-vocabulary support remains during fixture regeneration.
- Framework conformance tests updated to the new vocabulary after fixture regeneration.
- Focused `deno task test` / `deno task check` in Weave for touched runtime code, plus repo-appropriate ontology validation.

## Non-Goals

- Do not implement `weave import` as part of this ontology cleanup.
- Do not redesign `RepositorySourceLocator` beyond property/domain/comment updates needed by the spec rename.
- Do not add a full PROV-O event model for resolution attempts.
- Do not make ordinary reads, page rendering, or config resolution append observations by default.
- Do not model failed resolution attempts unless a later audit/provenance task requires it.
- Do not keep long-lived compatibility aliases for pre-v1 vocabulary unless implementation sequencing forces a short transitional bridge.
- Do not collapse config policy targets into artifact resolution specs; policy targeting remains its own config vocabulary.
- Do not rename this task note to completed unless explicitly requested.

## Implementation Plan

- [ ] Update SFLO core ontology class, subclass, property, and comment names from `ArtifactResolutionTarget` to `ArtifactResolutionSpec`.
- [ ] Rename core target coordinate predicates and update all ontology comments/ranges/domains.
- [ ] Add `sflo:hasFallbackArtifactResolutionSpec` and remove or retire enum-only fallback policy authoring vocabulary.
- [ ] Add `sflo:observedArtifactResolutionSpec` and remove or retire mirrored observed target coordinate predicates.
- [ ] Update SFLO config ontology with `sfcfg:ConfigSource` and config-source property ranges.
- [ ] Add or update config policy exact-target predicates separately from resolution-spec predicates.
- [ ] Update SFLO SHACL shapes and tests for specs, source subclasses, fallback specs, config sources, and observations.
- [ ] Update Weave RDF constants, parser code, serializers, and focused tests for source registries and observations.
- [ ] Update Weave config-source parsing/rendering if the config policy task starts using `sfcfg:ConfigSource`.
- [ ] Update Semantic Flow Framework specs/examples/conformance manifests that assert old artifact-resolution vocabulary.
- [ ] Regenerate affected fixture branches after Weave emits the new shape.
- [ ] Update relevant docs/task notes to reference this task as superseding the intermediate observation vocabulary shape.
- [ ] Run focused checks for each touched repo and provide commit messages per repo.
