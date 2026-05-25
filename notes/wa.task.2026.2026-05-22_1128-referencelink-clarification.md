---
id: iu634gaso05bniod1z8j8hp
title: 2026 05 22_1128 ReferenceLink Clarification
desc: ''
updated: 1779647404677
created: 1779647404677
---

## Goals

- Clarify that `sflo:ReferenceLink` is for curated links to RDF reference data, not for arbitrary page content, Markdown prose, images, HTML snippets, or source/acquisition provenance.
- Replace the confusing `referenceTarget` / `referenceTargetState` terms with a model where `ReferenceLink` points to a `ReferenceSource`, and `ReferenceSource` reuses the shared `ArtifactResolutionTarget` vocabulary.
- Keep the useful general resolution machinery (`hasTargetArtifact`, `hasRequestedTargetState`, `hasTargetRepositorySource`, `targetAccessUrl`, digest/mode/fallback policy) without making `ReferenceLink` itself an operational resolution target.
- Preserve the conceptual split between `ReferenceCatalog`, `KnopSourceRegistry`, `ExtractionSource`, `ResourcePageSource`, `ImportSource`, `IntegrationSource`, and other source/provenance bindings.
- Avoid future ontology churn by making the RDF-only reference direction explicit before adding more fixture rungs, import examples, or reference-driven page rendering.
- Rename the local/API request terminology away from "target" where it names the reference source, while leaving `target*` vocabulary intact inside `ArtifactResolutionTarget` where it means "target of byte resolution."
- Coordinate shared `ArtifactResolutionTarget` subclass and observation vocabulary through [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] rather than making ReferenceLink carry the whole resolution-model cleanup.

## Summary

The current `ReferenceLink` model has the right broad shape but the wrong direct property names. `referenceLinkFor` correctly names the subject/referent of the link, while `referenceTarget` currently names the RDF source artifact used as supporting data. That makes the word "target" misleading at exactly the point where readers expect "target" to mean the object of the reference relation.

The intended model should be:

```turtle
<alice> sflo:hasReferenceLink <alice/_knop/_references#reference001> .

<alice/_knop/_references#reference001> a sflo:ReferenceLink ;
  sflo:referenceLinkFor <alice> ;
  sflo:hasReferenceRole sflo:referenceRole_canonical ;
  sflo:hasReferenceSource <alice/_knop/_references#reference001-source> .

<alice/_knop/_references#reference001-source> a sflo:ReferenceSource ;
  sflo:hasTargetArtifact <alice/bio> .
```

When a reference should pin the source to a settled state, the source relator should use the shared resolution vocabulary:

```turtle
<alice/_knop/_references#reference001-source> a sflo:ReferenceSource ;
  sflo:hasTargetArtifact <alice/bio> ;
  sflo:hasRequestedTargetState <alice/bio/_history001/_s0002> .
```

`ReferenceSource` should be a subclass of `ArtifactResolutionTarget`. That keeps one targeting/resolution vocabulary across application concerns while keeping the curated reference relation itself clean:

- `ExtractionSource` is the extraction/grounding application concern and already subclasses `ArtifactResolutionTarget`.
- `ResourcePageSource` is the page-composition application concern and already subclasses `ArtifactResolutionTarget`.
- `ReferenceSource` should be the reference-data application concern and should also subclass `ArtifactResolutionTarget`.
- `ImportSource` should be the explicit acquisition/materialization concern and should also subclass `ArtifactResolutionTarget`.
- `IntegrationSource` should be the existing-source registration concern for `weave integrate` and should also subclass `ArtifactResolutionTarget`.

`ReferenceLink` should remain the semantic relator. It should not itself become an `ArtifactResolutionTarget`.

The broader source-family and observation cleanup is tracked in [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]]. This task depends on that direction for `ReferenceSource`, but it should stay focused on ReferenceLink semantics, ReferenceCatalog serialization, API/CLI terminology, and fixture rewrites.

## Discussion

### Why `referenceTarget` is confusing

