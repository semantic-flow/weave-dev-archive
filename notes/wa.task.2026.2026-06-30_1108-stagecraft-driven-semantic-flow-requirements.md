---
id: stagecraft-driven-sf-20260630
title: 2026 06 30 1108 Stagecraft Driven Semantic Flow Requirements
desc: ''
created: 1782842880000
---

## Goals

- Capture concrete Stagecraft persistence needs before changing Weave or SFLO vocabulary for roleplaying-data use cases.
- Use Stagecraft as a real downstream consumer to prioritize Semantic Flow and Weave engineering work.
- Keep the existing publication/runtime correctness backlog honest while allowing application-driven needs to reorder slices when they expose real blockers.

## Summary

Stagecraft development has begun in earnest, and Semantic Flow is expected to serve as the persisted roleplaying-data solution. This makes Stagecraft the first active application-shaped consumer after the ontology/publication fixtures. The near-term engineering posture should be consumer-driven but cautious: collect concrete workflows, data shapes, mutation patterns, query needs, and publication/inspection expectations before adding vocabulary or CLI/runtime behavior.

The paper sprint is parked for now. The FOMI/FOIS drafts are still useful as conceptual scratch, but they should not drive implementation. Stagecraft should drive by producing real use cases: campaign/world data, characters, scenes, logs, generated artifacts, references, revisions, provenance, and inspection requirements.

## Discussion

Useful pressure from Stagecraft likely differs from ontology publication in several ways:

- application data may change frequently and need practical current-state updates without noisy historical-state churn
- some resources may need exact citation, audit trails, or release-like snapshots, while others may only need current persistence
- generated or assistant-produced content may need provenance, revision, and attribution records
- roleplaying data may include nested or highly connected resources where identifier minting, ResourcePages, and references need to stay ergonomic
- local-first authoring and application runtime persistence may matter more than static public publication at first
- private campaign data raises a stronger boundary between inspectable local mesh state and anything suitable for publication

Stagecraft should not automatically force broad new ontology layers. First check whether existing Semantic Flow concepts already fit:

- `SemanticMesh` for the persisted namespace or workspace
- `Knop` for stable identifiers and support surfaces
- `DigitalArtifact` for files, generated documents, transcripts, bundles, maps, notes, and data exports
- `ArtifactHistory` and `HistoricalState` for snapshots worth citing or auditing
- current-only support artifacts for operational metadata and frequently changing data
- `ReferenceCatalog` and source registries for curated references and provenance
- ResourcePages for local inspection and future static publication

## Open Issues

- Which Stagecraft resources need stable public-style identifiers, and which only need internal IDs?
- What data should be RDF-native versus serialized application payloads carried as `DigitalArtifact`s?
- Which changes need append-onlyish history, and which should remain current-only?
- Does Stagecraft need fast local query/index support beyond the current file-oriented runtime?
- What privacy/publication boundary is required for campaigns, sessions, player data, generated text, and notes?
- Should Stagecraft use whole-repo, sidecar, or application-managed mesh topology for persisted data?

## Decisions

- Treat Stagecraft as a near-term consumer that may reprioritize Weave/Semantic Flow work.
- Do not add Stagecraft-specific vocabulary or runtime features until a concrete workflow demonstrates the need.
- Keep append-onlyish inventory, history-policy coherence, source provenance, and ResourcePage inspection as likely high-value shared substrate.

## Contract Changes

- None yet. This is a requirements-capture task.

## Testing

- No tests required until this produces implementation slices.
- Future slices should include small Stagecraft-shaped fixtures when they introduce reusable behavior.

## Non-Goals

- Do not design a full roleplaying ontology here.
- Do not make Stagecraft private application requirements part of portable Semantic Flow semantics without checking generality.
- Do not let paper-draft terminology override live ontology/framework wording.
- Do not prioritize RDFa/JSON-LD embedding or polished publication features over persistence correctness unless Stagecraft directly needs them.

## Implementation Plan

- [ ] Inventory the first Stagecraft persisted-data workflows and identify which ones are mesh, artifact, history, reference, source, or page problems.
- [ ] Decide the initial Stagecraft mesh topology: whole-repo, sidecar, branch-published, or application-managed local mesh.
- [ ] Map a few representative resources to existing Semantic Flow concepts before proposing vocabulary changes.
- [ ] Identify the first Weave blocker that prevents Stagecraft from persisting or inspecting useful roleplaying data.
- [ ] If a blocker is general, create a focused Weave task note and implementation slice.
