---
id: yjja497yn507gmnvo4xgib2
title: 2026 05 03 Enumeration Type Instances
desc: ''
updated: 1777955567657
created: 1777870909784
---

## Goals

- Replace hierarchical PascalCase controlled-value IRIs such as `ReferenceRole/Canonical` and `ArtifactResolutionMode/Current` with flat underscore-separated individuals.
- Avoid slash-based enum values so Turtle, JSON-LD, SPARQL, and CLI/generated-code consumers do not need escaped prefixed names such as `sflo:ArtifactResolutionMode\/Current`.
- Keep class IRIs such as `ReferenceRole`, `ArtifactResolutionMode`, and `JobKind` stable unless a separate modeling problem requires changing the classes themselves.
- Use human-readable `rdfs:label` values for display legibility rather than encoding display text in the IRI.
- Coordinate the enum migration with the next fixture ladder rerunging pass, now expected after the next config synthesis pass, so Alice Bio and Fantasy Rules are not rerung twice for closely related ontology and config vocabulary churn.

## Summary

The current ontologies model controlled values as hierarchical IRIs with PascalCase terminal segments, for example:

```ttl
<ReferenceRole/Canonical> a <ReferenceRole> .
<ArtifactResolutionMode/Current> a <ArtifactResolutionMode> .
<JobKind/MeshCreate> a <JobKind> .
```

That shape is readable as full IRIs, but it is awkward for prefixed Turtle and inconsistent with the current desire for ordinary term-like identifiers. The intended replacement is flat controlled-value individuals using the enum class lowerCamel name, an underscore, and a lowerCamel value suffix:

```ttl
<referenceRole_canonical> a <ReferenceRole> ;
  rdfs:label "Canonical" .

<artifactResolutionMode_current> a <ArtifactResolutionMode> ;
  rdfs:label "Current" .

<jobKind_meshCreate> a <JobKind> ;
  rdfs:label "mesh create" .
```

This keeps enum instances visually distinct from classes without requiring path hierarchy. It also lets generated RDF use the ontology's preferred namespace prefix `sflo:` cleanly, for example `sflo:referenceRole_canonical`.

The migration affects ontology files, Weave constants/tests, fixture RDF, Accord manifests, and docs/spec examples that assert exact IRIs.

## Discussion

Three alternatives were considered:

- hierarchical camelCase values such as `ReferenceRole/canonical`
- flat values such as `referenceRoleCanonical`
- hyphenated values such as `referenceRole-canonical`

Hierarchical camelCase keeps a type/value visual grouping, but it still requires escaping in prefixed Turtle and carries the same cross-library risk as the current pattern. Flat values avoid that problem and align with ordinary RDF vocabulary practice. The display label stays available through `rdfs:label`, so the IRI does not need to preserve `Canonical` or `Current` as a PascalCase segment.

Pure lowerCamel values such as `referenceRoleCanonical` are code-generation-friendly, but they are less legible once enum class names and value names both contain multiple words. Hyphenated forms such as `referenceRole-canonical` are similarly readable in RDF, but many generated programming-language bindings would need bracket access or name transformation because `-` is an operator in common target languages. The preferred shape is therefore `referenceRole_canonical`: class lowerCamel, underscore separator, and value lowerCamel.

The change should not be treated as a compatibility migration. Semantic Flow is still pre-v1, and carrying aliases for every old enum IRI would make generated RDF and fixture tests harder to reason about.

Known affected controlled vocabularies include at least:

- `ReferenceRole`: `ReferenceRole/Canonical`, `ReferenceRole/Supplemental`, `ReferenceRole/Deprecated`
- `ArtifactResolutionMode`: `ArtifactResolutionMode/Pinned`, `ArtifactResolutionMode/Current`
- `ArtifactResolutionFallbackPolicy`: `ArtifactResolutionFallbackPolicy/ExactOnly`, `ArtifactResolutionFallbackPolicy/AcceptLatestInRequestedHistory`
- `JobKind`: `JobKind/MeshCreate`, `JobKind/KnopCreate`, `JobKind/KnopAddReference`, `JobKind/Integrate`, `JobKind/Version`, `JobKind/Validate`, `JobKind/Generate`, `JobKind/Weave`
- `JobStatus`: `JobStatus/Accepted`, `JobStatus/Running`, `JobStatus/Succeeded`, `JobStatus/Failed`, `JobStatus/Cancelled`
- `RoleType`: `RoleType/DataExtraction`, `RoleType/QualityAssurance`, `RoleType/FinalApproval`, `RoleType/Coordination`

The Fantasy Rules ontology also carries domain-specific enum-like individuals such as alignment values. Those are not the primary target of this task unless they follow the same class-enumeration anti-pattern inside Semantic Flow ontology files. Domain vocabulary may legitimately use slash-path term IRIs when the domain wants dereferenceable term pages.

## Open Issues

- No remaining modeling blockers are known before implementation.

## Inventory Notes

Active Semantic Flow ontology files only contain slash-shaped controlled-value IRIs for the six vocabularies already listed in this note:

- `ReferenceRole`
- `ArtifactResolutionMode`
- `ArtifactResolutionFallbackPolicy`
- `JobKind`
- `JobStatus`
- `RoleType`