In the current model, `referenceLinkFor` is the actual subject of the relationship. For Alice Bio, the link is about `<alice>`, while `referenceTarget <alice/bio>` names the RDF artifact that supports or describes Alice.

That is backwards from how the word "target" reads in normal relation modeling. The target of "reference link for Alice" sounds like Alice, not the supporting source document. The term becomes especially confusing when a `ReferenceLink` is later used by page generation, because the thing currently called "target" is actually the source from which facts may be read.

The `target*` words are less problematic inside `ArtifactResolutionTarget`, because there they have a local technical meaning: the target of byte resolution. So the cleanup should not try to purge `target` globally. It should move `target` back behind the resolution relator boundary.

### RDF-only reference scope

The direction for this task is intentionally narrower than the most general possible reference model.

`ReferenceLink` should refer to RDF reference data: a governed `RdfDocument`, a `HistoricalState` / `LocatedFile` / manifestation of RDF bytes, or a future explicitly permitted RDF source resolved through repository or URL coordinates. It should not become the general way to attach non-RDF material to a ResourcePage.

Non-RDF content should use the right application concern:

- Markdown page prose: `ResourcePageDefinition` / `ResourcePageSource`, or a governed imported payload artifact used by page composition.
- Import/acquisition provenance: `KnopSourceRegistry` with an `ImportSource` source binding.
- Existing-source integration provenance: `KnopSourceRegistry` with an `IntegrationSource` source binding.
- Extraction grounding: `ExtractionSource`.
- RDF source references that describe or support a subject: `ReferenceLink` plus `ReferenceSource`.

This means Bob's Markdown bio should not be modeled as a `ReferenceLink`. If it is page prose, it belongs in page-source machinery. If it should become a governed artifact, import it as a payload such as `bob/bio` or `bob/page-main` and then point page composition at that artifact. Do not make `/bob` look like a Markdown bio payload just to avoid creating the right content boundary.

### No `ReferenceFormat` for this slice

There is real future value in distinguishing RDF, HTML/RDFa, HTML with JSON-LD, Markdown, images, and other media. But adding `ReferenceFormat` now would reopen the scope that this task is trying to close.

For this clarification, the useful rule is simpler:

- `ReferenceSource` is for RDF data.
- Runtime/page code should only treat a reference source as fact-producing RDF when the resolved target is an RDF artifact/source under the current ontology and SHACL rules.
- HTML-with-RDFa or HTML-with-JSON-LD can be revisited when there is an actual fixture or renderer use case.

So do not add `referenceFormat_rdf`, `referenceFormat_html`, `referenceFormat_markdown`, or similar vocabulary in this task.

### Keep `ReferenceRole`

Renaming `ReferenceRole` to `ReferenceAuthorityRole` would overstate what the current roles mean. `canonical`, `supplemental`, and `deprecated` describe how the reference source should be used relative to the subject. They are not all authority claims.

Keep `ReferenceRole` and the current role instances for this task. Do not add `referenceRole_pageSource`; page composition is not reference-link role semantics.

### Relationship to `ArtifactResolutionTarget`

`ArtifactResolutionTarget` is the shared policy-bearing relator for resolving bytes for an application concern. It already works well for:

- `ExtractionSource`: source RDF used to ground an extracted resource.
- `ResourcePageSource`: bytes used for a page region.
- `ImportSource`: source bytes explicitly acquired and copied into a governed local working surface.
- `IntegrationSource`: existing source bytes registered where they already live.

`ReferenceSource` should join that family. `ImportSource` and `IntegrationSource` should join it as source-registry concerns. All of them should reuse:

- `hasTargetArtifact`
- `hasTargetLocatedFile`
- `hasTargetDistribution`
- `hasRequestedTargetHistory`
- `hasRequestedTargetState`
- `hasArtifactResolutionMode`
- `hasArtifactResolutionFallbackPolicy`
- `targetAccessUrl`
- `hasTargetRepositorySource`
- `hasRepositorySourceFloatingLocator`
- `expectsContentDigest`
- `hasResolutionObservation`, when reference-source verification is intentionally recorded

