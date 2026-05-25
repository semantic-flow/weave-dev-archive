---
id: qv49c7mz2t8r6apb1nyk5h3
title: 2026 05 24_1256 Artifact Resolution Observations
desc: ''
updated: 1779666744662
created: 1779652560000
---

## Goals

- Make `ArtifactResolutionTarget` the shared requested-coordinate / resolution-policy relator for application concerns that resolve bytes.
- Add explicit application-concern subclasses for source roles that should not be represented as anonymous generic resolution targets: `ExtractionSource`, `ReferenceSource`, `ResourcePageSource`, `ImportSource`, and `IntegrationSource`.
- Separate requested coordinates and expectations from observed resolution evidence by introducing an `ArtifactResolutionObservation` relator.
- Preserve `expectsContentDigest` as requested/expected evidence on `ArtifactResolutionTarget`, while moving observed digest, observed state, observed file, observed path, timestamp, and observer information onto observations.
- Keep observations optional and intentionally recorded. Runtime reads such as page rendering should not mutate mesh RDF merely because bytes were resolved.
- Provide a clean migration path from the current extraction-scoped `hasObservedSourceState`, `hasObservedSourceManifestation`, `hasObservedSourceLocatedFile`, `observedSourceLocalRelativePath`, and `observedSourceDigest` terms.
- Avoid using import or ReferenceLink implementation notes as the hidden home for this cross-cutting ontology decision.

## Summary

The current core ontology has a useful `ArtifactResolutionTarget` relator for requested coordinates and resolution policy, but its observed evidence terms were added for `ExtractionSource` first and therefore live directly on `ExtractionSource`:

```turtle
<#extraction-source> a sflo:ExtractionSource ;
  sflo:hasTargetArtifact <alice/bio> ;
  sflo:hasRequestedTargetState <alice/bio/_history001/_s0002> ;
  sflo:hasObservedSourceState <alice/bio/_history001/_s0002> ;
  sflo:hasObservedSourceLocatedFile <alice/bio/_history001/_s0002/ttl/alice-bio.ttl> ;
  sflo:observedSourceDigest "sha256:..." .
```

That shape becomes awkward once `ImportSource`, `IntegrationSource`, and `ReferenceSource` also need observed evidence. It also mixes two different lifecycles:

- requested coordinates and expectations: what the operation or policy asks to resolve
- observations: what a specific resolution event actually saw, when, and by whom

The preferred direction is:

```turtle
<#import-source> a sflo:ImportSource ;
  sflo:targetAccessUrl "https://raw.githubusercontent.com/example/data.ttl"^^xsd:anyURI ;
  sflo:expectsContentDigest "sha256:expected..." ;
  sflo:hasResolutionObservation <#import-source-observation-001> .

<#import-source-observation-001> a sflo:ArtifactResolutionObservation ;
  sflo:observedAt "2026-05-24T19:56:00Z"^^xsd:dateTime ;
  sflo:observedBy <https://example.org/agents/weave-cli> ;
  sflo:observedContentDigest "sha256:observed..." .
```

For artifact/state resolution:

```turtle
<#extraction-source> a sflo:ExtractionSource ;
  sflo:hasTargetArtifact <alice/bio> ;
  sflo:hasRequestedTargetState <alice/bio/_history001/_s0002> ;
  sflo:hasResolutionObservation <#extraction-source-observation-001> .

<#extraction-source-observation-001> a sflo:ArtifactResolutionObservation ;
  sflo:hasObservedTargetState <alice/bio/_history001/_s0002> ;
  sflo:hasObservedTargetManifestation <alice/bio/_history001/_s0002/ttl> ;
  sflo:hasObservedTargetLocatedFile <alice/bio/_history001/_s0002/ttl/alice-bio.ttl> ;
  sflo:observedContentDigest "sha256:..." .
```

The relationship between requested coordinates and observations should live on the requested-coordinate relator with `hasResolutionObservation`. A separate global observation registry is not needed in the first pass. If later audit or deduplication pressure appears, an index can be added without changing what an observation means.

## Discussion

### Why not just broaden `observedSourceDigest`

