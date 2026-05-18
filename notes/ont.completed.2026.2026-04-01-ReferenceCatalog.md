---
id: 0gd1qpsh26f58zduqmni0v5
title: 2026-04-01-ReferenceCatalog
desc: ''
updated: 1775098297595
created: 1775098297595
---

## Goals

- Introduce a dedicated `ReferenceCatalog` support artifact for `ReferenceLink` relators.
- Remove `ReferentMetadata` / `hasReferentMetadata` from the live core model rather than keeping two overlapping homes for RDF about the referent.
- Keep `ReferenceCatalog` narrow and mechanical: it is for `ReferenceLink`s, not for arbitrary descriptive RDF and not for `owl:sameAs`.
- Allow both `Knop` and `SemanticMesh` to own a `ReferenceCatalog`, but do that through SHACL rather than by reintroducing a vague shared superclass.

## Summary

This task is complete. The live ontology, SHACL, supporting notes, Alice Bio fixture branches, and conformance manifests have all been updated to use `ReferenceCatalog` instead of `ReferentMetadata`.

The current model has two competing directions for RDF about a referent:

- keep a broad `ReferentMetadata` artifact
- or push substantive referent RDF into a separate payload dataset / payload artifact, with its own Knop where appropriate

This task takes the sharper direction:

- remove `ReferentMetadata` and `hasReferentMetadata`
- add `ReferenceCatalog` and `hasReferenceCatalog`
- reserve `ReferenceCatalog` for `ReferenceLink` relators only
- treat broader RDF about a referent as payload data, not as support-artifact metadata

The practical consequence is that the first Alice Bio reference branch should stop putting `ReferenceLink`s in Knop inventory and instead introduce a dedicated reference-catalog artifact.

## Discussion

### Why remove `ReferentMetadata`

`ReferentMetadata` overlaps too heavily with the payload-artifact path that the current project direction already prefers.

If RDF about the referent matters enough to curate as content, the cleaner model is:

- make it a payload dataset / payload artifact
- and, when needed, give it its own Knop-managed surface

Keeping `ReferentMetadata` alongside that creates a recurring modeling question with no clean rule:

- should these triples live in a support artifact or in a payload artifact?

Removing it sharpens the boundary:

- support artifacts are mechanical and operational
- payload artifacts carry substantive RDF content

### Why add `ReferenceCatalog`

`ReferenceLink`s still need a dedicated home because they are semi-mechanical support structures:

- they are about the referent
- but they are managed as support artifacts in the mesh
- they should not be mixed into `KnopInventory`

`ReferenceCatalog` is the clean narrow artifact for that purpose.

### Ownership versus subject matter

The owner of a `ReferenceCatalog` is a support resource:

- typically a `Knop`
- optionally a `SemanticMesh`

But the contents of a `ReferenceCatalog` are about the referent or mesh subject, not about the support object as such.

That means the core relation is still:

- `referenceLinkFor` points to the resource that the link is about

For example, for Alice:

- `referenceLinkFor <alice>`

not:

- `referenceLinkFor <alice/_knop>`

### Domain handling

Do not reintroduce `ArtifactContainer`.

If `hasReferenceCatalog` needs to work for both `Knop` and `SemanticMesh`, the cleanest current approach is:

- omit an `rdfs:domain` axiom on `hasReferenceCatalog`
- enforce permitted owners in SHACL

That keeps the live ontology from faking a common superclass merely for slot symmetry.

### Serialization direction

The current direction is:

- Knop-owned catalogs live at `D/_knop/_references/references.ttl`
- mesh-owned catalogs live at `_mesh/_references/references.ttl`
- stable `ReferenceLink` identities may be fragment IRIs rooted at the catalog resource, such as `<D/_knop/_references#reference001>`

This keeps `ReferenceCatalog` aligned with other support artifacts while allowing one self-contained reference namespace per catalog.

## Open Issues

- Decide whether `ReferenceCatalog` should eventually get a direct owner-level shortcut analogous to `hasWorkingKnopInventoryFile` / `hasWorkingMeshInventoryFile`, or whether `hasWorkingLocatedFile` on the artifact is enough.
- Decide later whether reference verification needs its own object model; for now `verifiedAt` / `verifiedBy` remain latest-verification convenience fields on `ReferenceLink`.

## Decisions

