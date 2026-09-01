---
id: da42352e-35a9-42f5-81bf-85ba8be94c1b
title: Initial Versioned Mesh-Support Inventory Append
desc: 'Append initial settled mesh-support history and page facts without rewriting carried MeshInventory blocks'
created: 1788235271000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Route the history-bearing arm of `planInitialMeshSupportResourcePageWeave` through the shared inventory append planner.
- Preserve the exact pre-weave MeshInventory bytes/facts while appending initial settled metadata, inventory, optional config, history/state/manifestation, located-file, and ResourcePage facts.
- Retain the already-ruled MeshMetadata progression placement and the migrated page-only/current-only behavior.

## Summary

After versioned first-payload closure, `src/core/weave/mesh_support_pages.ts` contains the remaining current MeshInventory block mutation used by ordinary live fixtures. `planInitialMeshSupportResourcePageWeave` splits and normalizes the carried inventory, replaces `_mesh` and every support artifact block, and upserts initial history/page/file blocks. The 2026-08-31 producer correction removed mutable progression from those blocks, but unknown settled facts on `_mesh`, `_mesh/_meta`, `_mesh/_inventory`, or `_mesh/_config` can still be lost.

The no-initial-history page-only arm already uses `planInventoryAppend`; this child migrates only the initial history-bearing arm.

## Discussion

Build requested settled facts from the existing support-resource model:

- mesh/support ResourcePage claims and page file types
- `hasArtifactHistory`, history ordinal, and state membership
- historical state/manifestation relationships and ordinals
- snapshot located-file declarations

Do not put `currentArtifactHistory`, `nextHistoryOrdinal`, `latestHistoricalState`, or `nextStateOrdinal` back into MeshInventory. Continue rendering those pointers for every versioned support artifact in `_mesh/_meta/meta.ttl`, including the metadata self-snapshot behavior already covered by [[wa.completed.2026.2026-08-31_1714-mesh-support-progression-producer-correction]].

## Open Issues

- The metadata self-snapshot and inventory snapshot bytes must remain exactly the current planned bytes after append rendering. Report any actual recursion rather than weakening the ownership split.
- If mixed versioned/current-only policy needs two append passes, preserve one prepared carried prefix and one final append plan; do not reintroduce subject-block mutation for convenience.

## Decisions

- Reuse `prepareCurrentInventory`, `planInventoryAppend`, and `renderInventoryAppendPlan`.
- Generate no blank nodes. Carried config-rule or other blank-node subgraphs remain opaque prefix data.
- Stale inventory-owned progression fails closed pending explicit repair; ordinary weave never deletes it as a hidden migration.
- New-file metadata/config/inventory snapshots are not themselves append-migration targets; exactness is measured against the final planned current documents.

## Contract Changes

- Initial versioned mesh-support MeshInventory growth becomes append/no-op/fail-closed instead of block replacement.
- Unknown carried facts/comments/repeated blocks survive byte-for-byte.
- Output ordering and affected fixture bytes change intentionally; progression ownership and public command/result surfaces do not.

## Testing

- Fail-on-old exact-prefix/semantic-union test with opaque `_mesh` and support-subject facts, comments, repeated blocks, and carried blank-node subgraphs.
- Exact no-op and single-valued conflict/stale-progression refusal coverage.
- Versioned, current-only, config-present/absent, and mixed-policy cases with exact metadata/inventory snapshot assertions.
- Runtime zero-write refusal coverage plus existing support-page and mesh-create/weave regressions.
- Regenerate and Accord-validate substantively affected Alice, sidecar, and branch-published tails from their initial support-history rung; discard timestamp-only rehearsal output.
- Run `deno task ci` and `deno task build:npm-lib` before landing.

## Non-Goals

- No KnopInventory/payload/PageDefinition migration, legacy extract/raw import disposition, progression-owner ruling, fixture topology, ontology, CLI, public API, repair/retraction, or OS-level append work.

## Implementation Plan

- [ ] Record fail-on-old prefix, no-op, conflict, stale-progression, and mixed-policy tests.
- [ ] Build one bounded requested-fact document for initial settled support history/page membership.
- [ ] Route the history-bearing arm through the shared prepared append path and remove dead local block mutation helpers.
- [ ] Prove snapshot/self-snapshot bytes and runtime zero-write refusal behavior.
- [ ] Regenerate/publish only substantive fixture changes and update durable guidance/release-note boarding.
- [ ] Run full gates and return the remaining-writer delta.