The config ontology also defines controlled vocabularies. Their current values are already flat camelCase rather than hierarchical slash paths, but they should be normalized in this pass so enum-like values consistently use the same underscore-separated class/value shape:

- `LocalPathBase`: `meshRootPathBase` -> `localPathBase_meshRoot`, `userHomePathBase` -> `localPathBase_userHome`, `absolutePathBase` -> `localPathBase_absolutePath`
- `LocalPathLocatorKind`: `workingLocalRelativePathLocatorKind` -> `localPathLocatorKind_workingLocalRelativePath`, `targetLocalRelativePathLocatorKind` -> `localPathLocatorKind_targetLocalRelativePath`
- `RemoteLocatorKind`: `workingAccessUrlLocatorKind` -> `remoteLocatorKind_workingAccessUrl`, `targetAccessUrlLocatorKind` -> `remoteLocatorKind_targetAccessUrl`

The config rename is not required to remove slash paths, but it avoids preserving a second enum-value naming convention.

## Decisions

- Use flat underscore-separated individuals for controlled enum values.
- Prefix the individual local name with the enum class name in lowerCamel, followed by `_`, followed by the value name in lowerCamel, for example `referenceRole_canonical` and `artifactResolutionMode_current`.
- Keep enum classes themselves as PascalCase class IRIs.
- Prefer `rdfs:label` for display strings such as "Canonical", "Current", and "mesh create".
- Do not use slash hierarchy for enum values.
- Do not use pure lowerCamel enum values such as `referenceRoleCanonical`.
- Do not use hyphenated enum values such as `referenceRole-canonical`.
- Normalize already-flat config ontology controlled values to the same underscore-separated class/value shape in this pass.
- Remove old enum IRIs outright; do not preserve them as deprecated individuals or compatibility aliases.
- Do not repair fixture branches as part of this task; defer fixture regeneration until after the next config synthesis pass so enum and config vocabulary fallout can be absorbed together.
- Defer explicit SHACL enum-value enforcement unless a later validation task decides ordinary RDF/tests are insufficient.
- Leave historical task notes alone; update only live docs/specs and current tests.

## Contract Changes

- Semantic Flow enum instance IRIs move from hierarchical PascalCase paths or older unseparated config names to flat underscore-separated names.
- RDF examples, generated support artifacts, Accord manifests, SHACL tests, and Weave constants must use the new enum IRIs.
- JSON-LD contexts and API examples that mention enum values must be updated to the new term IRIs.
- Existing fixtures that assert old enum IRIs are no longer canonical once fixture regeneration is complete.

## Testing

- Validate changed ontology Turtle files with RDF syntax validation.
- Run ontology SHACL validation if available for the affected files, but do not add new enum-value SHACL constraints in this pass.
- Update Weave unit/integration/e2e tests that assert enum IRIs.
- Search the workspace for old enum IRI fragments such as `ReferenceRole/`, `ArtifactResolutionMode/`, `ArtifactResolutionFallbackPolicy/`, `JobKind/`, `JobStatus/`, and `RoleType/` after the migration, plus old config value names such as `meshRootPathBase`, `workingLocalRelativePathLocatorKind`, `targetLocalRelativePathLocatorKind`, `workingAccessUrlLocatorKind`, and `targetAccessUrlLocatorKind`.
- After deferred fixture regeneration, rerun the fixture conformance tests and validate regenerated Accord manifests.

## Non-Goals

- Renaming enum classes such as `ReferenceRole` or `ArtifactResolutionMode`.
- Designing a general SKOS concept scheme for every controlled vocabulary.
- Changing domain ontology term IRIs that intentionally use slash paths for dereferenceable public terms.
- Preserving old enum IRIs as long-term aliases.
- Solving page-selection or ReferenceLink behavior directly; this task only settles the controlled-value naming convention those features should use.

## Implementation Plan

- [x] Inventory all enum-like controlled values in Semantic Flow ontology files and classify which are in scope.
- [x] Update core ontology enum individuals to flat underscore-separated names.
- [x] Update job ontology enum individuals to flat underscore-separated names.
- [x] Update provenance ontology enum individuals to flat underscore-separated names.
- [x] Update config ontology controlled-value individuals to flat underscore-separated names.
- [x] Update JSON-LD examples, live docs/spec examples, and ontology notes that refer to old enum IRIs.
- [x] Update Weave constants and rendered Turtle templates for `ReferenceRole`, `ArtifactResolutionMode`, and any other enum values it emits or parses.
- [x] Update Weave tests that assert old enum IRIs.
- [c] Rerung Alice Bio from the first affected ReferenceLink branch when coordinated with the page-selection split.
- [c] Rerung Fantasy Rules from the first affected term-extraction branch when coordinated with the page-selection split.
- [d] Regenerate fixtures after the ontology/code migration is complete; deferred until after the next config synthesis pass.
- [d] Record exact reproduction commands for regenerated fixture transition manifests or conformance README entries; deferred with fixture regeneration.
- [c] Skip a standalone enum-only validation pass; fixture-backed tests and exact generated outputs will be updated after the next config synthesis pass and fixture regeneration.