Broadening only `observedSourceDigest` would fix the immediate import issue, but it would leave the model half-extraction-specific and half-generic:

- `observedAt` already has domain `ArtifactResolutionTarget`.
- `expectsContentDigest` already has domain `ArtifactResolutionTarget`.
- state, manifestation, located-file, local-path, and digest observation terms currently have domain `ExtractionSource`.

That asymmetry would keep producing questions every time a new `ArtifactResolutionTarget` subclass needs evidence. If we are accepting pre-v1 churn, this is the better moment to split observations properly.

### Requested coordinates should stay stable

`ArtifactResolutionTarget` subclasses should be treated as requested-coordinate / resolution-policy relators. They can be durable and usually immutable in the sense that changing from one URL, repository path, requested state, or expected digest to another is a meaningful source-binding change.

Observation records are different. They are appendable event/evidence records. Repeated import, repeated verification, or repeated extraction can produce multiple observations for the same requested coordinates.

This matters for `import --replace-working`: the `ImportSource` may still describe the same URL and expected digest policy, while a new observation records what the repeated acquisition saw. If the request coordinates change materially, minting a new `ImportSource` or replacing the existing source binding is cleaner than rewriting old observation evidence.

### Observation attachment point

The first-pass relationship should be:

```turtle
sflo:hasResolutionObservation
  rdfs:domain sflo:ArtifactResolutionTarget ;
  rdfs:range sflo:ArtifactResolutionObservation .
```

That keeps observations discoverable from the source binding they support. It also lets `ReferenceSource`, `ImportSource`, `IntegrationSource`, `ExtractionSource`, and `ResourcePageSource` all share the same evidence model.

Do not put observations directly in `KnopSourceRegistry` as free-floating records in the first pass. They may be serialized in the same RDF file as the owning source binding, but semantically they should hang from the relevant `ArtifactResolutionTarget`.

### Observation vocabulary

The likely first-pass class and properties are:

- `ArtifactResolutionObservation`
- `hasResolutionObservation`
- `hasObservedTargetState`
- `hasObservedTargetManifestation`
- `hasObservedTargetLocatedFile`
- `observedTargetLocalRelativePath`
- `observedContentDigest`
- `observedAt`
- `observedBy`

`observedAt` should move from `ArtifactResolutionTarget` to `ArtifactResolutionObservation`.

`observedBy` can use `dcterms:Agent` as its range for consistency with the existing `verifiedBy` pattern. A future provenance pass may choose richer PROV-O relations or distinguish human operator, runtime software, and host process. That is not needed for the first pass.

### Names

Use "target" in the observation terms because the observation is about what an `ArtifactResolutionTarget` resolved. This is different from `ReferenceLink` terminology, where "target" was confusing because it sounded like the object of the reference relation.

Recommended replacements:

- `hasObservedSourceState` -> `hasObservedTargetState`
- `hasObservedSourceManifestation` -> `hasObservedTargetManifestation`
- `hasObservedSourceLocatedFile` -> `hasObservedTargetLocatedFile`
- `observedSourceLocalRelativePath` -> `observedTargetLocalRelativePath`
- `observedSourceDigest` -> `observedContentDigest`

Because this is pre-v1, do not add long-lived compatibility aliases unless implementation sequencing absolutely requires a short transitional period.

### Application concerns

`ArtifactResolutionTarget` should remain abstract/generic. Concrete application concerns should use subclasses where the source binding has domain meaning:

- `ExtractionSource`: RDF bytes used to ground or extract a Knop-managed resource.
- `ReferenceSource`: RDF bytes used by a curated `ReferenceLink`.
- `ResourcePageSource`: bytes used for a ResourcePage region.
- `ImportSource`: outside/current source bytes actively acquired and copied into a governed working surface.
- `IntegrationSource`: existing source bytes registered where they already live.

The ontology can still allow a plain `ArtifactResolutionTarget` where a truly generic binding is needed, but new Weave operations should prefer the narrower class when the concern is known.

### Serialization

`ExtractionSource`, `ImportSource`, and `IntegrationSource` observations should normally live in the Knop source registry RDF file beside the source binding:

- `D/_knop/_sources/sources.ttl`

