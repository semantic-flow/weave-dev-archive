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

- Moving metadata/config progression exposed one ResourcePage reader that assumed those pointers remained in inventory. It was corrected to combine settled history structure from MeshInventory with mutable progression from MeshMetadata; the ownership split was not weakened.
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

## Closure Receipt

Delivered 2026-08-31.

- Weave PR #69 merged as `8392751`. Initial versioned mesh-support planning now keeps only settled artifact/history/state/manifestation membership in MeshInventory and writes current/latest/next progression for every versioned mesh support artifact into MeshMetadata.
- ResourcePage history and raw-source assembly now reads mesh-support progression from MeshMetadata while retaining settled structure from MeshInventory. This closes the only reader dependency exposed by the ownership move.
- Semantic Flow Framework PR #1 merged as `a748ac9`, aligning exact progression placement assertions, current workspace-rule vocabulary, the carried Alice MeshInventory ordinal, and the nondeterministic observation-time exception.
- Regenerated and Accord-validated Alice `a.03` through `a.30`, sidecar `a.03` through `a.17`, and branch-published `a.02` through `a.15` excluding the independent source-lane rung `a.10`. Published every affected checkpoint ref; branch publication `gh-pages` was replaced under an exact old-SHA lease.
- Merged accepted final rungs into non-branch fixture `main`: Alice `7bd589b` has the exact `a.30-founding-corrected` tree; sidecar `0832430` has the exact `a.17-all-remaining-terms-woven` tree. Branch-fantasy-rules `main` remains the source lane.
- Full Weave `deno task ci` passed with 918 tests, zero failures, and LCOV generated. GitHub CI, npm-lib, CodeQL, and codecov/patch all passed before merge.
- Current regenerated first-Knop inputs contain no MeshInventory-owned `currentArtifactHistory`, `nextHistoryOrdinal`, `latestHistoricalState`, or `nextStateOrdinal`; [[wa.task.2026.2026-08-31_1111-versioned-first-knop-mesh-inventory-append]] is unblocked and stale legacy shapes remain fail-closed rather than silently repaired.

## Implementation Plan

- [x] Record fail-on-old placement tests and exact affected fixture refs.
- [x] Split settled inventory facts from mesh-metadata progression in the initial support producer.
- [x] Run focused/full Weave validation before fixture writes.
- [x] Regenerate and validate every affected fixture tail deliberately.
- [x] Land receipts and unblock the preserved first-Knop append WIP.
