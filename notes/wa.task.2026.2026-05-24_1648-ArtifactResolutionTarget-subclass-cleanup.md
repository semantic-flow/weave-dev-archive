---
id: skfj38zp53nwu4ecskggesh
title: 2026-05-24_1648-ArtifactResolutionTarget-subclass-cleanup
desc: ''
updated: 1779939408168
created: 1779666550543
---

## Goals

- Capture the cross-cutting follow-through around `sflo:ArtifactResolutionTarget` subclasses without overloading the concrete integrate or import implementation tasks.
- Keep each application concern explicit: `ExtractionSource`, `ReferenceSource`, `ResourcePageSource`, `IntegrationSource`, and `ImportSource` should not collapse back into anonymous generic resolution targets.
- Align Weave planner/runtime/parser terminology with the ontology split between application-concern sources and the generic `target*` byte-resolution vocabulary.
- Remove temporary compatibility bridges after the Semantic Flow Framework fixture meshes and conformance manifests regenerate.
- Preserve the sequencing decision: integrate source binding first, import provenance second, framework fixture regeneration after both.

## Summary

The ontology cleanup in [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] introduced the shared pattern:

```turtle
<#some-source> a sflo:IntegrationSource ;
  sflo:targetLocalRelativePath "src/example.ttl" ;
  sflo:hasResolutionObservation <#some-source-observation-001> .

<#some-source-observation-001> a sflo:ArtifactResolutionObservation ;
  sflo:observedContentDigest "sha256:..." .
```

That model deliberately separates:

- application concern: `ExtractionSource`, `ReferenceSource`, `ResourcePageSource`, `IntegrationSource`, or `ImportSource`
- requested coordinates and policy: inherited from `ArtifactResolutionTarget`
- observed evidence: carried by `ArtifactResolutionObservation`

The first implementation pass landed the ontology and moved extraction/reference rendering far enough for Weave tests to keep passing, but several edges remain intentionally deferred. This note is the coordination ledger for those edges.

## Discussion

### What belongs here

This task should track cross-cutting cleanup, naming consistency, parser/API shape, temporary compatibility removal, and sequencing. It should not be the task that implements all concrete source workflows.

Concrete implementation homes:

- `IntegrationSource`: [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]]
- `ImportSource`: [[wa.completed.2026.2026-05-21_0907-import]]
- `ReferenceSource`: [[wa.task.2026.2026-05-22_1128-referencelink-clarification]]
- observations and core vocabulary: [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]]

This task should answer the shared questions those concrete tasks should not each answer differently.

### Subclass responsibilities

`ArtifactResolutionTarget` should remain the generic policy-bearing relator for resolving bytes. New Weave output should use a narrower subclass whenever the application concern is known.

- `ExtractionSource`: RDF bytes used to ground or extract a Knop-managed resource. Already moved to nested observations in Weave, but internal parser field names still expose `observedSource*`.
- `ReferenceSource`: RDF data used by a `ReferenceLink`. Live RDF shape now uses `ReferenceLink` -> `hasReferenceSource` -> `ReferenceSource`, but user-facing/core request names still need to move away from "reference target" where they mean source.
- `ResourcePageSource`: bytes used for a resource-page region. This is the likely home for non-RDF page prose when the bytes are already governed or resolved locally; it is not a substitute for import provenance.
- `IntegrationSource`: existing source bytes registered where they already live by `weave integrate`. This has landed and should be the pattern import follows.
- `ImportSource`: outside/current source bytes actively acquired and copied into a governed working surface by `weave import`. This should follow the integrate migration so both share source-registry plumbing.

### Source registry shape

`sflo:KnopSourceRegistry` currently links source bindings with `sflo:hasSourceBinding` whose range is `ArtifactResolutionTarget`. That gives useful flexibility, but Weave should not emit a bare `ArtifactResolutionTarget` when it knows a binding is extraction, integration, or import provenance.

Open design pressure:

- Should SHACL warn or reject a `KnopSourceRegistry` `hasSourceBinding` object that is only typed `ArtifactResolutionTarget`?
- Should `ExtractionSource` also be linked through `hasSourceBinding` everywhere, or should `hasExtractionSource` remain a separate convenience relation?
- Should parser APIs expose a generic source-binding union with a `kind`, or keep operation-specific readers?
- Should internal parser fields rename from `observedSource*` to `observedTarget*` / `observedContentDigest`, or stay stable until a larger API break?

The practical first answer is conservative: implement `IntegrationSource` and `ImportSource` through `hasSourceBinding`, keep parsing tolerant enough for existing fixtures, and make stricter SHACL/API decisions after fixture regeneration.

### Terminology

Keep `target*` inside `ArtifactResolutionTarget` because there it means "target of byte resolution." Avoid `target` in public/API names where the user-facing thing is a source binding.

Examples:

- `ReferenceLink` should have a reference source, not a reference target.
- `ImportSource` records an acquisition source; `targetAccessUrl` is still acceptable inside the relator because it is the URL whose bytes the relator resolves.
- `IntegrationSource` records existing source bytes; `targetLocalRelativePath` is still acceptable inside the relator because it is the local target of resolution.

