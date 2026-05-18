---
id: mrglp1bvoit0wpr69uhvz25
title: 2026-03-26-ArtifactHistory
desc: ''
updated: 1774544367566
created: 1774544367566
---

This task is complete. The live ontology, SHACL, supporting notes, and Alice Bio fixture all now use the explicit `ArtifactHistory` model and the `_historyNNN` / `_sNNNN` path convention described here.

## Context

The current core artifact model links `HistoricalState` directly from `DigitalArtifact` and treats `_history` material as support structure under the hosting `Knop`.

That is still workable for sparse single-lineage versioning, but it has started to show real pressure points:

- a history landing page should ideally correspond to an explicit resource rather than being an exception to the usual `ResourcePage` pattern
- the next auto-issued state ordinal has no good home if `KnopInventory` stays current-surface-only
- multiple histories are now a realistic use case, including replacement histories after redaction/correction or partial migration into a new namespace
- the current model does not give the diachronic lineage itself a first-class handle for temporal extent, lineage metadata, or allocation state
- the current `ArtifactContainer` class looks unused and under-motivated in the active core, and its continued presence makes the history discussion fuzzier rather than clearer

This task explores a narrower reintroduction: `ArtifactHistory`, not the broader older `ArtifactFlow`.

## Working Goal

Introduce an optional explicit `ArtifactHistory` resource that can:

- own `hasHistoricalState` / `latestHistoricalState`
- carry history-level operational metadata such as the next auto-issued state ordinal
- support a corresponding history `ResourcePage`
- make room for multiple histories per artifact without forcing that complexity on every artifact

The goal is to keep sparse cases possible while making explicit history lineages first-class when they are intentionally materialized.

## Current Direction

- `ArtifactHistory` should be optional, not mandatory for every `DigitalArtifact`.
- `ArtifactFlow` should remain out of the active core; the narrower term here is `ArtifactHistory`.
- `ArtifactContainer` should be removed from the active core rather than extended to cover `ArtifactHistory`.
- If explicit histories are adopted, `hasHistoricalState` and `latestHistoricalState` should hang off `ArtifactHistory`, while `DigitalArtifact` links to one or more histories through a new relation.
- `DigitalArtifact` should also have a convenience pointer to the current/default history, because once multiple histories exist, following `hasArtifactHistory` and then `latestHistoricalState` is no longer enough to identify the active lineage.
- The existing property names `hasHistoricalState` and `latestHistoricalState` can stay; the main change is their authoritative domain.
- No separate `RdfDocument` support artifact for `ArtifactHistory` is planned in this first pass.
- History-level semantic and operational metadata should initially be serialized in the owning `KnopMetadata` artifact, not in `KnopInventory`, and not in a separate sub-knop history metadata artifact.
- `KnopInventory` should stay focused on current-surface structure and file/resource enumeration rather than allocation state.
- Auto-generated historical-state identifiers should use a reserved `_sNNNN` pattern.
- Auto-generated history identifiers should use a reserved `_historyNNN` pattern, while still allowing user-chosen history names.

## Proposed Ontology Delta

- Remove class `ArtifactContainer`.
- Remove `ArtifactContainer` as a superclass of `SemanticMesh` and `Knop`.
- Add class `ArtifactHistory`.
- Add object property `hasArtifactHistory`.
  - expected domain: `DigitalArtifact`
  - expected range: `ArtifactHistory`
- Add object property `currentArtifactHistory`.
  - expected superproperty: `hasArtifactHistory`
  - expected character: functional subproperty of `hasArtifactHistory`
  - expected domain: `DigitalArtifact`
  - expected range: `ArtifactHistory`
  - expected usage: optional but strongly expected when an artifact has an active explicit history lineage, and the normative target for future weave/version operations
- Change `hasHistoricalState` so its authoritative domain is `ArtifactHistory`.
- Change `latestHistoricalState` so its authoritative domain is `ArtifactHistory`.
- Keep `previousHistoricalState` on `HistoricalState`.
- Add datatype property `historyOrdinal`.
  - expected domain: `ArtifactHistory`
  - expected range: non-negative integer
- Add datatype property `nextHistoryOrdinal`.
  - expected domain: `DigitalArtifact`
  - expected range: non-negative integer
- Add datatype property `nextStateOrdinal`.
  - expected domain: `ArtifactHistory`
  - expected range: non-negative integer

## Modeling Direction

`ArtifactHistory` should be treated as an explicit support resource in the mesh surface and typed only as a `SemanticFlowResource`, not as a governing `DigitalArtifact`.

It should also not be treated as an `ArtifactContainer`; explicit structural relations are the better fit than a vague shared container superclass.

Likely consequences:

- a `DigitalArtifact` may have zero, one, or many `ArtifactHistory` resources
- a `DigitalArtifact` may optionally designate one of those histories as its current/default history through `currentArtifactHistory`
- each `ArtifactHistory` may have zero, one, or many `HistoricalState`s
- `latestHistoricalState` becomes lineage-local rather than artifact-global
- history replacement or parallel histories become representable without pretending that all states belong to one undifferentiated artifact-level stream

