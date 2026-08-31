---
id: b36f1124-b7e3-4421-8f29-90a390bd4492
title: Add-Reference Inventory Append
desc: 'Preserve carried KnopInventory bytes and facts while adding the first ReferenceCatalog'
created: 1788163560000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Remove the live data-loss path in `knop add-reference` that replaces the current KnopInventory with a canonical template.
- Preserve the complete current KnopInventory byte-for-byte as the output prefix, including unknown facts, comments, prefixes, and support artifacts.
- Append only missing planner-approved ReferenceCatalog facts, no-op semantic duplicates at the append helper boundary, and fail before writes on conflicting single-valued settled facts.

## Summary

Audit against Weave `main` at `6643ae1` found that `renderUpdatedKnopInventoryTurtle` classifies the current KnopInventory, renders a new woven/current-only/unwoven template, and then calls `renderKnopInventoryWithPreservedSupportArtifacts`. That preservation helper only knows source registry, reference catalog, and FoundingReferentData families. An unrelated carried predicate or comment is therefore absent from the replacement template and can be silently lost.

The shared `planInventoryAppend` / `renderInventoryAppendPlan` path already provides semantic duplicate detection, single-valued conflict reporting, exact current-byte preservation, compact self-contained suffixes, and render-vs-plan proof. This bite must route the first ReferenceCatalog addition through that shared path from the original current KnopInventory rather than treating a canonical replacement document as the output.

Implemented on `lane/add-reference-inventory-append` at `58a8b54`; awaiting review and landing. The original current KnopInventory is prepared once, existing shape/support-family validation remains active, the shape-specific renderer supplies only the requested fact graph, and `renderInventoryAppendPlan` writes the exact carried prefix plus planner-approved missing facts. The redundant operation-local Turtle parser is removed.

## Discussion

The existing shape-specific renderers remain useful as a bounded source of the facts required for each supported unwoven/current-only/versioned shape. The implementation may parse their desired output as requested facts, but the append plan must be prepared against the original current bytes and must return only the original graph union planner-approved missing facts. It must not write the canonical renderer's replacement bytes.

This bite deliberately preserves the current placement of any mutable progression facts emitted for a newly created versioned ReferenceCatalog. Moving current/latest/next predicates to metadata is a later plan-level contract change. Preserving that behavior here keeps the data-loss fix reviewable and prevents a storage-model migration from hiding inside an append-mechanics change.

## Open Issues

- None requiring a ruling for this bite. If desired-output Turtle contains a fact that cannot safely be treated as a requested settled fact, report the exact predicate/path and stop rather than inventing a second append abstraction.

## Decisions

- The original current KnopInventory, not a canonical re-render, is the prepared append input.
- Reuse `planInventoryAppend` and `renderInventoryAppendPlan`; do not extend the support-preservation block machinery.
- Preserve existing acceptance, ReferenceCatalog shape, created files, result ordering, and current progression placement.

## Contract Changes

- Successful `knop add-reference` preserves arbitrary carried KnopInventory bytes as an exact prefix instead of retaining only support families known to Weave.
- Missing ReferenceCatalog facts append in a self-contained suffix. A contradiction on a single-valued settled predicate refuses before filesystem writes with the existing and requested facts named.
- Output byte ordering changes intentionally from canonical replacement to prefix-preserving append; RDF outcomes and public result paths remain otherwise unchanged.

## Testing

- Fail-on-old core test: add a first reference to a KnopInventory carrying an unrelated predicate, comment, and nonstandard prefix; require the result to start with the exact input and retain the carried graph.
- Fail-on-old conflict test: carry a contradictory single-valued locator/support relationship and require a diagnostic naming existing and requested facts.
- Exact graph test for each supported unwoven/current-only/versioned input shape: result equals current graph union the expected ReferenceCatalog facts, with no deletion.
- Runtime integration test: conflict produces zero created directories/files and leaves current KnopInventory byte-identical.
- Keep existing root, exact-state/working reference, semantically equivalent Turtle, success logging, and atomic write tests green.
- Run focused tests first, then `deno task fmt:check`, `deno task lint`, `deno task check`, and `deno task test`; run `deno task ci` before landing.

Fail-on-old receipt: the exact-prefix core test, conflicting-locator core test, and runtime zero-write conflict test produced 0 passed / 3 failed on unchanged production code. The old path dropped the carried comment/prefix/opaque fact and accepted the contradictory ReferenceCatalog locator.

Implementation receipts at `58a8b54`:

- focused core/runtime suite: 23 passed / 0 failed;
- `deno task test`: 890 passed / 0 failed;
- `deno task fmt:check`, `deno task lint`, and `deno task check`: green;
- `deno task ci`: 890 passed / 0 failed, LCOV generated; only the known deleted-temporary-source coverage notices appeared.

The semantic-union oracles cover unwoven, current-only, and versioned inputs. Exact-prefix coverage carries a nonstandard prefix, operator comment, opaque predicate, and carried blank node. Runtime conflict coverage proves the KnopInventory remains byte-identical and no `_references` directory is created.

## Non-Goals

- No MeshInventory, PageDefinition, extract, integrate, ResourcePage generation, fixture-topology, or ontology change.
- No movement of current/latest/next progression facts between inventory and metadata.
- No repair/retraction surface, OS-level append, locking, rollback redesign, or public API change.
- No new inventory mutation abstraction and no broad cleanup of shape-specific template renderers outside the add-reference call path.

## Implementation Plan

- [x] Record the carried-fact and single-valued-conflict tests failing on current `main` before production edits.
- [x] Prepare the original KnopInventory once and derive the supported shape's requested ReferenceCatalog facts.
- [x] Plan and render the append through the shared planner/renderer, mapping conflicts to `KnopAddReferenceInputError` without writes.
- [x] Add graph-union coverage across unwoven/current-only/versioned shapes and runtime zero-write conflict coverage.
- [x] Update developer guidance and board the intentional prefix-preserving byte change here for the next release notes; no next-release stub exists yet.
- [x] Run focused and full validation, record receipts, and return the plan-level delta: G1 implementation is green; review/landing remains.