The first implementation does not need to support every resolver shape at runtime. It should at least cover the existing fixture shape: in-mesh `hasTargetArtifact`, optionally pinned with `hasRequestedTargetState`.

Observation details such as observed digest, observed state, timestamp, and observer belong to `ArtifactResolutionObservation` in [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]], not directly on `ReferenceLink` or `ReferenceSource`.

### Relationship to `RepositorySourceLocator`

`RepositorySourceLocator` and `RepositorySourceFloatingLocator` are locator/evidence objects, not application concerns. They fit under any `ArtifactResolutionTarget` subclass that needs repository coordinates.

For `ReferenceSource`, that means a future reference can identify an RDF source in a repository without inventing reference-specific repository vocabulary:

```turtle
<thing/_knop/_references#reference001-source> a sflo:ReferenceSource ;
  sflo:hasTargetRepositorySource <thing/_knop/_references#reference001-repo-source> ;
  sflo:expectsContentDigest "sha256:..." .

<thing/_knop/_references#reference001-repo-source> a sflo:RepositorySourceLocator ;
  sflo:sourceRepositoryUrl "https://github.com/example/data"^^xsd:anyURI ;
  sflo:sourceRepositoryRef "main" ;
  sflo:sourceRepositoryCommit "..." ;
  sflo:sourceRepositoryPath "rdf/thing.ttl" .
```

This remains resolution metadata. It does not authorize ambient network access. Runtime access policy still belongs to the operational config line described in [[wa.task.2026.2026-04-11_1723-operational-config-for-runtime-resolution]].

### Where `ReferenceSource` should live

`ReferenceSource` records are part of the curated reference relation, so they should be serialized in the `ReferenceCatalog` alongside their owning `ReferenceLink`.

That does not broaden `ReferenceCatalog` into arbitrary descriptive RDF. A `ReferenceCatalog` may contain `ReferenceLink` nodes and the immediate `ReferenceSource` / locator helper nodes needed to describe those links. Broader facts about the referent still belong in payload artifacts or datasets.

`KnopSourceRegistry` remains the home for source/acquisition/provenance bindings such as import materialization, integration source binding, and extraction provenance. It should not become the default home for curated reference sources, or the reference model will be split across `_references` and `_sources` for no gain.

### Fixture precedent

The current fixture precedent already behaves like RDF reference sources even though the vocabulary says `referenceTarget`:

- Alice Bio: `<alice>` references `<alice/bio>`.
- Branch Fantasy Rules: ontology terms reference the `ontology` RDF artifact.
- Branch Fantasy Rules: `CharacterShape` references the `shacl` RDF artifact.
- Gunaar examples reference the `examples/gunaar` RDF artifact.

This task should align vocabulary with that practice rather than expanding the concept to non-RDF content.

## Open Issues

- What exact fragment convention should Weave use for source relators? Recommendation: use a stable companion fragment derived from the link fragment, such as `#reference001-source`, so a source remains visibly attached to its link.
- What ResourcePage selection policy should decide which reference RDF facts are incorporated into a generated page? Recommendation: leave this for a later page-selection task; this task should only make the source relation unambiguous.

## Decisions

