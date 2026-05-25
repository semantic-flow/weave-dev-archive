---
id: ghw2c6oiej0ey3ugiqzr62
title: 2026 05 24_1301 Integrate Source Binding Update
desc: ''
updated: 1779652863000
created: 1779652863000
---

## Goals

- Update `weave integrate` source-binding output to use `sflo:IntegrationSource` once the shared vocabulary in [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] has landed.
- Preserve the product distinction between `integrate` and `import`: integrate registers existing source bytes where they live; import actively acquires bytes and copies them into a governed working surface.
- Align integrate source bindings with the same `ArtifactResolutionTarget` family used by `ExtractionSource`, `ReferenceSource`, `ResourcePageSource`, and `ImportSource`.
- Move intentionally recorded integrate evidence to `sflo:ArtifactResolutionObservation` instead of placing observed evidence directly on requested-coordinate relators.
- Keep ambient remote fetching out of integrate until a concrete runtime-resolution policy task explicitly enables it.
- Land this migration before implementing `weave import` source-registry output and before regenerating Semantic Flow Framework fixture meshes.

## Summary

`weave integrate` should remain the command for registering an existing source surface without copying it into the mesh first. Today the model has been drifting toward generic `ArtifactResolutionTarget` source bindings. That works mechanically, but it hides an important application concern: these are integration bindings, not import acquisition records, reference links, page-source declarations, or extraction grounding.

After [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] lands, integrate should emit source-registry bindings shaped like:

```turtle
<#integration-source-001> a sflo:IntegrationSource ;
  sflo:targetLocalRelativePath "src/alice-bio.ttl" ;
  sflo:hasResolutionObservation <#integration-source-001-observation-001> .

<#integration-source-001-observation-001> a sflo:ArtifactResolutionObservation ;
  sflo:observedAt "2026-05-24T20:01:03Z"^^xsd:dateTime ;
  sflo:observedContentDigest "sha256:..." .
```

Observation records should only be written when integrate intentionally verifies or records evidence as part of its operation. Ordinary later reads should not append observations.

## Decisions

- Use `sflo:IntegrationSource` for integrate source-registry bindings.
- Keep `sflo:IntegrationSource` as an `sflo:ArtifactResolutionTarget` subclass.
- Use the shared target/resolution vocabulary for local path, repository, floating locator, artifact, state, digest expectation, resolution mode, and fallback policy.
- Use `sflo:ArtifactResolutionObservation` for observed digest, observed local path/file/state, timestamp, and observer evidence.
- Keep `sflo:ImportSource` separate: import copies bytes into a governed working surface; integrate records existing bytes where they already live.
- Do not make integrate a remote-fetch command in this task.
- Current sequencing preference: update integrate source binding first, then implement import provenance, then regenerate Semantic Flow Framework fixtures/spec manifests for both changes together.

## Implementation Progress

- Weave integrate source-registry rendering now emits `sflo:IntegrationSource` instead of anonymous `sflo:ArtifactResolutionTarget` records for integrate bindings.
- Repository-backed integrate now attaches the computed local byte digest as an `sflo:ArtifactResolutionObservation` linked from the `IntegrationSource`.
- Working-only and floating repository integrate bindings remain observation-free unless the operation intentionally records digest/evidence.
- Source-registry parsing now exposes `listIntegrationSourceInventoryStates` so callers can retrieve integration source coordinates, repository locators, expected digest, and linked observation evidence without inspecting raw RDF class names.
- CLI/runtime docs now describe integrate source bindings as `IntegrationSource` records and explain repository-backed digest observations.
- Semantic Flow Framework fixture/spec manifest updates remain deferred until import provenance lands, so fixture mesh regeneration can absorb both source-family changes in one pass.

## ArtifactResolutionTarget Follow-Through

Broader `ArtifactResolutionTarget` subclass cleanup is tracked in [[wa.task.2026.2026-05-24_1648-ArtifactResolutionTarget-subclass-cleanup]]. This task should stay focused on `IntegrationSource` and source-registry plumbing for `weave integrate`.

The shared cleanup that belongs here is rendering, parsing, diagnostics, and tests for integrate bindings. This task should stop treating integrate bindings as anonymous generic `ArtifactResolutionTarget` records where the operation knows they are integration bindings.

## Contract Changes

- Weave:
  - Update integrate planning/rendering to create `sflo:IntegrationSource` source bindings in `_knop/_sources/sources.ttl`.
  - Update source-registry parser APIs so existing callers can retrieve integration source coordinates without depending on raw RDF class names.
  - Update diagnostics and docs to say "integration source" where the command is describing existing-source registration.
- SFLO dependency:
  - Depends on `sflo:IntegrationSource`, `sflo:ArtifactResolutionObservation`, and `sflo:hasResolutionObservation` from [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]].
- Semantic Flow Framework:
  - Update any integrate-related behavior specs or conformance manifests that assert the old source-binding class or direct observed evidence shape.

## Testing

- Unit-test integrate planning for local, sidecar, and branch-based source bindings typed as `sflo:IntegrationSource`.
- Unit-test source-registry parsing for `sflo:IntegrationSource` records with and without linked observations.
- Regression-test that `weave integrate` does not copy source bytes into a governed working file; that remains import's job.
- Regression-test that `weave integrate` does not fetch remote URLs merely because a locator shape can represent them.
- Run focused integrate/source-registry tests and the normal Weave checks for touched code.

## Non-Goals

- Do not implement `weave import`.
- Do not implement ambient remote fetching for integrate.
- Do not redesign local path access grants beyond what is needed for the new class shape.
- Do not make every later source read append an observation.
- Do not implement ResourcePage source selection or ReferenceLink page-fact selection.
- Do not use this task to finish all `ArtifactResolutionTarget` subclass follow-through.
- Do not rename this task note to a completed note unless explicitly requested.

## Implementation Plan

- [x] Wait for or coordinate with [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] so `sflo:IntegrationSource` and observation vocabulary exist.
- [x] Re-read [[wd.general-guidance]], [[wu.repository-options]], relevant integrate docs, and this note before editing.
- [x] Audit current source-registry rendering/parsing for places that treat integrate bindings as generic `ArtifactResolutionTarget` records.
- [x] Update integrate source-registry rendering to emit `sflo:IntegrationSource`.
- [x] Update source-registry parsing to handle the new class and linked observations.
- [x] Update user/developer docs that describe integrate source bindings.
- [d] Update framework specs/manifests if they assert integrate source-binding RDF. Deferred until import provenance lands and fixture meshes regenerate in one pass.
- [x] Add focused tests for integrate planning, parsing, and no-fetch behavior.
- [x] Run relevant checks.
- [x] Provide a commit message with bullets for ontology dependency, Weave integrate behavior, docs, and tests.
