---
id: d91d162b-bd15-41d1-9034-be6d7442581e
title: Current-Only Payload-Like MeshInventory Append
desc: 'Give shared current-only first-payload and extracted-term page claims exact append/no-op semantics'
created: 1788198600000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Route `renderFirstPayloadWovenCurrentOnlyMeshInventoryTurtle` through the shared inventory append planner.
- Preserve the exact carried MeshInventory bytes, including trailing bytes, while appending only missing mesh membership and target/Knop page facts.
- Return exact input bytes when every requested fact already exists, without duplicating ResourcePage subject declarations.

## Summary

The shared current-only renderer already avoids subject-block replacement, but it still performs its own RDF membership scan, trims the current file, and unconditionally appends ResourcePage subject blocks. Repeated helper use can therefore duplicate page declarations, and a semantic no-op still changes bytes/trailing-newline shape.

This function serves both current-only first-payload weave and current-only sequential/homogeneous extracted-term planning. Migrating it now closes the current-only counterpart to the two extracted versioned/batch children without changing candidate selection or extraction-source resolution.

Implemented on `lane/current-only-payload-like-mesh-inventory-append` at `200b6e9`; awaiting review and landing. The helper now prepares the original MeshInventory, requests only mesh membership plus target/Knop page claims and page types, and returns the shared planner's exact-prefix append/no-op result. The bespoke quad-membership scan, unconditional page-block append, and `trimEnd` byte churn are removed.

## Discussion

The renderer owns five additive fact groups for one designator: `_mesh hasKnop D/_knop`; `D hasResourcePage D/index.html`; `D/_knop hasResourcePage D/_knop/index.html`; and the two ResourcePage/LocatedFile type declarations. The existing Knop type and working-inventory locator are validated earlier and are not rewritten or restated by this helper.

All requested facts use named-node subjects and objects. The helper emits no blank nodes. Any blank-node subgraphs already present in the current inventory remain untouched prefix bytes and are not submitted as append requests.

## Open Issues

- None. This path requests only multi-valued membership/type/page facts, so no new single-valued conflict classification is needed in this child.

## Decisions

- Reuse `prepareCurrentInventory`, `planInventoryAppend`, and `renderInventoryAppendPlan`; remove the helper's bespoke quad-membership scan.
- Keep the shared function name and all current-only routing unchanged.
- Preserve current-only semantics: no MeshInventory history/state facts or MeshMetadata progression are introduced.

## Contract Changes

- Current-only first-payload/extracted MeshInventory page claims preserve exact current bytes and append one self-contained suffix only when facts are missing.
- A semantic duplicate request returns the exact current file, including its original trailing bytes.
- RDF outcomes and public/runtime result paths remain unchanged; only duplicate/churn behavior is tightened.

## Testing

- Fail-on-old exact-prefix test with custom prefixes, comments, opaque facts, trailing-byte shape, and a carried blank-node subgraph.
- Fail-on-old exact semantic no-op test requiring byte identity when every requested fact exists.
- Exact semantic-union test proving only the five owned fact groups are added.
- Integration coverage for current-only first-payload and current-only extracted paths, retaining zero MeshInventory history states.
- Run focused tests first, then `deno task fmt:check`, `deno task lint`, `deno task check`, `deno task test`, and `deno task ci` before landing.

Fail-on-old receipt: exact-prefix append with carried trailing spaces and exact semantic no-op produced 0 passed / 2 failed against unchanged production code. The old helper trimmed the carried bytes and duplicated ResourcePage subject declarations.

Implementation receipts at `200b6e9`:

- direct current-only renderer tests: 2 passed / 0 failed;
- current-only first-payload core regression: 1 passed / 0 failed;
- pending-heavy current-only/versioned extracted suite: 13 passed / 0 failed;
- `deno task test`: 902 passed / 0 failed;
- `deno task fmt:check`, `deno task lint`, and `deno task check`: green;
- `deno task ci`: 902 passed / 0 failed, LCOV generated; only the known deleted-temporary-source coverage notices appeared.

The requested suffix contains only named-node facts. The adversarial current input retains its opaque blank-node subgraphs solely through exact prefix preservation; no blank node is generated or submitted as a requested append fact.

## Non-Goals

- No versioned MeshInventory, first-Knop, later-payload, PageDefinition, extract, integrate, or mesh-support renderer migration.
- No current-mode extraction-source support, candidate classification, history policy, progression storage, fixture topology, ontology, CLI, or public API change.
- No new blank-node generation, blank-node validation policy, OS-level append, or repair/retraction surface.

## Implementation Plan

- [x] Record exact-prefix, no-op, and semantic-union tests failing on current `main` before production edits.
- [x] Replace the bespoke membership scan/string append with one bounded requested-fact plan.
- [x] Prove shared first-payload and extracted current-only integrations remain history-free and semantically stable.
- [x] Update developer guidance and board the no-op/output-byte change here for the next release notes; no next-release stub exists yet.
- [x] Run focused/full validation and return the plan-level delta: implementation is green; review/landing remains before the next child is cut.
