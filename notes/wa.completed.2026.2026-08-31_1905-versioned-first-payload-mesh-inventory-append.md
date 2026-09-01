---
id: 79d58a98-593b-4ce0-a237-c66386bbe6c8
title: Versioned First-Payload MeshInventory Append
desc: 'Preserve carried MeshInventory bytes and facts across singular and batched versioned first-payload weave'
created: 1788228331000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Route `renderFirstPayloadWovenMeshInventoryTurtle` through the shared append planner and preserve the original MeshInventory as the exact output prefix.
- Cover both singular versioned first-payload weave and `renderBatchedFirstPayloadWovenMeshInventoryTurtle`, which delegates each versioned target to the singular renderer.
- Remove the versioned first-payload mesh-root/artifact/Knop/history block replacement and complete legacy fallback after current accepted shapes are proven sufficient.

## Summary

The fresh post-first-Knop writer audit on 2026-08-31 found the next concentrated MeshInventory rewrite in `src/core/weave/mesh_inventory_renderers.ts`. `renderFirstPayloadWovenMeshInventoryTurtle` still splits the whole inventory into blocks, reconstructs `_mesh`, replaces the target payload, Knop, MeshInventory artifact/history, and several page blocks, and falls back to a complete hard-coded inventory when expected blocks are absent. Arbitrary carried facts on any replaced subject can be lost.

The batched first-payload renderer folds the singular renderer across targets, so migrating the singular primitive closes both paths without changing batch routing. Current-only first-payload/extracted page claims already use the append planner and are not part of this child.

## Discussion

Build one requested settled-fact document containing only:

- `_mesh hasKnop D/_knop`
- payload artifact types, current working locator, optional deterministic repository floating locator, and ResourcePage claim
- Knop type, working inventory locator, and ResourcePage claim
- working located-file declarations
- next MeshInventory state/manifestation membership and ResourcePage/file facts

MeshInventory progression remains authoritative in MeshMetadata. As with [[wa.completed.2026.2026-08-31_1111-versioned-first-knop-mesh-inventory-append]], current first-payload inputs carrying any inventory-owned `currentArtifactHistory`, `nextHistoryOrdinal`, `latestHistoricalState`, or `nextStateOrdinal` must fail before writes rather than be retained or silently repaired.

## Open Issues

- **RESOLVED:** the implementation stayed inside MeshInventory rendering, fact/locator validation, tests, and fixtures; candidate classification, current-mode extracted behavior, diagnostics, and batch selection were unchanged.
- **RESOLVED:** every live current fixture reached the bounded append path. The complete legacy fallback and its block/root reconstruction helpers were removed.

## Decisions

- Reuse `prepareCurrentInventory`, `planInventoryAppend`, and `renderInventoryAppendPlan`; do not introduce another append abstraction.
- Repository floating locators retain the ruled named `<D/_knop/_sources#payload-source-repository-locator>` identity. This child generates no blank nodes.
- Preserve payload RDF/non-RDF typing, MeshMetadata progression, result paths, and candidate/batch behavior.
- Treat output ordering and the retention of previously dropped carried facts as intentional fixture changes.

## Contract Changes

- Versioned first-payload MeshInventory updates become append/no-op/fail-closed rather than subject-block replacement.
- Singular and batched versioned first-payload outputs retain all carried bytes/facts as the exact prefix.
- Conflicting single-valued working locators and stale inventory-owned progression refuse before runtime writes.

## Testing

- Fail-on-old exact-prefix and semantic-union coverage with comments, repeated subjects, opaque facts, and a carried blank-node subgraph.
- Exact no-op and conflicting working-locator tests for local-file and named floating-repository payloads.
- Singular and multi-target versioned first-payload coverage proving deterministic accumulated append facts and one MeshInventory progression chain.
- Runtime zero-write conflict/stale-progression tests plus existing first-payload batch and nested-source regressions.
- Regenerate and Accord-validate each substantively affected live fixture tail from its earliest versioned first-payload weave rung; discard timestamp-only rehearsals.
- Run `deno task ci` and `deno task build:npm-lib` before landing.

## Non-Goals

- No payload/KnopInventory renderer migration, later-payload advancement, current-only first-payload changes, candidate routing, diagnostics, page-generation policy, ontology, CLI, or public API change.
- No repair/retraction surface, compatibility shim, fixture topology redesign, OS-level append, or blank-node policy change.

## Closure Receipt

Delivered 2026-08-31.

- Weave PR #73 merged as `eac9167` after 932-test CI, npm-lib, CodeQL, codecov/patch, and CodeRabbit review. Both review nitpicks landed: parsed-RDF negative assertions and shared locator-term derivation between rendering and compatibility validation.
- `renderFirstPayloadWovenMeshInventoryTurtle` now prepares the carried MeshInventory once, requests only payload/Knop/page/locator/next-state facts, and renders through the shared append planner. The delegated versioned batch path accumulates targets on the same exact carried prefix and emits one MeshInventory progression graph.
- The renderer returns exact input bytes for semantic no-op and rejects conflicting local/repository locator values, conflicting locator kinds, and stale inventory-owned progression before writes. Runtime tests prove locator/progression refusals leave the workspace byte-identical.
- Local, extra-mesh literal, named floating-repository, RDF, non-RDF, binary, root, nested, singular, and batch paths retain their existing candidate/result behavior. The implementation generates no blank nodes; carried blank-node subgraphs remain untouched prefix data.
- Regenerated, Accord-validated, and published the substantively affected Alice `a.07` through `a.30` tail. Alice fixture `main` merge `ef90916` has the exact accepted `a.30-founding-corrected` tree. Sidecar and branch-published rehearsals changed only generated HTML timestamps and were deliberately discarded.
- Full local `deno task ci` passed with 932 tests, zero failures, and LCOV generated; `deno task build:npm-lib` passed. [[wd.codebase-overview]] and [[wd.todo]] carry durable behavior and release-note boarding.

## Implementation Plan

- [x] Record fail-on-old singular/batched prefix, no-op, conflict, and stale-progression tests.
- [x] Build bounded first-payload requested facts and route the singular renderer through the shared append planner.
- [x] Prove batched accumulation and remove dead block/fallback helpers only when no sibling renderer uses them.
- [x] Add runtime zero-write coverage and retain existing first-payload planning/fixture behavior.
- [x] Regenerate only substantively affected fixture tails and publish accepted refs.
- [x] Update durable guidance/release-note boarding and run full gates.
