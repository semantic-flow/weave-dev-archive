---
id: 6c7b696a-f3de-426b-b304-245e1b9ec5c9
title: Versioned First-Knop MeshInventory Append
desc: 'Preserve carried MeshInventory bytes and facts while weaving first-Knop support history'
created: 1788199860000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Route versioned first-Knop MeshInventory growth through the shared append planner.
- Preserve the exact carried MeshInventory while appending only new mesh membership, identifier/Knop/page, and MeshInventory-history facts.
- Remove mesh-root, target, Knop, inventory artifact/history, and page subject-block replacement from `renderFirstKnopWovenMeshInventoryTurtle`.

## Summary

The versioned first-Knop renderer still parses Turtle into blocks, reconstructs the `_mesh` subject to preserve known Knop memberships, replaces MeshInventory artifact/history and target Knop blocks, chooses insertion anchors, and falls back to a complete legacy template when expected blocks are absent. Unknown mesh-root, target, Knop, or history facts can be dropped.

All facts this path adds use named-node/literal terms: `_mesh hasKnop`, identifier and Knop page claims, Knop type/working-inventory locator, inventory/page file types, and the next MeshInventory state/manifestation membership. It has no repository locator dependency and can proceed while the plan's generated-blank-node ruling remains open.

Paused 2026-08-31 before commit. The fail-on-old tests were implemented and the append path itself passed, but the real Alice `a.04` → `a.05` acceptance comparison exposed an incompatible responsibility: `a.04` carries `currentArtifactHistory`, `nextHistoryOrdinal`, `latestHistoricalState`, and `nextStateOrdinal` in MeshInventory, and the old first-Knop renderer intentionally deletes them while `_mesh/_meta` becomes authoritative. Exact-prefix preservation would retain forbidden mutable progression facts. WIP is preserved locally on `lane/versioned-first-knop-mesh-inventory-append` as stash `wip: first-Knop append blocked by progression migration ruling`.

Dave ruled option (B) on 2026-08-31. [[wa.completed.2026.2026-08-31_1714-mesh-support-progression-producer-correction]] has now stopped Weave from creating the stale pointers and regenerated every affected fixture tail. This child is unblocked; it must reject any remaining old shape before writes instead of silently deleting or grandfathering those facts.

## Discussion

Use a bounded first-Knop requested-fact builder rather than passing the legacy complete document to `planInventoryAppend`. The append planner should make anchor selection, root-block reconstruction, and legacy fallback unnecessary for valid prepared inventory input.

The existing first-Knop shape assertions and progression resolver remain the acceptance boundary. If the legacy fallback is reachable from a currently accepted shape that lacks facts the bounded request does not own, capture that fixture and report it rather than retaining an untested whole-document renderer.

## Open Issues

- **RESOLVED:** [[wa.completed.2026.2026-08-31_1714-mesh-support-progression-producer-correction]] stopped emitting stale inventory-owned progression. This child now rejects any remaining legacy pointers before append planning and runtime writes.

## Decisions

- Emit only named-node/literal requested facts and reuse the shared append planner/renderer.
- Keep versioned history policy, MeshMetadata progression, generated pages, and result paths unchanged.
- Remove dead block/root reconstruction helpers only when no sibling renderer still uses them.

## Contract Changes

- Versioned first-Knop weave preserves arbitrary carried MeshInventory facts/bytes instead of replacing owned subject blocks.
- Missing facts append; semantic duplicates no-op at the renderer boundary; contradictory single-valued locators/progression ordinals refuse before writes.
- Output ordering changes intentionally while owned RDF facts, one-state progression, pages, and result paths remain equivalent.

## Testing

- Fail-on-old exact-prefix test carrying opaque `_mesh`, identifier, Knop, history facts, comments, and a blank-node subgraph.
- Fail-on-old working-inventory-locator conflict test.
- Exact semantic-union and exact no-op tests for one versioned first-Knop transition.
- Runtime zero-write conflict coverage plus existing first-Knop/current-history fixture tests.
- Run focused tests first, then `deno task fmt:check`, `deno task lint`, `deno task check`, `deno task test`, and `deno task ci` before landing.

## Non-Goals

- No payload, extracted, PageDefinition, extract, integrate, mesh-support, repository-locator, or progression-storage migration.
- No blank-node generation/acceptance change, candidate routing, fixture topology, ontology, CLI, or public API change.
- No OS-level append or repair/retraction surface.

## Closure Receipt

Delivered 2026-08-31.

- Weave PR #71 merged as `7382eeb` after green CI, npm-lib, CodeQL, codecov/patch, and a no-finding CodeRabbit review.
- `renderFirstKnopWovenMeshInventoryTurtle` now prepares the original MeshInventory once, requests only first-Knop membership/page/next-state facts, and renders through the shared append planner. The legacy block/root reconstruction and complete fallback template are removed.
- Exact-prefix and semantic-union tests carry comments, repeated subject blocks, opaque facts, and blank-node subgraphs without generating new blank nodes. Semantic no-op returns exact input bytes; conflicting working-inventory locators refuse.
- A dedicated pre-planning guard rejects `currentArtifactHistory`, `nextHistoryOrdinal`, `latestHistoricalState`, or `nextStateOrdinal` anywhere in a first-Knop MeshInventory input. Runtime tests prove both locator conflict and legacy progression refusal leave the workspace byte-identical.
- Regenerated, Accord-validated, and published the substantively affected Alice `a.05` through `a.30` tail. Alice fixture `main` merge `de53ab2` has the exact accepted `a.30-founding-corrected` tree. Sidecar and branch-published rehearsals changed only generated HTML timestamps and were deliberately discarded rather than published.
- Full local `deno task ci` passed with 924 tests, zero failures, and LCOV generated; `deno task build:npm-lib` passed.
- The first-Knop output-byte/graph change is boarded in [[wd.todo]] for the next release notes, and [[wd.codebase-overview]] records the new append/fail-closed behavior.

## Implementation Plan

- [x] Record exact-prefix, conflict, no-op, and semantic-union tests failing on current `main` before production edits — 0 passed / 4 failed, then the fixture comparison exposed the blocker above.
- [x] Land [[wa.completed.2026.2026-08-31_1714-mesh-support-progression-producer-correction]], then prove current Weave output reaches this child without stale inventory-owned progression.
- [x] Build only first-Knop owned requested facts and plan/render them against the original inventory.
- [x] Remove obsolete fallback/anchor/root reconstruction code proven dead for this path.
- [x] Add runtime zero-write conflict coverage and retain first-Knop progression/page regressions.
- [x] Update developer guidance and board the output-byte change for the next release notes.
- [x] Run focused/full validation and return any plan-level sequencing delta.