`ReferenceSource` observations, if explicitly recorded, should normally live in the `ReferenceCatalog` beside the `ReferenceLink` and `ReferenceSource`:

- `D/_knop/_references/references.ttl`

`ResourcePageSource` observations should not be recorded by ordinary page rendering. A future explicit page-source verification command could record observations in the page definition or a source registry, but that is out of scope for this first ontology cleanup.

### Do not over-record observations

The existence of `ArtifactResolutionObservation` should not imply that every resolver call mutates RDF. Persist observations only when the operation contract says provenance/evidence is part of the output:

- import acquisition
- extraction grounding
- explicit integrate/source verification when requested
- explicit reference verification when requested

Normal `weave`, `generate`, and page rendering should not append observation records merely by reading source bytes.

## Open Issues

- Should there be a `latestResolutionObservation` convenience pointer, or should consumers sort `hasResolutionObservation` values by `observedAt` when they need the latest evidence? Recommendation: omit it initially.
- Should `observedBy` distinguish human operator, runtime software, and host process? Recommendation: use one generic property first and revisit if provenance queries need more.
- Should `observedContentDigest` allow multiple algorithms on the same observation? Recommendation: allow multiple digest literals unless SHACL pressure argues for one.
- Should observations include observed byte size, media type, or content type? Recommendation: defer until import/content-kind work needs it.
- Should a failed resolution have an observation record? Recommendation: not in the first pass; command logs can report failures, while successful observations record evidence that became part of the mesh.

## Decisions

- Create `ArtifactResolutionObservation` as the relator for observed resolution evidence.
- Link observations from `ArtifactResolutionTarget` with `hasResolutionObservation`.
- Move `observedAt` semantics to the observation relator rather than the requested-coordinate relator.
- Add `observedBy` on observations.
- Rename observed source terms to observed target/content terms and scope them to `ArtifactResolutionObservation`.
- Keep `expectsContentDigest` on `ArtifactResolutionTarget`.
- Treat observations as optional, appendable evidence records.
- Do not make ordinary read/render operations record observations by default.
- Add `ReferenceSource`, `ImportSource`, and `IntegrationSource` as `ArtifactResolutionTarget` subclasses; keep existing `ExtractionSource` and `ResourcePageSource` aligned with the same pattern.
- Track ReferenceLink-specific behavior in [[wa.task.2026.2026-05-22_1128-referencelink-clarification]].
- Track import command behavior in [[wa.task.2026.2026-05-21_0907-import]].
- Track integrate source-binding migration in [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]]; `IntegrationSource` should be added here, but rewriting all integrate output belongs there unless the same implementation pass naturally touches it.
- Track cross-cutting subclass follow-through in [[wa.task.2026.2026-05-24_1648-ArtifactResolutionTarget-subclass-cleanup]].

## Implementation Progress

- SFLO ontology now defines `ArtifactResolutionObservation`, `hasResolutionObservation`, generic observed target/content evidence terms, `observedBy`, and the `ReferenceSource` / `ImportSource` / `IntegrationSource` subclasses.
- SFLO SHACL now validates `ReferenceLink` -> exactly one `ReferenceSource`, `ReferenceSource` RDF target artifacts, and shared `ArtifactResolutionObservation` evidence.
- Weave now renders `ReferenceLink` catalogs through `hasReferenceSource` and `ReferenceSource`, and parses current ReferenceCatalog pages through that shape.
- Weave extraction/source-registry rendering now records observed evidence in an `ArtifactResolutionObservation` linked from the `ExtractionSource`.
- Weave source-registry parsing still returns the existing internal `observedSource*` model fields for callers, but those values now come from nested `ArtifactResolutionObservation` RDF.
- Weave fixture readers temporarily normalize older checked-in fixture-branch RDF from `referenceTarget` / direct observed-source properties into the new shape. This keeps tests meaningful while full fixture branch regeneration remains deferred.
- Remaining follow-through:
  - use [[wa.task.2026.2026-05-24_1648-ArtifactResolutionTarget-subclass-cleanup]] as the cross-cutting ledger for source-family cleanup
  - rename public/core/runtime `knop add-reference` request terminology from reference target to reference source
  - migrate integrate output to `IntegrationSource` (done in [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]])
  - implement import provenance as `ImportSource` plus `ArtifactResolutionObservation`
  - update Semantic Flow Framework API/spec/conformance manifests to stop asserting retired ReferenceLink and observed-source predicates; defer this until import provenance lands so fixture mesh regeneration can happen in one pass after both source-family migrations are available
  - regenerate fixture branches from scratch after manifests are updated