- `ReferenceLink` is only for curated links to RDF reference data that supports, describes, or otherwise provides data about `referenceLinkFor`.
- Do not add `RdfReferenceLink`. Narrow the existing `ReferenceLink` concept instead.
- Add `ReferenceSource` as an `ArtifactResolutionTarget` subclass.
- Add `hasReferenceSource` from `ReferenceLink` to `ReferenceSource`.
- Require exactly one `hasReferenceSource` per `ReferenceLink` in the first slice; model multiple supporting sources as multiple links until a concrete use case needs one link with several equivalent source relators.
- Replace `referenceTarget` with `hasReferenceSource` plus the generic target properties on `ReferenceSource`.
- Replace `referenceTargetState` with `hasRequestedTargetState` on `ReferenceSource`.
- Remove `referenceUriLiteral` in the coordinated ontology pass. Fixtures and manifests should be updated and complete from-scratch fixture regeneration should run afterward.
- Keep `target*` vocabulary inside `ArtifactResolutionTarget`; do not use `target` directly on `ReferenceLink`.
- Runtime support for ReferenceSource resolution should start with in-local-mesh governed RDF artifacts. Direct `targetAccessUrl`, repository locator, and floating locator reference sources can remain modeled but unsupported by runtime page/reference processing until explicit operational policy and tests exist.
- Add `ImportSource` as the source-registry application concern for explicit acquisition/materialization.
- Add `IntegrationSource` as the source-registry application concern for `integrate` registering existing source bytes where they already live.
- Use [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] as the owning task for the generic `ArtifactResolutionTarget` source-family cleanup and `ArtifactResolutionObservation` vocabulary.
- Keep `ReferenceRole` and the current role vocabulary. Do not rename it to `ReferenceAuthorityRole` and do not add `referenceRole_pageSource`.
- Do not add `ReferenceFormat` in this task.
- Serialize `ReferenceSource` records in the `ReferenceCatalog` beside their owning `ReferenceLink`.
- Keep `KnopSourceRegistry` for source/acquisition/provenance bindings, not curated reference links.
- Bob Markdown page content should not use `ReferenceLink`.
- Pre-v1 compatibility shims are not required. Prefer a clean coordinated rename across ontology, framework fixtures, mesh fixtures, Weave code, CLI, and docs.

## Implementation Progress

- SFLO and Weave have moved the live RDF shape to `ReferenceLink` -> `hasReferenceSource` -> `ReferenceSource`.
- Weave rendering/parsing and ResourcePage reference panels now understand the new source relator shape.
- Tests for old fixture branches use a temporary fixture-reader normalization layer so old checked-in fixture RDF can still serve as comparison material until the full fixture regeneration pass.
- Still open in this task:
  - rename public/core/runtime request fields and CLI flag from `referenceTargetDesignatorPath` / `--reference-target-designator-path` to source terminology
  - update Semantic Flow Framework behavior spec, API example/schema language, and conformance manifests
  - regenerate mesh fixture branches so the fixture repositories themselves no longer carry retired `referenceTarget` / `referenceTargetState` predicates

## Contract Changes

- SFLO core ontology:
  - Add `ReferenceSource` as `rdfs:subClassOf ArtifactResolutionTarget`.
  - Coordinate `ImportSource`, `IntegrationSource`, and shared observation vocabulary through [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] if they land in the same ontology pass.
  - Add `hasReferenceSource` with domain `ReferenceLink` and range `ReferenceSource`.
  - Remove or supersede `referenceTarget`, `referenceTargetState`, and `referenceUriLiteral`.
  - Update comments for `ReferenceLink`, `ReferenceCatalog`, and `ReferenceRole` to reflect RDF-only reference-data semantics.
- SFLO SHACL:
  - Require exactly one `referenceLinkFor` as today.
  - Require at least one `hasReferenceRole` as today.
  - Require exactly one `hasReferenceSource` for the first slice.
  - Require the `hasReferenceSource` object to be typed `ReferenceSource`.
  - Require a first-slice `ReferenceSource` to identify RDF data, at least when using `hasTargetArtifact`, by requiring the target artifact to be an `sflo:RdfDocument`.
  - Validate `hasRequestedTargetState` through the existing `ArtifactResolutionTarget` rules instead of a reference-specific state property.
- SFLO notes:
  - Update [[ont.reference-links]] and [[ont.summary.core]].
  - Add a decision-log entry explaining the rename and the `ReferenceSource <: ArtifactResolutionTarget` pattern.
- Semantic Flow Framework:
  - Update the `knop.addReference` behavior spec to use `referenceSource` terminology.
  - Update API examples and conformance manifests that currently assert `referenceTarget`.
  - Update fixture documentation so Bob page-source/import examples are not described as ReferenceLink cases.