- Remove `ReferentMetadata` and `hasReferentMetadata` from the core ontology.
- Add `ReferenceCatalog` as a `DigitalArtifact`, `RdfDocument`, and `SemanticFlowResource`.
- Add `hasReferenceCatalog` as the slot property for attaching a `ReferenceCatalog`.
- Keep `ReferenceCatalog` restricted to `ReferenceLink` relators only.
- Do not treat `owl:sameAs` or other broad descriptive/assertive RDF as part of `ReferenceCatalog` in this pass.
- Treat substantive RDF about the referent as payload data rather than as a support-artifact metadata bucket.
- Support `Knop` and `SemanticMesh` as valid owners of `hasReferenceCatalog` through SHACL rather than by adding a new common superclass.
- `referenceLinkFor` should point to the actual resource the link is about, not to the Knop support object.
- `referenceTargetState` may be used when a `ReferenceLink` should pin to a specific `HistoricalState` rather than only to a broader artifact/resource identity.
- Use `D/_knop/_references/references.ttl` for Knop-owned catalogs and `_mesh/_references/references.ttl` for mesh-owned catalogs.
- Allow stable catalog-rooted fragment IRIs such as `<D/_knop/_references#reference001>` for `ReferenceLink` identities.
- Allow at most one `ReferenceCatalog` per `Knop` or `SemanticMesh` in this pass.
- Implement `SemanticMesh` ownership in ontology and SHACL now, but leave mesh-owned examples for a later fixture.

## Contract Changes

- The core ontology contract changes:
  - remove `ReferentMetadata`
  - remove `hasReferentMetadata`
  - add `ReferenceCatalog`
  - add `hasReferenceCatalog`
- SHACL changes:
  - remove any future dependency on `ReferentMetadata`
  - allow optional `hasReferenceCatalog` on `Knop`
  - allow optional `hasReferenceCatalog` on `SemanticMesh`
  - validate `ReferenceCatalog` typing and `ReferenceLink` ownership patterns
- Example and conformance contracts change:
  - Alice Bio `08` should use `ReferenceCatalog`
  - Alice Bio `09` should weave `ReferenceCatalog`
  - the corresponding Accord manifests must be updated to match

## Testing

- Validate the updated [semantic-flow-core-ontology.ttl](../semantic-flow-core-ontology.ttl) with `riot --validate`.
- Validate the updated [semantic-flow-core-shacl.ttl](../semantic-flow-core-shacl.ttl) with `riot --validate`.
- Run SHACL against representative graphs that include:
  - a `Knop` with one `ReferenceCatalog`
  - a `SemanticMesh` with one `ReferenceCatalog`
  - a `ReferenceLink` whose `referenceLinkFor` points to the referent rather than the Knop
- Update the Alice Bio fixture and confirm the revised `08` / `09` manifests still validate against Accord SHACL.
- Re-run the relevant Alice Bio Turtle validation and working-vs-latest snapshot equality checks once `09` is updated.

## Non-Goals

- Reintroducing `ArtifactContainer` or any equivalent generic owner superclass
- Turning `ReferenceCatalog` into a general referent-relations bucket
- Preserving `ReferentMetadata` as a deprecated alias in the live core model
- Solving every future relation type that might one day be about a referent
- Defining a new mesh-relative reference-path property in the same pass unless the example work proves it necessary

## Implementation Plan

- [x] Update [semantic-flow-core-ontology.ttl](../semantic-flow-core-ontology.ttl) to remove `ReferentMetadata` and `hasReferentMetadata`.
- [x] Add `ReferenceCatalog` and `hasReferenceCatalog` to [semantic-flow-core-ontology.ttl](../semantic-flow-core-ontology.ttl).
- [x] Update [semantic-flow-core-shacl.ttl](../semantic-flow-core-shacl.ttl) so `Knop` and `SemanticMesh` may each optionally own one `ReferenceCatalog`.
- [x] Add SHACL support that keeps `ReferenceLink` usage aligned with the revised model, especially that `referenceLinkFor` points to the actual subject resource rather than the Knop support object.
- [x] Update [[ont.summary.core]] to replace `ReferentMetadata` with `ReferenceCatalog` in the artifact-level support story.
- [x] Add a decision entry to [[ont.decision-log]] capturing the removal of `ReferentMetadata` and the introduction of `ReferenceCatalog`.
- [x] Update [[ont.use-case.biographical-data-publishing]] so the Alice Bio example reflects a `ReferenceCatalog` artifact rather than referent metadata or inventory-held links.
- [x] Update the Alice Bio fixture task note at [[wd.task.2026.2026-03-25-mesh-alice-bio]] so `08` and `09` reference the new artifact shape.
- [x] Update the framework conformance task note at [[sf.task.2026.2026-03-29-conformance-for-mesh-alice-bio]] if the manifest assumptions need to change.
- [x] Refactor the `08-alice-bio-referenced` fixture state in `mesh-alice-bio` so it introduces a `ReferenceCatalog` artifact and moves the `ReferenceLink` out of Knop inventory.
- [x] Correct the `ReferenceLink` semantics in the Alice Bio example so `referenceLinkFor` points at the referent resource.
- [x] Update [08-alice-bio-referenced.jsonld](../../semantic-flow-framework/examples/alice-bio/conformance/08-alice-bio-referenced.jsonld) to match the refactored `08` branch.
- [x] Refactor the `09-alice-bio-referenced-woven` target so it weaves the `ReferenceCatalog` artifact rather than inventing more inventory-local link state.
- [x] Update the `09` conformance manifest once the woven reference-catalog branch is settled.
