---
id: 2c83759d-b0f6-4fb7-b7c7-61da3cb6a096
title: Batched Extracted MeshInventory Append
desc: 'Preserve carried MeshInventory bytes and facts while adding a homogeneous extracted-term batch'
created: 1788191100000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Remove target-block deletion and reconstruction from batched untargeted extracted-term MeshInventory writes.
- Preserve the complete current MeshInventory byte-for-byte as the output prefix, including unknown facts, comments, prefixes, repeated subject blocks, and blank nodes.
- Append only missing planner-approved term, Knop, page, and optional MeshInventory-history facts; fail before writes on conflicting single-valued settled facts.

## Summary

`renderBatchedFirstExtractedKnopWovenMeshInventoryTurtle` is the highest-risk remaining MeshInventory renderer. It sorts target designators, splits the current Turtle into blocks, removes every target subject block and matching `_mesh hasKnop` block, reconstructs those facts from templates, and then replaces MeshInventory artifact/history blocks when history is versioned. Any target-local fact Weave does not know how to re-render can disappear.

The completed add-reference child proved the migration pattern: retain existing shape validation, render the supported desired graph as a bounded requested-fact source, prepare the original current inventory, and let `planInventoryAppend` / `renderInventoryAppendPlan` produce the exact original prefix plus only missing facts. This child applies that pattern only to the homogeneous batched extracted-term renderer used by PR #41's scale path.

Completed 2026-08-31. Implemented at `1770767` and merged through Weave PR #51 as `14090ba`. The renderer now constructs only its owned term, Knop, page, and optional MeshInventory history/state facts, prepares the original current inventory, and returns the shared planner's exact-prefix append/no-op/conflict result. The old target-block filter, reconstruction helpers, and anchor-based insertion path are removed.

## Discussion

The existing replacement renderer may be extracted as a private desired-graph builder for this bite. Its bytes must never become the returned inventory. Parse its facts as the request, plan them against the original current MeshInventory, and render the append through the shared proof-checked renderer.

For a versioned MeshInventory, the requested graph includes settled history/state/manifestation membership and generated-page facts. Current/latest/next progression remains in `_mesh/_meta/meta.ttl` through the existing separate renderer and is not moved here. For current-only policy, only term/Knop/file/page membership facts append.

## Open Issues

- None requiring a ruling for this bite. If the old desired graph contains mutable progression facts or a fact whose cardinality cannot be classified safely, report the exact predicate and stop rather than broadening the plan.

## Decisions

- Preserve canonical target ordering and current-only/versioned output semantics from the landed extracted-term batch path.
- Use the shared append planner and renderer; do not create a MeshInventory-specific mutation abstraction.
- Keep MeshMetadata progression, page generation, candidate classification, and batch eligibility unchanged.

## Contract Changes

- A successful homogeneous extracted-term batch preserves arbitrary carried MeshInventory bytes as an exact prefix instead of deleting and reconstructing target blocks.
- Missing settled facts append in a self-contained suffix; contradictory single-valued settled facts refuse before runtime writes with requested and existing facts named.
- Output byte ordering changes intentionally while the RDF graph, result paths, candidate ordering, MeshInventory state cardinality, and generated pages remain equivalent.

## Testing

- Fail-on-old target-block test: carry a comment, opaque predicate, repeated subject block, and blank node on a batched extracted target; require the result to start with the exact input and retain the full graph.
- Fail-on-old conflict test: carry a contradictory single-valued Knop inventory-file locator that the old reconstruction silently replaces; require the requested/existing-fact diagnostic.
- Exact semantic-union tests for current-only and versioned MeshInventory policies, including reverse-discovery canonical target order and one versioned MeshInventory state.
- Planner-level no-op test where every requested fact already exists; require exact input bytes.
- Runtime/integration zero-write conflict coverage and the existing N=40/120 extracted batch/memory regression.
- Run focused tests first, then `deno task fmt:check`, `deno task lint`, `deno task check`, `deno task test`, and `deno task ci` before landing.

Fail-on-old receipt: exact-prefix append, exact semantic no-op, conflicting Knop inventory locator, and versioned graph-union tests produced 0 passed / 4 failed against unchanged production code. The old renderer removed/reordered target blocks, normalized away a carried prefix, accepted the contradictory locator, and rewrote versioned subjects.

Implementation receipts at `1770767`:

- new renderer tests: 4 passed / 0 failed;
- pending-heavy batch/memory suite: 12 passed / 0 failed, including N=40/120 retained-growth coverage and runtime zero-write conflict refusal;
- focused reverse-order core batch regression: 1 passed / 0 failed;
- `deno task test`: 895 passed / 0 failed;
- `deno task fmt:check`, `deno task lint`, and `deno task check`: green;
- `deno task ci`: 895 passed / 0 failed, LCOV generated; only the known deleted-temporary-source coverage notices appeared.
- GitHub PR #51: CI, npm-lib, CodeQL, and patch coverage passed; CodeRabbit completed with no actionable comments and low merge risk. Its generic private-helper docstring warning is not a repository gate and did not justify style-only comments.

Exact-prefix coverage carries a nonstandard prefix, target-local comments, opaque predicates, repeated subject blocks, and blank-node subgraphs. Semantic-union coverage spans current-only and versioned policy, asserts one next MeshInventory state membership, and confirms mutable progression predicates remain outside inventory.

## Non-Goals

- No sequential extracted, first/later payload, first-Knop, PageDefinition, extract, integrate, or mesh-support renderer migration.
- No candidate eligibility, ordering, page generation, history policy, progression storage, fixture topology, ontology, CLI, or public API change.
- No OS-level append, general Turtle canonicalization, repair/retraction mode, or compatibility shim.

## Implementation Plan

- [x] Record exact-prefix, conflict, no-op, and graph-union tests failing on current `main` before production edits.
- [x] Replace whole-document desired rendering with a bounded requested-fact graph without changing owned RDF semantics.
- [x] Prepare the original MeshInventory and plan/render requested facts through the shared append path.
- [x] Cover current-only and versioned batches plus runtime zero-write conflict and existing scale regressions.
- [x] Update developer guidance and board the intentional byte-shape change here for the next release notes; no next-release stub exists yet.
- [x] Run focused/full validation and return the plan-level delta: implementation is green; review/landing remains before the next child is cut.