- Weave code:
  - Rename core/runtime request fields from `referenceTargetDesignatorPath` to `referenceSourceDesignatorPath`.
  - Rename CLI flag `--reference-target-designator-path` to `--reference-source-designator-path`.
  - Update `weave extract --add-source-references` internals and docs so extracted terms create `ReferenceLink` + `ReferenceSource` rather than `referenceTarget` facts.
  - Update ReferenceCatalog parsing/rendering to read and write `hasReferenceSource` plus `ReferenceSource` resolution facts.
  - Update ResourcePage reference panels to say "reference source" rather than "reference target".
- Fixture meshes:
  - Rewrite existing `references.ttl` files to use `hasReferenceSource` and companion `ReferenceSource` nodes.
  - Regenerate woven pages/manifests where rendered HTML or RDF assertions change.

## Testing

- Ontology/SHACL validation should reject a `ReferenceLink` with no `hasReferenceSource`.
- Ontology/SHACL validation should reject or warn on a `ReferenceSource` whose `hasTargetArtifact` does not identify an `sflo:RdfDocument`.
- Unit-test Weave ReferenceCatalog parsing for the new shape, including reordered Turtle, fragment companion sources, and optional `hasRequestedTargetState`.
- Unit-test CLI/request validation for the renamed `referenceSourceDesignatorPath` field/flag.
- Integration-test `weave knop add-reference alice --reference-source-designator-path alice/bio --reference-role canonical`.
- Integration-test `weave extract --all-terms --add-source-references` for current and pinned source-state cases.
- Regenerate or update Alice Bio and Fantasy Rules conformance assertions that currently query `referenceTarget`.
- Add regression coverage that non-RDF Markdown page content is handled through `ResourcePageSource` / import workflow and not through `ReferenceLink`.
- Run ontology validation plus the focused Weave tests that cover `knop add-reference`, extraction references, ReferenceCatalog parsing, and ResourcePage rendering.
- After implementation, run the normal per-repo checks required by the touched repos.

## Non-Goals

- Do not implement non-RDF ReferenceLinks.
- Do not add `ReferenceFormat`.
- Do not add `referenceRole_pageSource`.
- Do not make `ReferenceLink` itself an `ArtifactResolutionTarget`.
- Do not move curated reference source records into `KnopSourceRegistry`.
- Do not implement remote `targetAccessUrl` fetching for references unless a separate runtime-resolution task explicitly enables it under operational config.
- Do not change Bob's Markdown bio into a ReferenceLink case.
- Do not redesign import provenance or `KnopSourceRegistry` as part of this task, beyond keeping the boundaries clear.
- Do not make this task the owner of generic resolution-observation vocabulary; that belongs to [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]].
- Do not implement ResourcePage fact-selection policy for reference RDF in this task.
- Do not rename this task note to a completed note unless explicitly requested.

## Implementation Plan

- [ ] Re-read [[wd.general-guidance]], [[ont.reference-links]], [[ont.summary.core]], [[ont.decision-log]], the `knop.addReference` behavior spec, and this note before editing.
- [ ] Update SFLO ontology with `ReferenceSource` and `hasReferenceSource`, remove/supersede direct `ReferenceLink` target properties, and coordinate shared source subclasses/observations with [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]].
- [ ] Update SFLO SHACL for the new `ReferenceLink` / `ReferenceSource` shape.
- [ ] Update SFLO notes and decision log.
- [ ] Update Semantic Flow Framework specs, API examples, and conformance manifests.
- [ ] Update Weave core/runtime ReferenceCatalog models, add-reference planning, extraction source-reference generation, and page rendering.
- [ ] Rename the Weave CLI/API field and docs from reference target to reference source.
- [ ] Update Alice Bio and Fantasy Rules fixture meshes and generated pages that contain ReferenceCatalogs.
- [ ] Add focused tests for ontology validation, parser/rendering, CLI validation, `knop add-reference`, and `extract --add-source-references`.
- [ ] Run the relevant checks per touched repo.
- [ ] Provide commit messages per repo, with separate bullets for ontology, framework fixtures/specs, Weave code/docs, and fixture mesh regeneration.