## Contract Changes

- SFLO core ontology:
  - Add `ArtifactResolutionObservation`.
  - Add `hasResolutionObservation` from `ArtifactResolutionTarget` to `ArtifactResolutionObservation`.
  - Add `observedBy`.
  - Move or replace `observedAt` so it applies to `ArtifactResolutionObservation`.
  - Replace extraction-scoped observed terms with observation-scoped generic terms: `hasObservedTargetState`, `hasObservedTargetManifestation`, `hasObservedTargetLocatedFile`, `observedTargetLocalRelativePath`, and `observedContentDigest`.
  - Add `ReferenceSource`, `ImportSource`, and `IntegrationSource` as `ArtifactResolutionTarget` subclasses.
- SFLO SHACL:
  - Validate `hasResolutionObservation` objects as `ArtifactResolutionObservation`.
  - Validate observation state, manifestation, located-file, path, digest, timestamp, and observer fields.
  - Move existing extraction-source observed-evidence checks to the observation shape.
  - Keep `ExtractionSource` requirements for the source artifact as RDF-specific extraction policy, not generic observation policy.
- Semantic Flow Framework:
  - Update behavior specs and conformance manifests that currently assert extraction-scoped observed terms after integrate source binding and import provenance both land.
  - Add examples or assertions for import observations when import fixtures land.
- Weave:
  - Update extraction source rendering and parsing to use `ArtifactResolutionObservation`.
  - Update source-registry parsing APIs so callers can still retrieve observed evidence without caring about the RDF nesting shape.
  - Next, update integrate source-binding output to use `IntegrationSource` and observations where observation evidence is intentionally recorded.
  - When import lands, record import acquisition evidence as an observation linked from `ImportSource`.

## Testing

- Ontology/SHACL validation for a valid `ArtifactResolutionTarget` with one observation.
- Ontology/SHACL validation for invalid observation state, manifestation, located-file, timestamp, and digest shapes.
- Weave unit tests for rendering extraction observations in the new nested shape.
- Weave parser regression tests that recover observed evidence from nested observations.
- Conformance manifest updates for existing extraction-source observations.
- Import tests should verify that observed digest evidence is recorded on an observation linked from `ImportSource`.
- Integrate migration tests should be added in a later integrate task, unless this ontology cleanup directly touches integrate output.

## Non-Goals

- Do not implement `weave import`.
- Do not refactor all integrate output in this ontology task.
- Do not implement ReferenceLink RDF fact-selection for ResourcePages.
- Do not implement a global observation registry.
- Do not add a full PROV-O event model.
- Do not make ordinary page generation or source reads mutate RDF by recording observations.
- Do not add content-kind/media-type modeling beyond digest and existing target coordinates.
- Do not rename this task note to a completed note unless explicitly requested.

## Implementation Plan

- [x] Re-read [[ont.summary.core]], [[ont.reference-links]], [[ont.decision-log]], [[wd.general-guidance]], and this note before editing.
- [x] Update SFLO core ontology with `ArtifactResolutionObservation`, `hasResolutionObservation`, generic observed target/content properties, and the missing source subclasses.
- [x] Update SFLO SHACL for observations and source subclass constraints.
- [x] Update SFLO notes and decision log.
- [d] Update Semantic Flow Framework specs/manifests for extraction observation shape. Deferred until import provenance lands, so fixture meshes can be regenerated in one pass after both source-family migrations are available.
- [x] Update Weave extraction rendering/parsing and focused tests.
- [x] Update ReferenceLink and import task notes if implementation details change during the ontology pass.
- [x] Coordinate with [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]] if the implementation pass does not update integrate output to `IntegrationSource`.
- [x] Run relevant ontology validation and Weave focused tests.
- [x] Provide commit messages per touched repo.
