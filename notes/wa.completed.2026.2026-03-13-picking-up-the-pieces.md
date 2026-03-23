---
id: ajkwikaxm4h81dzhvnyo3d7
title: 2026 03 13 Picking up the Pieces
desc: ''
updated: 1773897130432
created: 1773620895575
---

## Goal

Recover a coherent Semantic Flow core model by simplifying the old heavy **Nomen**/**Knop** split, using `D/` as the public designator placeholder, using `D/_knop/` as the Knop IRI, and carrying that model far enough into ontology and documentation work that follow-on docs/code can proceed on a stable foundation.

## Status

- [x] Switch examples from `T/` to `D/` so the placeholder clearly means "designator"
- [x] Decide that `D/_knop/` denotes the Knop, not the identifier-as-identifier
- [x] Collapse `Nomen` into `Knop` in the current core ontology pass
- [x] Reintroduce a thin `Nomen` alongside `Knop`
- [x] Draft and land ontology deltas in the ontology checkout
- [x] Simplify the core artifact model to `AbstractArtifact -> HistoricalState -> ArtifactManifestation -> LocatedFile`
- [x] Add `hasWorkingLocatedFile` as the sparse working-surface hook
- [x] Replace metadata/inventory artifact-species with role classes
- [x] Keep `Knop` as the one-to-one support object paired with a `Nomen`
- [x] Keep `designatorPath` and `designates` on `Nomen`
- [x] Remove `payloadSlug`
- [x] Replace generic `contains...` slot properties with explicit `has...` slot properties
- [x] Refresh the Alice exemplar in the ontology repo
- [x] Add an LLM-oriented ontology core summary in the ontology repo
- [x] Decide to reset the canonical docs rather than incrementally harmonize sprawling legacy docs
- [x] Record the top-level settled decisions in [[wd.decision-log]]
- [c] Introduce an explicit subordinate identifier handle such as `D/_knop/#identifier` in the current core pass
- [c] Define a first-class alias/canonical-target pattern in the current core pass
- [x] Rewrite core docs around Nomen/Knop terminology in Weave
- [x] Identify the minimum code worth carrying forward, if any
- [x] Decide whether the remaining docs/code work should stay under this task or split into a follow-on task once implementation starts

## Summary

The older model carried two overlapping first-class support resources:

* the public designator IRI `D/`, which denotes the referent
* a semantic support object (`Nomen`) for talking about that designator in RDF
* a semantic/artifact container (`Knop`) for payload, metadata, inventory, and support artifacts

The settled direction removes the old heavier `Nomen` layer, but keeps a thinner naming-side node as **Nomen**:

* `D/` remains the designator IRI
* `D/_nomen/` denotes the thin naming resource `Nomen`
* `D/_knop/` denotes the mesh-managed Knop associated with `D/`
* all mesh-managed support artifacts live under `D/_knop/`
* a `Nomen` carries `designatorPath` and optional `designates`
* a `Knop` is the 1-to-1 mesh-managed support object linked to that `Nomen`
* a `Knop` may be purely referential, or it may also host a payload artifact
* `SemanticMesh` is the Semantic Flow surface, not the whole host hierarchy
* `_mesh/` denotes the mesh surface itself
* payload, metadata, and inventory are modeled as role classes over `DigitalArtifact` facets
* the core artifact chain is `AbstractArtifact -> HistoricalState -> ArtifactManifestation -> LocatedFile`
* sparse working support uses `hasWorkingLocatedFile` rather than `WorkingState` / `CurrentState`

This keeps the single-referent discipline at the public IRI while reducing conceptual duplication in the implementation model.

## Settled Decisions

- Use `D/` as the canonical placeholder for a public designator in docs and examples.
- Let `D/_nomen/` denote the thin naming resource `Nomen`.
- Let `D/_knop/` denote the Knop, not a separate identifier-object.
- Remove the old heavier `Nomen` layer rather than keeping the earlier duplicated Nomen/Knop model.
- Reintroduce a thin `Nomen` class rather than reviving a heavier `Nomen` or more abstract identifier-handle layer.
- Keep `designatorPath` and `designates` on `Nomen`.
- Treat a subordinate identifier handle as a future escape hatch, not a current core requirement.
- Treat alias/canonical-target semantics as deferred rather than core-blocking for this pass.
- Model `SemanticMesh` as the Semantic Flow surface, with `_mesh/` as its conventional handle.
- Keep `ArtifactContainer` for now because both `SemanticMesh` and `Knop` host support resources.
- Keep `Knop` as the one-to-one support object associated with a Semantic Flow identifier and paired `Nomen`.
- Use the simplified artifact model:
  - `AbstractArtifact`
  - `HistoricalState`
  - `ArtifactManifestation`
  - `LocatedFile`
- Keep `AbstractArtifact` as the diachronic artifact identity.
- Use `hasWorkingLocatedFile` instead of `WorkingState` / `CurrentState`.
- Treat `PayloadArtifact`, `KnopMetadata`, `KnopInventory`, `MeshMetadata`, and `MeshInventory` as role classes over artifact facets.
- Remove `payloadSlug`; the identifier-side path information now lives in `designatorPath` on `Nomen`.
- Prefer explicit `has...` slot properties over a generic `containsSemanticFlowResource` property in the current core surface.
- Keep inventory singular in the current model.
- Reset the canonical docs rather than trying to harmonize the sprawling legacy note set.

## Discussion

### Why collapse Nomen into Knop

The current ontology and docs are cleaner if the mesh-managed support object stays `Knop`, while mesh-relative naming semantics live on a thin `Nomen`:

* `D/` denotes the referent
* `D/_nomen/` denotes the Nomen for `D/`
* `D/_knop/` denotes the Knop that supports `D/` inside the mesh
* `D/_knop/...` hosts metadata, inventory, payload-support structure, history, and other mesh-managed artifacts
* identifier-side semantics no longer need to be conflated with Knop support semantics

In the filesystem-backed model, the jobs formerly associated with the heavier `Nomen` layer are either now carried by the thinner `Nomen`, implied by structure, or can live in Knop-managed metadata:

* `designatorPath` now lives on `Nomen`
* `designates` now lives on `Nomen`
* identifier-about-identifier metadata can accumulate on `Nomen` without requiring the old heavier duplicated layer

This reduces duplication between:

* Nomen metadata vs Knop metadata
* Nomen inventory vs Knop inventory
* Nomen containment vs Knop containment
* “binding” logic that is really just mesh structure

### Filesystem posture

For a designator `D/`:

* `D/index.html` remains the primary human dereference point
* `D/_knop/index.html` can explain the support structure and expose raw metadata/inventory links
* payload, support artifacts, and `_history/` live under `D/_knop/`
* `_mesh/` is the mesh-level support surface

## Open Issues

* Should the root designator also have a root Knop at `/_knop/`?
* How should we model a mesh-managed Knop whose referent is a DigitalArtifact hosted elsewhere?
* How much backward compatibility do we want for existing `_nomen/` references in docs or generated structures?
* What is the minimum live doc set that should become canonical next?
* What is the minimum code worth carrying forward from the kato-side work, if any?

### Recommendation: kato repo reset

Recommendation: **start over for the kato repo documentation and code, but not by losing history**.

That means:

* keep the conversation notes, task notes, and a few high-value project-level notes
* do not try to preserve the sprawling current conceptual docs as authoritative
* do not spend time adapting small legacy code unless it clearly supports the new Nomen/Knop model
* rewrite a compact canonical doc set around the new model, then reintroduce only what still earns its keep

Rationale:

* too many existing docs encode the Nomen/Knop split and will keep reintroducing confusion
* incremental harmonization across a sprawling, outdated note set will likely cost more than a focused rewrite
* the small amount of existing code appears less valuable than a clean restatement of the model

Suggested preservation strategy:

* preserve current docs on the existing branch or under an explicit legacy/archive area
* define a small new canonical surface
* treat old notes as historical reference, not live specification

## Remaining Work

- [x] Rewrite the highest-leverage core docs around Nomen/Knop terminology.
- [x] Decide the minimal live doc set that should become canonical next.
- [x] Decide whether the docs/code follow-on should stay under this task or move to a fresh task note.

## Validation / Evidence

- [x] Update the ontology checkout to remove the old heavier `Nomen` surface and replace it with the thinner current one.
- [x] Update SHACL to reflect the simplified artifact model.
- [x] Update the Alice exemplar in the ontology repo.
- [x] Add a crisp LLM-oriented ontology summary in the ontology repo.
- [x] Run stale-term greps against the edited ontology/SHACL/docs surface.
- [x] Run fuller RDF/SHACL validation in the ontology repo.

## Notes On Scope

This task still makes sense as the umbrella for the March 13 conceptual cleanup. The remaining open work is no longer "figure out the model"; it is "finish restating docs and decide what code, if any, survives." If that work becomes large enough to need its own scoped execution plan, split it later rather than prematurely.

## Non-Goals

* Reworking the entire ontology in this repo
* Preserving every old documentation page as active/canonical
* Maintaining compatibility with every experimental `_nomen/` layout ever discussed
* Reviving the old heavier Nomen model just to preserve older terminology
* Porting forward legacy kato-side code without a clear current use case
