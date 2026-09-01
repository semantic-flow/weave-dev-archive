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

Weave PR #75 merged as `34b97a3`. `planInitialMeshSupportResourcePageWeave` now prepares the carried MeshInventory once, builds one bounded named-node fact request for every applicable support policy, and delegates append/no-op/conflict behavior to the shared planner. The exact carried bytes remain the output prefix, including comments, repeated support-subject blocks, opaque facts, trailing bytes, and carried blank-node subgraphs.

The already-migrated page-only arm shares the same prepared value and stale-progression check. Initial-history detection now uses settled `hasArtifactHistory` relationships rather than the removed mutable pointer, so a second plan over the first plan's exact inventory and metadata outputs creates or updates nothing.

## Discussion

Build requested settled facts from the existing support-resource model:

- mesh/support ResourcePage claims and page file types
- `hasArtifactHistory`, history ordinal, and state membership
- historical state/manifestation relationships and ordinals
- snapshot located-file declarations

Do not put `currentArtifactHistory`, `nextHistoryOrdinal`, `latestHistoricalState`, or `nextStateOrdinal` back into MeshInventory. Continue rendering those pointers for every versioned support artifact in `_mesh/_meta/meta.ttl`, including the metadata self-snapshot behavior already covered by [[wa.completed.2026.2026-08-31_1714-mesh-support-progression-producer-correction]].

## Resolved Issues

- Metadata self-snapshot and inventory snapshot bytes equal the final planned current documents; no recursion was required.
- Mixed versioned/current-only policy uses one fact document and one append plan. Current-only resources receive page facts, while only versioned resources receive history/state/manifestation/snapshot facts and MeshMetadata progression.
- Review caught and closed the removed-pointer sentinel gap: settled `hasArtifactHistory` now selects the already-initialized path, while legacy inventory-owned mutable progression still refuses before branch selection.

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

- [x] Record fail-on-old prefix, no-op, conflict, stale-progression, mixed-policy, and repeated-plan tests.
- [x] Build one bounded requested-fact document for initial settled support history/page membership.
- [x] Route the history-bearing arm through the shared prepared append path and remove dead local block mutation helpers.
- [x] Prove snapshot/self-snapshot bytes and runtime zero-write refusal behavior.
- [x] Regenerate/publish the substantively affected fixture tails and update durable guidance/release-note boarding.
- [x] Run full gates and return the remaining-writer delta.

## Completion Receipt

- Weave: PR #75, merge `34b97a3`; implementation commits `db1d811` and `29e61cc`.
- Verification: local and GitHub CI each passed 939 tests; format, lint, type-check, coverage, npm-library build, CodeQL, and codecov/patch passed.
- Review: CodeRabbit's checklist-format nit and repeated-initialization finding were fixed; the review-fix rerun was rate-limited but all mandatory rerun gates passed.
- Fixtures: Accord accepted and the fixture repos published Alice `a.03`–`a.30`, sidecar `a.03`–`a.17`, and branch-published `a.02`–`a.15` plus `gh-pages`. Alice and sidecar `main` exactly match their accepted final rung trees.
- Remaining-writer delta: no current MeshInventory weave arm still block-rewrites the carried document. The parent plan now moves to separate legacy extract/raw import disposition, then the Knop-local progression ownership ruling before remaining versioned KnopInventory-family migration.