One likely consequence for serialization is that the `KnopMetadata` artifact becomes the initial home for triples such as:

- `DigitalArtifact hasArtifactHistory ArtifactHistory`
- `DigitalArtifact currentArtifactHistory ArtifactHistory`
- `DigitalArtifact nextHistoryOrdinal ...`
- `ArtifactHistory hasHistoricalState HistoricalState`
- `ArtifactHistory latestHistoricalState HistoricalState`
- `ArtifactHistory historyOrdinal ...`
- `ArtifactHistory nextStateOrdinal ...`

`KnopInventory` may still mention currently materialized history files, pages, and paths, but it should not be the authority for history ordinality or the next auto-issued state ordinal.

## Serialization And Path Direction

Current preferred direction:

- default initial history example: `D/_history001`
- default auto-generated historical state example: `D/_history001/_s0001`
- `D/_history001/index.html` should be the `ResourcePage` for that `ArtifactHistory`

Design intent:

- reserve `_historyNNN` and `_sNNNN` for generated defaults
- allow user-chosen history and state names where explicit naming is desired
- keep `v...` available for user-facing version semantics rather than occupying it with the default machine naming scheme

Numbered history resources can remain directly under `D/`; there is no need for a separate collection layer in the current direction.

This still leaves room for later named histories such as a redacted or migrated lineage without requiring every mesh to use them.

## Current Decisions

- `ArtifactHistory` is worth modeling explicitly if the history itself has identity, a page, allocation state, or replacement/multiplicity semantics.
- `ArtifactContainer` should be removed rather than reused for `ArtifactHistory`.
- `ArtifactHistory` should be typed only as a `SemanticFlowResource`.
- `DigitalArtifact` should carry a `currentArtifactHistory` pointer when explicit histories are in play, and that pointer should be a functional subproperty of `hasArtifactHistory` as well as the normative target for future weave/version operations.
- `DigitalArtifact` should also carry `nextHistoryOrdinal` for symmetric default allocation of numbered history resources.
- The default machine naming scheme should not consume `v...`.
- `hasHistoricalState` can keep its current name even if its authoritative domain changes.
- `KnopMetadata` is the serialization home for history-level semantic and operational metadata in this model; avoid dedicated history metadata artifacts and avoid sub-knop history metadata artifacts in this pass.
- `KnopInventory` should remain current-surface-oriented.
- Default examples should use `_historyNNN` and `_sNNNN`.

## Open Questions

- Are there history-level metadata needs beyond `historyOrdinal`, `nextStateOrdinal`, and temporal extent that still fit comfortably in `KnopMetadata` without reintroducing a dedicated history metadata artifact?

## Candidate Documentation Updates

- update `semantic-flow-core-ontology.ttl` to remove `ArtifactContainer`, add `ArtifactHistory`, `hasArtifactHistory`, `currentArtifactHistory`, `historyOrdinal`, `nextHistoryOrdinal`, `nextStateOrdinal`, and revise comments/domain expectations for history relations
- update `semantic-flow-core-shacl.ttl` so validation matches the revised core model, especially around `hasHistoricalState`, `latestHistoricalState`, and any assumptions that still target `DigitalArtifact` directly
- update `notes/ont.summary.core.md` to describe the revised artifact/history model and new path examples
- update `notes/ont.use-case.biographical-data-publishing.md` so history resources and states use the new explicit `ArtifactHistory` framing rather than direct artifact-to-state structure
- add a decision entry in `notes/ont.decision-log.md` adopting `ArtifactHistory` and the `_historyNNN` / `_sNNNN` default naming direction while keeping `ArtifactFlow` out of the active core
- update [wd.task.2026.2026-03-25-mesh-alice-bio.md](../../../documentation/notes/wd.task.2026.2026-03-25-mesh-alice-bio.md) so the branch plan and example paths for woven history generation match the new model

## Suggested Plan

1. Decide the minimal ontology shape for `ArtifactHistory`, especially whether it needs any superclass beyond `SemanticFlowResource`.
2. Remove `ArtifactContainer` from the core and strip its remaining subclass uses.
3. Define `hasArtifactHistory`, `currentArtifactHistory`, and move the authoritative use of `hasHistoricalState` / `latestHistoricalState` onto `ArtifactHistory`.
4. Define the minimal ordinal vocabulary: `historyOrdinal` on `ArtifactHistory`, `stateOrdinal` on `HistoricalState`, `nextHistoryOrdinal` on `DigitalArtifact`, and `nextStateOrdinal` on `ArtifactHistory`.
5. Update `semantic-flow-core-shacl.ttl` so shapes validate the revised ontology rather than the current direct-`DigitalArtifact` history model.
6. Update the path examples and Alice Bio use case to use explicit histories and padded `_historyNNN` / `_sNNNN` names.
7. Record the decision in the ontology decision log before changing the mesh example branches that depend on it.
