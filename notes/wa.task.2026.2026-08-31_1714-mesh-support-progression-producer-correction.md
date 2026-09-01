---
id: 584a0351-3410-4195-abf8-10732006306a
title: Mesh Support Progression Producer Correction
desc: 'Move mutable initial mesh-support progression out of MeshInventory before fail-closing stale first-Knop inputs'
created: 1788221656764
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Stop `planInitialMeshSupportResourcePageWeave` from emitting `currentArtifactHistory`, `nextHistoryOrdinal`, `latestHistoricalState`, and `nextStateOrdinal` into MeshInventory.
- Keep settled mesh-support artifact/history/state/manifestation membership in MeshInventory while moving mutable progression for mesh metadata, inventory, and optional config into `_mesh/_meta/meta.ttl`.
- Regenerate affected fixture ladders from their earliest initial-support weave rung and unblock [[wa.task.2026.2026-08-31_1111-versioned-first-knop-mesh-inventory-append]].

## Summary

Dave ruled option (B) on 2026-08-31: correct producers and fixtures first, then make later append paths fail closed on stale legacy shapes pending explicit repair/regeneration. The current initial mesh-support history producer still calls `appendInitialSupportHistoryFactsToBlock` and `renderInitialSupportHistoryBlock`, which place both settled membership and mutable current/latest/next pointers in MeshInventory. It separately writes only MeshInventory progression to MeshMetadata, leaving metadata/config progression without the ruled authoritative home.

This producer creates the stale Alice `a.03-mesh-created-woven` shape that is inherited by `a.04` and exposes the first-Knop append conflict. Fixture correction without producer correction would merely recreate the defect.

## Discussion

Split initial support facts by responsibility:

- MeshInventory retains `hasArtifactHistory`, `historyOrdinal`, `hasHistoricalState`, manifestation/file relationships, ResourcePage claims, and types.
- MeshMetadata carries `currentArtifactHistory` / `nextHistoryOrdinal` on each versioned mesh-support artifact and `latestHistoricalState` / `nextStateOrdinal` on each selected history.

The updated metadata snapshot for `_mesh/_meta` should remain the same bytes written as the current metadata file, preserving the existing self-snapshot behavior. Current-only support policies remain history-free and must not gain progression.

After core behavior is green, identify every fixture scenario whose first support-page weave invokes this producer. Regenerate from the earliest affected rung through each dependent tail using the fixture-ladder workflow and exact Accord manifests; do not hand-edit branch outputs.

## Open Issues

- If moving metadata/config progression into MeshMetadata exposes a recursive snapshot inconsistency or a reader that assumes those pointers remain in inventory, report it before weakening the ruled ownership split.
- Broad published SFLO/URPX release regeneration remains release-owned and is not implied by fixture reruns.

## Decisions

- Producer correction precedes fixture correction and first-Knop append.
- Ordinary operations never delete stale pointers as a hidden migration.
- Post-correction stale shapes fail closed; explicit repair/regeneration is a separate later surface.
- All mesh-owned support progression belongs in `_mesh/_meta/meta.ttl`; inventory keeps settled membership only.

## Contract Changes

- Initial versioned mesh-support weave changes MeshInventory and MeshMetadata RDF placement.
- Generated current/snapshot metadata bytes gain authoritative progression for every versioned mesh-owned support artifact.
- Affected fixture branches change deliberately from their earliest producer rung onward.
- Current-only policy behavior and public command/result surfaces remain unchanged.

## Testing

- Fail-on-old tests proving MeshInventory omits all four mutable predicates for metadata, inventory, and config while retaining settled membership.
- Exact metadata assertions for current/latest/next progression on every versioned mesh-support artifact.
- Mixed versioned/current-only policy coverage and semantic rerun behavior.
- Fixture-ladder regeneration plus Accord validation for every affected tail.
- Proof that the corrected first-Knop input reaches its append child without stale progression.
- Full `deno task ci` before landing.

## Non-Goals

- No first-Knop append WIP, named-locator work, Knop-owned progression migration, explicit repair command, release publication, ontology change, or general fixture topology redesign.

## Implementation Plan

- [ ] Record fail-on-old placement tests and exact affected fixture refs.
- [ ] Split settled inventory facts from mesh-metadata progression in the initial support producer.
- [ ] Run focused/full Weave validation before fixture writes.
- [ ] Regenerate and validate every affected fixture tail deliberately.
- [ ] Land receipts and unblock the preserved first-Knop append WIP.
