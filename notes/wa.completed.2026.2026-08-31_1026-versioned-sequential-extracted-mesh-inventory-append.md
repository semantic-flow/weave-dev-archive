---
id: 2e680bde-5ea0-4d52-a671-4afa57e32d36
title: Versioned Sequential Extracted MeshInventory Append
desc: 'Preserve carried MeshInventory bytes and facts while weaving one extracted term through the sequential versioned path'
created: 1788197160000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Remove subject-block replacement from the versioned sequential extracted-term MeshInventory renderer.
- Preserve the complete current MeshInventory as an exact byte prefix while appending only the selected term/Knop/page and next MeshInventory-state facts.
- Reuse the bounded owned-fact pattern proven by the batched extracted child, including semantic no-op and fail-closed single-valued conflicts.

## Summary

`renderGenericFirstExtractedKnopWovenMeshInventoryTurtle` still splits the current MeshInventory, chooses parent/page anchors, replaces the target Knop plus MeshInventory artifact/history blocks, and upserts the selected term, pages, next state, manifestation, and files. This is the versioned sequential path used by explicit single targets and by mixed/recursive sets that intentionally do not enter homogeneous batching.

The function currently lacks a `meshBase` argument because its legacy block renderer never parsed RDF. This child should pass the already-known mesh base from `planFirstExtractedKnopWeave`, construct only the facts the sequential versioned path owns, and plan/render them against the original current inventory through `planInventoryAppend` / `renderInventoryAppendPlan`.

Completed 2026-08-31. Implemented at `85343c6` and merged through Weave PR #53 as `12233bc`. The internal renderer now receives `meshBase`, reuses the bounded extracted-term requested-fact builder for one target, and returns the shared planner's exact-prefix append/no-op/conflict result. Parent/page anchors, target/Knop/history replacement, and the now-dead parent-designator helper are removed from this path.

## Discussion

The owned target facts match the batched extracted child for one designator: target ResourcePage claim, Knop type/working-inventory/page facts, inventory LocatedFile types, and target/Knop page types. The versioned arm additionally owns the MeshInventory artifact/history membership, next state/manifestation/file/page facts, and previous-state link.

Parent-designator anchor selection is serialization machinery, not RDF behavior, and should disappear from this path. Canonical output position is replaced by exact carried-prefix preservation plus one self-contained suffix. Current-only sequential extracted weave continues through the shared `renderFirstPayloadWovenCurrentOnlyMeshInventoryTurtle` and is explicitly outside this child.

## Open Issues

- None requiring a ruling. If the direct path owns a fact not present in the batched request shape, identify and test that fact rather than carrying anchor/block behavior forward.

## Decisions

- Add `meshBase` to the internal renderer signature and keep the exported/runtime/public contracts unchanged.
- Reuse the shared append planner/renderer and the same single-valued predicate classification as the batched extracted path.
- Preserve sequential routing, target selection, page generation, history-index regeneration, and MeshMetadata progression unchanged.

## Contract Changes

- Versioned sequential extracted-term weave preserves arbitrary carried MeshInventory bytes and target-local facts instead of replacing their subject blocks.
- Missing owned facts append; exact duplicates no-op at the renderer boundary; contradictory single-valued facts refuse before runtime writes.
- Output byte placement changes intentionally while the selected designator, created/updated paths, history progression, generated pages, and owned RDF facts remain equivalent.

## Testing

- Fail-on-old exact-prefix test with opaque target facts, comments, repeated subject blocks, and a blank-node subgraph.
- Fail-on-old working-inventory-locator conflict test with requested/existing fact diagnostics.
- Exact semantic-union test for one versioned sequential target and one next MeshInventory state.
- Direct semantic no-op test returning exact input bytes.
- Runtime/integration zero-write conflict test using an explicit extracted target; retain mixed/recursive sequential-routing and history-index regressions.
- Run focused tests first, then `deno task fmt:check`, `deno task lint`, `deno task check`, `deno task test`, and `deno task ci` before landing.

Fail-on-old receipt: exact-prefix versioned union, exact semantic no-op, conflicting Knop inventory locator, and explicit-target runtime zero-write tests produced 0 passed / 4 failed against unchanged production code.

Implementation receipts at `85343c6`:

- sequential renderer tests: 3 passed / 0 failed;
- combined extracted renderer plus pending-heavy suites: 20 passed / 0 failed;
- complete core weave suite: 112 passed / 0 failed;
- focused extracted-bob runtime fixture: 1 passed / 0 failed;
- `deno task test`: 899 passed / 0 failed;
- `deno task fmt:check`, `deno task lint`, and `deno task check`: green;
- `deno task ci`: 899 passed / 0 failed, LCOV generated; only the known deleted-temporary-source coverage notices appeared.
- GitHub PR #53: CI, npm-lib, CodeQL, and patch coverage passed; CodeRabbit accepted the lane under its review-rate-limit path with no findings.

Three serialization-shaped core assertions and one integration fixture were converted to exact-prefix plus RDF-semantic acceptance. Sequential selection, mixed/recursive routing, history-index regeneration, source evidence, root/nested targets, and prior snapshot behavior remain green.

## Non-Goals

- No current-only extracted, homogeneous batch, first-Knop, first/later payload, PageDefinition, extract, integrate, or mesh-support renderer migration.
- No current-mode extraction-source support, candidate classification, batch eligibility, progression storage, fixture topology, ontology, CLI, or public API change.
- No OS-level append, repair/retraction surface, or general Turtle canonicalization.

## Implementation Plan

- [x] Record exact-prefix, conflict, no-op, and versioned-union tests failing on current `main` before production edits.
- [x] Add the internal mesh-base input and construct only sequential extracted owned facts.
- [x] Plan/render against the original current MeshInventory and delete obsolete anchor/block machinery when no longer used.
- [x] Add explicit-target runtime zero-write conflict coverage and retain sequential/history regressions.
- [x] Update developer guidance and board the byte-shape change here for the next release notes; no next-release stub exists yet.
- [x] Run focused/full validation and return the plan-level delta: implementation is green; review/landing remains before the next child is cut.