### Fixture sequencing

The fixture/conformance update should wait until import provenance lands. Integrate source binding has landed, so import is now the remaining source-family gate before framework fixture meshes should churn.

Until then, Weave may keep narrow fixture normalization for old checked-in fixture branches, but that bridge should be removed in the fixture-regeneration pass.

## Open Issues

- Should `KnopSourceRegistry` allow bare `ArtifactResolutionTarget` source bindings, or should concrete Weave output always use `ExtractionSource`, `IntegrationSource`, or `ImportSource`?
- Should `hasExtractionSource` be retained as a separate relation, or should extraction grounding be consistently modeled as `hasSourceBinding` to an `ExtractionSource`?
- Should shared parser/runtime models use `observedSource*` field names for compatibility or rename to observation-oriented names?
- Should `ResourcePageSource` get more explicit content-kind/media-type policy before non-RDF imported page prose relies on it heavily?
- Should Semantic Flow Framework conformance manifests assert each concrete source subclass, or assert only behavior-level outcomes and leave class-shape checks to ontology/SHACL tests?

## Decisions

- Use concrete subclasses for known Weave source roles rather than emitting bare `ArtifactResolutionTarget` records.
- Land `IntegrationSource` migration before `ImportSource` provenance; this ordering has been satisfied.
- Defer Semantic Flow Framework fixture mesh regeneration until import provenance lands.
- Keep `target*` property names inside `ArtifactResolutionTarget`; clean up confusing "target" names at the operation/API boundary.
- Keep temporary fixture normalization narrow and explicitly removable.

## Contract Changes

- Weave:
  - Audit source-registry rendering/parsing for generic `ArtifactResolutionTarget` assumptions.
  - Keep completed `weave integrate` output aligned with `sflo:IntegrationSource`.
  - Implement `weave import` provenance as `sflo:ImportSource` using the completed integrate source-registry pattern.
  - Rename public/core/runtime ReferenceLink request terminology from reference target to reference source where the RDF source is meant.
  - Decide whether to rename internal observed-evidence parser fields or keep compatibility wrappers.
- SFLO:
  - Consider SHACL guidance for `KnopSourceRegistry` `hasSourceBinding` values after concrete Weave output has moved to subclasses.
  - Consider whether `hasExtractionSource` remains separate from `hasSourceBinding`.
- Semantic Flow Framework:
  - Regenerate fixture meshes and conformance manifests after import lands.
  - Remove stale `referenceTarget`, `referenceTargetState`, `referenceUriLiteral`, and direct observed-source predicates from active fixture expectations.
- Documentation:
  - Update user/developer docs so `integrate`, `import`, `ReferenceLink`, and page-source wording uses source terminology consistently.

## Testing

- Focused Weave tests should assert concrete subclass output for extraction, integration, import, reference, and page-source paths as those paths land.
- Source-registry parser tests should cover linked `ArtifactResolutionObservation` records and missing-observation cases.
- SHACL guardrails should reject retired ReferenceLink predicates and validate `ReferenceLink` -> exactly one `ReferenceSource`.
- Fixture-regeneration tests should remove temporary normalization once new framework fixtures are canonical.
- Integration and import tests should include no-ambient-fetch regressions so `targetAccessUrl` and repository locators do not accidentally authorize page rendering or generate-time network access.

## Non-Goals

- Do not implement `weave integrate`; that landed in [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]].
- Do not implement `weave import`; use [[wa.completed.2026.2026-05-21_0907-import]].
- Do not redesign the full content-kind/media-type model for Markdown, HTML, images, or arbitrary binary payloads.
- Do not make `ReferenceLink` support non-RDF reference data in this cleanup.
- Do not make ordinary page rendering, source reads, or reference resolution append observations.
- Do not regenerate Semantic Flow Framework fixtures until import provenance lands.
- Do not rename this task note to a completed note unless explicitly requested.

## Implementation Plan

- [ ] Re-read [[wd.general-guidance]], [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]], [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]], [[wa.completed.2026.2026-05-21_0907-import]], and [[wa.task.2026.2026-05-22_1128-referencelink-clarification]] before editing code.
- [ ] Inventory Weave source-registry and ReferenceCatalog parser/rendering APIs that still treat known source concerns as generic `ArtifactResolutionTarget` records.
- [x] Land [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]].
- [ ] Land import provenance from [[wa.completed.2026.2026-05-21_0907-import]] using the settled source-registry shape.
- [ ] Finish ReferenceLink API/CLI terminology cleanup from [[wa.task.2026.2026-05-22_1128-referencelink-clarification]].
- [ ] Decide whether to rename internal observed-evidence parser fields or keep compatibility wrappers.
- [ ] Decide whether SHACL should constrain `KnopSourceRegistry` source bindings to concrete source subclasses.
- [ ] Regenerate Semantic Flow Framework fixture meshes and manifests after import lands.
- [ ] Remove temporary fixture normalization code once canonical fixtures use the new source-family shapes.
- [ ] Run relevant Weave, SFLO, and framework checks for the touched repos.
- [ ] Provide commit messages per repo.
